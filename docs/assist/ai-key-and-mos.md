# AI Key & Mouse Optimization (MOS) Subsystems

Keychron Assistant introduces two specialized native subsystems that augment
the hardware capabilities of Keychron keyboards and mice:
1. **AI Key Subsystem (`service::ai_key`)**: Dedicated hardware trigger for AI prompts and workflows.
2. **MOS Subsystem (`service::mos`)**: Native mouse smooth scrolling and acceleration optimization.

---

## 1. AI Key Subsystem (`service::ai_key`)

Several newer Keychron keyboards feature a physical **AI Key** (or allow mapping
any key to an AI trigger). Rather than just acting as a standard hotkey, Assistant
integrates this key directly into the Keychron Launcher LangChain AI ecosystem.

### Architecture

```
┌─────────────────────────────────┐
│     Physical Hardware Key       │ (Keychron Keyboard AI Key)
└────────────────┬────────────────┘
                 │ Raw HID Report (PB usage / 0x7f00 slot)
                 ▼
┌─────────────────────────────────┐
│     device_listener / Assist    │
└────────────────┬────────────────┘
                 │ Look up AiKeyConf in assist.json
                 ▼
┌─────────────────────────────────┐
│ Assistant + Launcher WebSocket  │
│ AIKey config RPC (Get/Set/Put/  │
│ Remove/Clear)                   │
└─────────────────────────────────┘
```

### Setting up the key in Launcher

1. Connect the keyboard in Keychron Launcher and open **AI Key settings**.
2. Create a configuration, select/record an AI-key slot, enter a prompt template
   (the default is `{#}`), and select model plus window/dialog behavior.
3. Save. Decompiled Launcher code sends `AIKey` WebSocket messages: `Get` loads
   configs for the current numeric device ID (`vp_id`); `Set` adds a config;
   `Put` updates by index; `Remove` deletes by index. Config records include
   `key`, `template`, `window`, `dialog`, `model`, and numeric `vp_id`.
4. In the firmware keymap, assign the intended physical key to Keychron's
   Assist `QK_CR_X` keycode path. `QK_CR_X` emits Consumer-page usage
   `0x01cc + X`; it is not QMK's `PB_X` programmable-button mechanism.

The Launcher bundle also builds `Aikey` entries in its Quick-Start assist
configuration from saved AI Key records. Do not assume an undocumented
`AIKey/Trigger` WebSocket command: the inspected Launcher settings component
only confirms the config RPCs above. The precise hardware-event-to-AI execution
path must be established from the native implementation before claiming parity.

### Configuration Schema (`struct AiKeyConf`)

Each configured AI key is stored in `assist.json` (or `assist_key.json`):

```rust
pub struct AiKeyConf {
    pub key: String,         // Keycode slot (e.g. "0x7f01" or Consumer code)
    pub template: String,    // Customizable prompt template string
    pub window: bool,        // Whether to automatically open/focus the AI chat window
    pub dialog: bool,        // Whether to pop up a minimal floating dialog
    pub model: String,       // Target LLM (e.g. "deepseek-v4-pro", "deepseek-v4-flash")
    pub vp_id: String,       // Vendor and Product ID of the source device
}
```

#### Template List Wrapper (`struct AiKeyConfList`)

```json
{
  "list": [
    {
      "key": "0x7f01",
      "template": "Summarize the selected text concisely into bullet points.",
      "window": true,
      "dialog": false,
      "model": "deepseek-v4-pro",
      "vp_id": "13364:4100"
    },
    {
      "key": "0x7f02",
      "template": "Translate the selected text into idiomatic English.",
      "window": false,
      "dialog": true,
      "model": "deepseek-v4-flash",
      "vp_id": "13364:4100"
    }
  ]
}
```

### Protocol Commands (`AiKeyCmd` & `AiKeyActionType`)

- `AiKeyWsMsg`: Carries command payloads over the WebSocket bridge.
- Window/dialog settings use the Launcher values `New` and `Keep`. The available
  evidence does not establish an `AIKey/Trigger` WebSocket command.

---

## 2. MOS Mouse Optimization Subsystem (`service::mos`)

The MOS subsystem (named after the macOS "Mos" utility) resolves the common
issue where third-party mice produce jerky, discrete scroll notches on desktop
operating systems (especially macOS) compared to smooth trackpads.

### Configuration Schema (`struct MosConf`)

MOS configuration consists of 9 parameters controlling the interpolation physics
and hotkey overrides:

```rust
pub struct MosConf {
    pub speed: f64,            // Overall scroll speed scaling multiplier
    pub step: f64,             // Interpolation step distance
    pub duration: f64,         // Duration of the smoothing deceleration curve (seconds)
    pub enable: bool,          // Master toggle for smooth scrolling
    pub auto_enable: bool,     // Automatically enable when a third-party mouse is detected
    pub detect_distance: f64,  // Distance threshold before smoothing engages
    pub interval: u32,         // Frame interval in milliseconds (typically 16 ms for 60 Hz)
    pub reverse_key: u32,      // Modifier key to temporarily reverse direction
    pub speed_key: u32,        // Modifier key to temporarily boost speed
}
```

#### Default Values (`MosConf::default()`)

```json
{
  "speed": 1.0,
  "step": 10.0,
  "duration": 0.5,
  "enable": true,
  "auto_enable": false,
  "detect_distance": 5.0,
  "interval": 16,
  "reverse_key": 0,
  "speed_key": 0
}
```

### WebSocket RPC (`MosWsEventType`)

The Launcher controls MOS settings via WebSocket frames with `evt_type: "Mos"`:

```json
{
  "evt_type": "Mos",
  "evt_data": {
    "cmd": "Set",
    "data": {
      "speed": 1.5,
      "step": 12.0,
      "duration": 0.4,
      "enable": true,
      "auto_enable": true,
      "detect_distance": 5.0,
      "interval": 16,
      "reverse_key": 56,
      "speed_key": 59
    }
  }
}
```

- **`Set`:** Updates active runtime parameters and persists to `assist.json`.
- **`Get`:** Returns current `MosConf`.
- **`Reset`:** Restores factory default physics parameters.

---

## 3. Platform Implementation

### macOS Engine (`src/service/mos/mac.rs`)

1. **Event Interception (`CGEventTap`):**
   - Installs a low-level tap on `kCGAnnotatedSessionEventTap` for `kCGEventScrollWheel`.
   - Reads `kCGScrollWheelEventDeltaAxis1` (vertical) and `Axis2` (horizontal).
2. **Event Suppression:**
   - If smooth scrolling is active, Assistant consumes (suppresses) the original discrete wheel event so macOS does not perform its default abrupt jump.
3. **Deceleration & Smoothing (`SmoothScroll`):**
   - Pushes raw wheel ticks into an interpolation state buffer.
   - Computes an ease-out deceleration curve over `duration` milliseconds.
4. **Synthetic Event Injection:**
   - A high-frequency timer (`SCROLL_TIMER` running at `interval` ms) generates synthetic, fractional scroll events using `CGEventCreateScrollWheelEvent2` with pixel-level precision (`kCGScrollWheelEventScrollPhaseContinuous`).
5. **Reverse & Speed Hotkeys (`ToggleKey`):**
   - If `reverse_key` is held (e.g. Option or Command), the scroll direction is inverted on the fly without changing system settings.

### Windows Engine (`src/service/mos/win.rs`)

- Uses Raw Input (`RegisterRawInputDevices` for `HID_USAGE_PAGE_GENERIC`, `HID_USAGE_GENERIC_MOUSE`).
- Intercepts `WM_INPUT` messages containing `RI_MOUSE_WHEEL`.
- Subdivides the discrete `WHEEL_DELTA` (120 units) into smooth, sub-tick movements injected via `SendInput`.
