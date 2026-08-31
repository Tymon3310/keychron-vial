# Mouse / Trackball Protocol

This document describes the mouse and trackball protocols discovered in the
Keychron Launcher, including the NAPE command set for trackball configuration,
polling rate protocols, and mouse-specific HID communication.

> **Source:** Reverse-engineered from the Keychron Launcher JavaScript bundle.
> These protocols are not implemented in our Vial GUI fork.

## Overview

Keychron mice and trackballs communicate via two distinct HID collections
depending on their polling rate:

| Collection    | Usage Page | Usage | Protocol | Report Size |
|---------------|------------|-------|----------|-------------|
| `mouse`       | `0xFFC1` (65473) | 1 | 1K | 32 bytes |
| `mouse_4k`    | `0xFF0A` (65290) | 1 | 4K/8K | 64 bytes |

Mouse communication can be either direct USB or through a 2.4 GHz receiver
(dongle), controlled by `workMode`:

| workMode | Transport    | Transceiver Class |
|----------|-------------|-------------------|
| 0        | Direct USB  | Simple or Feature Report |
| 1        | Via dongle  | Queued with retry |

## Protocol Detection

4K mice undergo additional protocol detection to distinguish 4K from 8K
Nordic:

```
Probe: [1, 0, 129, 1, ..., checksum]
If response[6,7] == [0x34, 0x34] → "8k_nordic"
If response[6,7] == [0x2D, 0x36] → "8k_nordic"
Otherwise                         → "4k"
```

Detected protocol values: `"1k"`, `"4k"`, `"8k_nordic"`, and `"kc_lite"`.

### kc_lite Protocol Variant

A fourth protocol family (`kc_lite`) exists for budget mice. It uses 63-byte
payloads on report ID **179 (`0xB3`)** with single-letter command bytes:

| Cmd byte | Operation | Payload | ACK |
|----------|-----------|---------|-----|
| 65 (`0x41`) | DPI / rate level set | `[65, level, level, values[0..5], min(levelNum,3)]` | `data[0]==228 (0xE4)` or `65` |
| 66 (`0x42`) | Advance params | `[66, lod, sensor_wave, sensor_line, sensor_motion, _, scrollDir+1, _, fps20k+1]` | `data[0]==228 && data[2]==66` |
| 67 (`0x43`) | Debounce | `[67, ms]` (+ per-key: `[2]=17, [3]=key_index, [4]=ms`) | `data[0]==228 && data[2]==67` |

Macros use their own 4-byte step encoding:
`[0, kind<<7 | type, value, 0]`; delay steps are `[0, 15, ms_lo, ms_hi]`;
modifier keys are flagged with type `2` and bit masks 1/2/4/8/16/32/64/128
for Ctrl/Shift/Alt/Gui.

Its asynchronous monitor reports differ from the other protocols:

```
data[0] = 231 (0xE7):
  data[1] = 1 → light event: mode=data[2], speed=data[4],
                brightness=data[5], rgb=data[6..8]
  data[1] = 2 → base event: workMode=data[2], connect=data[3],
                power state/value=data[4..5], dpi level/levelNum=data[6]/data[8],
                pollingRate level=data[7]
data[0] = 229 (0xE5): profile change, current = data[1]
```

## Mouse Report IDs

| Report ID | Hex    | Usage |
|-----------|--------|-------|
| 0         | `0x00` | Standard keyboard-style reports |
| 179       | `0xB3` | kc_lite protocol commands |
| 181       | `0xB5` | Mouse reports (incl. 8K haptic motor) |

## Packet Checksum (8K Protocol)

8K mouse packets use a trailing checksum byte at offset 63:

```
buf[63] = (161 - (sum of bytes 0..62)) & 0xFF
```

The constant 161 (`0xA1`) is the checksum seed.

---

## NAPE Command Set

The "NAPE" commands configure trackball and mouse features. They are sent
under the `KC_MISC_CMD_GROUP` (0xA7) command with sub-command values
in the `0x20`--`0x39` range.

> **Note:** These sub-commands share the same top-level command byte (0xA7)
> as the keyboard misc commands (0x01--0x1C), but target mouse/trackball
> firmware. The name "NAPE" appears to be an internal Keychron codename
> for the trackball product line.

### Sub-Command Map

| Sub-cmd | Hex    | Direction | Description |
|---------|--------|-----------|-------------|
| `KC_USER_CMD_NAPE_GET_ORI`        | `0x20` | Read  | Get trackball orientation |
| `KC_USER_CMD_NAPE_GET_DPI`        | `0x21` | Read  | Get DPI profile settings |
| `KC_USER_CMD_NAPE_SET_DPI`        | `0x22` | Write | Set DPI profile selection |
| `KC_USER_CMD_NAPE_SET_DPI_VALUE`  | `0x23` | Write | Set DPI value for a profile |
| `KC_USER_CMD_NAPE_GET_DPI_VALUE`  | `0x24` | Read  | Get DPI value for a profile |
| `KC_USER_CMD_NAPE_SET_TAPHOLDS`   | `0x25` | Write | Set tap-hold button config |
| `KC_USER_CMD_NAPE_GET_TAPHOLDS`   | `0x26` | Read  | Get tap-hold button config |
| `KC_USER_CMD_NAPE_SET_COMBOS`     | `0x27` | Write | Set button combo config |
| `KC_USER_CMD_NAPE_GET_COMBOS`     | `0x28` | Read  | Get button combo config |
| `KC_USER_CMD_NAPE_SET_GESTURE`    | `0x29` | Write | Set gesture config |
| `KC_USER_CMD_NAPE_GET_GESTURE`    | `0x2A` | Read  | Get gesture config |
| `KC_USER_CMD_NAPE_SET_PROFILE`    | `0x2B` | Write | Set active mouse profile |
| `KC_USER_CMD_NAPE_GET_PROFILE`    | `0x2C` | Read  | Get active mouse profile |
| `KC_USER_CMD_NAPE_SET_LAYER`      | `0x2D` | Write | Set active layer |
| `KC_USER_CMD_NAPE_DEL_COMBOS`     | `0x2E` | Write | Delete button combo |
| `KC_USER_CMD_NAPE_DEL_TAPHOLDS`   | `0x2F` | Write | Delete tap-hold entry |
| `KC_USER_CMD_NAPE_BAT_REPORT`     | `0x30` | Write | Send battery report config |
| `KC_USER_CMD_NAPE_GET_BAT_REPORT` | `0x31` | Read  | Get battery report |
| `KC_USER_CMD_NAPE_SET_ORI`        | `0x34` | Write | Set trackball orientation |
| `KC_USER_CMD_NAPE_GET_LAYER_ORI`  | `0x38` | Read  | Get per-layer orientation |
| `KC_USER_CMD_NAPE_SET_LAYER_ORI`  | `0x39` | Write | Set per-layer orientation |

### Additional Mouse Sub-Commands

| Sub-cmd | Hex    | Direction | Description |
|---------|--------|-----------|-------------|
| `Get_Force_Gesture_Scroll` | `0x33` (51) | Read | Get force gesture scroll setting |
| `Set_Force_Gesture_Scroll` | `0x32` (50) | Write | Set force gesture scroll setting |

---

## Feature Details

### DPI Configuration

Mice support multiple DPI profiles (typically 4-6). Each profile has
independent X and Y DPI values. The Launcher shows:
- Current DPI profile index
- DPI value per profile
- Maximum report rate supported

### Orientation (Trackball)

The trackball sensor orientation can be rotated to match the physical
mounting angle. Get/set via `NAPE_GET_ORI` / `NAPE_SET_ORI`.

Angles are quantised in 45° steps: the wire value is `angle / 45`
(0=0°, 1=45°, ... 7=315°).

**Per-layer orientation:** sub-commands 56/57 (`0x38`/`0x39`) extend
this with per-layer angles:

```
GET_LAYER_ORI : [0xA7, 56, layer]            → data[2] = angle / 45
SET_LAYER_ORI : [0xA7, 57, layer, angle/45]
```

The global commands (32/52) still apply when no layer is selected; the UI
falls back to them for the "default" angle.

### Trackball State Monitor

Trackballs push asynchronous state using `KC_GET_CURRENT_LAYER` (0xA3) as
the report marker:

```
data[0] = 0xA3
data[1] = active layer
data[2] = orientation units (angle = 45 * value)
data[3] = current DPI index/value
```

The Launcher diffs this stream to emit `LAYER_CHANGE`, `ANGLE_CHANGE`, and
`DPI_CHANGE` monitor events (see [wireless-communication.md](wireless-communication.md)).

### Haptic Motor (8K Mice)

8K-protocol mice expose a haptic motor configuration sent as 20-byte
reports on report ID **181 (`0xB5`)**:

```
Get:
  Request : data[0] = 105 (0x69)
  Response: data[0] = 105
            data[1] = levelTotal
            data[2] = levelCount (defaults to levelTotal)
            data[3..4] = level pair 1 [strength, duration]
            data[5..6] = level pair 2 [strength, duration]
Set:
  Request : data[0] = 89 (0x59)
            data[1] = levelCount (default 6)
            data[2..5] = two [strength, duration] pairs
  ACK     : report with data[0] == 228 (0xE4) && data[2] == 89
```

Not supported on `1k`/`4k` protocols.

### Tap-Hold

Button tap-hold configuration allows a single button to perform different
actions on tap (quick press) vs hold (sustained press). Entries can be
added and deleted.

### Button Combos

Button combinations (pressing multiple buttons simultaneously) can be
mapped to specific actions. Entries can be added and deleted.

### Gestures

Trackball gesture recognition -- specific movement patterns (e.g., swipe,
scroll) are mapped to actions.

### Force Gesture Scroll

A special scroll mode where trackball gestures are interpreted as scroll
inputs. Toggled via sub-commands `0x32` / `0x33`.

### Battery Report

For wireless mice, battery state can be queried:
- `chargeFlag` -- whether the device is charging
- `batteryLevel` -- remaining battery percentage

### Profiles

Mice support multiple saved profiles (DPI, button mapping, etc.), similar
to keyboard analog matrix profiles.

---

## Dongle Version Command

Mice can be paired to a dongle, and the Launcher queries the dongle version directly. The request uses `data[0] = 0x00`, `data[2] = 0x81 (129)`, `data[3] = 0x00`.

```
Request:  [0x00, 0x00, 0x81, 0x00]
Response: Filters for data[0] == 0x00 && data[3] == 0x00
          Byte 8: Version Minor/Patch (Minor = (data[8] & 240) >> 4, Patch = data[8] & 15)
          Byte 9: Version Major (Major = data[9])
          Result string: `${Major}.${Minor}.${Patch}`
```

## Mouse State Packet Parsing

When a mouse connects (direct USB or via dongle), the Launcher parses a
state packet containing system parameters:

| Field                 | Description |
|-----------------------|-------------|
| `state`               | Connection state |
| `vid` / `pid`         | Device identification |
| `reportRateMax`       | Maximum polling rate supported |
| `chargeFlag`          | Charging indicator |
| `batteryLevel`        | Battery percentage |
| `profileIndex`        | Active profile |
| `dpiIndex`            | Active DPI profile |
| `dpiX` / `dpiY`       | Current DPI values |
| `reportRate`          | Current polling rate |

### Base Info (Device Data)

Mice have an extensive settings object for system properties, acquired using a
request containing `[0x04, 0, 0x83, 0x03]`:

| Offset | Field                 | Description |
|--------|-----------------------|-------------|
| 0-1    | `systemSleepTime`     | Idle sleep timeout (LE16, seconds) |
| 2-3    | `bleSystemSleepTime`  | BLE idle sleep timeout (LE16, seconds) |
| 4      | `reportRate`          | Current polling rate |
| 5      | `debounceTime`        | Button debounce (ms) |
| 6      | `quickResponse`       | Quick response mode |
| 7      | `buttonWakeupEnable`  | Wake on button press |
| 8      | `moveWakeupEnable`    | Wake on mouse movement |
| 9      | `wheelWakeupEnable`   | Wake on scroll wheel |
| 10     | `wheelReverse`        | Invert scroll wheel direction |
| 11     | `irButtonMode`        | IR button mode |
| 12     | `mouseLeftKeyMode`    | Left click mode |
| 13     | `mouseRightKeyMode`   | Right click mode |
| 14     | `rfPowerMode`         | RF transmission power |
| 15     | `bleNum`              | Bluetooth host slot number |
| 16     | `rptFlag`             | Supported polling rates flag |
| 17-22  | `rptTable`            | 6-byte array of available polling rates |

### Polling Rate

Polling rate is reported as a level index with available levels:

```
levelsVal: [0, 1, 2, 3, 4, 5]  (indices into rate table)
reportRateMax: (hardware maximum)
pollingRate.level: [current_level, ...]
```

---

## Known Mouse Products

| Product Name         | Product ID | Dongle PID | Protocol |
|----------------------|------------|------------|----------|
| M3mini4K             | `0x0620`   | `0xD037`   | 4K       |
| M44K                 | `0x0621`   | `0xD040`   | 4K       |
| M3mini4K (aluminium) | `0x0622`   | `0xD041`   | 4K       |
| M3M24k               | `0x0623`   | `0xD045`   | 4K       |
| M6 4K                | `0x0624`   | `0xD046`   | 4K       |
| M34K                 | `0x07A0`   | `0xD03C`   | 4K       |

---

## Related Documents

- [Launcher Overview](overview.md) -- transceiver classes, buffer construction
- [Bridge / Dongle Protocol](bridge-dongle-protocol.md) -- mouse receiver slots
- [Command Map Comparison](command-map.md) -- NAPE command enum
- [AI Assistant](ai-assistant.md) -- mouse tool catalog built on these commands
