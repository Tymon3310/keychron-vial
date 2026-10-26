# ZMK and Other Non-QMK Keyboards

Not every keyboard the Keychron Launcher configures runs Keychron's QMK. This
page covers the others: the **Ultra** series and **B Pro** series (Keychron's
ZMK fork on Zephyr), Telink-based boards, and the older firmware families the
Launcher still has code paths for.

None of these run Vial. They are documented here because they share the
Launcher protocol with the QMK boards.

> **Sources:**
> - Keychron Launcher bundle (`https://launcher.keychron.com`)
> - Keychron's ZMK fork, [`github.com/Keychron/zmk`](https://github.com/Keychron/zmk),
>   branches `rtl8762g` (Ultra series) and `keychron_bpro` (B Pro series)
> - Launcher backend: product records
>   (`https://launcher.keychron.com/vapi/v2/product/<vpId>`), keyboard
>   definitions (`https://launcher.keychron.com/static/device/<vpId>/json/v3.json`)
>   and the official firmware images they link to (`cdn.shopify.com`)
>
> The Launcher bundle itself never says "ZMK". Everything ZMK-specific below
> comes from Keychron's ZMK source and the official firmware images.

---

## Firmware Families at a Glance

| Family | Chip | Firmware | Definition `bootloader` | Firmware file | Update path in the Launcher |
|--------|------|----------|-------------------------|---------------|------------------------------|
| Q / V / K Pro / K Max / HE / 8K / … | STM32, WB32, … | Keychron QMK | (none) / `WB` / … | `.bin` | WebUSB DFU (see [dfu-protocols.md](dfu-protocols.md)) |
| **Ultra** (Q/V/K/J … Ultra 8K) | Realtek RTL8762G | Keychron ZMK (`rtl8762g` branch) | `KLD` | `*_ota.bin` (plain image) | `AA 55` DFU over HID usage page `0x8C`, or Realtek CFU for old patch versions |
| **B Pro** (B1 / B2 / B6 Pro) | Nordic nRF52840 | Keychron ZMK (`keychron_bpro` branch) | `PCA` | Nordic DFU `.zip` (`.bin` + `.dat` + `manifest.json`) | Nordic serial DFU over **Web Serial** |
| B2 Pro (one JIS variant) | Telink | Proprietary Telink firmware | `telink` | `.bin` (Telink `KNLT` image) | `AA 55` DFU over HID |
| Others | XJ, Nations N32, NXP LPC5516 | Proprietary | `XJ`, `XJ_NEW`, `Nations`, `NXP` | | Dedicated flashers (see [dfu-protocols.md](dfu-protocols.md)) |

The B Pro images contain Zephyr's Nordic BLE controller, and the Ultra images
contain Zephyr and ZMK event names (`zmk_keycode_state_changed`, ...), so the
"ZMK" label is confirmed from the shipped binaries, not just from the source.

---

## Protocol Detection: `KC_GET_PROTOCOL_VERSION` (`0xA0`)

```
Request : [0xA0]
Reply   : data[1] = Keychron protocol version
          data[3] = "instruction set" (stored as keyboard.instruction_set)
```

What each firmware answers:

| Firmware | `data[1]` | `data[3]` (instruction set) |
|----------|-----------|-----------------------------|
| Keychron QMK (`keyboards/keychron/common`) | `2` | `2` (`QMK_COMMAND_SET`) |
| Keychron ZMK, Ultra (`rtl8762g`) | `1` | `1` |
| Keychron ZMK, B Pro (`keychron_bpro`) | `1` | `0` (not set) |

Instruction sets `3` and `4`, and protocol versions above `3` (which switch
the Launcher to the wide "V4" feature bitmap, see
[command-map.md](command-map.md#v4-support-feature-bitmap)), are handled by
the Launcher but are not produced by any public Keychron firmware.

### Feature Bitmap (`0xA2`) for Protocol ≤ 3

The Launcher reads two feature bytes from a position that depends on the
instruction set:

| Instruction set | Bytes read |
|-----------------|------------|
| `1`, `3` | `data[1]`, `data[2]` |
| `2`, `4` | `data[2]`, `data[3]` |
| other (e.g. `0`) | none (all features off) |

This matches the firmware: Keychron QMK writes a `0` at `data[1]` and its
flags at `data[2..3]`; Keychron ZMK (Ultra) writes its flags at `data[1]`.

| Byte / bit | Launcher name | Keychron QMK / ZMK name |
|------------|---------------|-------------------------|
| 0 / 0 | `Get_Default_Layer` | `FEATURE_DEFAULT_LAYER` |
| 0 / 1 | `Support_Le` | `FEATURE_BLUETOOTH` |
| 0 / 2 | `Support_Rf` | `FEATURE_P24G` |
| 0 / 3 | `Support_Mock_Key` | `FEATURE_ANALOG_MATRIX` |
| 0 / 4 | `Upload_Default_Layer` | `FEATURE_STATE_NOTIFY` (QMK) / `FEATURE_INFO_CHAGNED_NOTIFY` (ZMK) |
| 0 / 5 | `Support_DEBOUNCE` | `FEATURE_DYNAMIC_DEBOUNCE` |
| 0 / 6 | `Support_Snap` | `FEATURE_SNAP_CLICK` |
| 0 / 7 | `Support_Keychron_Rgb` | `FEATURE_KEYCHRON_RGB` |
| 1 / 0 | `Performance_Mode` (ignored when instruction set is `2`) | `FEATURE_QUICK_START` (QMK) |
| 1 / 3 | `Bootloader_Jump` (only for instruction set `2` or `4`) | -- |

The instruction-set exceptions exist because QMK uses byte 1 bit 0 for Quick
Start, which the Launcher would otherwise read as "performance mode".

---

## Keychron ZMK's Launcher Module

Both ZMK branches add `app/src/launcher/`, which makes the keyboard look like
a VIA keyboard to the Launcher:

- **Raw HID:** usage page `0xFF60`, usage `0x61`, 32-byte reports, like QMK.
- **VIA commands `0x01`--`0x15`** with VIA protocol version `0x000C`, so the
  Launcher treats it as a "v3" device (`protocol >= 11`).
- **Dynamic keymap in RAM-backed "EEPROM":** a 2560-byte buffer holding
  layout options, keymaps, encoder maps and 16 macros.
- **QMK keycodes on the wire.** The firmware converts them to ZMK behaviours
  itself (`map_keycode.c`): `QK_MOMENTARY` → `momentary_layer`,
  `QK_LAYER_TAP` → `layer_tap`, `QK_MOD_TAP` → `mod_tap`,
  `QK_ONE_SHOT_MOD` → `sticky_key`, `QK_MACRO` → a macro behaviour, mouse
  keys → `mouse_key_press`, and so on. The Launcher needs no ZMK-specific
  keycode tables.
- **Custom keycodes** start at `QK_KB_0`: `BT_HST1..3`, `KC_BOOT`,
  `PAIR_24G`, macOS/Windows shortcuts (`UC_LOPT`, `UC_LCMD`, `UC_TASK_VIEW`,
  …), battery info, optional Mac/Win layer keys.
- **Lighting** uses the VIA lighting channels (backlight, RGB light, RGB
  matrix, LED matrix, audio) and, when enabled, Keychron RGB `0xA8`.
- **`0xA1`** returns an ASCII version string ending in `-ZK` plus the build
  date and time.
- **`0xAB`** factory test is implemented.

Not implemented: `0xA9` (Hall Effect), `0xAA` (wireless module DFU through the
keyboard), the `0xA7` sub-commands from `0x0F` up (DIP switch, NKRO, latch
keys, macro repeat, bootloader jump, pointing device).

### `0xA7` Sub-Commands in Keychron ZMK

Same numbering as Keychron QMK's `KC_MISC_CMD_GROUP`. Replies are
`[0xA7, sub, status, ...]` with status `0` = OK, `1` = fail.

| Sub | Name | Ultra | Reply / request |
|-----|------|-------|-----------------|
| `0x01` | protocol | yes | `data[3]` = misc version: `3` if the board has a separate 2.4 GHz polling rate, else `1` |
| `0x03`/`0x04` | keyboard language get/set | no (in the enum, not handled) | |
| `0x05` | debounce get | yes | `data[3] = 1` (type "TeLink", see below), `data[4]` press ms, `data[5]` release ms |
| `0x06` | debounce set | yes | `[0xA7, 6, press, release]`; press ≤ 50, release 1--50 |
| `0x07`--`0x0A` | Snap Click info/get/set/save | when enabled | same layout as QMK |
| `0x0B` | sleep timers get | yes | `data[3..4]` idle time, `data[5..6]` sleep time (LE16, seconds) |
| `0x0C` | sleep timers set | yes | idle ≥ 5, sleep ≥ 60, else fail |
| `0x0D` | polling rate get | yes | dual layout below |
| `0x0E` | polling rate set | yes | `[0xA7, 0x0E, usb_div, rf_div, x]`, each ≤ 6 |

Dual polling-rate reply (`0x0D`) on boards with a separate 2.4 GHz rate:

```
data[3..4]  USB max rate (LE16, e.g. 8000)   data[5] capability mask   data[6] USB divider   data[7] read-only (0)
data[8..9]  2.4G max rate (LE16)             data[10] capability mask  data[11] 2.4G divider data[12] read-only (0)
data[13..14] Bluetooth rate (125)            data[15..16] 0x40 0x40    data[17] read-only (1)
```

Rate = max rate >> divider. The keyboard also sends this report unprompted
when the rate changes on the device.

### Debounce Variants

The Launcher supports three debounce wire formats and probes for them:

| Variant | Detection | Set |
|---------|-----------|-----|
| **QMK** | `[0xA7, 5, 0xFF]` → `data[3] = 0`, `data[4]` sub-type, `data[5]` time | `[0xA7, 6, subType, ms]` |
| **"TeLink"** (Keychron ZMK and Telink firmware) | `[0xA7, 5, 0xFF]` → `data[3] = 1`, `data[4]` press, `data[5]` release | `[0xA7, 6, press, release]` |
| **Old** | `[0xB1]` → `data[1]` scan, `data[2]` press, `data[3]` release (USB only) | `[0xB0, scan, press, release]` |

---

## Firmware Updates

### Ultra (`KLD`)

- Ultra boards expose a second HID interface on usage page **`0x8C`** with
  report IDs **`0xB2`** (out) and **`0xB1`** (in), the same page Keychron
  receivers use.
- The firmware page sends the whole `*_ota.bin` through the `AA 55/56` DFU
  protocol described in
  [dfu-protocols.md](dfu-protocols.md#bluetooth-module-dfu) and
  [../firmware/kfw-format.md](../firmware/kfw-format.md): `0x60` model info,
  `0x61` DFU version, `0x62`/`0x63` start, `0x64` 16-byte chunks, `0x65` CRC,
  `0x66` image switch. For `KLD` boards the model-name check on the file is
  skipped.
- Keychron ZMK adds `0x6D` (chip UUID), `0x6E` (Realtek patch version and
  bank) and `0x6F` (build info) to that command set.
- If `0x6E` reports a Realtek patch version at or below a threshold built into
  the Launcher and the product record has a patch firmware, the page switches
  to **Realtek CFU** mode instead: the user picks HID device `0BDA:4762` and a
  CFU package (`offer` + payload files) is sent as described in
  [dfu-protocols.md](dfu-protocols.md).
- The `*_ota.bin` images are not encrypted (Zephyr/ZMK strings are readable).

### B Pro (`PCA`)

- Firmware is a legacy Nordic DFU package: `manifest.json` (`dfu_version`
  0.5, `device_type` 82), an init packet `.dat` and the application `.bin`.
- The Launcher asks for a serial port with `navigator.serial.requestPort()`
  and runs the Nordic SLIP/HCI DFU described in
  [dfu-protocols.md](dfu-protocols.md).

### Telink (`telink`)

- Uses the same firmware page component as `KLD` (the `AA 55` path, with the
  model-name check against the file).
- The image is a plain Telink binary with the `KNLT` magic at offset 8.

---

## UI Differences

- Keyboards whose definition has `bootloader: "KLD"` (`chipType`) hide the
  "default layer" marker in the layer selector and show a different layer
  indicator.
- In the keycode picker, `KLD` boards don't get the `DF` (default layer)
  keycode.
- Definitions for these boards usually only list `"DEBOUNCE"` in `feature`,
  so the keyboard class is the base one (not `HE` or `SCREEN`).

---

## Product IDs

All use VID `0x3434`. Taken from the `CONFIG_USB_DEVICE_PID` values in
Keychron's ZMK branches and checked against the Launcher backend (product
names are the backend's; many are still filed under a pre-production "DVT"
category there).

| Series | PIDs | `bootloader` |
|--------|------|--------------|
| B1 Pro | `0x0711`--`0x0713`, `0x071A`--`0x071C` | `PCA` |
| B2 Pro | `0x0721`--`0x0723`, `0x0726`, `0x072A`--`0x072D` | `PCA` (`0x0722`: `telink`) |
| B6 Pro | `0x0761`--`0x0763`, `0x076B`--`0x076D` | `PCA` |
| V Ultra | `0x0C00` (V0), `0x0C10`--`0x0C12` (V1), `0x0C20` (V2), `0x0C30`--`0x0C32` (V3), `0x0C50`--`0x0C52` (V5), `0x0C60`--`0x0C62` (V6), `0x0CA0` (V10) | `KLD` |
| Q Ultra | `0x1210`--`0x1212` (Q1), `0x1230`--`0x1232` (Q3), `0x1251`--`0x1252` (Q5), `0x1260`--`0x1262` (Q6), `0x12D0` (Q13) | `KLD` |
| K Ultra | `0x1610`--`0x1611` (K1), `0x1620`--`0x1622` (K2), `0x1630`--`0x1631` (K3), `0x1640`, `0x1642` (K4), `0x1650`--`0x1651` (K5), `0x1680`--`0x1682` (K8), `0x16A0` (K10) | `KLD` |
| J Ultra | `0x042B` (J9), `0x0447` (J2) | `KLD` |
| K SE2 (ZMK) | `0x1336` (K3 SE2), `0x1356` (K5 SE2) | none in the definition |
| Others on the `rtl8762g` branch | `0x0D20` (Z270 Ultra shield; the backend names this PID "K2X RGB"), `0x1321` (RS45), `0x1323` (QN104 and RS87) | -- |

`0x1321` and `0x1323` have no backend record. The branch also contains a
Realtek reference shield (`rtk_keyboard`, PID `0x4762`).

---

## Vial / VIA Compatibility

- These boards do not run Vial and can't be ported to `vial-qmk`.
- Keychron ZMK speaks the VIA protocol (`0x01`--`0x15`, protocol `0x000C`)
  with QMK keycodes, so a VIA-compatible tool with the right definition JSON
  can in principle edit the keymap. Vial-specific commands are not
  implemented.

## Related Documents

- [Launcher Overview](overview.md) -- WebHID architecture and transport
- [Command Map Comparison](command-map.md) -- sub-command enums and V4 feature flags
- [DFU Protocols](dfu-protocols.md) -- all firmware update paths
- [Wireless Module Firmware Files](../firmware/kfw-format.md) -- the `AA 55` DFU protocol
- [Protocol Overview](../protocol-overview.md) -- master protocol reference
