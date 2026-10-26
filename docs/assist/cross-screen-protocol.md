# Cross-Screen Protocol (Virtual KVM / RDP)

Keychron Assistant implements a decentralized, low-latency Virtual KVM system
("Cross-Screen", internally named `remote_desk` / `RDP`). It enables two computers
(running Windows, macOS, or Linux) on the same local network to share a single
keyboard and mouse seamlessly by moving the cursor across the edge of the screen,
similar to Synergy, Barrier, or Logitech Flow.

---

## 1. Architecture & Protocol Topology

```
┌─────────────────────────┐                           ┌─────────────────────────┐
│       Host PC A         │                           │       Host PC B         │
│     (192.168.1.10)      │                           │     (192.168.1.20)      │
│                         │                           │                         │
│  UDP Port 7654          │   Multicast Discovery     │  UDP Port 7654          │
│  239.255.255.250:7654   │◄─────────────────────────►│  239.255.255.250:7654   │
│                         │   DISCOVER_JSON:{...}     │                         │
│                         │                           │                         │
│                         │   Pairing Handshake       │                         │
│                         │──────────────────────────►│  Modal Prompt Dialog    │
│                         │   RequestPair / Confirm   │  (pair-confirm window)  │
│                         │◄──────────────────────────│                         │
│                         │                           │                         │
│  [Screen Right Edge]    │   UDP Input Event Stream  │  [Screen Left Edge]     │
│  Cursor hits boundary   │   RdpTocType::Init        │  Cursor enters screen   │
│  Entry ratio: 0.45      │──────────────────────────►│  Slave mode: controlled │
│                         │                           │                         │
│  Mouse & Keystrokes     │   MouseMove, MouseClick   │  Enigo / SecureInput    │
│  captured via hooks     │──────────────────────────►│  injects synthetic OS   │
│                         │   Keyboard, MouseScroll   │  events into Win/Mac    │
│                         │                           │                         │
│  Cursor returns left    │   RdpTocType::Release     │  Control returned to    │
│                         │──────────────────────────►│  Host PC B             │
└─────────────────────────┘                           └─────────────────────────┘
```

The system operates entirely peer-to-peer over the local area network (LAN)
without requiring internet access or cloud relay infrastructure.

---

## 2. Peer Discovery (`udp_server`)

### Transport Parameters

- **Discovery Port:** `7654` — beacons only (`DISCOVER_JSON:` / legacy `DISCOVER:`),
  sent to IPv4 multicast `239.255.255.250:7654` plus subnet and limited
  broadcast, answered on the same port.
- **Pairing/Session Port:** `40001` — everything else (`RequestPair`,
  `RequestConfirm`, input events, file transfer, heartbeats). Seen in
  traffic from a Windows peer; the number is not stored as a constant in
  the binary.
- **Local Bind Address:** `0.0.0.0:7654` (in code) and the session port above.

### Envelope Encoding Details

Example from a Windows peer:

```json
{"evt_type":"RemoteDesk","evt_data":{"evt_type":"Disconnect","evt_data":""},"source_addr":""}
```

- `source_addr` is always a string (`""` when unset), never `null`.
- Unit-like payloads (`Disconnect`, and by extension `PairCancel`,
  `Heartbeat`) serialize as `""`, not `null`.

### Discovery Message Format

When peer discovery is active (`switch_rdp_scan(true)`), Assistant broadcasts
beacon packets periodically:

```
DISCOVER_JSON:<Serialized JSON PeerClient>
```

Windows peers were observed sending a legacy plain-text variant instead:

```
DISCOVER:<hostname>:<os_info>
```

e.g. `DISCOVER:DESKTOP-ABC123:Windows 11 (26200)`.

#### Peer Record (`struct PeerClient`)

```json
{
  "device_id": "b3e2a104-58f2-49a1-9c60-8f921ab07e99",
  "name": "Tymon-MacBook-Pro",
  "addr": "192.168.1.10:7654",
  "os_info": "MacOS 14.5",
  "screen_resolution": {
    "width": 3024,
    "height": 1964
  },
  "interface_name": "en0",
  "network_type": "wifi",
  "mac_address": "3c:22:fb:11:22:33",
  "session_id": "9a1f28b7-6401-44bb-bc89-11823901bca0"
}
```

- `network_type`: `"wifi"`, `"ethernet"`, or `"other"`.
- `os_info`: Reports `"Windows"`, `"MacOS"`, or `"Linux"` and OS release.
- When Assistant receives a `DISCOVER_JSON:` beacon from a new IP, it emits a
  `LanDeviceDiscover` event to the frontend UI with the peer details.

---

## 3. Pairing Handshake & Mutual Trust

Before input events can be forwarded between devices, the two machines must pair.

```
Device A (Initiator)                                Device B (Receiver)
        │                                                    │
        │  RequestPair (dir, monitors, trust_uuid)           │
        ├───────────────────────────────────────────────────►│
        │                                                    │ (Shows pair-confirm
        │                                                    │  window popup)
        │                                                    │
        │  RequestConfirm (sessionId, trust_uuid)            │
        │◄───────────────────────────────────────────────────┤ (User clicks "Accept")
        │                                                    │
        │  Active Connection Established                     │
        │◄══════════════════════════════════════════════════►│
```

### 1. `RequestPair` Packet (`struct RdpRequestPairInfo`)

Sent by the initiator to start a cross-screen relationship. Example from a
Windows peer:

```json
{
  "addr": "",
  "device_id": "cb60d8ba-7684-4dc6-a3e1-16f549713702",
  "name": "DESKTOP-VFK8EUO",
  "os_info": "Windows 11 (26200)",
  "screen_resolution": {
    "width": 2560,
    "height": 1440
  },
  "monitors": [
    {
      "width": 2560,
      "height": 1440,
      "is_primary": true,
      "index": 0
    }
  ],
  "dir": "Left",
  "automatic": false,
  "session_id": "4420-1791234085600094600"
}
```

Notes:

- `addr` is empty on the wire; the receiver uses the UDP source. There is no
  `network_type` / `mac_address` / `interface_name` here (those live in the
  discovery beacon instead), and `trust_uuid` is omitted when there is no
  trust relationship yet.
- `Monitor` has exactly 4 fields (`width`, `height`, `is_primary`, `index`).

### 2. User Confirmation Dialog (`pair-confirm`)

Upon receiving `RequestPair`, Receiver opens the `pair-confirm` window:
- Shows peer machine name, IP, OS, and direction.
- Has a countdown timer (expires in 60 seconds).
- User can check **"Trust this device and accept future requests automatically"**.

### 3. `RequestConfirm` Packet (`struct RdpConfirmInfo`)

If accepted, Receiver responds with:

```json
{
  "addr": "192.168.1.20:7654",
  "device_id": "8f39a012-7bb1-41d9-9533-5c0291ba44a1",
  "name": "Tymon-MacBook-Pro",
  "os_info": "MacOS 14.5",
  "session_id": "49ba01e1-c88f-4311-9e8a-df01b7a82910",
  "screen_resolution": { "width": 3024, "height": 1964 },
  "trust_uuid": "e4b1092a-33b0-4f91-8841-a129048a12ef"
}
```

- If rejected, Receiver sends `RequestReject` (`struct RdpRejectInfo { addr, trust_uuid, reason }`).
- If Initiator cancels before confirmation, it sends `PairCancel`.

### 4. Automatic Connection & Trust Tokens

If both devices check the "Trust" option, mutual `trust_uuid` tokens are persisted
in `rdp_settings.json`. On future launches, `RequestPair` carries `automatic: true`.
When both sides have matching configured trust IDs, the pairing dialog is bypassed
and the connection is authorized automatically.

---

## 4. Screen-Crossing Mechanics & Coordinates

### Screen Crossing Edges (`RdpDir`)

The screen boundary direction defines where the mouse leaves the local display:

| `RdpDir` | Description | Typical Multi-Monitor Setup |
|----------|-------------|-----------------------------|
| `Left` | Mouse exits through left screen edge | Target PC is positioned to the left |
| `Right` | Mouse exits through right screen edge | Target PC is positioned to the right |
| `Top` | Mouse exits through top screen edge | Target PC is positioned above |
| `Bottom` | Mouse exits through bottom screen edge | Target PC is positioned below |
| `Off` | Cross-screen disabled | Standalone operation |

### Normalized Ratio Coordinates (`0.0 ~ 1.0`)

> **Note:** Edge positions are exchanged as **normalized entry/exit ratios**
> in the range `[0.0, 1.0]`, not as pixel coordinates.

When the mouse hits the boundary:
$$\text{Entry Ratio} = \frac{\text{Cursor Position on Edge Axis}}{\text{Length of Edge}}$$

For a vertical edge (Left or Right), $\text{ratio} = y / \text{height}$.
On the target machine, the incoming cursor position is calculated as:
$$y_{\text{remote}} = \text{Entry Ratio} \times \text{Remote Height}$$

If an older client transmits absolute coordinates:
`跨屏边缘比例超出 0~1（收到 ），对端很可能是未更新的旧版本，仍在发送绝对像素`

---

## 5. Input Event Stream (`RdpTocType`)

While control is shifted to the remote peer, all local keyboard and mouse inputs
are intercepted (using `SetWindowsHookEx` on Windows or `CGEventTap` on macOS)
and forwarded over UDP.

### Message Variants

The UDP channel encodes input events as serialized `RdpTocType` variants:

```rust
pub enum RdpTocType {
    Init(f32, RdpDir),
    MouseMove(f32, f32),
    MouseClick(MouseAction, InputState),
    MouseScroll(i32, i32),
    Keyboard(u32, InputState, u32),
    Release(f32, RdpDir),
}
```

#### 1. `Init(entry_ratio, edge)`
- **Direction:** Master $\to$ Slave
- **Payload:** `(f32, RdpDir)` (e.g., `(0.45, "Right")`)
- **Action:** Remote peer transitions to controlled/slave mode (`本机转为被控`). Warps the remote mouse cursor to the corresponding opposite edge (`Left`) at $y = 0.45 \times \text{height}$.

#### 2. `MouseMove(dx, dy)`
- **Direction:** Master $\to$ Slave
- **Payload:** Relative motion deltas or normalized relative offset.
- **Action:** Moves remote mouse cursor using `Enigo` / `SendInput` / `CGEventPost`.

#### 3. `MouseClick(button, state)`
- **Direction:** Master $\to$ Slave
- **`MouseAction`:**
  - `0`: Left button
  - `1`: Right button
  - `2`: Middle button
  - `3`: Button 4 (Back)
  - `4`: Button 5 (Forward)
  - `5`: Button 8 / custom
- **`InputState`:** `Press` (`1`) or `Release` (`0`).

#### 4. `MouseScroll(horizontal, vertical)`
- **Direction:** Master $\to$ Slave
- **Payload:** Scroll wheel deltas (including high-resolution / tilt wheel events).

#### 5. `Keyboard(keycode, state, modifiers)`
- **Direction:** Master $\to$ Slave
- **Payload:** `(keycode: u32, state: InputState, modifiers: u32)`
- **Keycode Translation:** The sender normalizes keycodes. On Windows, virtual keycodes (`VK_...`) or `rdev` keys are translated. On macOS, `win_vk_to_cg_keycode` or `rdev_key_to_cg_keycode` converts codes to Apple `CGKeyCode` before injection.

#### 6. `Release(exit_ratio, edge)`
- **Direction:** Master $\to$ Slave
- **Payload:** `(f32, RdpDir)`
- **Action:** Remote peer releases mouse control back to local cursor (`光标交还本机`). Master restores local cursor at the edge coordinates.

---

## 6. Connection Health & Watchdog

To prevent a PC from being stuck in controlled mode if the network drops or the
peer crashes:

1. **Heartbeat Sender (`spawn_heartbeat_sender`):**
   - Transmits a lightweight `Heartbeat` packet every 1,000 ms over UDP.
2. **Peer Watchdog (`spawn_peer_watchdog`):**
   - If no heartbeat or input packet is received within the watchdog timeout
     (typically 5,000 ms), the connection triggers `network_timeout`.
   - The controlled machine immediately unlocks and restores local mouse/keyboard control:
     `[跨屏] 接收到对端关闭通知，本端释放鼠标控制权`

---

## 7. Network Change Detection (`spawn_network_change_watcher`)

Cross-screen sessions monitor local network interface changes (e.g., laptop
switching from Ethernet to Wi-Fi, or roaming between access points):

- **Baseline Fingerprint (`PeerRouteFingerprint`):** Records active route, subnet, and default gateway.
- **Change Detection:** Polled via `sysinfo` and `if-addrs`.
- **Handling Rules:**
  - If network interface changes but route to peer remains reachable: logs `检测到网卡/IP列表变化，但当前跨屏链路未变化，忽略本次波动` and maintains connection.
  - If route to peer is broken: logs `跨屏] 网络环境变化，终止当前连接状态` and alerts frontend UI with `NetworkChanged`.

---

## 8. Emergency Shortcuts

To guarantee that the user can never be permanently locked out if a peer becomes
unresponsive while controlling the cursor:

- **Disconnect Shortcut (`set_rdp_disconnect_shortcut`):**
  - Default: `Ctrl + Alt + D` (customizable in UI, cannot be disabled).
  - Immediately terminates the cross-screen link and returns cursor to the host PC.
- **Lock Computer Shortcut (`set_system_lock_shortcut`):**
  - Dispatches `LockScreen` command over network and locks the local OS workstation (`LockWorkStation` on Windows, SAC screen saver on macOS).
