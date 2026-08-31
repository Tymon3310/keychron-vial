# Wireless DFU (0xAA) & Factory Test (0xAB)

These two command groups handle wireless module firmware updates and factory
testing. They are less commonly used by end-user-facing GUI code.

> **See also:** The [Launcher DFU Protocols](../launcher/dfu-protocols.md)
> document details all 5 firmware update paths reverse-engineered from the
> Keychron Launcher, including the Bluetooth DFU packet format, Nordic nRF
> DFU over SLIP/HCI, STM32 WebDFU, and more.

## 0xAA -- Wireless DFU

The Wireless DFU command (`KC_WIRELESS_DFU = 0xAA`) is used for over-the-air
firmware updates of the Bluetooth/2.4 GHz wireless module (lkbt51 or ckbt51).

### Notes

- The wireless module is a separate SoC that communicates with the main MCU
  over UART.
- DFU for the wireless module is handled by the module's own bootloader,
  with the main MCU acting as a passthrough for HID data.
- The GUI-side implementation for wireless DFU is separate from the STM32
  DFU flasher (which uses `dfu-util` for the main MCU).
- Detailed sub-command documentation is firmware-internal; the GUI does not
  currently expose wireless DFU through the standard editor interface.

---

## 0xAB -- Factory Test

The Factory Test command (`KC_FACTORY_TEST = 0xAB`, Launcher name
`KC_FACTORY`) enables a special test mode used during keyboard
manufacturing.

### Packet Framing (verified in `factory_test.c`)

Both directions use the full 32-byte raw-HID report with a 16-bit additive
checksum in the last two bytes:

```
data[0]     = 0xAB
data[1]     = sub-command (FACTORY_TEST_CMD_*)
data[2..28] = payload
data[29]    = unused (excluded from checksum)
data[30]    = checksum low byte   } checksum = sum(data[1..28]) & 0xFFFF
data[31]    = checksum high byte  }
```

Requests failing the checksum are silently dropped. Sub-commands ≤
`FACTORY_TEST_CMD_RADIO_CARRIER` (7) are accepted only over wired USB.

### Firmware Handler

The factory test dispatcher is in `keyboards/keychron/common/factory_test.c`.
It handles:

- Key matrix scanning verification
- LED testing (individual and full-board)
- Wireless module connectivity checks (`INT_PIN` test: LKBT51/CKBT51)
- DIP switch verification (`GET_TRANSPORT`)
- ADC calibration for analog keyboards (`CHARGING_ADC`)

### Notes

- Factory test mode is typically only accessible via a specific key combination
  or firmware flag.
- Commands in this group are not intended for end-user use and may cause
  unexpected behavior if sent to a production keyboard outside of factory
  test mode.
- The official Keychron Launcher actively uses `0xAB`: it names the command
  `KC_FACTORY`, defines the same `FACTORY_TEST_CMD_*` enum (values 1--9), and
  uses two sub-commands:
  - **`GET_TRANSPORT` (5)** to detect the DIP/mode-switch position.
    Firmware replies `[0xAB, 5, transport, usb_power_sense, ... cksum]`,
    where transport = `get_transport()`. The Launcher reads byte `[2]`:
    **1=USB, 2=BT, 4=G (2.4 GHz)**. Its request only sets `data[30]=5`
    with `data[31]=0`, which happens to satisfy the checksum for an
    otherwise-empty payload (sum = 5).
  - **`GET_DEVICE_ID` (9)**: firmware replies
    `[0xAB, 9, 12, <12-byte UUID from UID_BASE>, ... cksum]`; the Launcher
    concatenates bytes `[3..13]` (11 of the 12 UUID bytes) as decimal
    digits into a chip-ID string used e.g. for per-device profile storage.

### Sub-Command Reference

| Value | Name | Handled by firmware? |
|-------|------|----------------------|
| `0x01` | `BACKLIGHT` | yes (test backlight mode) |
| `0x02` | `OS_SWITCH` | yes (report OS switch state) |
| `0x03` | `JUMP_TO_BL` | stubbed out (commented) |
| `0x04` | `INT_PIN` | yes (wireless builds) |
| `0x05` | `GET_TRANSPORT` | yes (used by Launcher) |
| `0x06` | `CHARGING_ADC` | yes (`BATTERY_CHARGE_DONE_DETECT_ADC`) |
| `0x07` | `RADIO_CARRIER` | yes (wireless builds) |
| `0x08` | `GET_BUILD_TIME` | yes (returns QMK_BUILDDATE) |
| `0x09` | `GET_DEVICE_ID` | yes (12-byte MCU UUID; used by Launcher) |

See also [launcher/command-map.md](../launcher/command-map.md).

---

## STM32 DFU Flashing (GUI-side)

While not part of the HID protocol, the GUI includes an STM32 DFU flasher
(`editor/dfu_flasher.py`) for keyboards with STM32 MCUs. This is triggered
when:

1. The `MISC_DFU_INFO` flag is set in the misc feature mask, **or**
2. The MCU info string (from `DFU_INFO_GET`) contains "STM32"

The flasher uses `dfu-util` externally and operates outside the HID channel
(the keyboard must be rebooted into DFU bootloader mode first).
