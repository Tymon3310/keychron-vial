# Misc Command Group (0xA7)

The Misc command group (`KC_MISC_CMD_GROUP = 0xA7`) bundles a variety of
settings sub-commands under a single top-level command ID. The sub-command
is specified in `data[1]`.

> **Note:** The Launcher uses the name `KC_OTHER_SETTING_VERSION` for this
> command (0xA7). It also defines additional sub-commands (`0x20`--`0x34`)
> for mouse/trackball devices ("NAPE" commands). See
> [Mouse / Trackball Protocol](../launcher/mouse-protocol.md) and
> [Command Map Comparison](../launcher/command-map.md).

All responses in this group follow the pattern:

```
Byte  Field           Description
----  -----           -----------
 0    0xA7            Command echo
 1    sub_command     Sub-command echo
 2    status          0 = success, 1 = fail
 3..  payload         Sub-command-specific data
```

## Sub-Command Map

| Sub-cmd | Value  | Direction | Description               |
|---------|--------|-----------|---------------------------|
| `MISC_GET_PROTOCOL_VER`  | `0x01` | Read  | Get misc protocol version + feature flags |
| `DFU_INFO_GET`           | `0x02` | Read  | Get MCU chip info         |
| `LANGUAGE_GET`           | `0x03` | Read  | Get language setting      |
| `LANGUAGE_SET`           | `0x04` | Write | Set language setting      |
| `DEBOUNCE_GET`           | `0x05` | Read  | Get debounce settings     |
| `DEBOUNCE_SET`           | `0x06` | Write | Set debounce settings     |
| `SNAP_CLICK_GET_INFO`    | `0x07` | Read  | Get snap click pair count |
| `SNAP_CLICK_GET`         | `0x08` | Read  | Get snap click entries    |
| `SNAP_CLICK_SET`         | `0x09` | Write | Set snap click entry      |
| `SNAP_CLICK_SAVE`        | `0x0A` | Write | Save snap click to EEPROM |
| `WIRELESS_LPM_GET`       | `0x0B` | Read  | Get wireless LPM settings |
| `WIRELESS_LPM_SET`       | `0x0C` | Write | Set wireless LPM settings |
| `REPORT_RATE_GET`        | `0x0D` | Read  | Get USB report rate       |
| `REPORT_RATE_SET`        | `0x0E` | Write | Set USB report rate       |
| `DIP_SWITCH_GET`         | `0x0F` | Read  | Get DIP switch state      |
| `DIP_SWITCH_SET`         | `0x10` | Write | Set DIP switch state      |
| `FACTORY_RESET`          | `0x11` | Write | Reset all Keychron settings to defaults |
| `NKRO_GET`               | `0x12` | Read  | Get NKRO state            |
| `NKRO_SET`               | `0x13` | Write | Set NKRO enabled/disabled |

---

## 0x01 -- Get Misc Protocol Version

Returns the misc sub-protocol version number and a feature flags bitmask
that indicates which misc sub-commands are available.

### Request

```
[0xA7] [0x01]
```

### Response

```
Byte  Field                 Description
----  -----                 -----------
 3    version_low           Protocol version LSB
 4    version_high          Protocol version MSB
 5    features_low          Misc feature flags byte 0
 6    features_high         Misc feature flags byte 1
```

### Misc Feature Flags

| Bit | Mask   | Constant            | Feature                |
|-----|--------|---------------------|------------------------|
| 0   | `0x01` | `MISC_DFU_INFO`    | MCU/DFU information    |
| 1   | `0x02` | `MISC_LANGUAGE`     | Language setting       |
| 2   | `0x04` | `MISC_DEBOUNCE`     | Dynamic debounce       |
| 3   | `0x08` | `MISC_SNAP_CLICK`   | Snap Click (SOCD)      |
| 4   | `0x10` | `MISC_WIRELESS_LPM` | Wireless power mgmt    |
| 5   | `0x20` | `MISC_REPORT_RATE`  | USB report rate        |
| 6   | `0x40` | `MISC_QUICK_START`  | Quick start mode       |
| 7   | `0x80` | `MISC_NKRO`         | NKRO toggle            |

Current protocol version: `0x0002`.

---

## 0x02 -- DFU Info Get

Returns MCU chip information for firmware update purposes.

### Request

```
[0xA7] [0x02]
```

### Response

```
Byte  Field           Description
----  -----           -----------
 2    status          0 = success
 3    info_type       1 = DFU_INFO_CHIP
 4    length          Length of chip name string
 5..  chip_name       ASCII chip name (e.g. "STM32L432")
```

### Notes

- The chip name is used to determine DFU flashing compatibility.
- If the name contains "STM32", the GUI enables the DFU flasher tab.

---

## 0x05 / 0x06 -- Debounce Get / Set

### Debounce Get Request

```
[0xA7] [0x05]
```

### Debounce Get Response

```
Byte  Field           Description
----  -----           -----------
 2    status          0 = success
 3    (reserved)      Always 0 (QMK marker)
 4    debounce_type   Debounce algorithm type (0-6)
 5    debounce_time   Debounce time in milliseconds
```

### Debounce Set Request

```
[0xA7] [0x06] [debounce_type] [debounce_time]
```

### Debounce Types

| Value | Constant                              | Name                              |
|-------|---------------------------------------|-----------------------------------|
| 0     | `DEBOUNCE_SYM_DEFER_GLOBAL`          | Symmetric Defer (Global)          |
| 1     | `DEBOUNCE_SYM_DEFER_PER_ROW`        | Symmetric Defer (Per Row)         |
| 2     | `DEBOUNCE_SYM_DEFER_PER_KEY`        | Symmetric Defer (Per Key)         |
| 3     | `DEBOUNCE_SYM_EAGER_PER_ROW`        | Symmetric Eager (Per Row)         |
| 4     | `DEBOUNCE_SYM_EAGER_PER_KEY`        | Symmetric Eager (Per Key)         |
| 5     | `DEBOUNCE_ASYM_EAGER_DEFER_PER_KEY` | Asymmetric Eager-Defer (Per Key)  |
| 6     | `DEBOUNCE_NONE`                      | None (no debouncing)              |

### Notes

- The debounce type is read-only in the sense that it is determined by the
  firmware build configuration. The `DEBOUNCE_SET` command changes the debounce
  time, not the algorithm.
- The debounce time is persisted to EEPROM by the firmware immediately.

---

## 0x07 / 0x08 / 0x09 / 0x0A -- Snap Click (SOCD)

Snap Click provides Simultaneous Opposing Cardinal Directions (SOCD) handling
for regular (non-analog) keyboards.

### Get Info (0x07)

```
Request:  [0xA7] [0x07]
Response: data[2]=status, data[3]=pair_count
```

### Get Entries (0x08)

Reads snap click pair entries in batches.

```
Request:  [0xA7] [0x08] [start_index] [count]
Response: data[2]=status, data[3..]=entries (3 bytes each)
```

Each entry is 3 bytes: `[type] [key1] [key2]`

### Set Entry (0x09)

```
Request:  [0xA7] [0x09] [index] [1] [type] [key1] [key2]
```

### Save (0x0A)

Persists snap click settings to EEPROM.

```
Request:  [0xA7] [0x0A]
```

### Snap Click Types

| Value | Constant                    | Name                                    |
|-------|-----------------------------|-----------------------------------------|
| 0     | `SNAP_CLICK_TYPE_NONE`     | Disabled                                |
| 1     | `SNAP_CLICK_TYPE_REGULAR`  | Last Key Priority (simple)              |
| 2     | `SNAP_CLICK_TYPE_LAST_INPUT` | Last Key Priority (re-activates held) |
| 3     | `SNAP_CLICK_TYPE_FIRST_KEY`  | Absolute Priority: Key 1              |
| 4     | `SNAP_CLICK_TYPE_SECOND_KEY` | Absolute Priority: Key 2             |
| 5     | `SNAP_CLICK_TYPE_NEUTRAL`   | Cancel (both keys cancel out)          |

---

## 0x0B / 0x0C -- Wireless LPM (Low Power Mode)

Controls backlight timeout and idle timeout for wireless operation.

### Get (0x0B)

```
Request:  [0xA7] [0x0B]
Response: data[2]=status, data[3..4]=backlit_time (LE16), data[5..6]=idle_time (LE16)
```

### Set (0x0C)

```
Request:  [0xA7] [0x0C] [backlit_time LE16] [idle_time LE16]
```

- `backlit_time`: Seconds before backlight turns off (minimum 5)
- `idle_time`: Seconds before keyboard enters idle/sleep (minimum 60)
- Both values are 16-bit little-endian unsigned integers.

---

## 0x0D / 0x0E -- Report Rate

Controls the USB polling rate.

### Get (0x0D)

**Version 1 (Single Rate)**:
```
Request:  [0xA7] [0x0D]
Response: data[2]=status, data[3]=rate_index, data[4]=supported_mask
```

**Version 2 (Dual Rate)**:
```
Request:  [0xA7] [0x0D]
Response: data[2]=status, data[3]=usb_rate_index, data[4]=usb_supported_mask, data[5]=fr_supported_mask, data[6]=fr_rate_index
```

### Set (0x0E)

**Version 1 (Single Rate)**:
```
Request:  [0xA7] [0x0E] [rate_index]
```

**Version 2 (Dual Rate)**:
```
Request:  [0xA7] [0x0E] [usb_rate_index] [fr_rate_index]
```

### Rate Index Values

| Value | Constant              | Rate      |
|-------|-----------------------|-----------|
| 0     | `REPORT_RATE_8000HZ` | 8000 Hz   |
| 1     | `REPORT_RATE_4000HZ` | 4000 Hz   |
| 2     | `REPORT_RATE_2000HZ` | 2000 Hz   |
| 3     | `REPORT_RATE_1000HZ` | 1000 Hz   |
| 4     | `REPORT_RATE_500HZ`  | 500 Hz    |
| 5     | `REPORT_RATE_250HZ`  | 250 Hz    |
| 6     | `REPORT_RATE_125HZ`  | 125 Hz    |

### Notes

- `supported_mask` is a 7-bit bitmask where bit N indicates rate index N is
  supported. Default is `0x7F` (all rates).
- The GUI checks `(mask >> rate_index) & 1` before allowing selection.

---

## 0x12 / 0x13 -- NKRO

Controls N-Key Rollover mode.

### Get (0x12)

```
Request:  [0xA7] [0x12]
Response: data[2]=status, data[3]=flags
```

Flags byte:
- Bit 0: NKRO enabled (1) / disabled (0)
- Bit 1: NKRO supported by hardware
- Bit 2: NKRO adaptive (automatically determined, not user-settable)

### Set (0x13)

```
Request:  [0xA7] [0x13] [enabled: 0 or 1]
```

### Notes

- When `adaptive` flag (bit 2) is set, the keyboard automatically selects
  6KRO or NKRO based on connection type. The GUI disables the NKRO toggle
  in this case.

---

## 0x11 -- Factory Reset

Resets all Keychron-specific settings to factory defaults.

```
Request:  [0xA7] [0x11]
Response: data[2]=status
```

This clears debounce, snap click, RGB, and analog settings stored in EEPROM.
