# Analog Matrix Extras (Launcher-Only)

This document describes Analog Matrix (Hall Effect) sub-commands that exist
in the Keychron Launcher but are **not present in our firmware fork**. These
represent newer firmware features that Keychron has added since our fork
diverged.

> **Source:** Reverse-engineered from the Keychron Launcher JavaScript bundle.
> For the base protocol, see [firmware/analog-matrix-protocol.md](../firmware/analog-matrix-protocol.md).

## Naming Differences

The Launcher uses the command name `KC_HE` (0xA9) instead of the firmware's
`KC_ANALOG_MATRIX`. See [command-map.md](command-map.md) for a full
comparison of all sub-command names.

Notable name differences:

| Value | Firmware Name        | Launcher Name              |
|-------|----------------------|----------------------------|
| 18    | `AMC_GET_PROFILE_RAW`| `AMC_GET_PROFILE_BUFFER`   |
| 20    | `AMC_SET_TRAVEL`     | `AMC_SET_TRAVAL` (typo)    |
| 22    | `AMC_SET_SOCD`       | `AMC_SET_ACT_ON_AXIS`      |
| 30    | `AMC_RESET_PROFILE`  | `AMC_CLEAR_PROFILE_BUFFER` |
| 31    | `AMC_SAVE_PROFILE`   | `AMC_SAVE_PROFILE_BUFFER`  |

---

## Launcher-Only Sub-Commands

### 0x1D -- AMC_RESET_SINGLE_KEY

Resets a single key's analog configuration back to defaults within a profile,
without resetting the entire profile.

```
Request:  [0xA9] [0x1D] [profile] [row] [col]  (likely)
Response: data[2] = status
```

This is useful for reverting individual key customizations (e.g., a single
DKS or gamepad assignment) without losing other per-key settings.

### 0x50 -- AMC_GET_AXIS_TYPE

Reads the axis type configuration. The exact meaning is unclear from the
minified code, but it likely relates to the joystick axis mapping mode
(linear vs. curve-based, or analog vs. digital threshold).

```
Request:  [0xA9] [0x50]
Response: data[2] = status, data[3..] = axis type data
```

### 0x51 -- AMC_SET_AXIS_TYPE

Sets the axis type configuration.

```
Request:  [0xA9] [0x51] [axis_type_data...]
Response: data[2] = status
```

---

## Full Sub-Command Map (Launcher)

For reference, here is the complete AMC enum as defined in the Launcher:

| Sub-cmd | Value | Hex    | In Firmware? | Description |
|---------|-------|--------|--------------|-------------|
| `AMC_GET_VERSION`            | 1   | `0x01` | Yes | Protocol version |
| `AMC_GET_PROFILES_INFO`      | 16  | `0x10` | Yes | Profile metadata |
| `AMC_SELECT_PROFILE`         | 17  | `0x11` | Yes | Switch profile |
| `AMC_GET_PROFILE_BUFFER`     | 18  | `0x12` | Yes | Read profile raw bytes |
| `AMC_SET_PROFILE_NAME`       | 19  | `0x13` | Yes | Set profile name |
| `AMC_SET_TRAVAL`             | 20  | `0x14` | Yes | Set actuation/RT settings |
| `AMC_SET_ADVANCE_MODE`       | 21  | `0x15` | Yes | Set DKS/Gamepad/Toggle |
| `AMC_SET_ACT_ON_AXIS`        | 22  | `0x16` | Yes* | Set SOCD pair (different name) |
| `AMC_RESET_SINGLE_KEY`       | 29  | `0x1D` | **No** | Reset single key config |
| `AMC_CLEAR_PROFILE_BUFFER`   | 30  | `0x1E` | Yes* | Reset profile (different name) |
| `AMC_SAVE_PROFILE_BUFFER`    | 31  | `0x1F` | Yes* | Save profile (different name) |
| `AMC_GET_CURVE`              | 32  | `0x20` | Yes | Joystick response curve |
| `AMC_SET_CURVE`              | 33  | `0x21` | Yes | Set response curve |
| `AMC_GET_REALTIME_TRAVEL`    | 48  | `0x30` | Yes | Live key travel value |
| `AMC_CALIBRATE`              | 64  | `0x40` | Yes | Start calibration |
| `AMC_GET_CALIBRATE_STATE`    | 65  | `0x41` | Yes | Calibration state |
| `AMC_GET_CALIBRATED_VALUE`   | 66  | `0x42` | Yes | Per-key calibration data |
| `AMC_GET_AXIS_TYPE`          | 80  | `0x50` | **No** | Get axis type config |
| `AMC_SET_AXIS_TYPE`          | 81  | `0x51` | **No** | Set axis type config |

\* Same byte value, different constant name.

---

## Notes

- The `AMC_RESET_SINGLE_KEY` (0x1D) command is absent from our firmware's
  `keychron_raw_hid.h` header. Adding it would require implementing a handler
  in the analog matrix module that clears a single `analog_key_config_t` entry
  back to the global defaults.
- The `AMC_GET/SET_AXIS_TYPE` (0x50/0x51) commands are in a completely
  different sub-command range from the others, suggesting they were added
  in a later firmware revision.
- The `AMC_SET_ACT_ON_AXIS` name for SOCD (0x16) suggests the Launcher may
  treat SOCD and analog axis activation as related concepts.
- The typo `AMC_SET_TRAVAL` (missing 'e') is present in the Launcher's enum
  definition and matches a similar typo (`okmc_traval_config_t`) in the
  firmware C headers.

---

## Related Documents

- [Firmware: Analog Matrix Protocol](../firmware/analog-matrix-protocol.md) -- base protocol docs
- [Firmware: Data Structures](../firmware/data-structures.md) -- analog_key_config_t, okmc_config_t
- [Command Map Comparison](command-map.md) -- full naming comparison
