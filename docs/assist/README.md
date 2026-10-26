# Keychron Assistant Documentation (`keychron-assist`)

This directory documents the internals, wire protocols, architecture, and
companion subsystems of **Keychron Assistant** (`keychron-assist`).

The documentation was produced by reverse-engineering the official Windows
and macOS installers, which the Launcher downloads from
`https://launcher.keychron.com/static/assist/KeychronAssist-<version>.zip`
(Windows) and `.../KeychronAssist-<version>.dmg` (macOS).

---

## Documentation Index

| Document | Topic & Scope |
|----------|---------------|
| [**Architecture & Overview**](overview.md) | Two-process architecture, Tauri v2 stack, package layouts (Win/Mac), deep linking (`launcherassist://`), directory storage schemas (`~/.Keychron_Screen/`), and embedded asset extraction |
| [**WebSocket Protocol**](websocket-protocol.md) | `ws://127.0.0.1:40003` companion RPC protocol: envelope framing, application streaming (`FetchStart`/`Fetch`/`FetchEnd`), keybinding management, `0x7f00` keycode mapping table, and receiver VPID mapping |
| [**Cross-Screen Protocol (Virtual KVM)**](cross-screen-protocol.md) | Peer-to-peer virtual KVM: UDP multicast discovery (`239.255.255.250:7654`), pairing handshake (`RequestPair`/`RequestConfirm`), trust tokens, normalized entry ratio coordinates (`0.0`--`1.0`), input streaming (`RdpTocType`), and network watchdog |
| [**File Transfer Protocol**](file-transfer-protocol.md) | Reliable datagram file streaming: chunked sliding-window protocol, batch acknowledgments (`FileBatchAck`), retransmissions, duplicate renaming, and 15-field transfer history schema |
| [**AI Key & MOS Subsystems**](ai-key-and-mos.md) | Dedicated hardware AI Key prompt integration (`AiKeyConf`) with Keychron Launcher's LangChain agent, plus MOS smooth scrolling and acceleration physics engine (`MosConf`) |
| [**Secure Input Service**](secure-input-service.md) | Privileged helper service (`secure_input_service`): bypassing Windows Secure Desktop (`WinSta0\Winlogon`) for UAC and lock-screen input injection via Win32 Named Pipes |
| [**Device Listener & HID Interception**](device-listener.md) | Low-level hardware drivers: `async-hid` on Windows, `IOHIDManager` + `CGEventTap` on macOS, Consumer Control usage mapping (`0x01cc`--`0x01ff`), mouse button tables, and default action suppression |
| [**Tauri IPC & Internal Commands**](internal-ipc.md) | Reference of the internal Tauri commands (`H()`), parameters, return types, and asynchronous notification events (`RustEvent`) |

---

## High-Level Summary

Keychron Assistant serves three primary roles in the Keychron ecosystem:

1. **Launcher Companion Bridge (`ws://127.0.0.1:40003`):**
   Allows the web-based Keychron Launcher to scan installed desktop apps,
   open system dialogs, bind physical keyboard buttons to launch apps or URLs,
   and persist quick-start configurations locally.

2. **Cross-Screen Virtual KVM & File Sharing (`0.0.0.0:7654`):**
   Enables two computers on the same LAN to share a single keyboard and mouse
   seamlessly when the cursor crosses a configured screen edge, and to stream
   files between machines.

3. **Hardware Augmentation (AI Key & Smooth Scrolling):**
   Provides hardware integration for dedicated keyboard AI keys (triggering
   custom prompts via LangChain and DeepSeek) and provides an advanced mouse
   smooth scrolling engine (MOS) on macOS and Windows.
