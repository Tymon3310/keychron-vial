# Command Map Comparison -- Firmware vs Launcher

This document compares the protocol command enums as defined in the Keychron
firmware C headers versus the Keychron Launcher JavaScript bundle. The
Launcher uses slightly different names for some commands and defines
additional commands not present in our firmware fork.

## Standard VIA Commands (shared)

These are identical between firmware and Launcher:

| Value | Firmware Constant                   | Launcher Constant                   |
|-------|-------------------------------------|-------------------------------------|
| 1     | `id_get_protocol_version`           | `GET_PROTOCOL_VERSION`              |
| 2     | `id_get_keyboard_value`             | `GET_KEYBOARD_VALUE`                |
| 3     | `id_set_keyboard_value`             | `SET_KEYBOARD_VALUE`                |
| 4     | `id_dynamic_keymap_get_keycode`     | `DYNAMIC_KEYMAP_GET_KEYCODE`        |
| 5     | `id_dynamic_keymap_set_keycode`     | `DYNAMIC_KEYMAP_SET_KEYCODE`        |
| 6     | `id_dynamic_keymap_clear_all`       | `DYNAMIC_KEYMAP_CLEAR_ALL`          |
| 7     | `id_backlight_config_set_value`     | `CUSTOM_MENU_SET_VALUE` / `BACKLIGHT_CONFIG_SET_VALUE` |
| 8     | `id_backlight_config_get_value`     | `CUSTOM_MENU_GET_VALUE` / `BACKLIGHT_CONFIG_GET_VALUE` |
| 9     | `id_backlight_config_save`          | `CUSTOM_MENU_SAVE` / `BACKLIGHT_CONFIG_SAVE` |
| 10    | `id_eeprom_reset`                   | `EEPROM_RESET`                      |
| 11    | `id_bootloader_jump`                | `BOOTLOADER_JUMP`                   |
| 12    | `id_dynamic_keymap_macro_get_count` | `DYNAMIC_KEYMAP_MACRO_GET_COUNT`    |
| 13    | `id_dynamic_keymap_macro_get_buffer_size` | `DYNAMIC_KEYMAP_MACRO_GET_BUFFER_SIZE` |
| 14    | `id_dynamic_keymap_macro_get_buffer`| `DYNAMIC_KEYMAP_MACRO_GET_BUFFER`   |
| 15    | `id_dynamic_keymap_macro_set_buffer`| `DYNAMIC_KEYMAP_MACRO_SET_BUFFER`   |
| 16    | `id_dynamic_keymap_macro_reset`     | `DYNAMIC_KEYMAP_MACRO_RESET`        |
| 17    | `id_dynamic_keymap_get_layer_count` | `DYNAMIC_KEYMAP_GET_LAYER_COUNT`    |
| 18    | `id_dynamic_keymap_get_buffer`      | `DYNAMIC_KEYMAP_GET_BUFFER`         |
| 19    | `id_dynamic_keymap_set_buffer`      | `DYNAMIC_KEYMAP_SET_BUFFER`         |
| 20    | `id_dynamic_keymap_get_encoder`     | `DYNAMIC_KEYMAP_GET_ENCODER`        |
| 21    | `id_dynamic_keymap_set_encoder`     | `DYNAMIC_KEYMAP_SET_ENCODER`        |

Note: Commands 7/8/9 have dual names in the Launcher (`CUSTOM_MENU_*` and
`BACKLIGHT_CONFIG_*` both map to the same values). This reflects VIA's
historical rename of these commands.

## Keychron Extension Commands

| Value | Hex    | Firmware Name            | Launcher Name              | Notes |
|-------|--------|--------------------------|----------------------------|-------|
| 160   | `0xA0` | `KC_GET_PROTOCOL_VERSION`| `KC_GET_PROTOCOL_VERSION`  | Same |
| 161   | `0xA1` | `KC_GET_FIRMWARE_VERSION`| `KC_FIRMWARE_VERSION`      | "GET_" dropped |
| 162   | `0xA2` | `KC_GET_SUPPORT_FEATURE` | `KC_GET_SUPPORT_FEATURE`   | Same |
| 163   | `0xA3` | `KC_GET_DEFAULT_LAYER`   | `KC_GET_CURRENT_LAYER`     | "DEFAULT" → "CURRENT" |
| 167   | `0xA7` | `KC_MISC_CMD_GROUP`      | `KC_MISC_CMD_GROUP`          | Same |
| 168   | `0xA8` | `KC_KEYCHRON_RGB`        | `KC_RGB`                   | Simplified |
| 169   | `0xA9` | `KC_ANALOG_MATRIX`       | `KC_HE`                    | "Analog Matrix" → "Hall Effect" |
| 170   | `0xAA` | `KC_WIRELESS_DFU`        | *(not in enum)*            | Handled via separate DFU classes |
| 171   | `0xAB` | `KC_FACTORY_TEST`        | `KC_FACTORY`               | In Launcher enum; see below |
| 172   | `0xAC` | *(not in firmware)*      | `KC_SCREEN`                | **Launcher only** -- see [screen-protocol.md](screen-protocol.md) |

### KC_FACTORY (0xAB) Sub-Commands

The Launcher names command `0xAB` as `KC_FACTORY` and defines a
`FACTORY_TEST_CMD_*` sub-command enum. Two of these are actively used by the
Launcher UI; the rest are defined but not referenced:

| Value | Hex    | Launcher Name                      | Used | Notes |
|-------|--------|------------------------------------|------|-------|
| 1     | `0x01` | `FACTORY_TEST_CMD_BACKLIGHT`       | no   | |
| 2     | `0x02` | `FACTORY_TEST_CMD_OS_SWITCH`       | no   | |
| 3     | `0x03` | `FACTORY_TEST_CMD_JUMP_TO_BL`      | no   | |
| 4     | `0x04` | `FACTORY_TEST_CMD_INT_PIN`         | no   | |
| 5     | `0x05` | `FACTORY_TEST_CMD_GET_TRANSPORT`   | **yes** | DIP/transport detection, see below |
| 6     | `0x06` | `FACTORY_TEST_CMD_CHARGING_ADC`    | no   | |
| 7     | `0x07` | `FACTORY_TEST_CMD_RADIO_CARRIER`   | no   | |
| 8     | `0x08` | `FACTORY_TEST_CMD_GET_BUILD_TIME`  | no   | |
| 9     | `0x09` | `FACTORY_TEST_CMD_GET_DEVICE_ID`   | **yes** | Chip ID readout (trackball profile storage) |

**GET_TRANSPORT (5)** -- detects the physical mode-switch position:

```
Request : [0xAB, 0x05, ..., data[30] = 0x05, data[31] = 0x00]
          (byte 30 is the additive-checksum low byte; for this empty
           payload the sum is just data[1] = 5, so it doubles as an
           echo of the sub-command)
Response: [0xAB, cmd, transport, usb_power_sense, ...]
          data[2] = 1 → USB, 2 → BT, 4 → G (2.4 GHz)
```

**GET_DEVICE_ID (9)** -- returns a device-unique chip ID string (firmware
sends the 12-byte MCU UUID from `UID_BASE` prefixed by its length):

```
Request : [0xAB, 0x09, ..., data[30] = 0x09]
Response: data[1] = 9, data[2] = 12 (UUID length),
          data[3..14] = UUID bytes
          Launcher concatenates decimal values of data[3..13] (11 bytes)
```

### Key Observations

1. **`KC_MISC_CMD_GROUP` (0xA7)**: The Launcher and firmware use the same name. Sub-command `0x01` returns the misc protocol version.

2. **`KC_HE` (0xA9)**: The Launcher calls this "Hall Effect" (`HE`), while the
   firmware uses "Analog Matrix" (`ANALOG_MATRIX`). Both refer to the same
   Hall Effect switch protocol.

3. **`KC_SCREEN` (0xAC)**: This command is **only in the Launcher** -- it does
   not exist in our firmware fork. It controls an external flash filesystem
   and RTC clock for keyboards with built-in screens/displays. See
   [screen-protocol.md](screen-protocol.md).

4. **`0xAA` and `0xAB`**: These are not dispatched through the Launcher's VIA
    command dispatcher; the Launcher handles Bluetooth DFU and factory test
    through dedicated classes that construct packets independently.
    `0xAB` is present in the enum as `KC_FACTORY` with a `FACTORY_TEST_CMD_*`
    sub-command enum (see table above).

## Analog Matrix (HE) Sub-Commands

| Value | Hex    | Firmware Name        | Launcher Name              | Notes |
|-------|--------|----------------------|----------------------------|-------|
| 1     | `0x01` | `AMC_GET_VERSION`    | `AMC_GET_VERSION`          | Same |
| 16    | `0x10` | `AMC_GET_PROFILES_INFO` | `AMC_GET_PROFILES_INFO` | Same |
| 17    | `0x11` | `AMC_SELECT_PROFILE` | `AMC_SELECT_PROFILE`       | Same |
| 18    | `0x12` | `AMC_GET_PROFILE_RAW`| `AMC_GET_PROFILE_BUFFER`   | "RAW" → "BUFFER" |
| 19    | `0x13` | `AMC_SET_PROFILE_NAME`| `AMC_SET_PROFILE_NAME`    | Same |
| 20    | `0x14` | `AMC_SET_TRAVEL`     | `AMC_SET_TRAVAL`           | Typo in Launcher: "TRAVAL" |
| 21    | `0x15` | `AMC_SET_ADVANCE_MODE`| `AMC_SET_ADVANCE_MODE`    | Same |
| 22    | `0x16` | `AMC_SET_SOCD`       | `AMC_SET_ACT_ON_AXIS`      | Different name -- SOCD vs "act on axis" |
| 29    | `0x1D` | *(not in firmware)*  | `AMC_RESET_SINGLE_KEY`     | **Launcher only** |
| 30    | `0x1E` | `AMC_RESET_PROFILE`  | `AMC_CLEAR_PROFILE_BUFFER` | "RESET" → "CLEAR_BUFFER" |
| 31    | `0x1F` | `AMC_SAVE_PROFILE`   | `AMC_SAVE_PROFILE_BUFFER`  | Added "BUFFER" |
| 32    | `0x20` | `AMC_GET_CURVE`      | `AMC_GET_CURVE`            | Same |
| 33    | `0x21` | `AMC_SET_CURVE`      | `AMC_SET_CURVE`            | Same |
| 34    | `0x22` | `AMC_GET_GAME_CONTROLLER_MODE` | *(hardcoded as 34)* | Launcher calls it `getJoyKeyboard` |
| 35    | `0x23` | `AMC_SET_GAME_CONTROLLER_MODE` | *(hardcoded as 35)* | Launcher calls it `enableJoyKeyboard` |
| 48    | `0x30` | `AMC_GET_REALTIME_TRAVEL` | `AMC_GET_REALTIME_TRAVEL` | Same |
| 64    | `0x40` | `AMC_CALIBRATE`      | `AMC_CALIBRATE`            | Same |
| 65    | `0x41` | `AMC_GET_CALIBRATE_STATE` | `AMC_GET_CALIBRATE_STATE` | Same |
| 66    | `0x42` | `AMC_GET_CALIBRATED_VALUE` | `AMC_GET_CALIBRATED_VALUE` | Same |
| 80    | `0x50` | *(not in firmware)*  | `AMC_GET_AXIS_TYPE`        | **Launcher only** |
| 81    | `0x51` | *(not in firmware)*  | `AMC_SET_AXIS_TYPE`        | **Launcher only** |

See [analog-matrix-extras.md](analog-matrix-extras.md) for details on the
Launcher-only commands.

### AMC_GET_VERSION Feature Bits

The `AMC_GET_VERSION` (0xA9 / 0x01) response carries the HE command-set
version in `data[2]` and a feature bitmap in `data[4]`. The Launcher
checks six bits:

| Bit | Feature              |
|-----|----------------------|
| 0   | `switchAxis`         |
| 1   | `resetSingleKey`     |
| 2   | `dkr`                |
| 3   | `turbo`              |
| 4   | `gamepadDisable`     |
| 5   | `supportMechanical`  |

`supportMechanical` enables an additional "HE mechanical" query
(sub-command **82 / `0x52`**, see below). The Launcher also dispatches
per-version command classes for HE versions 1--5.

### HE Mechanical/Magnetic Axis Bitmap

Sub-command **82 (`0x52`)** under `KC_HE` (0xA9) returns the per-key switch
technology map for hybrid HE/mechanical boards:

```
Request : [0xA9, 82]
Response: data[2] = row count
          data[3] = col count
          data[4 + 3*row .. +2] = 24-bit LE bitmap for that row
                                  bit set   → mechanical switch
                                  bit clear → magnetic (HE) switch
```

## FR (Forza Receiver / Dongle) Commands

These commands exist only in the Launcher -- there is no corresponding
firmware enum in our fork:

| Value | Hex    | Launcher Name                | Description |
|-------|--------|------------------------------|-------------|
| 177   | `0xB1` | `FR_GET_PROTOCOL_VERSION`    | Dongle protocol version + feature flags |
| 178   | `0xB2` | `FR_GET_STATE`               | Connected device slots |
| 179   | `0xB3` | `FR_GET_FW_VERSION`          | Dongle firmware version |
| 181   | `0xB5` | `FR_CTL_GAMEPAD_RPT_ENABLE`  | Toggle gamepad report forwarding |
| 186   | `0xBA` | `FR_DFU_OVER_VIA`            | DFU update tunneled through VIA |

See [bridge-dongle-protocol.md](bridge-dongle-protocol.md).

## NAPE (Mouse/Trackball) Sub-Commands

Sent under `KC_MISC_CMD_GROUP` (0xA7) with sub-command values
`0x20`--`0x39`. Not present in our firmware fork:

| Value | Hex    | Launcher Name                    |
|-------|--------|----------------------------------|
| 32    | `0x20` | `KC_USER_CMD_NAPE_GET_ORI`       |
| 33    | `0x21` | `KC_USER_CMD_NAPE_GET_DPI`       |
| 34    | `0x22` | `KC_USER_CMD_NAPE_SET_DPI`       |
| 35    | `0x23` | `KC_USER_CMD_NAPE_SET_DPI_VALUE` |
| 36    | `0x24` | `KC_USER_CMD_NAPE_GET_DPI_VALUE` |
| 37    | `0x25` | `KC_USER_CMD_NAPE_SET_TAPHOLDS`  |
| 38    | `0x26` | `KC_USER_CMD_NAPE_GET_TAPHOLDS`  |
| 39    | `0x27` | `KC_USER_CMD_NAPE_SET_COMBOS`    |
| 40    | `0x28` | `KC_USER_CMD_NAPE_GET_COMBOS`    |
| 41    | `0x29` | `KC_USER_CMD_NAPE_SET_GESTURE`   |
| 42    | `0x2A` | `KC_USER_CMD_NAPE_GET_GESTURE`   |
| 43    | `0x2B` | `KC_USER_CMD_NAPE_SET_PROFILE`   |
| 44    | `0x2C` | `KC_USER_CMD_NAPE_GET_PROFILE`   |
| 45    | `0x2D` | `KC_USER_CMD_NAPE_SET_LAYER`     |
| 46    | `0x2E` | `KC_USER_CMD_NAPE_DEL_COMBOS`    |
| 47    | `0x2F` | `KC_USER_CMD_NAPE_DEL_TAPHOLDS`  |
| 48    | `0x30` | `KC_USER_CMD_NAPE_BAT_REPORT`    |
| 49    | `0x31` | `KC_USER_CMD_NAPE_GET_BAT_REPORT`|
| 50    | `0x32` | `Set_Force_Gesture_Scroll`       |
| 51    | `0x33` | `Get_Force_Gesture_Scroll`       |
| 52    | `0x34` | `KC_USER_CMD_NAPE_SET_ORI`       |
| 56    | `0x38` | `KC_USER_CMD_NAPE_GET_LAYER_ORI` |
| 57    | `0x39` | `KC_USER_CMD_NAPE_SET_LAYER_ORI` |

The two new layer-orientation commands read/write the trackball rotation
angle **per mouse layer** (the global commands 32/52 remain for the default
angle):

```
GET_LAYER_ORI : [0xA7, 56, layer]           → data[2] = angle / 45
SET_LAYER_ORI : [0xA7, 57, layer, angle/45]
```

See [mouse-protocol.md](mouse-protocol.md).

## NAPE Settings Group (0xA7 / 0x20)

**Keyboards** with a built-in trackball or touchpad use a nested command
group: byte 1 is the fixed group ID
`0x20` (`Nape_Settings`) and byte 2 selects the operation. Every response
echoes `[0xA7, 0x20, sub]` and carries a status in `data[3]` (`0` = OK)
for set/delete commands. Timeout is 500 ms.

Standalone trackballs still use the flat NAPE set above. The first two bytes
collide (`[0xA7, 0x20]` is also flat `NAPE_GET_ORI`), so the device class
decides which layout is used.

| Sub | Hex | Name | Request (after `0xA7, 0x20, sub`) | Response |
|-----|-----|------|-----------------------------------|----------|
| 1  | `0x01` | `Get_Dpi` | -- | `data[3]` = current DPI stage |
| 2  | `0x02` | `Set_Dpi` | `stage` | status |
| 3  | `0x03` | `Set_Dpi_Value` | `stage, dpi_lo, dpi_hi` | status |
| 4  | `0x04` | `Get_Dpi_Value` | `stage` | `data[3..4]` = DPI (LE16) |
| 5  | `0x05` | `Get_Dpi_Counts` | -- | `data[3]` = number of stages |
| 6  | `0x06` | `Set_Dpi_Counts` | `count` | status |
| 7  | `0x07` | `Set_Tapholds` | `layer, row, col, tap(LE16), hold(LE16), tapTerm(LE16)` | status |
| 8  | `0x08` | `Get_Tapholds` | `layer, row, col` | `data[6..7]` tap, `data[8..9]` hold, `data[10..11]` tapTerm |
| 9  | `0x09` | `Set_Combos` | `index, timeout(LE16), layerMask, pos[4], tap(LE16), held(LE16)` | status |
| 10 | `0x0A` | `Get_Combos` | `index` | same layout as Set, from `data[4]` |
| 11 | `0x0B` | `Set_Gesture` | `up, down, left, right` (4 × LE16 keycodes) | status |
| 12 | `0x0C` | `Get_Gesture` | -- | `data[3..10]` = up, down, left, right |
| 14 | `0x0E` | `Del_Combos` | `index` | status |
| 15 | `0x0F` | `Del_Tapholds` | `layer, row, col` | status |
| 18 | `0x12` | `Set_Force_Gesture_Scroll` | `gesture(0/1), scroll(0/1)` | status |
| 19 | `0x13` | `Get_Force_Gesture_Scroll` | -- | `data[3]` gesture, `data[4]` scroll |
| 20 | `0x14` | `Get_Ori` | -- | `data[3..4]` = angle (LE16, degrees) |
| 21 | `0x15` | `Set_Ori` | `angle(LE16)` | status |
| 22 | `0x16` | `Get_Custom_Dpi` | -- | `data[3..4]` = DPI (LE16) |
| 23 | `0x17` | `Set_Custom_Dpi` | `dpi(LE16)` | status |
| 24 | `0x18` | `Get_Layer_Ori` | `layer` | `data[3..4]` = angle (LE16) |
| 25 | `0x19` | `Set_Layer_Ori` | `layer, angle(LE16)` | status |
| 29 | `0x1D` | `Get_Layer_Dpi` | `layer` | `data[3]` = stage |
| 30 | `0x1E` | `Set_Layer_Dpi` | `layer, stage` | status |
| 31 | `0x1F` | `Get_Layer_Dpi_Value` | `layer, stage` | `data[3..4]` = DPI (LE16) |
| 32 | `0x20` | `Set_Layer_Dpi_Value` | `layer, stage, dpi(LE16)` | status |
| 33 | `0x21` | `Get_Auto_Mouse` | -- | `data[3]` = enabled |
| 34 | `0x22` | `Set_Auto_Mouse` | `enabled` | status |
| 35 | `0x23` | `Get_Auto_Mouse_Params` | -- | `data[3]` threshold, `data[4..5]` timeout ms, `data[6]` layer, `data[7..10]` hold keys |
| 36 | `0x24` | `Set_Auto_Mouse_Params` | `threshold, timeout(LE16), layer, holdKeys[4]` (unused = `0xFF`) | status |
| 37 | `0x25` | `Get_Touch_Speed` | -- | `data[3]` = speed |
| 38 | `0x26` | `Set_Touch_Speed` | `speed` | status |

Notes:

- Unlike the trackball's 45° steps, keyboard angles are plain degrees
  0..359 on the wire. The keyboard `angle` page shows −90..+90 per layer and
  converts with `device = (page + 65 + 360) % 360` (`deviceZero = 65`, so a
  page angle of 0° is 65° on the wire).
- Combo `positions` are 4 key indices (unused = `0xFF`); the UI allows 2–4 keys and up to 30 combos. Tap-hold `tapTerm`
  defaults to 200 ms and moves in 50 ms steps.
- Tap-hold writes are serialised through a promise queue; the firmware is
  expected to answer each one before the next is sent.
- The "keyset" page (`advance` route) shows tabs per product flag:
  `toggle` (`Support_Latch`), `snap` (`Support_Snap`), and `modtap`,
  `simultaneous`, `trackball`, `touchpad` from the backend `ks_menu` list
  (see [overview.md](overview.md#product-dictionary-flags)).
- In the trackball tab, special keys are found **by name** in the board's
  `customKeycodes`: `GESTURE`, `SCROLL`, and `LC-HOLD`/`LC_HOLD` (left-click
  lock). A Vial port that wants these to show up must keep those names.

### Mouse / Trackball Special Keycodes

The Launcher maps high-level pointer and navigation functions to custom keycodes:

| Keycode | Identifier | Description |
|---------|------------|-------------|
| `CUSTOM(28)`--`CUSTOM(35)` | `0°`--`315°` | Quantized orientation angles (45° increments) |
| `CUSTOM(37)` | `mouse.key.nextDpi` | Next DPI stage |
| `CUSTOM(38)` | `mouse.key.prevDpi` | Previous DPI stage |
| `CUSTOM(39)` | `mouse.key.prevPollingRate` | Previous polling rate stage |
| `CUSTOM(40)` | `mouse.key.nextPollingRate` | Next polling rate stage |
| `CUSTOM(43)` | `trackball.key.direction` | Trackball direction / orientation toggle |
| `CUSTOM(44)` | `mouse.key.dpiLoop` | Cycle through DPI stages |
| `CUSTOM(45)` | `mouse.key.loopPollingRate`| Cycle through polling rate stages |
| `CUSTOM(46)` | `mouse.key.leftDouble` | Double-click left mouse button |
| `CUSTOM(47)` | `mouse.key.customDpi` | Custom DPI toggle |
| `MO(9)` | `trackball.key.gesture` | Momentary hold for trackball gesture layer |
| `MO(10)` | `trackball.key.scroll` | Momentary hold for trackball scroll layer |
| `KC_MS_BTN1_LOCK` | `mouse.key.leftLock` | Left Click Lock (latches primary button down) |

## New Keyboard Misc (0xA7) Sub-Commands

The Keychron Launcher exercises several `KC_MISC_CMD_GROUP` sub-commands above
`0x13` that are not yet implemented in our firmware fork:

| Value | Hex    | Name                | Packet | Description |
|-------|--------|---------------------|--------|-------------|
| 20 | `0x14` | Reset keymap single key | `[0xA7, 20, 2^layer, row, keycode_hi, keycode_lo, ...]` | Restore one key's keymap entry to default, then re-read the keymap buffer |
| 21 | `0x15` | Jump bootloader | `[0xA7, 21]`, response `data[3]` = status | Soft reboot into bootloader (gated on feature bit + instruction set 2/4) |
| 22 | `0x16` | Latch GetCount | `[0xA7, 22]` → `data[3]` = count, `data[4]` = limit | Latch-key slot usage/limit |
| 23 | `0x17` | Latch GetToggle | `[0xA7, 23, offset, count≤5]` → records at `data[5..]`, 5 bytes each: `[layer, row, col, type, repeat]` | Read latch keys (paged, ≤5 per request; filter `type != 0` and matching layer) |
| 24 | `0x18` | Latch SetToggle | `[0xA7, 24, 1, layer, row, col, type, repeat]` | Set one latch key; sent per key with ~50 ms spacing. Type: 0=DELETE, 1=KEY, 2=TURBO, 3=MACRO |
| 27 | `0x1B` | Macro repeat Get | `[0xA7, 27, 0, macro_count]` → `data[5 .. 5+count)` = repeat counts | Per-macro repeat count (1--100), gated on `Support_Macro_Repeat` |
| 28 | `0x1C` | Macro repeat Set | `[0xA7, 28, 1, macro_index, times]` | Set one macro's repeat count |
| 2 (+3) | `0x02 0x03` | Get DFU/bootloader name | `[0xA7, 2, 3]` → `data[4]` = length, `data[5..]` = ASCII name | Used by the firmware page to label the bootloader (see [dfu-protocols.md](dfu-protocols.md#bootloader-name-query)) |

Latch keys ("toggle keys", marketed as *Magnet toggle*) let a key latch
on/off like Caps Lock: types are regular key latches (`KEY`), macro latches
(`MACRO`), and turbo/repeat latches (`TURBO` with a repeat count). The
feature is advertised via bit 18 of the V4 support-feature bitmap.

### V4 Support-Feature Bitmap

For keyboards reporting protocol version > 3 in `KC_GET_PROTOCOL_VERSION`
(`data[1]`; `data[3]` holds the "instruction set"), `KC_GET_SUPPORT_FEATURE`
(0xA2) switches from the legacy single-byte bitmap to a wide format with two
little-endian words. Only the two radio bits come from the first word;
everything else is in the second:

```
word A = data[3..4] (LE 16-bit):
  bit 0  Support_Le
  bit 1  Support_Rf
word B = data[6..8] (LE 24-bit):
  bit 0  Get_Default_Layer
  bit 2  Support_Keychron_Rgb
  bit 6  Support_DEBOUNCE
  bit 7  Support_Snap          (Snap Tap / SOCD keyset)
  bit 8  Support_Sleep
  bit 9  Support_PollRate
  bit 12 Upload_Default_Layer
  bit 16 Bootloader_Jump
  bit 17 Performance_Mode
  bit 18 Support_Latch         (latch/toggle keys)
  bit 20 Support_Macro_Repeat
```

`data[5]` is not read.

Legacy (protocol ≤ 3) devices keep the old byte-wide layout documented in
[firmware/misc-commands.md](../firmware/misc-commands.md).

## OS Key Switch (client-side)

Enabled when the backend product flag `km_os_switch` is `true`. Nothing new
is sent to the firmware: the Launcher rewrites layer 0 with normal
`DYNAMIC_KEYMAP_SET_KEYCODE` calls.

1. It looks up the board's `customKeycodes` entries named `Left Option`,
   `Left Cmd`, `Right Option`, `Right Cmd` (case-insensitive). The keycode is
   `QK_KB + index`. If `Left Option` or `Left Cmd` is missing the feature is
   off.
2. It reads layer 0 at `[row 3, col 1]` and `[row 3, col 2]`. Option + Cmd
   there means the keymap is in Mac mode; `KC_LGUI` + `KC_LALT` means Windows.
3. If that differs from the host OS (from the user agent), it writes the
   other pair to those two keys and swaps every `KC_LALT/LGUI/RALT/RGUI` ↔
   Option/Cmd custom keycode elsewhere in layer 0.

Seen state is kept in `localStorage` (`kc_os_key_switch_seen_*`).

## RGB Sub-Commands (Launcher names)

The Launcher uses simplified names for some RGB sub-commands:

| Value | Hex    | Firmware Name                   | Launcher Name |
|-------|--------|---------------------------------|---------------|
| 6     | `0x06` | `RGB_GET_LED_IDX`               | `LedNumber`   |
| 7     | `0x07` | `PER_KEY_RGB_GET_TYPE`          | `GetEffect`   |
| 8     | `0x08` | `PER_KEY_RGB_SET_TYPE`          | `SetEffect`   |
| 9     | `0x09` | `PER_KEY_RGB_GET_COLOR`         | `GetLedColor`  |
| 10    | `0x0A` | `PER_KEY_RGB_SET_COLOR`         | `SetLedColor`  |

## Miscellaneous Keycodes

The Launcher embeds a hardcoded list of basic QMK keycodes and their corresponding numerical values. Notably, it contains specific references to:

| Keycode | Value | Description |
|---------|-------|-------------|
| `KC_HAEN` | 136 (`0x88`) | HanYeong (Korean) |
| `KC_HANJ` | 137 (`0x89`) | Hanja (Korean) |
| `KC_HENK` | 138 (`0x8A`) | JIS Henkan (Convert) |
| `KC_MHEN` | 139 (`0x8B`) | JIS Muhenkan (Non-convert) |

## State Notification (0xBC)

The Launcher listens for asynchronous state change notifications from the
dongle on command byte `0xBC` (188):

```
Byte  Field    Description
----  -----    -----------
 0    0xBC     State notification marker
 1    kb_1     Keyboard slot 1 state
 2    kb_2     Keyboard slot 2 state
 3    mouse    Bit 2: mouse connection state (shifted: data[3] >> 2 & 1)
```

This is not a command that the host sends -- it is an unsolicited report from
the dongle/bridge device when connection state changes.
