# ZMK & Zephyr RTOS Device Support in Keychron Launcher

This document describes how the Keychron Launcher web application handles
**ZMK-based** (Zephyr RTOS) keyboards and how its WebHID communication model
differs from traditional QMK-based models.

> **Source:** Reverse-engineered from the Keychron Launcher JavaScript bundle
> (snapshot in `/src/keychron-launcher`).

---

## Overview

While most Keychron custom keyboards (Q / V / K Pro / K Max series) run forks of
**QMK**, Keychron employs **ZMK firmware** (built atop Zephyr RTOS on Nordic
nRF52-series wireless MCUs) for specific product categories:

- **Keychron Ultra Series** (e.g. `Q Ultra`, `V Ultra`): Next-generation
  wireless custom mechanical keyboards featuring 8K wireless polling rates and
  extended battery life.
- **Keychron B Pro Series** (e.g. `B1 Pro`, `B6 Pro`): Ultra-slim / low-profile
  wireless keyboards.

The Keychron Launcher unifies both QMK and ZMK architectures under a single
WebHID interface by detecting an **Instruction Set** ID and switching packet
framing accordingly.

---

## Instruction Set & Protocol Detection

When the Launcher connects to any keyboard, it queries the protocol version:

```
Request : [0xA0] (KC_GET_PROTOCOL_VERSION)
Response: data[0] = 0xA0
          data[1] = Keychron protocol version (1, 2, 3, 4)
          data[3] = instruction_set identifier
```

### Instruction Set Mapping

| `instruction_set` (`data[3]`) | Firmware Architecture | Description / Target Lineup |
|-------------------------------|-----------------------|-----------------------------|
| `1` or `3`                    | **QMK**               | Standard QMK Keychron models |
| `2` or `4`                    | **ZMK (Zephyr RTOS)** | Ultra / B Pro wireless keyboards |

---

## Protocol Adaptations for ZMK

### 1. Feature Flag Offset (`0xA2` KC_GET_SUPPORT_FEATURE)

In protocol v2/v3 modes, the byte offset for reading feature bitmaps depends on
the `instruction_set`:

```javascript
// Launcher internal logic:
const instruction_set = this.keyboard.instruction_set;
const featureBytes = (instruction_set === 1 || instruction_set === 3)
    ? [data[1], data[2]]   // QMK layout
    : (instruction_set === 2 || instruction_set === 4)
    ? [data[2], data[3]]   // ZMK offset (shifted by 1 byte)
    : [0, 0];
```

### 2. Bootloader Jump (`0xA7` / `0x15`)

Traditional QMK boards jump to bootloader via VIA command `BOOTLOADER_JUMP`
(`0x0B`) or hardware physical reset switches. ZMK keyboards lack hardware DFU
switches and instead expose a software jump sub-command:

- **Sub-command:** `[0xA7, 0x15]` under `KC_MISC_CMD_GROUP`
- **Gating:** Enabled only when `instruction_set` is `2` or `4` and the
  `Bootloader_Jump` feature bit is set.
- **Action:** Instructs the Nordic MCU to reboot into the Nordic DFU / UF2
  bootloader.

### 3. Performance Mode (Radio Interval & Latency)

ZMK keyboards support dynamically toggling wireless radio performance modes
(optimizing between maximum battery life and lowest wireless latency):

- **Feature gating:** `Performance_Mode` is enabled when bit 17 of the V4 feature
  bitmap (or bit 0 of the secondary v2 word) is set, provided
  `instruction_set !== 2` (or via dedicated ZMK handlers).
- Manages Bluetooth connection interval and 2.4 GHz RF sleep delays directly
  through Zephyr RTOS power management subsystems.

### 4. Keymap & Dynamic Buffers

Because ZMK does not use QMK's `dynamic_keymap` EEPROM structures:
- Keycode indexes and layer mappings use ZMK-specific keycode lookup tables
  embedded in the Launcher bundle.
- Matrix coordinates map directly to ZMK behavioral bindings (e.g. `&kp`, `&mo`,
  `&tog`).


---

## Related Documents

- [Launcher Overview](overview.md) -- WebHID architecture and transport
- [Command Map Comparison](command-map.md) -- sub-command enums and V4 feature flags
- [Wireless Communication](wireless-communication.md) -- power modes and RF settings
- [Protocol Overview](../protocol-overview.md) -- master protocol reference
