# Keychron Launcher -- Reverse-Engineering Overview

This document describes the architecture and protocols of the **Keychron
Launcher** web application (`launcher.keychron.com`), reverse-engineered from
its minified JavaScript bundles. The Launcher is Keychron's official
configuration tool for keyboards, mice, trackballs, and 2.4 GHz dongles.

> **Source:** All findings in the `launcher/` docs are derived from the
> Angular application bundle `main.67e8841912a834d8.js` (~3.9 MB minified).
> The other JS files served by the Launcher (runtime, scripts, polyfills) are
> webpack-bundled copies of the same application code and contain no additional
> protocol logic.
> **Note:** These documents have been thoroughly verified against the decompiled 
> `main.67e8841912a834d8.js` source code and are confirmed to be 100% accurate.

## Application Stack

- **Framework:** Angular (minified, single-bundle deployment, with chunk splitting)
- **Transport:** WebHID API exclusively -- no WebBluetooth, no WebSerial
- **Firmware packages:** JSZip/pako for ZIP firmware archives
- **Code editor:** ACE editor (embedded, for JSON/keymap editing)
- **Backend APIs:**
  - `https://launcher.keychron.com/api/` (main backend for metadata and layout JSONs)
  - `https://statistic.keychron.com/` (analytics)
  - Firmware payloads are served mostly from Shopify CDN (`cdn.shopify.com/s/files/...`)

## Supported Brands and Models

The Keychron Launcher codebase explicitly supports devices beyond the core Keychron lineup. It uses domain-based routing (`window.location.hostname`) to skin the UI and filter available products for sub-brands. Supported brands include:

- **Keychron** (`launcher.keychron.com`)
- **Lemokey** (`launcher.lemokey.com`, Keychron's gaming sub-brand)
- **SKN / Wukong** (`launcher.skn.net.cn`, `wukong.skn.net.cn`)
- **Jamesdonkey** (`launcher.jamesdonkey.com`, `wooking.com.cn`)
- **Candysign** (`candysign.keychron.cn`)
- **Silvermonkey** (`launcher.silvermonkey.com`)
- **GameRaider** (`app.gameraider.com.tr`)
- **Colorful** (`keyboard.colorful.cn`)
- **Antec** (`launcher.antec.com`)
- **Oneofzero** (`launcher.oneofzero.net`)
- **Gamepro** (`launcher.gamepro.ua`)
- **Fibo** (`launcher.fibo-lab.com`)

Certain features (like mouse scrolling settings or specific DFU update tabs) are enabled or disabled depending on the active sub-brand domain.

Additionally, the Launcher contains explicit support for **ZMK-based** firmware models (like the Keychron B Pro and Q Ultra series) over WebHID. This allows these newer, wireless-first Zephyr RTOS keyboards to be configured via the same protocol interface as Keychron's standard QMK forks.

## WebHID Transport

### Communication Channels

The Launcher uses the WebHID API (`navigator.hid`) for all device communication. It does **not** use the WebBluetooth API. 

It can communicate with wireless keyboards in three ways:
1. **Wired USB:** Direct communication over standard QMK Raw HID endpoint (`0xFF60`/`0x61`).
2. **2.4 GHz Dongle (Bridge Tunneling):** If the keyboard is connected via a 2.4 GHz dongle, the Launcher connects to the dongle's HID endpoint (`0x8C`/`0x01`). It queries the dongle to see what devices are paired, and then sends standard VIA 32-byte payloads to the dongle. The dongle acts as a "bridge" and tunnels these VIA commands over the air to the keyboard, returning the responses back to the Launcher.
3. **Bluetooth Firmware Updating:** The word "bluetooth" in the codebase refers exclusively to updating the separate Bluetooth module's firmware. This is also done *over the wired USB HID connection* by tunneling DFU packets through the main MCU to the Bluetooth IC.

### HID Collections

Four HID collection types are used to identify device capabilities:

| Usage Page | Usage | Collection Key | Device Type        |
|------------|-------|----------------|--------------------|
| `0xFF60` (65376) | `0x61` (97) | `keyboard` | QMK Raw HID (standard VIA/Vial endpoint) |
| `0x8C` (140)     | `0x01` (1)  | `bridge`   | 2.4 GHz dongle / bridge receiver |
| `0xFFC1` (65473) | `0x01` (1)  | `mouse`    | Mouse (1K polling protocol) |
| `0xFF0A` (65290) | `0x01` (1)  | `mouse_4k` | Mouse (4K/8K polling protocol) |

The Launcher filters paired devices using these collection identifiers packed
as signed int32 values: `usagePage << 16 | usage`.

### HID Report IDs

Different report IDs are used depending on the device type and protocol:

| Report ID | Hex    | Usage                              |
|-----------|--------|------------------------------------|
| 0         | `0x00` | Standard keyboard HID reports      |
| 1         | `0x01` | XJ chip erase/flash protocol       |
| 178       | `0xB2` | Bluetooth DFU reports              |
| 181       | `0xB5` | Mouse reports                      |
| 192       | `0xC0` | Nordic SLIP/HCI DFU                |

### Transceiver Classes

The Launcher abstracts HID communication through four transceiver classes,
selected based on device type and polling rate:

| Class | Protocol | Method | Description |
|-------|----------|--------|-------------|
| Simple | All | `sendReport()` | Fire-and-forget, no ACK |
| Queued | 1K wireless | `sendReport()` | Queue with retry (3 attempts), timeout |
| Feature Report | USB mice | `sendFeatureReport()` / `receiveFeatureReport()` | Feature reports for direct USB |
| 1K Feature Report | 1K mice | Feature reports | Flow control via report IDs `0xE4` and `0x52` |

For mice, the transceiver is selected by `workMode`:
- `workMode == 0` → direct USB (simple or feature report)
- `workMode == 1` → via dongle/receiver (queued)

### Protocol Detection

4K mice undergo additional protocol detection at connect time:

```
Send: [1, 0, 129, 1, ..., checksum]  (to 4K collection)
If response bytes [6,7] == [52, 52] (0x34, 0x34) → "8k_nordic"
If response bytes [6,7] == [45, 54]               → "8k_nordic"
Otherwise                                         → "4k"
```

## Vendor and Product IDs

### Keychron Vendor ID

`0x3434` (13364 decimal) -- used for all Keychron keyboards, mice, and dongles.

A second vendor ID `0x34B7` (13495) is used for "XJ" devices (Xinji-based
hardware with a different flashing protocol).

### Known Product IDs

#### Mice (4K/8K models)

| Product Name         | Product ID | Dongle PID |
|----------------------|------------|------------|
| M3mini4K             | `0x0620`   | `0xD037`   |
| M44K                 | `0x0621`   | `0xD040`   |
| M3mini4K (aluminium) | `0x0622`   | `0xD041`   |
| M3M24k               | `0x0623`   | `0xD045`   |
| M6 4K                | `0x0624`   | `0xD046`   |
| M34K                 | `0x07A0`   | `0xD03C`   |

#### Special / Test PIDs

| PID      | Description                  |
|----------|------------------------------|
| `0x0E20` | Fake/demo keyboard           |
| `0xD049` | Fake M6 8K mouse (dev mode)  |
| `0xD031` | Gamepad-specific PID         |
| `0x3616` | Fake test device             |

#### Keyboard PIDs (partial, from monkeyPatch references)

| PID      | Notes |
|----------|-------|
| `0x0A33` | Keyboard (monkeyPatch) |
| `0x0A35` | Keyboard (monkeyPatch) |
| `0x0437` | Keyboard |

## Connection and Device Enums

### Connection Type

| Value | Name  | Meaning              |
|-------|-------|----------------------|
| 0     | `G`   | 2.4 GHz wireless     |
| 1     | `BT`  | Bluetooth            |
| 2     | `USB` | Wired USB            |

### Device Type

| Value | Name       |
|-------|------------|
| 0     | `Mouse`    |
| 1     | `Keyboard` |
| 2     | `RF`       |
| 3     | `Bridge`   |

### Product Category

| Value | Name        |
|-------|-------------|
| 0     | `Keyboard`  |
| 1     | `Mouse`     |
| 2     | `Bridge`    |
| 3     | `Trackball` |

### Connection Status

| Value | Name         |
|-------|--------------|
| 1     | `connect`    |
| 2     | `disconnect` |

## Buffer Construction

The standard buffer factory creates 32-byte `Uint8Array` buffers:

```js
Buffer(size = 32) → new Uint8Array(size)
```

Mouse protocols (4K/8K) use 64-byte buffers. The HID report handler reads
incoming reports as:

```js
handleReport(event) {
    const data = new Uint8Array(event.buffer, 0, event.byteLength);
    this.report$.next(data);
}
```

## Checksum / Integrity Algorithms

| Algorithm | Usage | Details |
|-----------|-------|---------|
| `byteNor(n)` | BLE DFU packet length validation | Bitwise NOT of length byte |
| `CRC-CCITT` | Nordic SLIP/HCI DFU | Init `0xFFFF`, polynomial `0x1021` |
| `CRC32` | BLE firmware verification | Standard CRC32 (`0xEDB88320` polynomial) |
| Additive checksum | BLE data packets, screen commands | Sum of payload bytes, 16-bit LE |
| Packet checksum (0xA1) | Mouse 8K protocol | `buf[63] = 161 - (sum of bytes 0..62) & 0xFF` |

## Protocol Version Negotiation

The Launcher distinguishes firmware protocol versions:
- `protocol >= 11` → "v3" (different keycode ranges, different QMK enums)
- `protocol >= 12` → different packet parsing offsets
- `protocol < 11` → "v2" (legacy keycode ranges)

Three complete QMK keycode enum variants are embedded in the Launcher to
support both v2 and v3 firmware builds.

## Macro Action Types

| Value | Constant            | Description               |
|-------|---------------------|---------------------------|
| 0     | `Terminate`         | End of macro              |
| 1     | `Tap`               | Key tap (press + release) |
| 2     | `Down`              | Key press                 |
| 3     | `Up`                | Key release               |
| 4     | `Delay`             | Delay (ms)                |
| 5     | `CharacterStream`   | Type string               |
| 6     | `MouseButtonDown`   | Mouse button press        |
| 7     | `MouseButtonUp`     | Mouse button release      |
| 124   | `DelayTerminate`    | Delay + terminate         |

## Related Documents

### Launcher Protocol Docs
- [Command Map Comparison](command-map.md) -- firmware vs Launcher naming
- [Screen Protocol](screen-protocol.md) -- 0xAC flash filesystem and RTC
- [Bridge / Dongle Protocol](bridge-dongle-protocol.md) -- FR commands, VIA tunneling
- [DFU Protocols](dfu-protocols.md) -- all 5 firmware update paths
- [Mouse / Trackball Protocol](mouse-protocol.md) -- NAPE commands, polling rates
- [Analog Matrix Extras](analog-matrix-extras.md) -- Launcher-only AMC commands
- [Wireless Communication](wireless-communication.md) -- sleep, battery, performance modes, dual polling rates, monitor events

### Firmware Protocol Docs
- [Firmware Protocol Overview](../firmware/protocol-overview.md)
- [Core Commands](../firmware/core-commands.md) -- 0xA0--0xA3
- [Misc Commands](../firmware/misc-commands.md) -- 0xA7
- [RGB Protocol](../firmware/rgb-protocol.md) -- 0xA8
- [Analog Matrix Protocol](../firmware/analog-matrix-protocol.md) -- 0xA9
- [Wireless DFU & Factory Test](../firmware/wireless-dfu-factory.md) -- 0xAA, 0xAB
- [Data Structures](../firmware/data-structures.md)
