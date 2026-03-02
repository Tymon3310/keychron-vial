# Wireless Communication

This document describes the wireless-specific communication protocols used by
the Keychron Launcher for Bluetooth, 2.4 GHz, and USB mode management. It
covers power management, battery reporting, connection mode switching,
performance modes, dual polling rates, and real-time wireless monitoring.

> **Source:** Reverse-engineered from the Keychron Launcher JavaScript bundle.
> See [Bridge / Dongle Protocol](bridge-dongle-protocol.md) for the FR command
> set and VIA tunneling. See [DFU Protocols](dfu-protocols.md) for wireless
> firmware updates.

## Connection Modes

Keychron wireless devices support three connection modes, represented by the
`LJ` enum:

| Value | Name  | Meaning                          |
|-------|-------|----------------------------------|
| 0     | `G`   | 2.4 GHz wireless (via dongle)    |
| 1     | `BT`  | Bluetooth Low Energy             |
| 2     | `USB` | Wired USB                        |

For mice, the `workMode` property is used instead:

| workMode | Transport  | Transceiver class         |
|----------|-----------|---------------------------|
| 0        | USB       | Direct (fire-and-forget)  |
| 1        | 2.4 GHz   | Queued (ACK-based retry)  |

### Connection Mode Detection

The Launcher reads connection state from keyboard reports:

```
report byte[N] == 2  →  USB
report byte[N] != 2  →  2.4G / wireless
```

### Menu Visibility by Connection State

| Menu path              | Requires                                        |
|------------------------|-------------------------------------------------|
| `firmware/flash`       | `state === USB`                                 |
| `firmware/bluetooth`   | `state === USB` AND `support_bluetooth_update`  |
| `firmware/frequency`   | `state === G` (2.4 GHz)                         |

---

## Wireless Keycodes

These keycodes switch the keyboard's connection mode or display battery
status. They are sent as key events from dedicated physical keys or function
layer bindings.

| Keycode | Hex      | Name  | Function                |
|---------|----------|-------|-------------------------|
| 32267   | `0x7E0B` | BTH1  | Switch to Bluetooth Host 1 |
| 32268   | `0x7E0C` | BTH2  | Switch to Bluetooth Host 2 |
| 32269   | `0x7E0D` | BTH3  | Switch to Bluetooth Host 3 |
| 32270   | `0x7E0E` | 2.4G  | Switch to 2.4 GHz mode  |
| 32271   | `0x7E0F` | Batt  | Display battery level   |

Standard HID power keys:
- `KC_PWR` = 165, `KC_SLEP` = 166, `KC_WAKE` = 167

---

## Feature Support Flags

Before using wireless features, the Launcher queries the keyboard's support
bitmap via `KC_GET_SUPPORT_FEATURE` (`0xA2`). The response is a multi-byte
bitmask parsed bit-by-bit:

| Bit Position | Flag Name            | Description                          |
|-------------|----------------------|--------------------------------------|
| `[1][1]`    | `Support_Rf`         | RF (wireless 2.4 GHz) support       |
| `[8]`       | `Support_Sleep`      | Sleep / power management             |
| `[9]`       | `Support_PollRate`   | Polling rate adjustment              |
| `[17]`      | `Performance_Mode`   | Performance mode selection           |
| `[1][0]`    | `Support_Le`         | LED control support                  |
| `[6]`       | `Support_DEBOUNCE`   | Debounce adjustment                  |
| `[7]`       | `Support_Snap`       | Snap Tap / SOCD support              |
| `[16]`      | `Bootloader_Jump`    | Jump-to-bootloader command           |
| `[18]`      | `Support_Toggle`     | Toggle key support                   |
| `[12]`      | `Upload_Default_Layer` | Upload default layer support       |

The `Support_Rf` flag gates all wireless-specific settings in the Launcher UI.

---

## Sleep / Power Management (Keyboards)

Wireless keyboards have configurable sleep behavior to conserve battery. These
commands use `KC_MISC_CMD_GROUP` (`0xA7`) sub-commands `0x0B` (11) and `0x0C` (12).

### Get Sleep (sub-command 0x0B)

#### Request

```
Byte  Value
----  -----
 0    0xA7     KC_MISC_CMD_GROUP
 1    0x0B     Get_Sleep (11)
```

#### Response

```
Byte  Field           Description
----  -----           -----------
 0    0xA7            Echo
 1    0x0B            Echo
 3-4  backlight       Backlight timeout (LE16, seconds)
 5-6  sleep           Sleep timeout (LE16, seconds)
 7-8  magnetScan      Magnet scan interval (LE16, seconds, HE keyboards only)
```

Parsed as:
```
backlight  = report[4] << 8 | report[3]
sleep      = report[6] << 8 | report[5]
magnetScan = report[8] << 8 | report[7]
```

### Set Sleep (sub-command 0x0C)

#### Request

```
Byte  Value
----  -----
 0    0xA7     KC_MISC_CMD_GROUP
 1    0x0C     Set_Sleep (12)
 2-3  backlight    Backlight timeout (LE16, seconds)
 4-5  sleep        Sleep timeout (LE16, seconds)
 6-7  magnetScan   Magnet scan interval (LE16, seconds)
```

### Validation Rules

- Sleep timeout minimum: **60 seconds**
- Backlight timeout minimum: **5 seconds**
- Backlight timeout must be ≤ sleep timeout
- If magnet scan is enabled: sleep > magnetScan, backlight < magnetScan

---

## Performance Mode

Wireless keyboards with the `Performance_Mode` feature flag support three
power/performance presets:

| Value | Mode          | Description                                |
|-------|---------------|--------------------------------------------|
| 0     | Battery       | Maximum battery life, reduced polling rate  |
| 1     | Normal        | Balanced power and responsiveness           |
| 2     | Performance   | Maximum polling rate, lowest latency        |

### Get Performance (sub-command 17)

```
Byte  Value
----  -----
 0    0xA7     KC_MISC_CMD_GROUP
 1    17       Performance Get
```

### Set Performance (sub-command 18)

```
Byte  Value
----  -----
 0    0xA7     KC_MISC_CMD_GROUP
 1    18       Performance Set
 2    mode     0 = Battery, 1 = Normal, 2 = Performance
```

---

## Dual Polling Rate (USB vs 2.4 GHz)

Wireless keyboards can have **separate polling rates** for USB and 2.4 GHz
connections. Two polling rate protocol versions exist.

### Available Rates

| Index | Rate (Hz) |
|-------|-----------|
| 0     | 8000      |
| 1     | 4000      |
| 2     | 2000      |
| 3     | 1000      |
| 4     | 500       |
| 5     | 250       |
| 6     | 125       |

### Poll Rate Detect (sub-command 1)

Determines the polling rate protocol version:

```
Byte  Value
----  -----
 0    0xA7     KC_MISC_CMD_GROUP
 1    1        PollRate Detect
```

#### Response

```
Byte  Field           Description
----  -----           -----------
 3-4  version         Protocol version (LE16): 3 = v1, other = v2
 5    support         Supported rates bitmap
 6    feature         Feature flags (bit 1 = single key reset support)
```

### Poll Rate Get (sub-command 13)

```
Byte  Value
----  -----
 0    0xA7     KC_MISC_CMD_GROUP
 1    13       PollRate Get
```

#### Response -- Version 1 (single rate)

```
Byte  Field           Description
----  -----           -----------
 3    current_usb     Current USB polling rate index
 4    support         Supported rates bitmap (1 byte, LSB = 8000 Hz)
```

#### Response -- Version 2 (dual rate)

```
Byte  Field           Description
----  -----           -----------
 3    current_usb     Current USB polling rate index
 4    support_usb     USB supported rates bitmap
 5    support_fr      2.4 GHz supported rates bitmap
 6    current_fr      Current 2.4 GHz polling rate index
```

### Poll Rate Set (sub-command 14)

#### Version 1

```
Byte  Value
----  -----
 0    0xA7     KC_MISC_CMD_GROUP
 1    14       PollRate Set
 2    rate     Polling rate index (applies to both USB and 2.4 GHz)
```

#### Version 2

```
Byte  Value
----  -----
 0    0xA7     KC_MISC_CMD_GROUP
 1    14       PollRate Set
 2    usb_rate   USB polling rate index
 3    fr_rate    2.4 GHz polling rate index
```

---

## Battery Level

### Keyboard Battery

Keyboard battery is read as part of the sleep settings response (sub-command
11 of `KC_MISC_CMD_GROUP`). The Batt keycode (`0x7E0F`) triggers an on-screen
battery level display on the keyboard itself.

### Mouse Battery (1K Protocol)

```
Report filter: report[0] === 1 && report[3] === 1
Response:
  report[5]  = power state (0 = discharging, non-zero = charging)
  report[6]  = power value (0-100 percentage)
  report[7]  = current profile
  report[10] = max report rate
```

### Mouse Battery (8K Nordic Protocol)

```
Report filter: (workMode===1 ? report[0]===65 : report[0]===1) && report[3]===1
Response:
  report[10] = charge flag (3 → map to 0 / discharging)
  report[11] = battery level (0-100)
  report[12] = current profile
  report[5]  = max report rate
```

### Mouse Battery (4K Protocol)

Read from the receiver state response (see
[Bridge / Dongle Protocol](bridge-dongle-protocol.md)):

```
chargeFlag    = from receiver info
batteryLevel  = from receiver info
reportRate    = from receiver info
```

### Battery Events

The Launcher subscribes to continuous battery updates via the
`BATTERY_CHANGE` event (enum value 4). Display:
- `powerValue`: percentage 0-100
- `power.state`: 0 = discharging, non-zero = charging
- Battery bar color: `#11C468` (green)

---

## Mouse Wireless System Parameters (8K Nordic)

Nordic-protocol mice have an extensive system parameters block read/written
via HID command group 4. These control wireless-specific behavior.

### Get System Params

```
Byte  Value
----  -----
 0    4        Command group
 2    131      0x83 (read flag)
 3    3        Sub-command: get system params
```

Buffer size: 64 bytes.

### Response (parsed from `report.slice(4)`)

| Offset | Field                | Type   | Description                     |
|--------|----------------------|--------|---------------------------------|
| 0-1    | `systemSleepTime`    | LE16   | System sleep timeout (seconds)  |
| 2-3    | `bleSystemSleepTime` | LE16   | BLE-specific sleep timeout      |
| 4      | `reportRate`         | uint8  | Current report rate index       |
| 5      | `debounceTime`       | uint8  | Debounce time (ms)              |
| 6      | `quickResponse`      | uint8  | Quick response mode (0/1)       |
| 7      | `buttonWakeupEnable` | uint8  | Wake from sleep on button press |
| 8      | `moveWakeupEnable`   | uint8  | Wake from sleep on movement     |
| 9      | `wheelWakeupEnable`  | uint8  | Wake from sleep on scroll       |
| 10     | `wheelReverse`       | uint8  | Reverse scroll direction        |
| 11     | `irButtonMode`       | uint8  | IR button mode                  |
| 12     | `mouseLeftKeyMode`   | uint8  | Left button mode                |
| 13     | `mouseRightKeyMode`  | uint8  | Right button mode               |
| 14     | `rfPowerMode`        | uint8  | RF transmit power level         |
| 15     | `bleNum`             | uint8  | Number of BLE connection slots  |
| 16     | `rptFlag`            | uint8  | Report rate support bitmask     |
| 17-22  | `rptTable`           | uint8[]| Report rate lookup table        |

### Set System Params

```
Byte  Value
----  -----
 0    4        Command group
 3    4        Sub-command: set system params
 4-5  systemSleepTime      (LE16)
 6-7  bleSystemSleepTime   (LE16)
 8    reportRate
 9    debounceTime
 10   quickResponse
 11   buttonWakeupEnable
 12   moveWakeupEnable
 13   wheelWakeupEnable
 14   wheelReverse
 15   irButtonMode
 16   mouseLeftKeyMode
 17   mouseRightKeyMode
 18   rfPowerMode
 19   bleNum
 20   rptFlag
 21-27 rptTable[0..6]
```

Buffer size: 64 bytes, with checksum at byte 63.

---

## Mouse Monitor Events

When connected wirelessly, mice broadcast real-time status updates as
unsolicited HID reports. These are separate from the bridge state
notifications (`0xE2`) documented in
[Bridge / Dongle Protocol](bridge-dongle-protocol.md).

### Event Codes

| Byte[0] | Decimal | Event Name        | Data                                  |
|---------|---------|-------------------|---------------------------------------|
| `0xE1`  | 225     | `monitor-light`   | RGB mode, speed, brightness, color    |
| `0xE2`  | 226     | `monitor-base`    | Work mode, connection, battery, DPI, polling rate |
| `0xE5`  | 229     | `monitor-profile` | Current profile index                 |

### monitor-base (`0xE2`) Fields

```
{
  workMode:    connection mode (USB/2.4G),
  connect:     connection status,
  power: {
    state:     charging state (0 = discharging),
    value:     battery percentage (0-100)
  },
  dpi: {
    level:     current DPI level index,
    levelNum:  total DPI levels available
  },
  pollingRate: {
    level:     current polling rate index
  }
}
```

### monitor-light (`0xE1`) Fields

```
{
  mode:        RGB effect mode,
  speed:       effect speed,
  brightness:  brightness level,
  rgb:         [r, g, b] color values
}
```

### Command ACK (`0xE4` / `0x52`)

Mouse commands receive acknowledgments via these report codes:

```
report[0] === 228 (0xE4) or report[0] === 82 (0x52)
  report[1] === 0  →  delayed retry (500ms, then next command)
  report[1] === 1  →  immediate response ready
```

---

## Transceiver Architecture

The Launcher selects different HID transceiver implementations based on device
protocol and connection mode.

### Selection Logic

```
if protocol === "1k":
    workMode === 1 ? AckFeatureReport : SequentialFeatureReport
else:
    workMode === 1 ? QueuedSendReport : DirectSendReport
```

### Transceiver Types

| Type                   | Protocol   | WorkMode | Transport                   | Flow Control         |
|------------------------|-----------|----------|-----------------------------|----------------------|
| Direct                 | 8k/4k    | 0 (USB)  | `sendReport()`              | None                 |
| Queued                 | 8k/4k    | 1 (2.4G) | `sendReport()` + queue      | Wait for `inputreport` response |
| Sequential Feature     | 1k       | 0 (USB)  | `sendFeatureReport()` / `receiveFeatureReport()` | 5s timeout |
| ACK Feature            | 1k       | 1 (2.4G) | `sendFeatureReport()` + `inputreport` monitor | ACK via `0xE4`/`0x52` |

### Queued Transceiver (Wireless 8K/4K)

Commands are queued as `[reportId, data, retryCount]` tuples. On receiving an
`inputreport` response, the queue shifts and processes the next command.
Timeout-based retry ensures reliability over the wireless link.

### ACK Feature Transceiver (Wireless 1K)

Monitors `inputreport` for acknowledgment codes:
- `0xE4` (228) or `0x52` (82) with `byte[1] === 0`: delayed retry (500ms)
- `0xE4` (228) or `0x52` (82) with `byte[1] === 1`: immediate
  `receiveFeatureReport()`

### Mouse Report IDs by Protocol

| Protocol   | Report ID | Notes                          |
|-----------|-----------|--------------------------------|
| 1K        | 81        | Feature reports                |
| Default   | 181 (`0xB5`) | Standard mouse commands     |
| 8K Nordic | 179 (`0xB3`) | Nordic-specific commands    |

### Mouse Buffer Checksums

4K and 8K Nordic protocols use a packet checksum:

```
function kt(data):
    sum = 0
    for i in 0..62: sum += data[i]
    data[63] = 161 - (sum & 0xFF)
```

And a wireless mode flag:

```
function ct(data, workMode):
    if workMode === 1: data[0] |= (1 << 6)   // sets bit 6
```

---

## Protocol Detection (Mice)

### Standard vs Nordic Detection

When a mouse connects via the `mouse_4k` HID collection (`0xFF0A`), the
Launcher sends a probe packet to determine the exact protocol:

```
Request:  Uint8Array(64): [1, 0, 129, 1, ...] + checksum at byte[63]
Response: filter for byte[0] === 1
```

Detection criteria:
- `byte[5]` (reportRateMax) > 4 → `"8k_nordic"`
- `byte[6,7]` = `[0x34, 0x34]` or `[0x2D, 0x36]` → `"8k_nordic"`
- Otherwise → `"4k"`

### Nordic Mouse Detection

```js
isNordicMouse = (usage === 1 && usagePage === 0xFF0A)
```

When `isNordicMouse === true`, the `write()` method uses report ID 178
(`0xB2`) instead of the standard report ID.

---

## Mouse Pairing Configuration

Mouse buttons can be configured as pairing triggers (hold to enter pairing
mode).

### 1K Protocol (reportId 81)

```
GET: cmd=54
Response:
  byte[1] = button bitmask (bits 0-4: left|right|middle|forward|backward)
  byte[2] = hold time

SET: cmd=70
  byte[0] = 70
  byte[1] = button config bitmask
  byte[2] = hold time (seconds)
```

### 4K/8K Protocol (reportId 0/64)

```
GET: cmd=4/68, subcmd=1
Response:
  byte[49] = button bitmask (left|middle|right|forward|backward)
  byte[50] = hold time

SET: cmd=11 (4K) / cmd=11 (8K Nordic)
  byte[0] = 11
  byte[1] = 1 (subcmd)
  byte[2] = button config bitmask
  byte[3] = hold time
```

---

## Related Documents

- [Bridge / Dongle Protocol](bridge-dongle-protocol.md) -- FR commands, VIA tunneling, device slots
- [DFU Protocols](dfu-protocols.md) -- Bluetooth module DFU, wireless firmware updates
- [Mouse / Trackball Protocol](mouse-protocol.md) -- DPI, gestures, profiles
- [Launcher Overview](overview.md) -- HID collections, transceiver classes
- [Firmware: Misc Commands](../firmware/misc-commands.md) -- firmware-side sleep, poll rate, debounce
- [Firmware: Core Commands](../firmware/core-commands.md) -- feature support flags
