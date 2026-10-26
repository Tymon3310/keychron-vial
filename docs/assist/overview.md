# Keychron Assistant Architecture & Overview

Keychron Assistant (`keychron-assist`, package identifier `com.keychron.assist`)
is the official native desktop companion application that works alongside the
Keychron Launcher web application (`launcher.keychron.com`) and Keychron
hardware peripherals (keyboards, mice, and 2.4 GHz wireless receivers).

This document details the architectural design, technology stack, binary
distribution, execution lifecycle, configuration layout, and security model
of Keychron Assistant, reverse-engineered from the official Windows (`.zip`)
and macOS (`.dmg`) installers served from
`https://launcher.keychron.com/static/assist/`.

---

## 1. System Architecture

Keychron Assistant is structured as a two-process desktop application with
multithreaded background networking and device monitoring:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                      Keychron Launcher (WebHID / Web)                    │
│                        https://launcher.keychron.com                     │
└────────────────────────────────────┬─────────────────────────────────────┘
                                     │ WebSocket (ws://127.0.0.1:40003)
                                     ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                     Keychron Assist Main Process                         │
│                    (keychron-assist / Tauri v2 GUI)                      │
│                                                                          │
│  ┌───────────────────────┐  ┌────────────────────┐  ┌─────────────────┐  │
│  │ Embedded Angular SPA  │  │ WebSocket Server   │  │ UDP Discovery   │  │
│  │ (Brotli-compressed    │  │ ws://127.0.0.1:    │  │ & Peer Engine   │  │
│  │  in binary context)   │  │ 40003 (JSON RPC)   │  │ 239.255.255.250 │  │
│  └───────────┬───────────┘  └──────────┬─────────┘  └────────┬────────┘  │
│              │ Tauri IPC (invoke)      │                     │           │
│  ┌───────────▼─────────────────────────▼─────────────────────▼────────┐  │
│  │ Rust Backend Services (keychron_assist_lib)                        │  │
│  │  • service::installed       (App scanning & quick-start store)     │  │
│  │  • service::ai_key          (AI key template dispatch)             │  │
│  │  • service::mos             (Mouse smooth scrolling & accel)       │  │
│  │  • service::remote_desk     (Cross-screen KVM & file streaming)    │  │
│  │  • service::device_listener (Raw HID / CGEventTap / async-hid)     │  │
│  └──────────────────────────────────────┬─────────────────────────────┘  │
└─────────────────────────────────────────┼────────────────────────────────┘
                                          │
                  ┌───────────────────────┴───────────────────────┐
                  │ IPC (Win32 Named Pipe / Unix Domain Socket)   │
                  ▼                                               ▼
┌──────────────────────────────────────┐     ┌─────────────────────────────┐
│ Windows: secure_input_service.exe    │     │ macOS: secure_input_service │
│ - Windows Service: SYSTEM account    │     │ - Privileged input helper   │
│ - Desktop switch: WinSta0\Winlogon   │     │ - CGEventPost injection     │
│ - SendInput() on UAC & Lock screen   │     │ - TCC Accessibility API     │
└──────────────────────────────────────┘     └─────────────────────────────┘
```

### Primary Components

1. **Main Application (`keychron-assist`)**:
   - Built on **Tauri v2** (`tauri`, `tauri-runtime-wry`, `tao`, `wry`).
   - Runs an asynchronous event runtime backed by **Tokio**.
   - Serves an embedded Angular Single-Page Application (SPA) extracted from `dist/keychron-assist/browser`, compressed using **Brotli** and loaded via custom protocol handling.
   - Manages desktop system tray integration (`tray-icon`, `muda`).
   - Implements native background daemons for device event monitoring, WebSocket RPC, UDP peer discovery, and cross-screen control.

2. **Privileged Helper (`secure_input_service`)**:
   - Shipped as a standalone companion binary inside `binaries/secure_input_service`.
   - On Windows: Installed and run as an `NT AUTHORITY\SYSTEM` service named `KeychronAssistSecureInput`. Connects through named pipe `\\.\pipe\KeychronAssistSecureInput`. Switches to the secure desktop (`Winlogon` / UAC prompt) to allow seamless mouse and keyboard event injection when the system is locked or displaying elevated dialogs.
   - On macOS: Runs with Accessibility permissions to inject CoreGraphics synthetic events across privileged windows.

---

## 2. Technology Stack & Dependencies

Analysis of the binary symbols and compile artifacts reveals the complete dependency graph:

| Component | Role in Assistant |
|-----------|-------------------|
| `tauri` | Desktop application framework (windowing, IPC, asset bundling) |
| `tauri-runtime-wry` | Webview renderer backend using platform webview (WebView2 on Win, WKWebView on macOS) |
| `tao` | Cross-platform window creation and event loop |
| `wry` | Webview rendering abstraction |
| `tokio` | Multithreaded async I/O runtime (sockets, pipes, tasks, timers) |
| `tungstenite` / `tokio-tungstenite` | Native WebSocket protocol implementation for port `40003` |
| `async-hid` | Low-level asynchronous HID device enumeration and raw report I/O on Windows |
| `windows-service` | Windows Service Control Manager (SCM) lifecycle integration |
| `enigo` | Cross-platform synthetic mouse/keyboard input injection |
| `core-graphics` | CoreGraphics event tap (`CGEventTap`) and event creation (`CGEventCreate`) |
| `display-info` | Monitor display enumeration and resolution query |
| `brotli-decompressor` | Decompression of inlined web assets at application launch |
| `sysinfo` | System information, network interface, and resource monitoring |
| `winreg` / `windows-registry` | Windows Registry access for software scanning and autostart |
| `serde` / `serde_json` | JSON serialization/deserialization of network and storage payloads |
| `rfd` | Native file dialog picker ("Select Application Executable", file transfer) |

---

## 3. Package & Binary Distribution

### Windows Package (`.zip` containing an NSIS installer)

- **Format:** Nullsoft Scriptable Install System (NSIS) self-extracting installer.
- **Installer payload files:**
  - `keychron-assist.exe` (PE32+ 64-bit GUI)
  - `secure_input_service.exe` (PE32+ 64-bit console)
  - `uninstall.exe`
  - `$PLUGINSDIR/` (NSIS Tauri plugin helpers: `nsis_tauri_utils.dll`, `NSISdl.dll`, etc.)
- **Installation Directory:** `%LOCALAPPDATA%\Programs\Keychron-Assistant\`
- **Autostart:** Configured via Windows Registry `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`.
- **Service Registration:** Registers `KeychronAssistSecureInput` in Windows Service Manager pointing to `secure_input_service.exe`.

### macOS Package (`.dmg`)

- **Format:** Apple Disk Image (APFS volume named `Keychron Assist`).
- **Bundle layout (`Keychron-Assistant.app`):**
  - `Contents/MacOS/keychron-assist` (Mach-O Universal 64-bit: `x86_64` + `arm64`)
  - `Contents/MacOS/secure_input_service` (Mach-O 64-bit `arm64`)
  - `Contents/Info.plist`:
    - `CFBundleIdentifier`: `com.keychron.assist`
    - `CFBundleURLSchemes`: `launcherassist`
    - `LSMinimumSystemVersion`: `10.13`
  - `Contents/Resources/icon.icns`
  - `Contents/_CodeSignature/CodeResources`

---

## 4. Deep-Link Protocol (`launcherassist://`)

Keychron Assistant registers a custom system URL protocol handler:

- **Scheme:** `launcherassist://`
- **Tauri Plugin:** `tauri-plugin-deep-link`
- **Actions handled:**
  - `launcherassist://show-window`: Reveals and focuses the Assistant main GUI window.
  - `launcherassist://hide-window`: Hides the GUI window to the system tray.
  - `launcherassist://open-app?...`: Dispatches an application launch request.

When a browser navigates to `launcherassist://`, the OS launches or activates
Keychron Assistant. If the app is already running, the single-instance listener
(`tauri-plugin-single-instance`) receives the deep link arguments and triggers
`[DeepLinkDebug] WebSocket OpenApp received, action=show-window`.

---

## 5. Storage & Configuration Files

Keychron Assistant stores its operational configuration, key bindings,
device associations, and cross-screen histories in a dedicated user directory.

### Directory Locations

- **Windows:** `%USERPROFILE%\.Keychron_Screen\`
- **macOS / Linux:** `~/.Keychron_Screen/`
- **Default File Transfer Download Directory:** `~/Downloads/Keychron_Screen/`

### Configuration Files

| File | Purpose | Format |
|------|---------|--------|
| `assist.json` | Master settings store (`GlobalJson`: quick-start bindings, MOS configuration, AI key configs, schema version) | JSON |
| `assist_key.json` | AI key templates and quick trigger definitions | JSON |
| `rdp_settings.json` | Cross-screen trust tokens, mutual pairing credentials, and autoconnect device IDs | JSON |
| `rdp_history.json` | Cross-screen connection history records (incoming/outgoing device metadata) | JSON |
| `file_transfer_history.json` | Cross-screen file transfer session log and completion metrics | JSON |
| `i18n_cache.json` | Cached language strings and locale preferences | JSON |
| `log.json` / `secure_input.log` | Operational debug traces and Windows Service execution log | Text / JSON lines |

### Master Configuration Schema (`assist.json`)

The top-level configuration object `GlobalJson` serializes into 4 primary blocks:

```json
{
  "version": "<app version>",
  "quick_start": {
    "apps": [
      {
        "name": "Google Chrome",
        "path": "C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe",
        "icon": "",
        "icon_path": ""
      }
    ],
    "keybinds": {
      "default": {
        "0x7f00": {
          "type": "App",
          "name": "Google Chrome",
          "path": "C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe",
          "icon": "",
          "icon_path": ""
        }
      }
    },
    "receiver_maps": {
      "13364:4200": {
        "receiver_vpid": "13364:4200",
        "device_vpid": "13364:4100",
        "device_type": 1
      }
    }
  },
  "mos": {
    "speed": 1.0,
    "step": 10.0,
    "duration": 0.5,
    "enable": true,
    "auto_enable": false,
    "detect_distance": 5.0,
    "interval": 16,
    "reverse_key": 0,
    "speed_key": 0
  },
  "ai_key": {
    "list": [
      {
        "key": "0x7f01",
        "template": "Summarize this selected text",
        "window": true,
        "dialog": false,
        "model": "deepseek-v4-pro",
        "vp_id": "13364:4100"
      }
    ]
  }
}
```

---

## 6. Embedded Web Assets & Window Management

The frontend interface of Keychron Assistant is an Angular SPA compiled and
inlined into the executable.

### Window Definitions

Tauri initializes two primary window targets defined in the Tauri context:

1. **`main` (`/index.html`)**:
   - The primary application interface.
   - Tabs: Quick Start Application (`/home`), Cross-Screen Management (`/cross`), Settings (`/settings`).
   - Default hidden on launch; accessible via tray menu or `launcherassist://show-window`.

2. **`pair-confirm` (`/index.html?window=pair-confirm`)**:
   - Modal popup window titled `Connection Request`.
   - Displayed immediately upon receiving a remote cross-screen pairing request (`RequestPair`).
   - Shows remote peer hostname, operating system, IP address, and controls:
     - `Accept Request` (with optional `Trust this device` checkbox)
     - `Reject`
   - Features custom platform transparency (`apply_pair_confirm_macos_transparency`, `configure_pair_confirm_view_layer`).

### Inlined Asset Extraction

Assets are embedded in the PE/Mach-O `.rdata` / `__const` sections and mapped via
`tauri_utils::assets::EmbeddedAssets`. Every asset is stored compressed with
Brotli. At runtime, Tauri intercepts requests to `tauri://localhost/*` and
decompresses the requested file directly into memory without touching the disk.

The embedded bundle contains:
- `index.html`, `styles-<hash>.css`, `global.css`
- `main-<hash>.js`, `chunk-<hash>.js`, `polyfills-<hash>.js`
- Locales: `en-US.json`, `zh-CN.json`, `ja-JP.json`, `ko-KR.json`
- Cross-screen assets: `connect-dark.png`, `connect-light.png`, `panel-win.svg`, `panel-mac.svg`, `scan-loading.svg`, `scan-0.svg`, `scan-1.svg`, `scan-2.svg`

---

## 7. Functional Subsystems Overview

| Subsystem | Port / Transport | Purpose | Documentation |
|-----------|------------------|---------|---------------|
| **Launcher WebSocket** | `ws://127.0.0.1:40003` | Bi-directional RPC with Keychron Launcher web UI | [websocket-protocol.md](websocket-protocol.md) |
| **Cross-Screen (KVM)** | UDP `239.255.255.250:7654` & `0.0.0.0:7654` | LAN peer discovery, pairing handshake, cursor crossing & input relay | [cross-screen-protocol.md](cross-screen-protocol.md) |
| **File Transfer** | UDP / Datagram batching | Sliding-window chunked file streaming between paired PCs | [file-transfer-protocol.md](file-transfer-protocol.md) |
| **AI Key & MOS** | Internal & WebSocket | AI prompt triggers & mouse smooth scroll/acceleration | [ai-key-and-mos.md](ai-key-and-mos.md) |
| **Secure Input** | Named Pipe / Unix Socket | SYSTEM-level input injection across Windows UAC / Lock screen | [secure-input-service.md](secure-input-service.md) |
| **Device Listener** | `async-hid` / `IOHIDManager` | Raw HID report interception for Keychron programmable keys | [device-listener.md](device-listener.md) |
| **Internal IPC** | Tauri invoke (`H()`) | Rust command handlers bridging frontend and services | [internal-ipc.md](internal-ipc.md) |
