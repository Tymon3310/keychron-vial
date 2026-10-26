# AI Assistant (Launcher "Assist" Panel)

The Keychron Launcher ships a LangChain-based AI chat panel that can directly
configure connected devices through tool calls. This document records the
architecture, backend endpoints, and the complete tool catalog.

> **Source:** Reverse-engineered from the Keychron Launcher JavaScript bundle
> (`https://launcher.keychron.com`).

## Overview

- The whole LangChain JS ecosystem (`@langchain/core`, OpenAI, Anthropic,
  Google, Mistral, Groq, Cohere, Ollama, Bedrock, ... adapters) is bundled,
  but only two models are exposed in the UI.
- Tool schemas are validated with Zod; tool execution goes through a
  device-context layer that maps high-level operations onto the HID
  protocols documented elsewhere in this directory.
- Brands can hide the feature via the `assist` entry in the per-domain
  `disMenuIds` list (most white-label domains disable it).

## Models and Backend

| Model ID                      | Provider  | Temperature | Thinking |
|-------------------------------|-----------|-------------|----------|
| `deepseek-v4-flash`           | deepseek  | 0.1         | configurable toggle |
| `deepseek-v4-flash-vision-exp`| deepseek  | 0.1         | configurable toggle |

*(Note: `deepseek-v4-pro` was removed in favor of `deepseek-v4-flash-vision-exp` for vision experiments).*

- **API base:** `https://launcher.keychron.cn/api` (OpenAI-compatible
  proxy; no provider API keys are embedded).
- **Thinking Mode:** DeepSeek requests carry `thinking: { type: "disabled" | "enabled" }`
  toggled via the UI button (`supportsDeepThinking` / `deepThinkingEnabled`). When enabled,
  assistant messages capture and render `reasoning_content`.
- **Telemetry:**
  - Question statistics are posted to `https://erpapik.keychron.cn/launcher/vapi/ai-question-statistics`.
  - Prompt cache statistics are posted to `https://erpapik.keychron.cn/launcher/vapi/ai-prompt-cache-statistics`.
- **Knowledge-base search:** queries the brand's `knowledgeBaseApi` host (e.g. `https://launcher.keychron.cn`).
- **Token Meter Easter Egg:** 5 clicks on the chat panel title bar within 2 seconds
  unlocks the live LLM token-usage meter (`inputTokens`, `outputTokens`, `totalTokens`).

## AI Per-Key RGB Lighting Generator (`ai-light`)

In addition to the conversational chat agent, the Launcher embeds an AI lighting
generator component (`ai-light`) inside the per-key RGB backlight configuration view:

- **Purpose:** Generates custom single-character or glyph illumination patterns
  on the keyboard matrix using natural language prompting.
- **Backend:** Calls `https://launcher.keychron.cn/api/v1/chat/completions`
  with model `deepseek-chat` and `temperature: 0.3`.
- **Matrix Prompting:** Translates the connected keyboard's matrix layout (`row` and `col` counts)
  into a system prompt requesting a JSON coordinate array:
  `{"coordinates": [[row, col], ...]}`.
- **Hardware Application:** Converts returned coordinate pairs to hardware LED indexes
  via `toRegionKey()` and saves the pattern to keyboard memory via `saveRegion(new Region({keys, color}))`.

---

## Keychron Assistant Companion App Integration

Quick-start app launching, smooth scrolling, multi-device cross-screen sharing,
and key-binding storage are delegated to the native companion desktop app
("Keychron Assistant").

The Launcher web app opens a local WebSocket connection to `ws://localhost:40003`
to bridge browser sandboxing and OS-level operations.

### Integration Roles

- **Quick-Start Keybinding Storage:** When a user maps a physical key to launch an app or URL
  (using slots `KC_F16`--`KC_F24` or `0x7f00` range), the QMK keycode is written to the device,
  while the action payload (executable path, URL, icon) is pushed to Keychron Assistant.
- **2.4 GHz Dongle / Receiver VPID Mapping:** The Launcher dynamically informs Assistant which
  wireless peripheral is active behind a 2.4 GHz receiver (`SetReceiverVpidMap`), allowing
  QuickStart key events received by the dongle to trigger the correct desktop binding.
- **Trackball Support:** QuickStart bindings are fully supported on Trackballs
  (`trackball/assist` route), utilizing slots `KC_F16`--`KC_F20`.
- **AI Key Configuration Sync:** Manages hardware-triggered AI prompt workflows (`evt_type: "AIKey"`),
  linking custom keys with prompt templates and target AI models.
- **Smooth Scrolling (MOS):** Synchronizes smooth scrolling physics, acceleration, and modifier-key
  shortcuts (`evt_type: "Mos"`) with the native mouse daemon.
- **System Actions (`SysCmd`):** Spawns local applications or opens operating system mouse settings.

> **Full Native Protocol Documentation:**
> For the complete native architecture, IPC commands, and low-level protocol specifications,
> refer to the dedicated [`assist/`](../assist/) documentation suite:
> - [`assist/overview.md`](../assist/overview.md) -- Architecture, Tauri v2 backend, and storage schemas
> - [`assist/websocket-protocol.md`](../assist/websocket-protocol.md) -- Authoritative command reference for all 6 event namespaces
> - [`assist/ai-key-and-mos.md`](../assist/ai-key-and-mos.md) -- AI prompt templates and MOS scroll acceleration
> - [`assist/cross-screen-protocol.md`](../assist/cross-screen-protocol.md) -- Virtual KVM screen sharing protocol
> - [`assist/file-transfer-protocol.md`](../assist/file-transfer-protocol.md) -- Peer-to-peer reliable file transfer


### Quick-Start Bindings

A quick-start binding replaces a physical key's keycode with a slot code
(e.g. `KC_F16`...`KC_F24` or custom `0x7f00`-range codes) that Assistant
intercepts to open an app or URL:

```
binding = { type: "App" | "Url" | "Aikey", name, path/url, title, icon }
```

- Written through the normal VIA keymap (`DYNAMIC_KEYMAP_SET_KEYCODE`);
  the metadata lives on the Assistant side.
- `"Aikey"` is a new binding type for AI-triggered actions; clearing
  quick-start bindings explicitly preserves Aikey configurations.

## Device Tool Catalog

~90 tools are registered. Grouped by feature area:

### Connection / session
`get_current_device`, `get_paired_devices`, `connect_paired_device`,
`disconnect_current_device`, `request_new_device`, `open_assistant`,
`get_system_type`

### Keyboard keymap / layers
`get_all_keys`, `get_keycodes`, `get_keyboard_layers`,
`switch_keyboard_layer`, `remap_keys`, `reset_keymap`,
`get_keyboard_knobs`, `set_keyboard_knob` (encoder CCW/CW per layer)

### Snap Tap & latch (keyset)
`get_keyset_config`, `set_keyset_snap_tap`, `remove_keyset_snap_tap`
(5 slots, types 2=`SC` last-key-wins, 3=`LKP`, 4=`AP 1st`, 5=`AP 2nd`,
6=`CM` cancel; requires HE version >= 3),
`get_latch_keys`, `set_latch_keys`, `remove_latch_keys`
(types `KEY`/`MACRO`/`TURBO`; gated on `Support_Latch`)

### Keyboard lighting
`get_current_light_config`, `set_keyboard_light`, `get_key_light_colors`,
`set_key_light_color`, `get_mix_rgb_regions`, `set_key_to_mix_region`,
`set_mix_rgb_region_config`, `get_keyboard_led_indication`,
`set_keyboard_led_indication`

### Macros / quick start
`get_keyboard_macros`, `set_keyboard_macro`, `clear_keyboard_macro`,
`bind_macro_to_key`, `get_quick_start_apps`, `open_quick_start_app`,
`get_app_pages`, `bind_quick_start_app`, `bind_quick_start_url`,
`get_quick_start_bindings`, `remove_quick_start_binding`,
`clear_quick_start_bindings`

### Keyboard settings
`get_keyboard_advance_config`, `set_keyboard_debounce`,
`set_keyboard_dip`, `set_keyboard_nkro`, `set_keyboard_polling_rate`,
`set_keyboard_sleep`, `set_keyboard_performance_mode`
(NKRO is reported as firmware-managed on some models and cannot be toggled)

### Hall Effect
`get_he_config`, `get_he_profiles`, `switch_he_profile`,
`rename_he_profile`, `set_he_actuation_distance`,
`set_he_rapid_trigger`, `set_he_directive`, `remove_he_directive`,
`clear_he_directives` (DKS/Turbo/DKR directives), `set_he_snap_tap`,
`set_he_turbo`, `set_he_dkr`

### Mouse
`get_mouse_info`, `get_mouse_keys`, `set_mouse_key`,
`get_mouse_key_functions`, `get_mouse_dpi_config`, `set_mouse_dpi`,
`reset_mouse_dpi`, `get_mouse_polling_rate_config`,
`set_mouse_polling_rate`, `reset_mouse_polling_rate`,
`get_mouse_profiles`, `switch_mouse_profile`, `rename_mouse_profile`,
`get_mouse_light_config`, `set_mouse_light`, `reset_mouse_light`,
`get_mouse_macros`, `create_mouse_macro`, `update_mouse_macro`,
`delete_mouse_macro`, `bind_mouse_macro_to_key`, `reset_mouse_keymap`,
`get_mouse_advance_config`, `set_mouse_lod` (lift-off distance),
`set_mouse_sensor` (wave / angle-snap / motion-sync / scroll direction),
`set_mouse_debounce` (global or buttons 0-4), `set_mouse_sleep_timeout`
(1-240 min, 8k only), `set_mouse_angle` (8k only), `get_mouse_angle`,
`reset_mouse_advance` (requires `confirm=true`)

### Knowledge base
`search_knowledge_base`, plus meta tools `request_tools`, `ask_user_choice`

Notable capability strings surfaced by these tools:

- Angle snapping is restricted to 8K-protocol mice.
- Macro repeat supports 1--100 repetitions (`loopTilCount` mode) where
  supported.
- HE actuation updates can be rejected per-device ("Device rejected the HE
  actuation distance update"), implying firmware-side validation.

## Related Documents

- [Keychron Assistant Docs](../assist/README.md) -- native companion desktop daemon architecture, protocols, and services
- [Launcher Overview](overview.md) -- architecture and transport
- [Command Map](command-map.md) -- underlying command IDs
- [Mouse / Trackball Protocol](mouse-protocol.md)
- [Analog Matrix Extras](analog-matrix-extras.md)
- [Wireless Communication](wireless-communication.md)
- [Protocol Overview](../protocol-overview.md) -- master protocol reference
