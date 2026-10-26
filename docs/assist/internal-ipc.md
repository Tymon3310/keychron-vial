# Tauri IPC & Internal Commands Reference

Keychron Assistant's embedded Angular user interface interacts with the native
Rust backend (`keychron_assist_lib`) through Tauri's Inter-Process Communication
(IPC) bridge. In the compiled JavaScript bundle, calls to `invoke("cmd", args)`
are wrapped via the minified helper `H("cmd", args)`.

This document lists the internal IPC commands (registered in the app's
`src/commands/mod.rs` and `src/lib.rs`, per the source paths embedded in the
binaries). Some are only present in the macOS or Windows build.

---

## 1. Event Stream & Notification System

In addition to request-response RPC, the Rust backend pushes asynchronous status
updates to the frontend using the Tauri event channel:

```javascript
// From the embedded Angular frontend:
listen("notify", r => { this.event$.next(r.payload); });
```

### Event Notification Envelope (`RustEvent`)

```rust
pub struct RustEvent<T> {
    pub evt_type: RustEventType,
    pub evt_data: T,
}

pub enum RustEventType {
    LanDeviceDiscover,    // Discovered a new peer via UDP
    WindowActive,         // Window focus / navigation change
    InstalledAppUpdate,   // Installed application cache was modified
    RDP,                  // Cross-screen pairing or connection state update
    FileTransfer,         // File transfer progress, chunk, or status change
    NetworkChanged,       // Local IP / default route change detected
    AppFound,             // Single application found during stream
    ScanComplete,         // Full application scan finished
}
```

---

## 2. Complete Command Catalog

### A. Application Scanning & Quick-Start

| Command | Parameters | Return Type | Description |
|---------|------------|-------------|-------------|
| `get_installed` | `{ refresh: bool }` | `Vec<Installed>` | Scans or retrieves cached installed desktop applications (`.exe` / `.app`) |
| `installed_app_add` | `{ installed: Installed }` | `()` | Manually registers an executable in the user's quick-start application catalog |
| `installed_app_remove` | `{ name: String }` | `()` | Removes an application entry from the quick-start list |
| `get_connected_kb_list` | None | `Vec<KeychronDeviceInfo>` | Queries currently connected Keychron keyboards and programmable key counts |
| `clear_key_bind` | `{ vpId: String }` | `()` | Clears saved quick-start key bindings for the specified device VPID |

---

### B. Cross-Screen (RDP) Discovery & Lifecycle

| Command | Parameters | Return Type | Description |
|---------|------------|-------------|-------------|
| `switch_rdp_scan` | `{ flag: bool }` | `()` | Starts (`true`) or stops (`false`) UDP multicast peer discovery on port `7654` |
| `lan_device_discover_start` | None | `()` | Explicitly initiates a local network device discovery cycle |
| `lan_broadcast_start` | None | `()` | Starts broadcasting `DISCOVER_JSON:` announcements on `239.255.255.250` |
| `validate_port_status` | None | `bool` | Verifies whether UDP port `7654` and WebSocket port `40003` are available |
| `get_local_device_info` | None | `PeerClient` | Returns local machine metadata (ID, hostname, IP, screen resolution, OS) |
| `get_plat` | None | `String` | Returns host operating system identifier (`"windows"`, `"macos"`, `"linux"`) |

---

### C. Cross-Screen Pairing & Connection Management

| Command | Parameters | Return Type | Description |
|---------|------------|-------------|-------------|
| `rdp_connect` | `{ addr: String, trustUuid?: String, automatic: bool, dir: RdpDir }` | `()` | Dispatches a `RequestPair` connection request to a discovered peer IP:port |
| `rdp_confirm` | `{ addr: String, dir: RdpDir, trustUuid?: String, deviceId: String, deviceName: String, sessionId: String }` | `()` | Confirms and accepts an incoming pairing request from a remote peer |
| `rdp_confirm_dialog` | `{ pair: RdpRequestPairInfo, trusted: bool, trustUuid?: String }` | `bool` | Invoked by the `pair-confirm` popup window when user clicks "Accept Request" |
| `rdp_reject` | `{ addr: String, trustUuid?: String }` | `()` | Sends a `RequestReject` notification to decline an incoming pairing attempt |
| `rdp_reject_dialog` | `{ pair: RdpRequestPairInfo }` | `()` | Invoked by the `pair-confirm` popup window when user clicks "Reject" |
| `rdp_cancel_pairing` | `{ addr: String }` | `()` | Cancels an outgoing pairing request before peer responds |
| `rdp_complete_connection` | `{ addr: String, deviceId: String, deviceName: String, sessionId: String }` | `()` | Finalizes handshake and establishes input event streaming channels |
| `rdp_disconnect` | None | `()` | Gracefully terminates active cross-screen connection and returns cursor control |
| `rdp_reset_stale_pairing_state` | None | `()` | Resets internal timeout and lock state if a handshake stalled |
| `rdp_should_initiate_automatic` | `{ addr: String }` | `bool` | Checks whether peer matches a trusted autoconnect credential in `rdp_settings.json` |
| `rdp_trust_current_peer` | `{ trustUuid: String }` | `()` | Permanently marks the currently connected peer as trusted |
| `sync_rdp_web_state` | `{ state: Value }` | `()` | Synchronizes UI connection state with the background Rust daemon |

---

### D. Screen Positioning & Input Control

| Command | Parameters | Return Type | Description |
|---------|------------|-------------|-------------|
| `rdp_update_screen_position` | `{ dir: RdpDir }` | `()` | Updates active screen-crossing boundary edge (`"Left"`, `"Right"`, `"Top"`, `"Bottom"`, `"Off"`) |
| `rdp_release_mouse` | `{}` | `()` | Releases mouse control back to local operating system |

---

### E. Cross-Screen History & Device Trust Settings

| Command | Parameters | Return Type | Description |
|---------|------------|-------------|-------------|
| `get_rdp_connection_history` | None | `Vec<ConnectionHistoryRecord>` | Loads previous connection history from `rdp_history.json` |
| `record_rdp_connection_history` | `{ device: ConnectionHistoryDevice }` | `()` | Appends or updates a peer in the connection history store |
| `clear_rdp_connection_history` | None | `()` | Clears all connection history records |
| `remove_rdp_trusted_device` | `{ deviceId: String }` | `()` | Revokes trust credential and removes device from autoconnect whitelist |
| `read_rdp_automatic_connection_device` | None | `Option<String>` | Reads the designated automatic connection device ID |
| `write_rdp_automatic_connection_device` | `{ deviceId: String }` | `()` | Sets the designated automatic connection device ID |
| `set_rdp_disconnect_shortcut` | `{ key: String }` | `()` | Sets the emergency disconnect hotkey combination (default: `Ctrl+Alt+D`) |
| `set_system_lock_shortcut` | `{ key: String }` | `()` | Sets the lock workstation hotkey combination |

---

### F. File Transfer Engine

| Command | Parameters | Return Type | Description |
|---------|------------|-------------|-------------|
| `switch_file_transfer` | `{ v: bool }` | `()` | Enables or disables file transfer functionality over cross-screen link |
| `select_transfer_files` | `{ maxFiles: usize }` | `SelectedTransferFilesResult` | Opens OS file dialog to pick up to 10 files for transmission |
| `send_file_by_path` | `{ id: String, path: String, name: String, historyRecordId: String }` | `()` | Enqueues and initiates streaming of a file to the connected peer |
| `cancel_file_transfer` | `{ id: String }` | `()` | Cancels an ongoing inbound or outbound file transfer |
| `transfer_file_exists` | `{ path: String }` | `bool` | Checks if a file path exists on disk prior to retransmitting |
| `open_rdp_file_folder` | None | `()` | Opens the received files folder in Explorer / Finder / file manager |
| `set_rdp_file_folder` | `{ path: String }` | `()` | Configures a custom destination directory for received files |
| `reveal_file_transfer_file` | `{ path: String }` | `()` | Highlights a specific transferred file in the OS file manager |
| `get_file_transfer_history` | `{ deviceId: String }` | `Vec<FileTransferHistoryRecord>` | Retrieves past file transfer records for a specific peer |
| `record_file_transfer_history` | `{ record: FileTransferHistoryRecord }` | `()` | Persists a transfer attempt and metrics to `file_transfer_history.json` |
| `remove_file_transfer_history_record` | `{ deviceId: String, recordId: String }` | `()` | Deletes a single transfer history entry |
| `clear_file_transfer_history` | `{ deviceId: String }` | `()` | Clears all file transfer logs for the specified peer |

---

### G. System Permissions & OS Settings

| Command | Parameters | Return Type | Description |
|---------|------------|-------------|-------------|
| `check_accessibility_settings` | None | `bool` | Checks whether macOS Accessibility / TCC permissions are granted |
| `open_accessibility_settings` | None | `()` | Opens macOS System Preferences to Privacy & Security $\to$ Accessibility |
| `check_input_permission` | None | `bool` | Verifies Input Monitoring permission status |
| `open_input_permission` | None | `()` | Opens macOS System Preferences to Input Monitoring |
| `open_mouse_setting` | None | `()` | Opens OS mouse settings panel |

---

### H. Language & Localization Cache

| Command | Parameters | Return Type | Description |
|---------|------------|-------------|-------------|
| `read_language_preference` | None | `String` | Reads active language code (`"en-US"`, `"zh-CN"`, `"ja-JP"`, `"ko-KR"`) |
| `write_language_preference` | `{ languageCode: String }` | `()` | Persists preferred language code to `settings.json` |
| `read_language_cache` | `{ languageCode: String }` | `Option<Value>` | Loads cached translations from `i18n_cache.json` |
| `write_language_cache` | `{ languageCode: String, translations: Value }` | `()` | Writes updated language strings to `i18n_cache.json` |

---

### I. AI Key & Diagnostics

| Command | Parameters | Return Type | Description |
|---------|------------|-------------|-------------|
| `clear_app_log` | None | `()` | Truncates and clears `log.json` / debug trace files |
