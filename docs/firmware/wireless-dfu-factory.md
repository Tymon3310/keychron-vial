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

The Factory Test command (`KC_FACTORY_TEST = 0xAB`) enables a special test
mode used during keyboard manufacturing.

### Firmware Handler

The factory test dispatcher is in `keyboards/keychron/common/factory_test.c`.
It handles:

- Key matrix scanning verification
- LED testing (individual and full-board)
- Wireless module connectivity checks
- DIP switch verification
- ADC calibration for analog keyboards

### Notes

- Factory test mode is typically only accessible via a specific key combination
  or firmware flag.
- The GUI does not expose factory test commands in the user interface.
- Commands in this group are not intended for end-user use and may cause
  unexpected behavior if sent to a production keyboard outside of factory
  test mode.

---

## STM32 DFU Flashing (GUI-side)

While not part of the HID protocol, the GUI includes an STM32 DFU flasher
(`editor/dfu_flasher.py`) for keyboards with STM32 MCUs. This is triggered
when:

1. The `MISC_DFU_INFO` flag is set in the misc feature mask, **or**
2. The MCU info string (from `DFU_INFO_GET`) contains "STM32"

The flasher uses `dfu-util` externally and operates outside the HID channel
(the keyboard must be rebooted into DFU bootloader mode first).
