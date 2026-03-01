# Data Structures Reference

This document describes the binary data structures used in the Keychron HID
protocol, as defined in the firmware C headers and parsed by the GUI Python code.

## Profile Memory Layout

Each analog profile is stored as a contiguous block in EEPROM. The layout is:

```
Offset  Size                     Field
------  ----                     -----
0       4                        analog_key_config_t global
4       rows * cols * 4          analog_key_config_t key_config[rows][cols]
4 + R*C*4                        okmc_config_t okmc[okmc_count]
4 + R*C*4 + okmc_count*19        socd_config_t socd[socd_count]
4 + R*C*4 + O*19 + S*3           uint8_t name[PROFILE_NAME_LEN]  (30 bytes)
4 + R*C*4 + O*19 + S*3 + 30      uint16_t crc16
```

Where `R*C = rows * cols`, `O = okmc_count`, `S = socd_count`.

Total profile size = `4 + rows*cols*4 + okmc_count*19 + socd_count*3 + 30 + 2`

---

## analog_key_config_t (4 bytes)

Stores actuation settings for a single key (or the global defaults).

### C Definition (from `analog_matrix_type.h`)

```c
typedef struct __attribute__((packed)) {
    uint32_t mode             : 2;   // AKM_* enum
    uint32_t act_pt           : 6;   // actuation point (0.1mm, 0-63)
    uint32_t rpd_trig_sen     : 6;   // rapid trigger press sensitivity (0.1mm)
    uint32_t rpd_trig_sen_deact : 6; // rapid trigger release sensitivity (0.1mm)
    uint32_t adv_mode         : 4;   // ADV_MODE_* enum
    uint32_t adv_mode_data    : 8;   // okmc_idx (DKS), js_axis (Gamepad)
} analog_key_config_t;
```

### Byte Layout

```
Byte 0:  [mode:2] [act_pt:6]
         bits 0-1: mode
         bits 2-7: actuation point

Byte 1:  [rpd_trig_sen:6] [rpd_trig_sen_deact_lo:2]
         bits 0-5: rapid trigger press sensitivity
         bits 6-7: release sensitivity bits [1:0]

Byte 2:  [rpd_trig_sen_deact_hi:4] [adv_mode:4]
         bits 0-3: release sensitivity bits [5:2]
         bits 4-7: advance mode type

Byte 3:  [adv_mode_data:8]
         For DKS: OKMC slot index
         For Gamepad: joystick axis index
         For Toggle/Clear: unused (0)
```

### Field Details

| Field              | Bits | Range | Unit   | Description                          |
|--------------------|------|-------|--------|--------------------------------------|
| `mode`             | 2    | 0-5   | enum   | AKM_GLOBAL(0), AKM_REGULAR(1), AKM_RAPID(2), AKM_DKS(3), AKM_GAMEPAD(4), AKM_TOGGLE(5) |
| `act_pt`           | 6    | 0-63  | 0.1mm  | Actuation point. 0 = inherit global. Default: 20 (2.0mm) |
| `rpd_trig_sen`     | 6    | 0-63  | 0.1mm  | Rapid Trigger press sensitivity. 0 = inherit global |
| `rpd_trig_sen_deact` | 6  | 0-63  | 0.1mm  | Rapid Trigger release sensitivity. 0 = inherit global |
| `adv_mode`         | 4    | 0-3   | enum   | ADV_MODE_CLEAR(0), ADV_MODE_OKMC(1), ADV_MODE_GAME_CONTROLLER(2), ADV_MODE_TOGGLE(3) |
| `adv_mode_data`    | 8    | 0-255 | index  | Interpretation depends on adv_mode   |

### Inheritance Rules

When a per-key config field is `0`, it inherits the value from the global
config at profile offset 0. The `adv_mode` and `adv_mode_data` fields are
per-key only and are never inherited.

---

## okmc_config_t (19 bytes)

Stores a Dynamic Keystroke (OKMC) slot configuration.

### Layout

```
Offset  Size  Field
------  ----  -----
0       3     okmc_traval_config_t travel
3       8     uint16_t keycode[4]  (LE16 each)
11      8     okmc_action_t action[4]  (2 bytes each)
```

Total: 3 + 8 + 8 = 19 bytes.

### okmc_traval_config_t (3 bytes, packed bitfields)

```
Byte 0:  [shallow_act:6] [shallow_deact_lo:2]
         bits 0-5: shallow actuation travel (0.1mm, 0-63)
         bits 6-7: shallow deactuation bits [1:0]

Byte 1:  [shallow_deact_hi:4] [deep_act_lo:4]
         bits 0-3: shallow deactuation bits [5:2]
         bits 4-7: deep actuation bits [3:0]

Byte 2:  [deep_act_hi:2] [deep_deact:6]
         bits 0-1: deep actuation bits [5:4]
         bits 2-7: deep deactuation travel (0.1mm, 0-63)
```

### okmc_action_t (2 bytes per keycode)

Each keycode has 4 action slots (one per DKS event: shallow press, shallow
release, deep press, deep release). Each action is a 4-bit nibble.

```
Byte 0:  [shallow_act:4] [shallow_deact:4]
         bits 0-3: action on shallow actuation
         bits 4-7: action on shallow deactuation

Byte 1:  [deep_act:4] [deep_deact:4]
         bits 0-3: action on deep actuation
         bits 4-7: action on deep deactuation
```

### OKMC Action Values

| Value | Binary | Constant              | Description              |
|-------|--------|-----------------------|--------------------------|
| 0     | `000`  | `OKMC_ACTION_NONE`    | No action                |
| 1     | `001`  | `OKMC_ACTION_RELEASE` | Key release              |
| 2     | `010`  | `OKMC_ACTION_PRESS`   | Key press                |
| 6     | `110`  | `OKMC_ACTION_TAP`     | Press then release       |
| 7     | `111`  | `OKMC_ACTION_RE_PRESS` | Release, press, release |

### DKS Travel Events

A DKS-configured key divides its travel into 4 zones:

```
                        0mm (rest)
                        |
      shallow_act  -----+----------  Key begins to travel
                        |
   shallow_deact  ------+----------  Key returns past shallow threshold
                        |
         deep_act  -----+----------  Key reaches deep travel
                        |
      deep_deact  ------+----------  Key returns past deep threshold
                        |
                       4mm (full travel)
```

At each transition, the firmware executes the corresponding action for each of
the 4 keycodes. This allows complex macros like "press A on shallow, press B
on deep, release both on return."

---

## socd_config_t (3 bytes)

Stores a Simultaneous Opposing Cardinal Directions pair for analog keyboards.

### C Definition

```c
typedef struct __attribute__((packed)) {
    uint8_t key_1_row : 3;  // row of first key (0-7)
    uint8_t key_1_col : 5;  // column of first key (0-31)
    uint8_t key_2_row : 3;  // row of second key (0-7)
    uint8_t key_2_col : 5;  // column of second key (0-31)
    uint8_t type;            // SOCD priority type (0-6)
} socd_config_t;
```

### Byte Layout

```
Byte 0:  [key_1_row:3] [key_1_col:5]
         bits 0-2: first key row
         bits 3-7: first key column

Byte 1:  [key_2_row:3] [key_2_col:5]
         bits 0-2: second key row
         bits 3-7: second key column

Byte 2:  [type:8]
         SOCD priority type enum
```

---

## snap_click_config_t (3 bytes)

Stores a Snap Click (SOCD for mechanical keyboards) entry.

### Layout

```
Byte 0:  type     (SNAP_CLICK_TYPE_* enum, 0-5)
Byte 1:  key1     (key index)
Byte 2:  key2     (key index)
```

### Notes

- Snap Click is the mechanical keyboard equivalent of SOCD.
- Unlike `socd_config_t` which uses packed row/col bitfields, snap click
  uses simple key indices.
- The key indices reference positions in a keyboard-specific key table.

---

## effect_config_t (8 bytes, Mixed RGB)

Stores one effect entry in a Mixed RGB region's effect list.

### Layout

```
Byte 0:    effect    (effect type index)
Byte 1:    hue       (HSV hue, 0-255)
Byte 2:    sat       (HSV saturation, 0-255)
Byte 3:    speed     (animation speed, 0-255)
Byte 4-7:  time      (display duration in milliseconds, LE32)
```

---

## os_indicator_config_t

Stores OS indicator LED configuration.

### Fields (from HID response)

| Field          | Size  | Description                              |
|----------------|-------|------------------------------------------|
| available_mask | 1     | Bitmask of available indicators (read-only) |
| disable_mask   | 1     | Bitmask of disabled indicators (user-settable) |
| hue            | 1     | HSV hue for all indicators (0-255)       |
| sat            | 1     | HSV saturation (0-255)                   |
| val            | 1     | HSV value/brightness (0-255)             |

Indicator bits: bit 0 = Caps Lock, bit 1 = Num Lock, bit 2 = Scroll Lock.

---

## calibrated_value_t

Stores per-key calibration data returned by `AMC_GET_CALIBRATED_VALUE`.

| Field        | Size  | Type    | Description                         |
|--------------|-------|---------|-------------------------------------|
| zero_travel  | 2     | uint16  | ADC value at zero key travel        |
| full_travel  | 2     | uint16  | ADC value at full key travel (4mm)  |
| scale_factor | 4     | float32 | Conversion factor: ADC -> 0.1mm     |

---

## Constants Reference

### Travel Units

- `FULL_TRAVEL_UNIT = 40` (4.0mm expressed in 0.1mm)
- `DEFAULT_ACTUATION_POINT = 20` (2.0mm)

### Protocol Versions

- `PROTOCOL_VERSION = 0x02` (main Keychron protocol)
- `MISC_PROTOCOL_VERSION = 0x0002` (misc sub-protocol)
- `PER_KEY_RGB_VER = 0x0001` (RGB sub-protocol)
- `KC_ANALOG_MATRIX_VERSION = 0x34340004` (analog sub-protocol; only LSB transmitted)

### Profile Constants

- `PROFILE_NAME_LEN = 30` bytes (null-terminated string)
- CRC16 follows the name field (2 bytes, computed over the entire profile)

### Response Status

- `KC_SUCCESS = 0`
- `KC_FAIL = 1`
