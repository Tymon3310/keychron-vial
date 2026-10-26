# Wireless Module & Receiver Firmware Files (`.kfw`, `.bin`)

Keychron ships firmware for the **Bluetooth/2.4 GHz module** inside its
keyboards and for **Keychron Link** receivers as `.kfw` files, flashed with
Keychron's Windows tool "Keychron Bt Firmware Upgrade". Newer receivers use
plain `.bin` images instead. This page describes both formats and the HID
protocol the updater uses.

> **Source:** Official downloads linked from Keychron's support pages:
> - [How to flash the firmware for the Keychron receiver](https://www.keychron.com/pages/how-to-flash-the-firmware-for-the-keychron-receiver)
>   (updater `.exe`, Keychron Link type C/D `.kfw`, Link-KM and Ultra-Link 8K `.bin`)
> - [V Max, Q Max, Q HE, K Max series Bluetooth firmware](https://www.keychron.com/pages/keychron-v-max-q-max-k-max-series-bluetooth-firmware)
>   (updater `.exe`, `spi_tmode` / `lkbt51` module `.kfw`)
> - [K Pro series Bluetooth firmware](https://www.keychron.com/pages/keychron-k-pro-series-bluetooth-firmware)
>   (older updater, `ckbt51` module `.kfw`)
>
> All files are hosted on `cdn.shopify.com/s/files/1/0059/0630/1017/files/`.
> Every `.kfw` file from these pages parses and passes all checksums as
> described below.

## Summary

- The `.kfw` container is a nibble-swapped tag/length/value header followed
  by the payload.
- The payload is **encrypted** with a 16-byte block cipher. Neither the
  updater nor the Launcher decrypts it: the encrypted bytes are sent as they
  are and the module's bootloader decrypts them. The key is on the module,
  not in the updater.
- So a Linux flasher for official `.kfw` files is possible (no key needed on
  the PC), but custom module firmware is not.

---

## 1. `.kfw` Container

Every byte of the file has its two nibbles swapped (`b = (b << 4 | b >> 4) & 0xFF`).
After swapping the header back:

| Offset | Content |
|--------|---------|
| 0--2 | magic `kfw` |
| 3 | format version (`0x01`) |
| 4... | header fields: tag (1 byte), length (1 byte), value |
| after tag 6 | payload, **not** nibble-swapped (header length is 43--45 bytes depending on string lengths) |

### Tags

| Tag | Meaning | Examples |
|-----|---------|----------|
| 1 | Model (ASCII) | `KCNTFDM3` (Link type C), `KCNRFDM3` (Link type D), `KCH_KB1` (LKBT51 / spi_tmode module), `CKBT51` |
| 2 | Firmware version (ASCII) | |
| 3 | Payload CRC (u32 LE) | |
| 4 | u32 LE, sent with the verify command (`0` in most files; non-zero in the CKBT51 file) | |
| 5 | Payload size (u32 LE) | |
| 6 | Header CRC (u32 LE) | |

### Checksums

- **Algorithm:** CRC-32, polynomial `0xEDB88320`, init `0xFFFFFFFF`,
  **no final XOR** (`crc = zlib.crc32(data) ^ 0xFFFFFFFF`). The Launcher's
  Bluetooth DFU uses the same variant.
- **Tag 3** covers the payload exactly as stored in the file.
- **Tag 6** covers the nibble-swapped header from offset 0 up to (not
  including) tag 6.
- **Tag 5** equals `file_size - header_length`.

### Parsing (as the updater does it)

1. Nibble-swap the first 4 bytes and check the `kfw` magic.
2. Nibble-swap the header and walk the tags until tag 6.
3. Check the header CRC (tag 6).
4. Take the payload from the **original** (unswapped) file bytes.
5. Check `header_length + size == file_size` and the payload CRC (tag 3).

The updater's file picker also accepts `.hex` (Intel HEX) and `.bin`.

---

## 2. The Payload Is Encrypted

- Entropy is about 7.99 bits/byte for every payload, swapped or not. There is
  no ARM vector table and no readable string.
- The Keychron Link type C and type D payloads have the same size and are
  ~88% identical. Every difference is 16-byte aligned, and a changed block
  never affects the next one. That rules out stream/XOR ciphers (byte-level
  differences) and CBC (a change would spread to all following blocks), and
  fits a 16-byte block cipher such as AES.
- No 16-byte block repeats inside a payload, although firmware normally has
  repeated padding. The cipher therefore also depends on the block position
  (a tweak or per-block counter, XTS-like), not plain ECB.
- The updater contains no cipher constants (no AES S-box/T-tables, no DES,
  TEA or SM4 tables). The only transform it applies is the nibble swap.

Decryption happens in the module's bootloader. Getting plaintext would need a
dump of that bootloader over SWD, which only works if readout protection is
off.

---

## 3. Updater HID Protocol

### Device Matching

| VID | Usage page / usage | PID | Mode | Report ID out / in |
|-----|--------------------|-----|------|--------------------|
| `0x3434` | `0xFF60` / `0x61` (QMK raw HID) | not `0xD0xx` | through the keyboard | `0x00` / `0x00` |
| `0x3434` | `0x8C` / `0x01` | `0xD000` | module bootloader directly | `0xB2` / `0xB1` |
| `0x3434` | `0x8C` / `0x01` | other | receiver | `0xB2` / `0xB1` |

In "through the keyboard" mode the keyboard firmware forwards the packets to
the module over UART/SPI (Keychron's QMK: `lkbt51_dfu_rx()` /
`ckbt51_dfu_rx()` in `keyboards/keychron/common/wireless/`, reached via the
`0xAA` command; see [wireless-dfu-factory.md](wireless-dfu-factory.md)).

### Packet Layout

33-byte reports, 500 ms write timeout, 800 ms read timeout:

```
[reportID] AA (55|56) LEN ~LEN SEQ CMD DATA... SUM_LO SUM_HI
```

- `55` = no reply expected, `56` = reply expected. Replies start with `AA 57`.
- `LEN` = length of (`CMD` + `DATA`) + 2; `~LEN` is its bitwise complement.
- `SUM` = 16-bit sum of the `CMD` and `DATA` bytes (the Launcher also adds
  `SEQ` for write packets, see [../launcher/dfu-protocols.md](../launcher/dfu-protocols.md#bluetooth-module-dfu)).

### Commands

| Cmd | Meaning |
|-----|---------|
| `0x60` | Device info (model, firmware version, hardware version) |
| `0x61` | DFU protocol version |
| `0x62` | Set update mode |
| `0x63` | Start update (1-byte argument: DFU version) |
| `0x64` | Write one data chunk |
| `0x65` | Verify: tag 3 CRC + tag 4 value, 8 bytes LE. Reply must match `AA 57 .. .. .. A3 .. 65 ..` |
| `0x66` | Reboot into the application |
| `0x67` | Reboot into the bootloader |
| `0x6F` | Firmware build time |

Keychron's QMK `LKBT51_CMD_*` enum numbers its own module commands
`0x60`--`0x65` differently; those are the keyboard↔module commands, not the
DFU commands above.

### Flash Sequence

1. `0x61` protocol version. If the module is already in its bootloader, go to step 5.
2. `0x67` switch to bootloader ("Switch to bootloader timeout" on failure).
3. Wait for the bootloader, then `0x60` info. Stop with "Incorrect bootloader"
   if the model differs from tag 1.
4. `0x61` protocol version again ("Unknown protocol version").
5. `0x62` set update mode ("Set update mode failed" / "Unsupport update mode").
6. `0x63` start update.
7. `0x64` × N, 16 bytes per chunk (the last one may be shorter), up to 5
   retries each:
   - DFU protocol v0: `[64, 16 data bytes]`
   - DFU protocol v1: `[64, retry flag, chunk index, 16 data bytes]`; the
     bootloader echoes the chunk index and the updater compares a running
     checksum with the reply.
   - Data is the raw (still encrypted) payload; 16 bytes is one cipher block.
8. `0x65` verify CRC ("CRC32 check failed").
9. `0x66` reboot, then `0x60` to show the new version.

There is no command to read flash back from the module.

---

## 4. Receiver `.bin` Images

Newer receivers are not shipped as `.kfw`:

| Receiver | Format |
|----------|--------|
| Keychron Link-KM (type A and type C) | Plain Telink image: `KNLT` magic at offset 8, readable strings (`tlk_8355_dongle`, `Keychron Link-KM`, model `KCNRFD00`). Not encrypted. |
| Keychron Ultra-Link 8K | Starts with a DER-encoded ECDSA signature (`30 45 02 21 ...`, 72 bytes) followed by a zero-padded header and plaintext ARM code (`Secure APP Entry`, `SCB->VTOR=...`). Signed, not encrypted. |

Neither has the `.kfw` header. The Launcher's update paths are described in
[../launcher/dfu-protocols.md](../launcher/dfu-protocols.md) and
[../launcher/bridge-dongle-protocol.md](../launcher/bridge-dongle-protocol.md#0xba----dfu-over-via);
which path each receiver uses was not traced.

---

## 5. Main MCU vs. Wireless Module

- `dfu-util` / QMK CLI only reach the keyboard's **main MCU** (for example an
  STM32F401) through ST's built-in ROM bootloader. That bootloader is the same
  on every chip and holds no Keychron code or key.
- `dfu-util` can read the main MCU's flash (`dfu-util -a 0 -s 0x08000000:0x80000 -U dump.bin`)
  if readout protection is off. That gives the QMK firmware only.
- The wireless module (LKBT51/CKBT51) has no USB connection. It is wired to
  the main MCU over UART/SPI and only reachable through the `AA 55` protocol
  above.

## Related Documents

- [Wireless DFU & Factory Test](wireless-dfu-factory.md) -- `0xAA` passthrough in the keyboard firmware
- [Launcher DFU Protocols](../launcher/dfu-protocols.md) -- the Launcher's own Bluetooth DFU path
