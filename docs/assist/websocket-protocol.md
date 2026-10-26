# WebSocket Companion Protocol (`ws://127.0.0.1:40003`)

The Keychron Assistant WebSocket server acts as the local bridge between
browser-based client applications (specifically the Keychron Launcher web app
at `launcher.keychron.com`) and native operating system features such as
application launching, executable enumeration, file browsing, and hardware
quick-start keybinding persistence.

---

## 1. Transport & Connection Parameters

- **Transport:** WebSocket over TCP (RFC 6455)
- **Bind Address:** `127.0.0.1:40003` (the Launcher connects to `ws://localhost:40003`)
- **Framing:** Text-based JSON frames encoded as UTF-8
- **Heartbeat / Ping:** Supported at WebSocket frame level

When the Keychron Launcher web application opens, it checks whether Assistant is
running by connecting to `ws://localhost:40003`. If the connection succeeds, the
Launcher displays a green "Assistant Connected" status badge; if it fails, it
presents a download prompt linking to the platform installer
(`https://launcher.keychron.com/static/assist/KeychronAssist-<version>.zip` or `.dmg`).

---

## 2. Wire Envelope Format

All messages transmitted over the WebSocket connection (in both directions)
follow a standard two-tier envelope structure:

```json
{
  "evt_type": "<EventType>",
  "evt_data": {
    "cmd": "<CommandName>",
    "data": "<Payload>"
  }
}
```

### Top-Level Event Types (`evt_type`)

| `evt_type` | Description |
|------------|-------------|
| `Common` | System metadata and version negotiation |
| `QuickStart` | Application listing, native file dialogs, and keybinding management |
| `SysCmd` | Direct system action execution (application/URL spawning, OS settings) |
| `AiKey` | AI prompt template synchronization and quick trigger dispatch |
| `Mos` | Mouse smooth scrolling and acceleration configuration |
| `CrossScreen` | Virtual KVM and peer-to-peer screen crossing events |

### Payload Encoding

The web Launcher `JSON.stringify`s almost every `data` payload, so objects,
numbers, and ids arrive as JSON-encoded strings (e.g.
`"{\"vp_id\":13364,\"key\":\"0x7f00\"}"`, `"13364"`, `""`). Empty string means
"no data". The backend decodes one JSON layer before dispatch; already-
structured values (AIKey/Mos clients send raw objects) pass through
unchanged. `AddApp`/`RemoveApp` are defined in the Launcher bundle but never
called, so the backend leaves them unhandled rather than inventing semantics.

### Keybinding Responses

`AddKeyBind`/`UpdateKeyBind`/`RemoveKeyBind` reply with native `ResultCode`
strings (`"Success"`, `"KeyDuplicate"`, `"CommandFailed"`), matching the
Launcher's explicit `"Success"!==result` / `"KeyDuplicate"===result` checks.
Re-adding an identical binding is idempotent (`"Success"`); re-adding the
same key with a different bind reports `"KeyDuplicate"`. AIKey `Set`/`Put`/
`Remove`/`Clear` reply with booleans, matching that client's `!1===t` checks.

---

## 3. Result Codes & Errors

Commands responding with status or error states use the native `ResultCode` enum:

```rust
pub enum ResultCode {
    Success,
    PathNotExists,
    CommandFailed,
    OpenFailed,
    AppNotFound,
    AppExist,
    KeyDuplicate,
    NotExist,
}
```

When serialized in response messages, these correspond to standard strings:
`"Success"`, `"PathNotExists"`, `"CommandFailed"`, `"OpenFailed"`,
`"AppNotFound"`, `"AppExist"`, `"KeyDuplicate"`, `"NotExist"`.

---

## 4. Common Commands (`evt_type: "Common"`)

### `GetVersion`

Queries the running Assistant version.

- **Request:**
  ```json
  {
    "evt_type": "Common",
    "evt_data": {
      "cmd": "GetVersion",
      "data": null
    }
  }
  ```
- **Response:**
  ```json
  {
    "evt_type": "Common",
    "evt_data": {
      "cmd": "GetVersion",
      "data": {
        "version": "<app version>"
      }
    }
  }
  ```

### `OpenApp`

Requests Assistant to show its native desktop window or activate a specific page.

- **Request:**
  ```json
  {
    "evt_type": "Common",
    "evt_data": {
      "cmd": "OpenApp",
      "data": "show-window"
    }
  }
  ```

---

## 5. System Commands (`evt_type: "SysCmd"`)

### `SpawnAppByPath`

Directly launches an application by its filesystem path or opens a URL in the default browser.

- **Request:**
  ```json
  {
    "evt_type": "SysCmd",
    "evt_data": {
      "cmd": "SpawnAppByPath",
      "data": "https://launcher.keychron.com"
    }
  }
  ```
- **Execution Logic:**
  - If `data` begins with `http://` or `https://`:
    - Windows: `ShellExecuteW(..., "open", url, ...)`
    - macOS: `open <url>`
    - Linux: `xdg-open <url>`
  - If `data` is a filesystem path:
    - Windows: `CreateProcessW` / `Command::new(path)`
    - macOS: `open -a <path>`
    - Linux: spawns executable or `.desktop` exec entry
- **Response:**
  ```json
  {
    "evt_type": "SysCmd",
    "evt_data": {
      "cmd": "SpawnAppByPath",
      "data": true
    }
  }
  ```

### `Keyboard` / `MouseSetting`

Opens the operating system's native keyboard or mouse configuration panel:
- On Windows: Launches `main.cpl` (Mouse Properties) or Windows Settings (`ms-settings:mousetouchpad`).
- On macOS: Opens System Settings at Mouse/Keyboard pane.

---

## 6. Quick-Start Commands (`evt_type: "QuickStart"`)

The Quick-Start subsystem allows users to bind physical keys to launch apps,
open URLs, or trigger AI workflows.

### Command List (`QuickStartWsCmd`)

| Command | Direction | Purpose |
|---------|-----------|---------|
| `Get` | Launcher $\to$ Assist | Trigger scanning and streaming of installed applications |
| `FetchStart` | Assist $\to$ Launcher | Signals start of app stream with total count |
| `Fetch` | Assist $\to$ Launcher | Streams an individual discovered application record |
| `FetchEnd` | Assist $\to$ Launcher | Signals end of application stream |
| `AddApp` | Launcher $\to$ Assist | Manually add a custom executable to the quick-start app list |
| `RemoveApp` | Launcher $\to$ Assist | Remove an app from the quick-start app list |
| `SelectAppByNative` | Launcher $\to$ Assist | Open OS file picker dialog to select an `.exe` / `.app` |
| `AddKeyBind` | Launcher $\to$ Assist | Create a new key-to-action binding for a device VPID |
| `UpdateKeyBind` | Launcher $\to$ Assist | Update an existing keybinding |
| `RemoveKeyBind` | Launcher $\to$ Assist | Delete a keybinding by slot key |
| `ClearKeyBind` | Launcher $\to$ Assist | Clear all keybindings for a given VPID or globally |
| `GetKeyBinds` | Launcher $\to$ Assist | Retrieve all configured bindings for a device VPID |
| `SetReceiverVpidMap` | Launcher $\to$ Assist | Map a 2.4 GHz receiver VPID to peripheral VPID |
| `ClearReceiverVpidMap` | Launcher $\to$ Assist | Remove a receiver-to-device VPID association |
| `InstalledAppUpdate` | Both | Event notification when installed software cache updates |

---

### Application Scanning Stream (`Get`)

Because scanning the filesystem / registry for installed applications can take
several seconds, Assistant streams results asynchronously using a chunked protocol:

```
Launcher                  Assistant
   │                          │
   │  QuickStart: Get         │
   ├─────────────────────────►│  (Scans Registry / Applications)
   │                          │
   │  QuickStart: FetchStart  │  count = 142
   │◄─────────────────────────┤
   │                          │
   │  QuickStart: Fetch       │  App #1 (Chrome)
   │◄─────────────────────────┤
   │  QuickStart: Fetch       │  App #2 (VS Code)
   │◄─────────────────────────┤
   │  ... (142 messages)      │
   │                          │
   │  QuickStart: FetchEnd    │  count = 142
   │◄─────────────────────────┤
```

#### Application Record (`struct Installed`)

```json
{
  "name": "Visual Studio Code",
  "path": "C:\\Users\\User\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe",
  "icon": "data:image/png;base64,iVBORw0KGgo...",
  "icon_path": "C:\\Users\\User\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe"
}
```

- **Windows Scanner:** Inspects `HKLM\Software\Microsoft\Windows\CurrentVersion\Uninstall`, `HKCU\Software\Microsoft\Windows\CurrentVersion\Uninstall`, Start Menu shortcuts (`%APPDATA%\Microsoft\Windows\Start Menu\Programs`), and extract icons via `windows-icons-0.3.0`.
- **macOS Scanner:** Scans `/Applications`, `/System/Applications`, and `~/Applications` for `.app` bundles, extracting icons from `Contents/Resources/*.icns`.
- **Linux Scanner (Prototype):** Scans `/usr/share/applications`, `/usr/local/share/applications`, and `~/.local/share/applications` for `.desktop` files.

---

### Native File Picker (`SelectAppByNative`)

When the user clicks "Browse Executable" in the Launcher UI:

1. **Launcher sends:**
   ```json
   {
     "evt_type": "QuickStart",
     "evt_data": {
       "cmd": "SelectAppByNative",
       "data": null
     }
   }
   ```
2. Assistant opens a native blocking file dialog (`rfd::FileDialogBuilder` or `zenity --file-selection` on Linux) filtered to executable files (`*.exe` on Windows, `*.app` on macOS).
3. **Assist responds with `AddApp` payload:**
   ```json
   {
     "evt_type": "QuickStart",
     "evt_data": {
       "cmd": "AddApp",
       "data": [
         {
           "name": "obs64",
           "path": "C:\\Program Files\\obs-studio\\bin\\64bit\\obs64.exe",
           "icon": "",
           "icon_path": "C:\\Program Files\\obs-studio\\bin\\64bit\\obs64.exe"
         }
       ]
     }
   }
   ```

---

### Keybinding Management (`AddKeyBind`, `GetKeyBinds`, etc.)

Keybindings map a physical key (or programmable macro button) on a specific
Keychron device (identified by Vendor & Product ID) to an executable path, URL,
or AI template.

#### Binding Descriptor (`struct AssistKeyBind`)

```json
{
  "vp_id": "13364:4100",
  "key": "0x7f00",
  "bind": {
    "type": "App",
    "opt_data": {
      "name": "Google Chrome",
      "path": "C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe",
      "icon": "",
      "icon_path": ""
    }
  }
}
```

#### Binding Types (`AssistOptType`)

- `"App"`: Launch application at `opt_data.path`.
- `"Url"`: Open web URL in default system browser (`opt_data.path`).
- `"Dir"`: Open directory folder in Explorer / Finder.
- `"Aikey"`: Trigger AI prompt template workflow through Launcher.

#### Retrieving Configured Keybinds (`GetKeyBinds`)

- **Request:**
  ```json
  {
    "evt_type": "QuickStart",
    "evt_data": {
      "cmd": "GetKeyBinds",
      "data": "13364:4100"
    }
  }
  ```
- **Response:**
  ```json
  {
    "evt_type": "QuickStart",
    "evt_data": {
      "cmd": "GetKeyBinds",
      "data": [
        {
          "key": "0x7f00",
          "bind": {
            "type": "App",
            "opt_data": {
              "name": "Google Chrome",
              "path": "C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe"
            }
          }
        },
        {
          "key": "0x7f01",
          "bind": {
            "type": "Url",
            "opt_data": {
              "name": "GitHub",
              "path": "https://github.com"
            }
          }
        }
      ]
    }
  }
  ```

  > **Open shape question (do not guess):** two Launcher call sites disagree
  > about this response. `syncQuickStartToReceiver` reads each item as a
  > single-key map to an array (`hn[$t]?.[0]`), while the AI `getQuickStartBindings`
  > tool reads flat `Xe.key` / `Xe.opt_type` / `Xe.opt_data` fields. Both cannot
  > be right, so at least one call site is stale. The backend keeps
  > `[{key, bind}]` because the native binary's `AssistKeyBind` has exactly 2
  > fields and it round-trips the `AddKeyBind` request shape. Revisit only with
  > native wire evidence.

---

## 7. Quick-Start Keycode Mapping (`0x7f00` Table)

To make quick-start keys work from firmware, Keychron firmware translates
slot keys into standard USB HID Consumer Control Programmable Buttons (`PB_1`..`PB_52`).

Keychron Assistant contains an embedded lookup table mapping internal
virtual slot codes `0x7f00`--`0x7f33` to USB Consumer Usages `0x01cc`--`0x01ff`:

| Virtual Slot Code | Hex Usage | Consumer Control Usage Name | Linux Kernel Key |
|-------------------|-----------|-----------------------------|------------------|
| `0x7f00` | `0x01cc` | `AL Programmable Button 1` | `KEY_MACRO1` (656) / `PB_1` |
| `0x7f01` | `0x01cd` | `AL Programmable Button 2` | `KEY_MACRO2` (657) / `PB_2` |
| `0x7f02` | `0x01ce` | `AL Programmable Button 3` | `KEY_MACRO3` (658) / `PB_3` |
| `0x7f03` | `0x01cf` | `AL Programmable Button 4` | `KEY_MACRO4` (659) / `PB_4` |
| `0x7f04` | `0x01d0` | `AL Programmable Button 5` | `KEY_MACRO5` (660) / `PB_5` |
| `0x7f05` | `0x01d1` | `AL Programmable Button 6` | `KEY_MACRO6` (661) / `PB_6` |
| `0x7f06` | `0x01d2` | `AL Programmable Button 7` | `KEY_MACRO7` (662) / `PB_7` |
| `0x7f07` | `0x01d3` | `AL Programmable Button 8` | `KEY_MACRO8` (663) / `PB_8` |
| `...` | `...` | `...` | `...` |
| `0x7f18` | `0x01e4` | `AL Programmable Button 25` | `KEY_MACRO25` (680) / `PB_25` |
| `...` | `...` | `...` | `...` |
| `0x7f33` | `0x01ff` | `AL Programmable Button 52` | `KEY_MACRO52` / `PB_52` |

### Keybind Execution Flow

1. Keychron Launcher assigns a button to Quick-Start slot `0x7f00`.
2. Launcher writes Consumer Usage `0x01cc` into the keyboard's VIA dynamic keymap table.
3. Launcher informs Assistant via WebSocket: `AddKeyBind` with slot `"0x7f00"` and target app.
4. When the physical key is pressed, firmware transmits a HID Consumer report containing `0x01cc`.
5. Assistant's background device listener (`device_listener`) intercepts the raw report, maps `0x01cc` back to `0x7f00`, looks up the bound action in `assist.json`, and spawns the target program.

---

## 8. Receiver VPID Tunneling (`SetReceiverVpidMap`)

When a Keychron keyboard or mouse connects wirelessly via a 2.4 GHz USB receiver,
the USB descriptor reports the **Receiver's** Vendor & Product ID (e.g., `0x3434:0x4200`),
not the physical peripheral's ID (e.g., `0x3434:0x4100`).

To ensure that keybindings configured for the keyboard are correctly applied when
receiving keystrokes through the dongle, Keychron Launcher pushes a VPID mapping table
upon connection:

```json
{
  "evt_type": "QuickStart",
  "evt_data": {
    "cmd": "SetReceiverVpidMap",
    "data": {
      "receiver_vpid": "13364:4200",
      "device_vpid": "13364:4100",
      "device_type": 1
    }
  }
}
```

- `device_type`: `1` for Keyboard, `2` for Mouse.
- When `device_listener` detects incoming reports from `receiver_vpid`, it substitutes `device_vpid` when indexing `keybinds[vpid]`.
- On page unload / disconnect, Launcher sends `ClearReceiverVpidMap`.
