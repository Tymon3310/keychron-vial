# Keychron HID Protocol Overview

This document describes the Keychron-specific HID protocol used for communication
between the Vial GUI (or Vial Web) and Keychron keyboards running the Keychron
fork of QMK+Vial firmware.

> **See also:** The [Launcher docs](../launcher/overview.md) document additional
> protocols reverse-engineered from the Keychron Launcher web app, including
> bridge/dongle communication, mouse protocols, screen commands, and DFU
> procedures not present in our firmware fork.

## Architecture

The Keychron protocol extends the standard VIA/Vial raw HID interface. All
communication uses 32-byte HID reports sent over the same Usage Page
(`0xFF60`) / Usage (`0x61`) endpoint that VIA and Vial already use.

```
 +-----------+        32-byte HID reports         +--------------------+
 |  Vial GUI | <================================> | Keychron QMK+Vial  |
 | (Python)  |   USB HID  /  BLE  /  2.4 GHz     |    firmware        |
 +-----------+                                    +--------------------+
```

### Firmware side

The firmware dispatcher lives in `keyboards/keychron/common/keychron_raw_hid.c`.
When a raw HID report arrives, QMK calls `raw_hid_receive()`. The Keychron
firmware overrides this to first check whether `data[0]` falls in the Keychron
command range (`0xA0`--`0xAB`). If so, it routes to the appropriate handler;
otherwise it falls through to the standard VIA/Vial handler.

Each sub-protocol (RGB, Analog Matrix, Misc, etc.) has its own handler file
under `keyboards/keychron/common/`.

### GUI side

The GUI implements the protocol in `protocol/keychron.py` as the
`ProtocolKeychron` mixin class, which is mixed into the `Keyboard` class
alongside VIA/Vial protocol mixins.

### Transport

The `kc_raw_hid_send()` function in the firmware abstracts transport:

- **USB (wired)**: Uses ChibiOS `chnWrite` on the raw HID channel.
- **Bluetooth / 2.4 GHz**: Routes through `lkbt51` or `ckbt51` wireless
  module drivers, which forward HID reports over UART to the wireless SoC.

On the GUI side, USB is accessed via `hidapi` and the WebUSB API (for
`vial-web`). Wireless connections appear as USB HID once the dongle/receiver
is plugged in.

## Packet Format

All packets are 32 bytes. The general format is:

```
Byte   Field         Description
----   -----         -----------
 0     command       Top-level command ID (0xA0 -- 0xAB)
 1     sub_command   Sub-command within the protocol group (where applicable)
 2     status/data   Response status (0 = success, 1 = fail) or first data byte
 3..31 data          Command-specific payload
```

### Request (Host -> Keyboard)

The host sends a 32-byte packet with `data[0]` set to the command ID.
For commands that have sub-commands (0xA7, 0xA8, 0xA9), `data[1]` is the
sub-command. Remaining bytes carry parameters.

### Response (Keyboard -> Host)

The keyboard echoes the command and sub-command bytes, followed by a status
code and response data. The status byte position varies slightly between
sub-protocols:

- **Misc (0xA7)**: `data[2]` = status (0 = success)
- **RGB (0xA8)**: `data[2]` = status (0 = success)
- **Analog Matrix (0xA9)**: `data[2]` = status for most commands;
  `AMC_GET_PROFILE_RAW` echoes the profile index instead
- **Core commands (0xA0--0xA3)**: No explicit status byte; `data[0] == 0xFF`
  indicates the command is unsupported

### Error Handling

If a command is not recognized (e.g., the keyboard does not support Keychron
extensions), the firmware returns `data[0] = 0xFF`. The GUI uses this to
detect non-Keychron keyboards during the initial handshake.

## Command Map

| Command | Value  | Description                         | Details                           |
|---------|--------|-------------------------------------|-----------------------------------|
| `KC_GET_PROTOCOL_VERSION` | `0xA0` | Get Keychron protocol version | [core-commands.md](core-commands.md) |
| `KC_GET_FIRMWARE_VERSION` | `0xA1` | Get firmware version string   | [core-commands.md](core-commands.md) |
| `KC_GET_SUPPORT_FEATURE`  | `0xA2` | Get feature flags bitmask     | [core-commands.md](core-commands.md) |
| `KC_GET_DEFAULT_LAYER`    | `0xA3` | Get DIP switch default layer  | [core-commands.md](core-commands.md) |
| `KC_MISC_CMD_GROUP`       | `0xA7` | Misc command group (19 sub-commands) | [misc-commands.md](misc-commands.md) |
| `KC_KEYCHRON_RGB`         | `0xA8` | Keychron RGB sub-protocol (15 sub-commands) | [rgb-protocol.md](rgb-protocol.md) |
| `KC_ANALOG_MATRIX`        | `0xA9` | Analog Matrix / Hall Effect (18 sub-commands) | [analog-matrix-protocol.md](analog-matrix-protocol.md) |
| `KC_WIRELESS_DFU`         | `0xAA` | Wireless module DFU           | [wireless-dfu-factory.md](wireless-dfu-factory.md) |
| `KC_FACTORY_TEST`         | `0xAB` | Factory test mode             | [wireless-dfu-factory.md](wireless-dfu-factory.md) |
| `KC_SCREEN`               | `0xAC` | Screen/flash filesystem       | [Launcher docs](../launcher/screen-protocol.md) (not in our firmware fork) |

## Initialization Sequence

When the GUI connects to a keyboard, `reload_keychron()` runs the following
handshake:

1. **Protocol version** (`0xA0`) -- if `data[0] == 0xFF`, abort (not Keychron)
2. **Feature flags** (`0xA2`) -- 2-byte bitmask of supported features
3. **Firmware version** (`0xA1`) -- null-terminated ASCII string
4. **MCU info** (`0xA7 / 0x02`) -- chip name (e.g. "STM32L432")
5. **Misc protocol version** (`0xA7 / 0x01`) -- misc sub-protocol version + feature flags
6. **Per-feature reload** -- for each feature flag that is set, the
   corresponding `_reload_*` method fetches the current settings

## Feature Detection

Features are detected via two independent bitmasks:

### Primary feature flags (`0xA2`, 2 bytes)

| Bit   | Constant               | Feature                |
|-------|------------------------|------------------------|
| 0     | `FEATURE_DEFAULT_LAYER`| DIP switch layer query |
| 1     | `FEATURE_BLUETOOTH`    | Bluetooth connectivity |
| 2     | `FEATURE_P24G`         | 2.4 GHz connectivity   |
| 3     | `FEATURE_ANALOG_MATRIX`| Hall Effect switches   |
| 4     | `FEATURE_STATE_NOTIFY` | State change notifications |
| 5     | `FEATURE_DYNAMIC_DEBOUNCE` | Dynamic debounce   |
| 6     | `FEATURE_SNAP_CLICK`   | Snap Click (SOCD)      |
| 7     | `FEATURE_KEYCHRON_RGB` | Per-key / Mixed RGB    |
| 8     | `FEATURE_QUICK_START`  | Quick start mode       |
| 9     | `FEATURE_NKRO`         | N-Key Rollover toggle  |

### Misc feature flags (`0xA7 / 0x01`, 2 bytes)

| Bit   | Constant             | Feature              |
|-------|----------------------|----------------------|
| 0     | `MISC_DFU_INFO`     | DFU/MCU info query   |
| 1     | `MISC_LANGUAGE`      | Language setting      |
| 2     | `MISC_DEBOUNCE`      | Dynamic debounce     |
| 3     | `MISC_SNAP_CLICK`    | Snap Click (SOCD)    |
| 4     | `MISC_WIRELESS_LPM`  | Wireless LPM settings |
| 5     | `MISC_REPORT_RATE`   | USB report rate      |
| 6     | `MISC_QUICK_START`   | Quick start mode     |
| 7     | `MISC_NKRO`          | NKRO toggle          |

Some features can be indicated by either bitmask (the GUI checks both with
an OR). For example, debounce is supported if `FEATURE_DYNAMIC_DEBOUNCE` is
set in the primary flags **or** `MISC_DEBOUNCE` is set in the misc flags.

## Related Documents

### Firmware Protocol
- [Core Commands](core-commands.md) -- 0xA0--0xA3
- [Misc Commands](misc-commands.md) -- 0xA7 sub-protocol
- [RGB Protocol](rgb-protocol.md) -- 0xA8 sub-protocol
- [Analog Matrix Protocol](analog-matrix-protocol.md) -- 0xA9 sub-protocol
- [Wireless DFU & Factory Test](wireless-dfu-factory.md) -- 0xAA, 0xAB
- [Data Structures](data-structures.md) -- struct/bitfield reference

### Launcher Reverse-Engineering
- [Launcher Overview](../launcher/overview.md) -- architecture and transport
- [Command Map Comparison](../launcher/command-map.md) -- firmware vs Launcher naming
- [Screen Protocol](../launcher/screen-protocol.md) -- 0xAC (not in our firmware)
- [Bridge / Dongle Protocol](../launcher/bridge-dongle-protocol.md) -- FR commands
- [DFU Protocols](../launcher/dfu-protocols.md) -- all 5 firmware update paths
- [Mouse / Trackball Protocol](../launcher/mouse-protocol.md) -- NAPE commands
- [Analog Matrix Extras](../launcher/analog-matrix-extras.md) -- Launcher-only AMC commands
