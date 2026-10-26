# Hardware Device Listener & HID Interception

Keychron Assistant maintains a low-level background device listener
(`service::device_listener` and `utils::keychron_device`) that continuously monitors
connected Keychron keyboards, mice, and 2.4 GHz receivers for quick-start buttons,
AI hotkeys, and mouse wheel ticks.

---

## 1. Device Identification & Filtering

Assistant filters connected USB and Bluetooth devices to specifically monitor
hardware manufactured by Keychron and partner brands:

### Vendor & Product IDs

| Identifier | Value (Hex) | Value (Decimal) | Description |
|------------|-------------|-----------------|-------------|
| **Keychron VID** | `0x3434` | 13364 | Standard Keychron keyboards, mice, and dongles |
| **Xinji (XJ) VID** | `0x34B7` | 13495 | White-label / wireless chipset partner |
| **Device Name Matching** | Case-insensitive substring match | `"keychron"`, `"lemokey"` |

### Device Type Classification

In internal types (`common::types::KeychronDeviceInfo`), devices are classified
into two primary categories:

```rust
pub enum DeviceType {
    Keyboard = 1,
    Mouse = 2,
}
```

---

## 2. Platform Backends

To achieve zero-lag event capture across operating systems, Assistant uses distinct
platform drivers:

| Platform | Low-Level HID Driver | System Event Hook | Role |
|----------|----------------------|-------------------|------|
| **Windows** | `async-hid` (`win32::device`) | `RegisterRawInputDevices` & `SetWindowsHookExW` | Raw HID report capture + default action suppression |
| **macOS** | `IOHIDManager` | `CGEventTap` | HID report callback + event tap suppression |
| **Linux (Prototype)** | `evdev` | `ioctl(EVIOCGRAB)` | Kernel input event device reading |

---

## 3. Keyboard Event Interception

Keychron keyboards expose quick-start keys and programmable buttons over the
standard **HID Consumer Control** collection (Usage Page `0x0C`):

### Consumer-reserved Assist slots (`QK_CR_X`)

In this QMK tree, `QK_CR_0..QK_CR_51` (`0x7f00..0x7f33`) are a consumer-keycode
range. `KEYCODE2SYSTEM()` translates them to Consumer-page usages
`0x01cc..0x01ff`. Keychron's Assist assignment uses this path. On Linux these
Consumer usages are commonly surfaced as `KEY_MACRO1..KEY_MACRO52` (input codes
656..707), subject to the keyboard's HID descriptors and kernel mapping.

```
Physical Keypress
       │
       ▼ (QK_CR_X in the keymap)
HID Consumer report: Usage Page 0x0C, Usage 0x01cc..0x01ff
       │
       ▼ (Intercepted by async-hid / IOHIDManager)
Lookup: Consumer usage 0x01cc -> QK_CR_0 slot 0x7f00
       │
       ▼
Look up Keybind for Device VPID in assist.json
       │
       ▼
Execute Action (Launch App, Open URL, Trigger AIKey)
```

### Do not conflate this with QMK `PB_X`

QMK's `PB_1..PB_32` are a separate keycode family (`0x7440..0x745f`). The
`process_programmable_button` path sends a Programmable Buttons bitmap through
`host_programmable_button_send()`; it is not the `QK_CR_X` Consumer-keycode
translation. Thus the *QMK keycode ranges and HID report paths are distinct*.
Whether a particular Linux device/driver exposes both as distinguishable evdev
events must be verified from that device's actual HID descriptors and event
stream; do not infer PB events from Consumer `KEY_MACRO*` codes or mouse button
codes.

---

## 4. Mouse Event Interception & Recognized Buttons

In `src/service/device_listener/win.rs` and `mac.rs`, Assistant maintains explicit
match branches for all standard and extended mouse actions:

| Recognized Identifier | Hardware Event | Description |
|-----------------------|----------------|-------------|
| `MOUSE_LEFT` | Button 0 / Left Click | Primary button |
| `MOUSE_RIGHT` | Button 1 / Right Click | Secondary button |
| `MOUSE_MIDDLE` | Button 2 / Middle Click | Scroll wheel click |
| `MOUSE_4` | Button 3 / Side Back | Thumb Back button |
| `MOUSE_5` | Button 4 / Side Forward | Thumb Forward button |
| `MOUSE_8` | Button 7 / Custom | Dedicated sniper / DPI shift button |
| `MOUSE_WHEEL_UP` | Wheel +Y | Vertical scroll forward |
| `MOUSE_WHEEL_DOWN` | Wheel -Y | Vertical scroll backward |
| `MOUSE_WHEEL_LEFT` | Wheel -X | Horizontal tilt scroll left |
| `MOUSE_WHEEL_RIGHT` | Wheel +X | Horizontal tilt scroll right |
| `MOUSE_TILT_LEFT` | Tilt Left | Optical/encoder tilt switch |
| `MOUSE_TILT_RIGHT` | Tilt Right | Optical/encoder tilt switch |

---

## 5. Event Suppression (Dropping Default Actions)

When a mouse button or key is bound to a custom quick-start action (for instance,
binding `MOUSE_4` to open Terminal), the operating system must **not** execute its
default behavior (navigating backward in a web browser).

### macOS Suppression Engine (`src/service/device_listener/mac.rs`)

1. **`mark_mouse_default_block`:** Sets a timestamped block marker (`MOUSE_DEFAULT_BLOCK_UNTIL`) when an intercepted event matches an active binding.
2. **`watch_mouse_default_event_blocker`:** A dedicated `CGEventTap` callback intercepts the corresponding OS mouse event:
   ```rust
   if should_drop_mouse_default(event) {
       consume_mouse_default_block();
       return None; // Drops event completely from macOS event stream
   }
   ```
3. **`consume_mouse_default_block`:** Clears the block flag once the event is suppressed.

### Windows Suppression Engine (`src/service/device_listener/win.rs`)

- Installs a low-level hook using `SetWindowsHookExW(WH_MOUSE_LL, ...)` and `SetWindowsHookExW(WH_KEYBOARD_LL, ...)`.
- If the virtual key or mouse button matches a configured binding in `assist.json`:
  - Executes the bound action in a separate Tokio async task.
  - Returns `1` from the hook callback to consume the event and prevent `CallNextHookEx` propagation.

---

## 6. Wireless Receiver VPID Tunneling

Keychron 2.4 GHz wireless receivers present a single unified composite HID interface.
When multiple peripherals share a single receiver or a mouse communicates over a dongle:

1. The raw HID device reports the **Receiver's** USB VID/PID (e.g. `0x3434:0x4200`).
2. When Keychron Launcher connects to the peripheral, it sends `SetReceiverVpidMap`
   over WebSocket to inform Assistant of the true peripheral VPID (e.g. `0x3434:0x4100`).
3. `device_listener` checks this mapping table on every report. If a match exists,
   it looks up keybindings under the peripheral's VPID rather than the receiver's ID.
