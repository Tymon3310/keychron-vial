# Secure Input Service (`secure_input_service`)

The Secure Input Service is a dedicated privileged helper process packaged alongside
Keychron Assistant. It solves one of the most difficult challenges in cross-platform
virtual KVM software: **injecting synthetic mouse and keyboard input into privileged
operating system contexts** (such as Windows Lock Screens, User Account Control (UAC)
prompts, and the Windows Logon desktop).

---

## 1. The Secure Desktop Problem

On Windows, operating system security enforces **User Interface Privilege Isolation (UIPI)**
and desktop isolation:

1. **Standard User Desktop (`WinSta0\Default`):** Normal user applications run here.
2. **Secure Desktop (`WinSta0\Winlogon`):** Used exclusively by the OS for logon,
   lock screens, credential prompts, and UAC elevation dialogs.

When a Windows computer is locked or displays a UAC elevation prompt, the OS switches
the active display surface to the Secure Desktop. Standard user applications (including
Keychron Assistant's main GUI process running under the user's account) are strictly
prohibited from sending synthetic input messages (`SendInput`, `PostMessage`) across
this boundary.

Without a privileged service, moving the cross-screen mouse cursor onto a locked PC
would leave the cursor completely frozen and unable to type passwords or click "Yes" on UAC.

---

## 2. Architecture & Service Details

### Windows Implementation (`secure_input_service.exe`)

```
┌────────────────────────────────────────────────────────┐
│  Keychron Assistant Main Process (keychron-assist.exe) │
│  - Runs as Standard User in Session 1/2                │
│  - Connects to Named Pipe                              │
└──────────────────────────┬─────────────────────────────┘
                           │
                           │ Win32 Named Pipe:
                           │ \\.\pipe\KeychronAssistSecureInput
                           ▼
┌────────────────────────────────────────────────────────┐
│  Secure Input Service (secure_input_service.exe)       │
│  - Windows Service: KeychronAssistSecureInput          │
│  - Account: NT AUTHORITY\SYSTEM (Session 0)            │
│  - Log: C:\ProgramData\KeychronAssistSecureInput\      │
│         secure_input.log                               │
│                                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │ SecureDesktopGuard                               │  │
│  │ 1. OpenWindowStationW("WinSta0")                 │  │
│  │ 2. SetProcessWindowStation()                     │  │
│  │ 3. OpenInputDesktop() / OpenDesktopW("Winlogon") │  │
│  │ 4. SetThreadDesktop()                            │  │
│  │ 5. SendInput() -> Injected into Lock/UAC Screen  │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
```

### Windows Service Specifications

- **Service Name:** `KeychronAssistSecureInput`
- **Display Name:** `Keychron Assist Secure Input`
- **Startup Type:** Automatic
- **Account:** `NT AUTHORITY\SYSTEM`
- **Log Path:** `C:\ProgramData\KeychronAssistSecureInput\secure_input.log`
- **Command-Line Helper Mode:** Can be invoked directly as:
  ```cmd
  secure_input_service.exe --input-helper
  ```

---

## 3. Named Pipe IPC Protocol

The main application and the service communicate over a local named pipe:

- **Pipe Name:** `\\.\pipe\KeychronAssistSecureInput`
- **Framing:** Serialized JSON messages matching `enum SecureInputMessage`

### Message Schema (`SecureInputMessage`)

```rust
pub enum SecureInputMessage {
    Key {
        vk: u32,       // Virtual Keycode (VK_...)
        action: u32,   // 1 = KeyDown, 0 = KeyUp
    },
    MouseMove {
        dx: i32,       // Relative horizontal pixel displacement
        dy: i32,       // Relative vertical pixel displacement
    },
    MouseClick {
        button: u32,   // 0 = Left, 1 = Right, 2 = Middle, 3 = X1, 4 = X2
        action: u32,   // 1 = ButtonDown, 0 = ButtonUp
    },
    MouseScroll {
        horizontal: i32,
        vertical: i32,
    },
}
```

#### JSON Wire Payloads

**Key Event:**
```json
{
  "Key": {
    "vk": 65,
    "action": 1
  }
}
```

**Mouse Motion:**
```json
{
  "MouseMove": {
    "dx": 14,
    "dy": -8
  }
}
```

**Mouse Button:**
```json
{
  "MouseClick": {
    "button": 0,
    "action": 1
  }
}
```

---

## 4. Desktop Switching Mechanics (`SecureDesktopGuard`)

To successfully deliver `SendInput` calls into the secure desktop, `secure_input_service`
implements an RAII guard pattern (`SecureDesktopGuard`) in `src/bin/secure_input_service.rs`:

```c
// Decompiled logical sequence of SecureDesktopGuard:

1. HWINSTA hOldWinsta = GetProcessWindowStation();
2. HDESK hOldDesktop = GetThreadDesktop(GetCurrentThreadId());

3. HWINSTA hWinSta0 = OpenWindowStationW(L"WinSta0", FALSE, MAXIMUM_ALLOWED);
4. SetProcessWindowStation(hWinSta0);

5. // Attempt to attach to current active input desktop (UAC or Default)
   HDESK hInputDesktop = OpenInputDesktop(0, FALSE, MAXIMUM_ALLOWED);
   if (!hInputDesktop) {
       // Fall back to explicit Winlogon desktop (Logon / Lock screen)
       hInputDesktop = OpenDesktopW(L"Winlogon", 0, FALSE, MAXIMUM_ALLOWED);
   }

6. SetThreadDesktop(hInputDesktop);

7. // Inject synthetic events directly into the active desktop surface
   SendInput(nInputs, pInputs, sizeof(INPUT));

8. // Restore previous thread desktop and window station
   SetThreadDesktop(hOldDesktop);
   SetProcessWindowStation(hOldWinsta);
   CloseDesktop(hInputDesktop);
   CloseWindowStation(hWinSta0);
```

If the main Assistant process cannot reach the service named pipe, it logs a warning
and falls back to standard user-space injection:
`连接 secure_input_service 命名管道失败（锁屏期间键鼠将无法注入）`

---

## 5. macOS Implementation

On macOS, `Contents/MacOS/secure_input_service` serves a similar role for
privilege elevation:

- **IPC Transport:** Unix Domain Socket (using `std::os::unix::net::SocketAddr`).
- **Permissions:** Leverages macOS Accessibility and Input Monitoring permissions:
  - `AXIsProcessTrusted()`
  - `AXIsProcessTrustedWithOptions({ kAXTrustedCheckOptionPrompt: true })`
- **Injection Backend:** Injects synthetic events using Apple CoreGraphics:
  - `CGEventCreateMouseEvent()`
  - `CGEventCreateKeyboardEvent()`
  - `CGEventCreateScrollWheelEvent2()`
  - `CGEventPost(kCGHIDEventTap, event)`
