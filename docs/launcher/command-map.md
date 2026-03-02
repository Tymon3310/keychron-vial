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
| 171   | `0xAB` | `KC_FACTORY_TEST`        | *(not in enum)*            | Factory-only |
| 172   | `0xAC` | *(not in firmware)*      | `KC_SCREEN`                | **Launcher only** -- see [screen-protocol.md](screen-protocol.md) |

### Key Observations

1. **`KC_MISC_CMD_GROUP` (0xA7)**: The Launcher and firmware now use the same name `KC_MISC_CMD_GROUP`. In older versions, the Launcher called this `KC_OTHER_SETTING_VERSION`, which hints that the command was originally for querying the "other settings" protocol version (sub-cmd `0x01`), and the misc sub-commands were added later.

2. **`KC_HE` (0xA9)**: The Launcher calls this "Hall Effect" (`HE`), while the
   firmware uses "Analog Matrix" (`ANALOG_MATRIX`). Both refer to the same
   Hall Effect switch protocol.

3. **`KC_SCREEN` (0xAC)**: This command is **only in the Launcher** -- it does
   not exist in our firmware fork. It controls an external flash filesystem
   and RTC clock for keyboards with built-in screens/displays. See
   [screen-protocol.md](screen-protocol.md).

4. **`0xAA` and `0xAB`**: These are not in the Launcher's VIA command enum.
   The Launcher handles Bluetooth DFU and factory test through dedicated
   classes that construct packets independently, not through the VIA command
   dispatcher.

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
`0x20`--`0x34`. Not present in our firmware fork:

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

See [mouse-protocol.md](mouse-protocol.md).

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
