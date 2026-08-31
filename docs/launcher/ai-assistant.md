# AI Assistant (Launcher "Assist" Panel)

The Keychron Launcher ships a LangChain-based AI chat panel that can directly
configure connected devices through tool calls. This document records the
architecture, backend endpoints, and the complete tool catalog.

> **Source:** Reverse-engineered from the Keychron Launcher JavaScript bundle
> (snapshot in `/src/keychron-launcher`).

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

| Model ID             | Provider  | Temperature | Thinking |
|----------------------|-----------|-------------|----------|
| `deepseek-v4-pro`    | deepseek  | 0.1         | disabled |
| `deepseek-v4-flash`  | deepseek  | 0.1         | disabled |

- **API base:** `https://launcher.keychron.cn/api` (OpenAI-compatible
  proxy; no provider API keys are embedded).
- DeepSeek-specific extension: requests carry
  `thinking: { type: "disabled" }`, and assistant messages replay
  `reasoning_content` when thinking is enabled.
- Telemetry: question statistics are posted to
  `https://erpapik.keychron.cn/launcher/vapi/ai-question-statistics`.
- Knowledge-base search (`search_knowledge_base`) queries the brand's
  `knowledgeBaseApi` host (e.g. `https://launcher.keychron.cn`); the FAQ /
  product knowledge base is fetched from there. A UI easter egg (5 clicks
  on the title bar within 2 s) reveals the LLM token-usage meter.

## Keychron Assistant Companion App

Quick-start app launching and key-binding storage are delegated to a native
companion app ("Keychron Assistant", expected version ≥ 1.1.5):

- **Transport:** WebSocket to `ws://localhost:40003`
  (a legacy config value `ws://localhost:23333` also exists).
- **Framing:** JSON messages `{ evt_type: "Common" | "QuickStart",
  evt_data: { cmd: string, data: string(JSON) } }`.
- **Commands:**

| Command                | Direction | Data payload |
|------------------------|-----------|--------------|
| `GetVersion`           | both      | `{ version }` |
| `Get`                  | both      | app-list fetch pages: `{ action: "FetchStart"\|"Fetch"\|..., count, data }` |
| `SpawnAppByPath`       | → app     | `{ path }` |
| `AddKeyBind` / `RemoveKeyBind` / `ClearKeyBind` | → app | key + binding descriptor |
| `SetReceiverVpidMap`   | → app     | `{ receiver_vpid, device_vpid, device_type }` |
| `ClearReceiverVpidMap` | → app     | `{ receiver_vpid }` |

- The receiver VPID map tells Assistant which dongle belongs to which
  device so quick-start keys work over 2.4 GHz; the launcher pushes the map
  on connect and clears it on page unload.

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

- [Launcher Overview](overview.md) -- architecture and transport
- [Command Map](command-map.md) -- underlying command IDs
- [Mouse / Trackball Protocol](mouse-protocol.md)
- [Analog Matrix Extras](analog-matrix-extras.md)
- [Wireless Communication](wireless-communication.md)
- [Protocol Overview](../protocol-overview.md) -- master protocol reference
