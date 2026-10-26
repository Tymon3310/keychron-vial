# Keychron Cross -- Cross-Screen Multi-Device Sharing Protocol

Keychron Launcher introduces **Keychron Cross** (the "Cross" feature), a
cross-computer sharing and peripheral redirection suite. It enables seamless
keyboard and mouse sharing across multiple computers (similar to Logitech Flow
or Barrier/Synergy), synchronized clipboard, drag-and-drop file transfer,
and shared global hotkeys.

> **Source:** Reverse-engineered from the Keychron Launcher web application bundle
> (`/cross` route) and the Keychron Assistant desktop app (`keychron-assist`).

---

## Architecture Overview

Keychron Cross employs a hybrid web/native architecture:

1. **Web Frontend (Launcher):**
   - Implemented in the Angular route `/cross` (`CrossModule`).
   - Accessible from the header navigation bar via `nav-cross-screen` (`assets/cross/c-nav.svg`).
   - Provides setup status, device pairing visualization, display arrangement, and configuration.

2. **Native Companion Service (Keychron Assistant):**
   - Runs locally as a desktop daemon.
   - Exposes a local WebSocket server at `ws://localhost:40003`.
   - Operates a local UDP peer discovery service, encrypted LAN data channel, and mouse/keyboard interception engine.
   - Manages RDP-based peripheral redirection, clipboard synchronization, and network file transfer.

```
┌─────────────────────────┐               ┌─────────────────────────┐
│ Keychron Launcher (Web) │               │ Keychron Launcher (Web) │
│  https://launcher...    │               │  https://launcher...    │
└────────────┬────────────┘               └────────────┬────────────┘
             │ WebSocket                               │ WebSocket
             │ (localhost:40003)                       │ (localhost:40003)
┌────────────┴────────────┐   Encrypted LAN   ┌────────┴────────────┐
│   Keychron Assistant    │◄─────────────────►│   Keychron Assistant    │
│       (Host A)          │   UDP/TCP Data    │       (Host B)          │
└────────────┬────────────┘                   └────────┬────────────┘
             │                                         │
    ┌────────┴────────┐                       ┌────────┴────────┐
    │ Keyboard / Mouse│                       │ Screen / OS B   │
    └─────────────────┘                       └─────────────────┘
```

---

## UI Components and Selectors

The `/cross` route uses these Angular components (by selector):

| Selector | Purpose |
|----------|---------|
| `nav-cross-screen` | Navigation bar icon. Header toggle button displaying `assets/cross/c-nav.svg`; navigates to `/cross` and remembers the prior route for quick return. |
| `app-cross` | Root container with `<router-outlet>`. |
| `app-cross-scan` | Main state coordinator; subscribes to WebSocket status events and toggles between offline, pairing, and connected views. |
| `app-cross-connect` | Displayed when Keychron Assistant is offline or disconnected; prompts user to launch or install the native assistant. |
| `cross-connect-step` | Interactive pairing lifecycle: displays local device name, discovering/pending countdown timer (59s), remote authorization, and established link status. |
| `cross-connected-view` | Active connection dashboard displaying directional flow, target host, clipboard stats, file transfer path, and active hotkeys. |

---

## WebSocket Protocol (`CrossScreen`)

All web-to-daemon communication for Keychron Cross uses the `"CrossScreen"` event channel over the existing Keychron Assistant WebSocket transport.

### Framing Envelope

```json
{
  "evt_type": "CrossScreen",
  "evt_data": {
    "cmd": "<CommandName>",
    "data": "<Payload>"
  }
}
```

### Client Commands (Launcher → Assistant)

| Command | `cmd` | `data` | Description |
|---------|-------|--------|-------------|
| **Get Status** | `"Get"` | `""` | Queries the current cross-screen connection state, active peer, and settings. |
| **Disconnect** | `"Disconnect"` | `""` | Requests the daemon to disconnect from the currently connected peer computer. |
| **Launch Assistant** | `"OpenApp"` | `null` | *(Sent with `evt_type: "Common"`)* Focuses or opens the native Keychron Assistant desktop window. |

The Launcher automatically sends `{"cmd": "Get", "data": ""}` with `evt_type: "CrossScreen"` upon establishing the WebSocket connection, and repeats every 2.5 seconds if waiting for status confirmation.

---

## State Synchronization Messages (Assistant → Launcher)

The daemon emits asynchronous status updates when peer discovery changes, connections establish, or configuration changes occur.

Messages are handled if `e.event === "StateSync"` or `e.cmd` is one of `["Get", "Status", "StatusChanged", "Update"]`.

### `StateSync` Payload Schema

```json
{
  "connected": true,
  "local_device": {
    "name": "MacBook-Pro",
    "addr": "192.168.1.105"
  },
  "device": {
    "name": "Office-Desktop",
    "addr": "192.168.1.120",
    "os_info": "Windows 11 Pro",
    "trusted": true
  },
  "trusted": true,
  "position": "right",
  "hold_ctrl_to_cross": false,
  "active_file_transfers": 0,
  "settings": {
    "clipboard_enabled": true,
    "clipboard_sync_mode": "bidirectional",
    "clipboard_recent_items": 12,
    "clipboard_max_items": 50,
    "file_transfer_enabled": true,
    "receiving_folder": "C:\\Users\\User\\Downloads\\KeychronCross",
    "trusted_device_count": 2,
    "automatic_connection_device": "dev_win_desktop_01",
    "automatic_connection_device_name": "Office-Desktop",
    "trusted_devices": [
      {
        "device_id": "dev_win_desktop_01",
        "name": "Office-Desktop",
        "addr": "192.168.1.120"
      }
    ],
    "disconnect_shortcut": {
      "key": "Ctrl+Alt+Q"
    },
    "system_lock_shortcut": {
      "key": "Ctrl+Alt+L"
    }
  }
}
```

### Internal Normalized Model

The Launcher normalizes this data into a standardized reactive state:

```typescript
interface CrossStatus {
  connected: boolean;
  localDeviceName: string;
  device?: {
    name: string;
    system: string;
    platform: "Win" | "Mac" | "Linux" | "";
    trusted: boolean;
  };
  keyboardMouse: {
    enabled: boolean;
    direction: "left" | "right" | "-";
    targetDevice: string;
    holdCtrlToCross: boolean;
  };
  clipboard: {
    enabled: boolean;
    syncMode: string;
    recentItems: number;
    maxItems: number;
  };
  fileTransfer: {
    enabled: boolean;
    dropBoxKey: string;     // Target save folder path
    ongoing: number;        // Count of active transfers
  };
  settings: {
    trustedDevices: number;
    autoConnectTarget: string;
    shortcuts: {
      exitApplication?: string;
      switchPc?: string;
      centerCursor?: string;
      masterSlaveSwitch?: string;
      lockPc?: string;
      customScreenshot?: string;
    };
  };
}
```

---

## Connection Lifecycle & Step Flow

The pairing process shown in `cross-connect-step` follows a 3-step state machine:

```
[ Step 1: Local Device Ready ]
           │
           ▼ (LAN Peer Discovery / Handshake)
[ Step 2: Peer Authorization ]  ── (59-second Countdown / Cancel)
           │
           ▼ (Peer Confirms)
[ Step 3: Connection Established ]
```

1. **Step 1: Local Device Identification:**
   - Detects local host name and OS platform (`getPlatform()`).
   - Uses asset `assets/cross/scan-0.svg`.
2. **Step 2: Peer Discovery and Approval:**
   - Displays animated scanner (`assets/cross/scan-1.svg`).
   - Countdown timer initializes to **59 seconds** (`requestCountdown = 59`).
   - The user or remote computer must accept the link request before timeout.
   - User can click `cancelConnectionRequest()` to abort.
3. **Step 3: Established Devices View:**
   - Displays connected device pills (`local-pill` and `remote-pill`) with animated connection pulse dots (`assets/cross/scan-2.svg`).
   - Can be collapsed into a compact header toolbar or expanded into a full view.
   - Shows screen arrangement flow: `assets/cross/screen-left.svg` / `assets/cross/screen-right.svg` with animated directional transfer arrow (`cross-arrow-move`).

---

## Native Assistant Backend Services

While the Launcher web interface manages user configuration and UI state visualization,
the underlying virtual KVM networking, encrypted communication, and input redirection are
executed by the native Keychron Assistant daemon:

- **LAN Peer Discovery:** UDP multicast on `239.255.255.250:7654` using `LanDeviceDiscover`.
- **Virtual KVM / RDP Input Tunnel:** Full cursor capture, screen edge crossing calculation,
  and input event streaming (`RdpTocType`).
- **P2P Reliable File Transfer:** Reliable chunked datagram transfer with batch ACKs
  (`FileBatchAck`) and automated history tracking.
- **Secure Input Injection:** Windows `secure_input_service` (SYSTEM account) / macOS Accessibility
  APIs for elevated and lock-screen cursor movement.

> **Comprehensive Daemon Specifications:**
> For exact wire formats, packet headers, UDP beacon structures, and Rust structs:
> - [`assist/cross-screen-protocol.md`](../assist/cross-screen-protocol.md) -- UDP discovery, TCP handshake, and virtual KVM input streaming
> - [`assist/file-transfer-protocol.md`](../assist/file-transfer-protocol.md) -- Reliable chunked file transmission and history logging
> - [`assist/secure-input-service.md`](../assist/secure-input-service.md) -- Privileged helper service and UAC/lock-screen input injection

---

## Related Documents

- [AI Assistant & Lighting](ai-assistant.md) -- Launcher in-browser AI tools and Assistant bridge
- [Launcher Overview](overview.md) -- Overall architecture and route definitions
- [Mouse / Trackball Protocol](mouse-protocol.md) -- Pointer hardware protocols
- [Assistant Architecture Overview](../assist/overview.md) -- Keychron Assistant native desktop companion
