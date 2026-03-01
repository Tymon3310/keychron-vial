# Analog Matrix Protocol (0xA9)

The Analog Matrix sub-protocol (`KC_ANALOG_MATRIX = 0xA9`) manages Hall Effect
(HE) analog switch keyboards. It controls per-key actuation settings, profiles,
SOCD pairs, Dynamic Keystroke (DKS/OKMC) slots, joystick response curves,
game controller mode, real-time travel monitoring, and calibration.

> **See also:** The [Launcher Analog Matrix Extras](../launcher/analog-matrix-extras.md)
> document lists additional AMC sub-commands found in the Keychron Launcher
> that are not present in our firmware fork (axis type, single-key reset, etc.).

All responses follow the pattern:

```
Byte  Field           Description
----  -----           -----------
 0    0xA9            Command echo
 1    sub_command     Sub-command echo
 2    status          0 = success (except AMC_GET_PROFILE_RAW)
 3..  payload         Sub-command-specific data
```

## Sub-Command Map

| Sub-cmd | Value  | Direction | Description                        |
|---------|--------|-----------|------------------------------------|
| `AMC_GET_VERSION`              | `0x01` | Read  | Analog protocol version      |
| `AMC_GET_PROFILES_INFO`       | `0x10` | Read  | Profile count and metadata   |
| `AMC_SELECT_PROFILE`          | `0x11` | Write | Switch active profile        |
| `AMC_GET_PROFILE_RAW`         | `0x12` | Read  | Read raw profile bytes       |
| `AMC_SET_PROFILE_NAME`        | `0x13` | Write | Set profile name             |
| `AMC_SET_TRAVEL`              | `0x14` | Write | Set actuation/RT settings    |
| `AMC_SET_ADVANCE_MODE`        | `0x15` | Write | Set DKS/Gamepad/Toggle mode  |
| `AMC_SET_SOCD`                | `0x16` | Write | Set SOCD pair config         |
| `AMC_RESET_PROFILE`           | `0x1E` | Write | Reset profile to defaults    |
| `AMC_SAVE_PROFILE`            | `0x1F` | Write | Save profile to EEPROM       |
| `AMC_GET_CURVE`               | `0x20` | Read  | Get joystick response curve  |
| `AMC_SET_CURVE`               | `0x21` | Write | Set joystick response curve  |
| `AMC_GET_GAME_CONTROLLER_MODE`| `0x22` | Read  | Get game controller mode     |
| `AMC_SET_GAME_CONTROLLER_MODE`| `0x23` | Write | Set game controller mode     |
| `AMC_GET_REALTIME_TRAVEL`     | `0x30` | Read  | Live key travel value        |
| `AMC_CALIBRATE`               | `0x40` | Write | Start calibration            |
| `AMC_GET_CALIBRATE_STATE`     | `0x41` | Read  | Get calibration state        |
| `AMC_GET_CALIBRATED_VALUE`    | `0x42` | Read  | Get per-key calibration data |

---

## 0x01 -- Get Version

```
Request:  [0xA9] [0x01]
Response: data[2] = version_byte (LSB of KC_ANALOG_MATRIX_VERSION)
```

The full firmware version constant is `0x34340004`; only the lowest byte (`0x04`)
is transmitted.

---

## 0x10 -- Get Profiles Info

Returns metadata about the profile system.

```
Request:  [0xA9] [0x10]
Response:
  data[2] = current_profile   (active profile index, 0-based)
  data[3] = profile_count     (total number of profiles)
  data[4..5] = profile_size   (bytes per profile, LE16)
  data[6] = okmc_count        (number of OKMC/DKS slots per profile)
  data[7] = socd_count        (number of SOCD pair slots per profile)
```

### Notes

- Profile size is computed as:
  `4 + rows*cols*4 + okmc_count*19 + socd_count*3 + 30 + 2`
  (global config + per-key configs + OKMC slots + SOCD pairs + name + CRC16)
- See [data-structures.md](data-structures.md) for detailed struct layouts.

---

## 0x11 -- Select Profile

Switches the active analog profile.

```
Request:  [0xA9] [0x11] [profile_index]
Response: data[2] = status
```

---

## 0x12 -- Get Profile Raw

Reads raw bytes from a profile's memory. This is the primary mechanism for
reading per-key actuation settings, OKMC slots, SOCD pairs, and profile names.

### Request

```
[0xA9] [0x12] [profile] [offset_lo] [offset_hi] [size]
```

- `offset`: 16-bit little-endian byte offset into the profile data
- `size`: Number of bytes to read (maximum 26 per call)

### Response

```
data[0] = 0xA9       (command echo)
data[1] = 0x12       (sub-command echo)
data[2] = profile    (echoed profile index -- NOT a status code)
data[3..5] = echoed offset/size
data[6..6+size] = raw profile bytes
```

### Notes

- This command does NOT use a success/fail status byte at `data[2]`. Instead
  it echoes the profile index.
- The GUI reads profiles in chunks of up to 24-26 bytes at a time.
- See [data-structures.md](data-structures.md) for the profile memory layout.

---

## 0x13 -- Set Profile Name

```
Request:  [0xA9] [0x13] [profile] [name_length] [name_bytes...]
Response: data[2] = status
```

- Maximum name length: 30 bytes (`PROFILE_NAME_LEN`)
- Name is UTF-8, null-terminated in firmware storage.

---

## 0x14 -- Set Travel

Sets actuation point and Rapid Trigger sensitivity for keys.

### Global (entire keyboard)

```
[0xA9] [0x14] [profile] [mode] [act_pt] [sens] [rls_sens] [1=entire] [0]
```

### Per-key (using row masks)

```
[0xA9] [0x14] [profile] [mode] [act_pt] [sens] [rls_sens] [0=per-key]
  followed by MATRIX_ROWS * 3 bytes: LE24 column bitmask per row
```

### Parameters

| Field      | Range  | Unit     | Description                            |
|------------|--------|----------|----------------------------------------|
| `mode`     | 0-5    | enum     | Key mode (see Analog Key Modes below)  |
| `act_pt`   | 1-39   | 0.1 mm   | Actuation point                        |
| `sens`     | 1-40   | 0.1 mm   | Rapid Trigger press sensitivity        |
| `rls_sens` | 1-40   | 0.1 mm   | Rapid Trigger release sensitivity      |

### Analog Key Modes

| Value | Constant      | Name            | Description                        |
|-------|---------------|-----------------|------------------------------------|
| 0     | `AKM_GLOBAL`  | Global          | Inherit from global profile config |
| 1     | `AKM_REGULAR` | Regular         | Fixed actuation point              |
| 2     | `AKM_RAPID`   | Rapid Trigger   | Dynamic actuation (press/release sens) |
| 3     | `AKM_DKS`     | Dynamic Keystroke | Multi-action per travel stage    |
| 4     | `AKM_GAMEPAD` | Gamepad         | Analog axis output                 |
| 5     | `AKM_TOGGLE`  | Toggle          | Toggle key on/off on actuation     |

### Notes

- `FULL_TRAVEL_UNIT = 40` (4.0mm total travel in 0.1mm units).
- `DEFAULT_ACTUATION_POINT = 20` (2.0mm).
- When `entire=1`, the settings apply to all keys and also update the global
  config at offset 0 in the profile.
- Row masks use 3 bytes per row (24-bit little-endian column bitmask).

---

## 0x15 -- Set Advance Mode

Configures special per-key modes: DKS (Dynamic Keystroke), Gamepad axis,
or Toggle.

### Clear (revert to Regular/Rapid)

```
[0xA9] [0x15] [profile] [0=ADV_MODE_CLEAR] [row] [col] [0]
```

### DKS / OKMC

```
[0xA9] [0x15] [profile] [1=ADV_MODE_OKMC] [row] [col] [okmc_index]
  [travel_b0] [travel_b1] [travel_b2]     -- okmc_traval_config_t (3 bytes)
  [kc0_lo] [kc0_hi] ... [kc3_lo] [kc3_hi] -- 4 keycodes (8 bytes)
  [act0_b0] [act0_b1] ... [act3_b0] [act3_b1] -- 4 actions (8 bytes)
```

Total DKS packet payload: 7 + 3 + 8 + 8 = 26 bytes.

See [data-structures.md](data-structures.md) for the `okmc_config_t` layout.

### Gamepad Axis

```
[0xA9] [0x15] [profile] [2=ADV_MODE_GAME_CONTROLLER] [row] [col] [js_axis]
```

`js_axis` values 0-11 are analog axes (X, Y, Z, RX, RY, RZ with +/-),
values 13-44 are digital buttons (Button 0-31). See the Gamepad Axis table
below.

### Toggle

```
[0xA9] [0x15] [profile] [3=ADV_MODE_TOGGLE] [row] [col] [0]
```

### Gamepad Axis Values

| Value | Constant           | Name       |
|-------|--------------------|------------|
| 0     | `GC_X_AXIS_LEFT`  | X- (Left)  |
| 1     | `GC_X_AXIS_RIGHT` | X+ (Right) |
| 2     | `GC_Y_AXIS_DOWN`  | Y- (Down)  |
| 3     | `GC_Y_AXIS_UP`    | Y+ (Up)    |
| 4     | `GC_Z_AXIS_N`     | Z-         |
| 5     | `GC_Z_AXIS_P`     | Z+         |
| 6     | `GC_RX_AXIS_LEFT` | RX- (Left) |
| 7     | `GC_RX_AXIS_RIGHT`| RX+ (Right)|
| 8     | `GC_RY_AXIS_DOWN` | RY- (Down) |
| 9     | `GC_RY_AXIS_UP`   | RY+ (Up)   |
| 10    | `GC_RZ_AXIS_N`    | RZ-        |
| 11    | `GC_RZ_AXIS_P`    | RZ+        |
| 12    | `GC_AXIS_MAX`     | (boundary) |
| 13-44 | `GC_AXIS_MAX+1+N` | Button 0-31|

### Game Controller Mode Flags

| Bit | Mask   | Constant        | Description         |
|-----|--------|-----------------|---------------------|
| 0   | `0x01` | `GC_MASK_XINPUT`| XInput mode enabled |
| 1   | `0x02` | `GC_MASK_TYPING`| Typing mode enabled |

---

## 0x16 -- Set SOCD

Configures a Simultaneous Opposing Cardinal Directions pair for analog keyboards.

```
Request:  [0xA9] [0x16] [profile] [row1] [col1] [row2] [col2] [index] [socd_type]
Response: data[2] = status
```

### SOCD Types (for HE keyboards)

| Value | Constant                      | Name                    |
|-------|-------------------------------|-------------------------|
| 0     | `SOCD_PRI_NONE`              | Disabled                |
| 1     | `SOCD_PRI_DEEPER_TRAVEL`     | Deeper Travel           |
| 2     | `SOCD_PRI_DEEPER_TRAVEL_SINGLE` | Deeper Travel (Single) |
| 3     | `SOCD_PRI_LAST_KEYSTROKE`    | Last Keystroke          |
| 4     | `SOCD_PRI_KEY_1`             | Key 1 Priority          |
| 5     | `SOCD_PRI_KEY_2`             | Key 2 Priority          |
| 6     | `SOCD_PRI_NEUTRAL`           | Neutral                 |

### Notes

- HE keyboards use distance-based SOCD resolution (e.g., "Deeper Travel"
  activates whichever key is pressed further).
- This is separate from Snap Click (0xA7 / 0x07-0x0A) which is for
  regular mechanical keyboards.

---

## 0x1E -- Reset Profile

Resets a profile to factory defaults.

```
Request:  [0xA9] [0x1E] [profile]
Response: data[2] = status
```

---

## 0x1F -- Save Profile

Persists a profile to EEPROM.

```
Request:  [0xA9] [0x1F] [profile]
Response: data[2] = status
```

Must be called after modifying travel, advance mode, SOCD, or name settings.

---

## 0x20 / 0x21 -- Joystick Response Curve

The curve defines how analog travel maps to joystick axis output. It consists
of 4 control points for a bezier-like interpolation.

### Get (0x20)

```
Request:  [0xA9] [0x20]
Response: data[2..9] = 4 x LE16 control points (each point is {x, y} packed as LE16)
```

> **Note:** Unlike most commands, the GET response has **no status byte**.
> `game_controller_get_curve()` writes 8 bytes of curve data directly at
> `data[2]` via `memcpy`. Each LE16 value encodes a `point_t` where the
> low byte is X (travel, 0--40) and the high byte is Y (output, 0--127).

### Set (0x21)

```
Request:  [0xA9] [0x21] [point0 LE16] [point1 LE16] [point2 LE16] [point3 LE16]
Response: data[2] = status
```

---

## 0x22 / 0x23 -- Game Controller Mode

### Get (0x22)

```
Request:  [0xA9] [0x22]
Response: data[2]=status, data[3]=mode_flags
```

### Set (0x23)

```
Request:  [0xA9] [0x23] [mode_flags]
Response: data[2] = mode_flags (echoed, NOT a status byte)
```

> **Firmware note:** `game_controller_mode_set()` returns a bool, but the
> AMC command handler does **not** write a status byte to `data[2]` for this
> sub-command (unlike the GET handler). The original `mode_flags` value
> remains in `data[2]`. The GUI should verify success by checking only that
> `data[0]` and `data[1]` echo the command and sub-command.

Mode flags: see Game Controller Mode Flags table above.

---

## 0x30 -- Get Realtime Travel

Returns live travel data for a single key. Used for debugging and visualization.

```
Request:  [0xA9] [0x30] [row] [col]
Response:
  data[2]  = status
  data[3]  = row (echoed)
  data[4]  = col (echoed)
  data[5]  = travel_mm     (current travel in 0.1mm units)
  data[6]  = travel_raw    (raw ADC-derived value)
  data[7..8]  = value      (current ADC reading, LE16)
  data[9..10] = zero       (calibrated zero-travel ADC, LE16)
  data[11..12] = full      (calibrated full-travel ADC, LE16)
  data[13] = state         (key state enum)
```

### Notes

- This command uses only 1 retry (vs. the usual 3) to minimize latency.
- The GUI can poll this repeatedly for a live travel heatmap.

---

## 0x40 -- Calibrate

Starts a calibration process.

```
Request:  [0xA9] [0x40] [calib_type]
Response: data[2] = status
```

### Calibration Types

| Value | Constant                     | Description                       |
|-------|------------------------------|-----------------------------------|
| 0     | `CALIB_OFF`                  | Cancel calibration                |
| 1     | `CALIB_ZERO_TRAVEL_POWER_ON` | Zero-travel cal (auto at boot)   |
| 2     | `CALIB_ZERO_TRAVEL_MANUAL`   | Zero-travel cal (manual trigger) |
| 3     | `CALIB_FULL_TRAVEL_MANUAL`   | Full-travel cal (manual trigger) |
| 4     | `CALIB_SAVE_AND_EXIT`        | Save calibration and exit        |
| 5     | `CALIB_CLEAR`                | Clear all calibration data       |

---

## 0x41 -- Get Calibration State

```
Request:  [0xA9] [0x41]
Response:
  data[2] = calibrated_bitmask
            bit 0: zero-travel calibrated
            bit 1: full-travel calibrated
  data[3] = cali_state (current calibration state enum)
  data[4..] = calib_state_matrix (3 bytes per row, 24-bit col bitmask)
```

### Notes

- `data[2]` is NOT a success/fail status code for this command.
- The `calib_state_matrix` indicates which keys have been calibrated
  (set bits = calibrated).

---

## 0x42 -- Get Calibrated Value

Returns calibration data for a specific key.

```
Request:  [0xA9] [0x42] [row] [col]
Response:
  data[2] = row (echoed)
  data[3] = col (echoed)
  data[4] = status (0 = success)
  data[5..6] = zero_travel (LE16, ADC value at zero travel)
  data[7..8] = full_travel (LE16, ADC value at full travel)
  data[9..12] = scale_factor (IEEE 754 float, LE32)
```

### Notes

- The status byte is at `data[4]` (not `data[2]`) for this command.
- `scale_factor` is used internally by the firmware to convert ADC readings
  to 0.1mm travel units.
