# Keychron Protocol Documentation

This directory documents the Keychron-specific HID protocols used by Keychron
keyboards, mice, trackballs, and 2.4 GHz dongles. The documentation is derived
from two independent sources:

1. **Firmware** -- the C source code in the `vial-qmk` Keychron fork
2. **Launcher** -- reverse-engineering of the Keychron Launcher web application
   (`launcher.keychron.com`)

## Quick Reference

All Keychron commands are sent as 32-byte HID reports over the raw HID
endpoint (Usage Page `0xFF60`, Usage `0x61`). The first byte (`data[0]`) is
the command ID:

| Command | Value  | Name (firmware)       | Name (Launcher)       | Docs |
|---------|--------|-----------------------|-----------------------|------|
| `0xA0`  | 160    | `KC_GET_PROTOCOL_VERSION` | `KC_GET_PROTOCOL_VERSION` | [firmware](firmware/core-commands.md) |
| `0xA1`  | 161    | `KC_GET_FIRMWARE_VERSION` | `KC_GET_FIRMWARE_VERSION` | [firmware](firmware/core-commands.md) |
| `0xA2`  | 162    | `KC_GET_SUPPORT_FEATURE`  | `KC_GET_SUPPORT_FEATURE`  | [firmware](firmware/core-commands.md) |
| `0xA3`  | 163    | `KC_GET_DEFAULT_LAYER`    | `KC_GET_DEFAULT_LAYER`    | [firmware](firmware/core-commands.md) |
| `0xA7`  | 167    | `KC_MISC_CMD_GROUP`       | `KC_OTHER_SETTING_VERSION` | [firmware](firmware/misc-commands.md) / [launcher](launcher/command-map.md) |
| `0xA8`  | 168    | `KC_KEYCHRON_RGB`         | `KC_RGB`                  | [firmware](firmware/rgb-protocol.md) / [launcher](launcher/command-map.md) |
| `0xA9`  | 169    | `KC_ANALOG_MATRIX`        | `KC_HE`                   | [firmware](firmware/analog-matrix-protocol.md) / [launcher](launcher/analog-matrix-extras.md) |
| `0xAA`  | 170    | `KC_WIRELESS_DFU`         | --                        | [firmware](firmware/wireless-dfu-factory.md) |
| `0xAB`  | 171    | `KC_FACTORY_TEST`         | --                        | [firmware](firmware/wireless-dfu-factory.md) |
| `0xAC`  | 172    | --                        | `KC_SCREEN`               | [launcher](launcher/screen-protocol.md) |

Additional protocol families documented from the Launcher only:

| Family | Commands | Description | Docs |
|--------|----------|-------------|------|
| FR (Forza Receiver) | `0xB1`--`0xBA` | 2.4 GHz dongle / bridge protocol | [launcher](launcher/bridge-dongle-protocol.md) |
| NAPE (mouse/trackball) | sub-cmds under `0xA7` | DPI, gestures, profiles, battery | [launcher](launcher/mouse-protocol.md) |
| DFU (5 paths) | various | STM32, STM32WB, Nordic, XJ, XJ-new | [launcher](launcher/dfu-protocols.md) |
| Bluetooth module DFU | `0xAA` header | BLE module firmware update over USB HID | [launcher](launcher/dfu-protocols.md) |

## Documentation Structure

### `firmware/` -- Derived from QMK C source

These documents describe the protocols as implemented in the `vial-qmk`
Keychron firmware fork and consumed by the `vial-gui` Python client.

| Document | Description |
|----------|-------------|
| [protocol-overview.md](firmware/protocol-overview.md) | Architecture, packet format, initialization sequence, feature detection |
| [core-commands.md](firmware/core-commands.md) | `0xA0`--`0xA3`: protocol version, firmware version, feature flags, default layer |
| [misc-commands.md](firmware/misc-commands.md) | `0xA7`: 19 sub-commands for debounce, snap click, NKRO, language, wireless LPM, report rate, quick start |
| [rgb-protocol.md](firmware/rgb-protocol.md) | `0xA8`: 15 sub-commands for per-key RGB, mixed mode, effect selection |
| [analog-matrix-protocol.md](firmware/analog-matrix-protocol.md) | `0xA9`: 18 sub-commands for Hall Effect switches -- profiles, calibration, rapid trigger |
| [wireless-dfu-factory.md](firmware/wireless-dfu-factory.md) | `0xAA` wireless DFU, `0xAB` factory test mode |
| [data-structures.md](firmware/data-structures.md) | Struct and bitfield layouts used by the firmware |

### `launcher/` -- Reverse-engineered from Keychron Launcher JS

These documents describe protocols found in the Launcher web application that
are **not present** (or only partially present) in our firmware fork.

| Document | Description |
|----------|-------------|
| [overview.md](launcher/overview.md) | Angular app architecture, WebHID transport, HID collections, vendor/product IDs, checksums |
| [command-map.md](launcher/command-map.md) | Complete VIA+KC+AMC+FR+NAPE+RGB enum comparison between firmware and Launcher naming |
| [screen-protocol.md](launcher/screen-protocol.md) | `0xAC` KC_SCREEN: flash filesystem and RTC for screen-equipped keyboards |
| [bridge-dongle-protocol.md](launcher/bridge-dongle-protocol.md) | FR commands `0xB1`--`0xBA`, device slots, VIA tunneling through dongles |
| [dfu-protocols.md](launcher/dfu-protocols.md) | All 5 firmware update paths + Bluetooth module DFU, with full packet formats |
| [mouse-protocol.md](launcher/mouse-protocol.md) | NAPE sub-commands `0x20`--`0x34` for DPI, gestures, profiles, battery |
| [analog-matrix-extras.md](launcher/analog-matrix-extras.md) | Launcher-only AMC commands: reset single key, axis type get/set |
| [wireless-communication.md](launcher/wireless-communication.md) | Sleep/power management, battery, performance modes, dual polling rates, monitor events |

## Key Differences Between Sources

| Aspect | Firmware (our fork) | Launcher |
|--------|---------------------|----------|
| Transport | `hidapi` (USB) + wireless module UART | WebHID API only |
| Buffer size | 32 bytes | 32 bytes (keyboards), 64 bytes (4K/8K mice) |
| Naming | `KC_ANALOG_MATRIX`, `KC_MISC_CMD_GROUP`, `KC_KEYCHRON_RGB` | `KC_HE`, `KC_OTHER_SETTING_VERSION`, `KC_RGB` |
| Protocol versions | v2 only | v2 and v3 (`protocol >= 11`) |
| Screen support | Not present | Full `0xAC` sub-protocol |
| Mouse support | Not present | NAPE commands, polling rate detection, 4K/8K protocols |
| Bridge/dongle | Not present | FR commands, device slots, VIA tunneling |
| Wireless comm | Sleep/poll rate via 0xA7 (partial) | Full sleep, battery, performance modes, dual polling, monitor events |
| DFU | Wireless module DFU (`0xAA`) | 5 distinct DFU paths + BLE module DFU |

## Vendor Information

- **Keychron Vendor ID:** `0x3434` (13364)
- **XJ (Xinji) Vendor ID:** `0x34B7` (13495)
- **HID Usage Page:** `0xFF60` (raw HID), `0x8C` (bridge), `0xFFC1` (mouse 1K), `0xFF0A` (mouse 4K/8K)
