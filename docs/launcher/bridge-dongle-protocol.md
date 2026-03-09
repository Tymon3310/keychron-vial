# Bridge / Dongle Protocol

This document describes the 2.4 GHz dongle (bridge/receiver) protocol used
by Keychron wireless keyboards and mice. The bridge device acts as a USB
intermediary, forwarding HID commands between the host computer and wirelessly
connected Keychron devices.

> **Source:** Reverse-engineered from the Keychron Launcher JavaScript bundle.
> This protocol is implemented in our Vial GUI fork.

## Overview

Keychron wireless devices connect to the host via a USB 2.4 GHz dongle
(the "bridge"). The bridge presents itself as a HID device with usage page
`0x8C` (140), usage `0x01`. The Launcher communicates with the bridge using
the FR (Forza Receiver) command set, then tunnels standard VIA protocol
commands through the bridge to the connected keyboard.

```
 +----------+    WebHID (0x8C)    +---------+    2.4 GHz    +----------+
 | Launcher | =================> | Bridge  | ~~~~~~~~~~~~> | Keyboard |
 |  (host)  | <================= | (dongle)| <~~~~~~~~~~~~ | / Mouse  |
 +----------+    HID reports      +---------+    wireless   +----------+
```

## HID Interface

| Property    | Value             |
|-------------|-------------------|
| Usage Page  | `0x8C` (140)      |
| Usage       | `0x01` (1)        |
| Report ID   | 0                 |
| Report Size | 32 bytes          |

The bridge collection is detected alongside other Keychron HID collections
during device enumeration:

```js
navigator.hid.requestDevice({
    filters: [
        { usage: 97, usagePage: 65376 },  // keyboard
        { usage: 1,  usagePage: 140 },    // bridge
        { usage: 1,  usagePage: 65473 },  // mouse
        { usage: 1,  usagePage: 65290 }   // mouse 4K
    ]
})
```

## FR Command Set

The FR (Forza Receiver) commands are sent to the bridge device. They occupy
the `0xB1`--`0xBA` range, separate from the keyboard's `0xA0`--`0xAC` range.

| Command | Value | Hex    | Description                          |
|---------|-------|--------|--------------------------------------|
| `FR_GET_PROTOCOL_VERSION`   | 177 | `0xB1` | Get receiver protocol version + feature flags |
| `FR_GET_STATE`              | 178 | `0xB2` | Get paired device slots              |
| `FR_GET_FW_VERSION`         | 179 | `0xB3` | Get receiver firmware version        |
| `FR_CTL_GAMEPAD_RPT_ENABLE` | 181 | `0xB5` | Enable/disable gamepad report forwarding |
| `FR_DFU_OVER_VIA`           | 186 | `0xBA` | DFU update tunneled through VIA      |

---

### 0xB1 -- Get Protocol Version

Returns the receiver's protocol version and a feature flags bitmask.

#### Request

```
Byte  Value
----  -----
 0    0xB1
```

#### Response

```
Byte  Field           Description
----  -----           -----------
 1    version_high    Protocol version MSB
 2    version_low     Protocol version LSB
 3    features_0      Feature flags byte 0 (bits 7-0)
 4    features_1      Feature flags byte 1
```

Version is computed as `(data[2] << 8) | data[1]` (little-endian from bytes
at positions 1,2 of the response).

#### Feature Flags (byte 3, from MSB to LSB)

| Bit | Name                              | Description |
|-----|-----------------------------------|-------------|
| 7   | `state_notify_over_via`           | Supports connection state notifications over VIA |
| 6   | `support_multi_device_connect`    | Can have multiple devices connected simultaneously |
| 5   | `support_mouse_driver_over_via`   | Mouse protocol tunneling supported |
| 4   | `support_via_disable_gamepad_input` | Can disable gamepad HID input via command |

---

### 0xB2 -- Get State

Returns the paired device slots on the receiver. Each receiver supports
up to 3 device slots (for keyboard bridge) or 3-5 slots (for mouse
receivers).

#### Request

```
Byte  Value
----  -----
 0    0xB2
```

#### Response (Keyboard Bridge -- 3 slots)

```
Byte  Field           Description
----  -----           -----------
 2-3  slot_0_vid      VID of device in slot 0 (big-endian)
 4-5  slot_0_pid      PID of device in slot 0 (big-endian)
 6    slot_0_status   Connection status (1 = connected, 0 = empty/disconnected)
 7-8  slot_1_vid      VID of device in slot 1
 9-10 slot_1_pid      PID of device in slot 1
 11   slot_1_status   Connection status
12-13 slot_2_vid      VID of device in slot 2
14-15 slot_2_pid      PID of device in slot 2
 16   slot_2_status   Connection status
```

Each slot is 5 bytes: `[vid_hi, vid_lo, pid_hi, pid_lo, status]`.

A VID/PID of `0x0000`/`0x0000` with status `0` indicates an empty slot.

#### Response (1K Mouse Receiver -- 5 bonded devices)

```
Byte  Field           Description
----  -----           -----------
 2    bond_count      Number of bonded devices
 3-4  dev_0_vid       VID (big-endian)
 5-6  dev_0_pid       PID (big-endian)
 7    dev_0_state     State (1 = connected)
 ...  (repeats for bond_count devices, 5 bytes each)
```

---

### 0xB3 -- Get Firmware Version

Returns the receiver's firmware version string.

---

### 0xB5 -- Gamepad Report Enable

Toggles whether the bridge forwards gamepad HID reports from the connected
keyboard. This is used to disable gamepad input when the Launcher is
configuring the device, to avoid phantom joystick inputs.

---

### 0xBA -- DFU Over VIA

Initiates a firmware update for the connected wireless device, tunneled
through the VIA protocol channel. This allows updating the keyboard's
firmware without disconnecting from the bridge.

---

## VIA Protocol Tunneling

Once the bridge reports a connected device (via `FR_GET_STATE`), the Launcher
sends standard VIA protocol commands through the bridge. The bridge firmware
transparently forwards these HID reports to the connected keyboard and
relays responses back.

### Connection Flow

1. **Open bridge HID device** (usage page `0x8C`)
2. **Send `FR_GET_PROTOCOL_VERSION`** (`0xB1`) -- get version and feature flags
3. **Send `FR_GET_STATE`** (`0xB2`) -- enumerate paired device slots
4. **Find connected device** -- scan slots for `status == 1`
5. **Start VIA communication** -- send VIA commands (bytes 0-31) via
   `sendReport(0, buffer)` to the bridge
6. **Bridge forwards** the 32-byte report to the keyboard over 2.4 GHz
7. **Keyboard responds** through the bridge back to the host

### VIA Version Check (tunneled)

```js
// After confirming bridge has a connected device:
const buf = new Uint8Array(32);
buf[0] = VIA.GET_PROTOCOL_VERSION;  // 0x01
bridge.sendReport(0, buf);
// Response byte[2] = VIA protocol version
```

### Error Handling

If no device is connected (`connect == false`), the Launcher displays a
"device not connected to receiver" error and does not attempt VIA
communication.

---

## State Notifications (0xBC)

When the `state_notify_over_via` feature flag is set, the bridge sends
unsolicited state change reports when devices connect or disconnect:

```
Byte  Field    Description
----  -----    -----------
 0    0xBC     State notification marker (188 decimal)
 1    kb_1     Keyboard slot 1 state
 2    kb_2     Keyboard slot 2 state
 3    mouse    Bit 2 (data[3] >> 2 & 1): mouse connection state
```

The Launcher subscribes to these notifications to update the UI in real time
when a keyboard or mouse connects/disconnects from the dongle.

---

## Mouse Receiver Variants

Mouse receivers come in two flavors, detected by their HID collection:

| Collection   | Protocol  | Description |
|-------------|-----------|-------------|
| `mouse` (`0xFFC1`) | 1K | Standard polling rate mouse receiver |
| `mouse_4k` (`0xFF0A`) | 4K/8K | High polling rate mouse receiver |

Both use FR_GET_STATE to enumerate bonded devices, but they differ in:
- Report sizes (32 vs 64 bytes)
- Transceiver class (queued vs feature-report)
- State parsing (3 slots vs 5 bonded devices)

The 8K Nordic protocol is auto-detected by sending a probe packet and
checking response bytes `[6,7]` for `[0x34, 0x34]` or `[0x2D, 0x36]`.

---

## Related Documents

- [Launcher Overview](overview.md)
- [DFU Protocols](dfu-protocols.md) -- includes FR_DFU_OVER_VIA details
- [Mouse / Trackball Protocol](mouse-protocol.md) -- mouse-specific commands
- [Command Map Comparison](command-map.md) -- FR command enum reference
