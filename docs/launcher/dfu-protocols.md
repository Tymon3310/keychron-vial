# DFU Protocols

The Keychron Launcher supports **five distinct firmware update paths**,
each targeting different hardware. This document describes all of them as
reverse-engineered from the Launcher JavaScript bundle.

> **Source:** Reverse-engineered from the Keychron Launcher. Our Vial GUI
> only implements one of these (STM32 DFU via `dfu-util`); see
> [firmware/wireless-dfu-factory.md](../firmware/wireless-dfu-factory.md).

## Firmware Payload Acquisition

Before any flashing can occur, the Launcher requests the firmware binaries from the Keychron API.

The primary API endpoint used to query firmware updates is:
```
GET https://launcher.keychron.com/api/merchandise/product/vpId/{vpId}
Headers:
  Client: launcher
  Env: Prod
```

The `vpId` is an integer calculated by combining the Vendor ID and Product ID:
`vpId = (vendorId * 65536) + productId`

For example, a Keychron device with `VID 0x3434` (13364) and `PID 0x0437` (1079) has a `vpId` of `875824183`.

If an update is available, the API returns a JSON response containing a download URL for the firmware binary (usually hosted on `cdn.shopify.com` or `sysmgr.keychron.cn`). The Launcher downloads this binary and then routes it to one of the five DFU paths below based on the device's hardware type.

## DFU Path Summary

| # | Class | Target Hardware | Transport | Trigger |
|---|-------|-----------------|-----------|---------|
| 1 | STM32 DFU | STM32 MCU | WebUSB (`dfu`/`dfuse` library) | Device name contains "STM" or "DFU in FS Mode" |
| 2 | STM32WB Bulk | STM32WB MCU | WebUSB bulk transfer (endpoint 1) | Device name contains "WB" |
| 3 | Nordic nRF | nRF wireless SoC | HID serial (SLIP/HCI, 115200 baud) | Device name contains "PCA" |
| 4 | XJ (Xinji) | XJ-based devices | HID report ID 1 | Device name contains "XJ" |
| 5 | XJ New | Newer XJ devices | HID report ID 0 | Device name contains "XJ_NEW" |

Additionally, a **Bluetooth DFU** protocol exists for updating the wireless
module firmware over the keyboard's USB HID connection.

### DFU Event Enum

All DFU classes report progress via a shared event system:

| Value | Name       | Description |
|-------|------------|-------------|
| 0     | `Erase`    | Erasing flash |
| 1     | `Flashing` | Writing firmware |
| 2     | `Complete` | Update successful |
| 3     | `Error`    | Update failed |

---

## 1. STM32 DFU (WebUSB)

Uses the standard USB DFU protocol (DFU 1.1 / DfuSe) via the `dfu.js` and
`dfuse.js` WebUSB library.

### Detection

- Triggered when the USB device name contains `"STM"` or `"DFU in FS Mode"`
- Checks `DFUVersion == 282` (0x011A) and `interfaceProtocol == 2` for DfuSe

### DFU Descriptor

The Launcher parses the DFU functional descriptor (`bDescriptorType == 33`):

| Field                  | Size | Description |
|------------------------|------|-------------|
| `bmAttributes`         | 1    | Capability bits |
| `wDetachTimeOut`       | 2    | Detach timeout (ms) |
| `wTransferSize`        | 2    | Max transfer size per packet |
| `bcdDFUVersion`        | 2    | DFU version (0x011A = DfuSe) |

`bmAttributes` bits:
- Bit 0: `CanDnload` -- device can receive firmware
- Bit 1: `CanUpload` -- device can send firmware (readback)
- Bit 2: `ManifestationTolerant` -- device survives manifest without reset
- Bit 3: `WillDetach` -- device detaches itself after DFU_DETACH

### Sequence

1. Open WebUSB device and claim DFU interface
2. Parse DFU descriptor for transfer size and capabilities
3. Erase target flash sectors
4. Download firmware in `wTransferSize`-byte chunks
5. Verify and manifest (reset to run new firmware)

---

## 2. STM32WB Bulk Transfer

A simpler protocol for STM32WB chips using raw WebUSB bulk transfers on
endpoint 1.

### Flash Base Address

`0x08000000` (standard STM32 flash start)

### Commands

| Bytes          | Description |
|----------------|-------------|
| `[0x5E, 0xF0]` | Erase chip command |
| `0x6A` header  | Write data command |

### Write Packet Format

```
Byte  Field           Description
----  -----           -----------
 0    0x6A            Write command
 1    0x00            Reserved
 2    0x00            Reserved
 3    0x00            Reserved
 4-7  address         Target address (LE32)
 8    0x00            Reserved
 9    0x01            Block count (1)
10    0x00            Reserved
11    0x00            Reserved
12..  data            Firmware data (256-byte chunks)
```

Total packet size: 12-byte header + 256 bytes data = 268 bytes.

### Sequence

1. Open WebUSB device, claim interface
2. Send erase command `[0x5E, 0xF0]`
3. Write firmware in 256-byte chunks at sequential addresses
4. Reset device

---

## 3. Nordic nRF DFU (SLIP/HCI)

Updates the Nordic nRF wireless SoC (e.g., nRF52840) using a serial
protocol over HID with SLIP framing.

### Transport

- Opens HID device at **115200 baud** (serial over HID)
- Report ID: `0xC0` (192)
- Uses SLIP encoding for packet framing

### SLIP Encoding

```
Original byte    Encoded as
--------------   ----------
0xC0 (END)       0xDB 0xDC
0xDB (ESC)       0xDB 0xDD
All others       Pass through
```

Every packet ends with `0xC0` (SLIP END marker).

### HCI Packet Format

```
Byte  Field           Description
----  -----           -----------
 0-3  header          SLIP header (4 bytes, encoded)
 4..  payload         Command payload
 N    crc16_lo        CRC-CCITT LSB
 N+1  crc16_hi        CRC-CCITT MSB
 N+2  0xC0            SLIP END marker
```

#### SLIP Header Construction

```
header[0] = seq | ((seq+1)%8 << 3) | (1 << 6) | (1 << 7)
header[1] = type | ((length & 0x0F) << 4)
header[2] = (length & 0xFF0) >> 4
header[3] = (~(header[0] + header[1] + header[2]) + 1) & 0xFF  // checksum
```

Where:
- `seq` = sequence number (mod 8, incremented per packet)
- `type` = 14 (0x0E) for DFU transport
- `length` = payload length

### CRC-CCITT

CRC-CCITT with initial value `0xFFFF` and polynomial `0x1021`:

```
for each byte b in data:
    crc = (crc >> 8 & 0xFF) | (crc << 8 & 0xFF00)
    crc ^= b
    crc ^= (crc & 0xFF) >> 4
    crc ^= (crc << 8) << 4
    crc ^= ((crc & 0xFF) << 4) << 1
return crc & 0xFFFF
```

### DFU Opcodes

| Value | Operation    | Description |
|-------|-------------|-------------|
| 1     | INIT        | Send init packet (`.dat` file from firmware ZIP) |
| 3     | CREATE      | Start DFU session (specify data type and length) |
| 4     | WRITE       | Write firmware data chunk |
| 5     | VALIDATE    | Validate written firmware |

### Firmware Package

Nordic DFU firmware is distributed as a ZIP file containing:
- `manifest.json` -- describes the update contents
- `*.dat` -- init packet (device type, version, etc.)
- `*.bin` -- firmware binary

### Sequence

1. Parse firmware ZIP (manifest, `.dat`, `.bin`)
2. **CREATE** -- send opcode 3 with data type 4 (DATA) and total binary length
3. **INIT** -- send opcode 1 with the `.dat` init packet contents
4. **WRITE** -- send opcode 4 with firmware data in 512-byte chunks (200ms delay between chunks)
5. **VALIDATE** -- send opcode 5 to verify firmware integrity
6. **Activate** -- close connection, device reboots into new firmware

---

## 4. XJ (Xinji) Chip Flash

Updates Xinji-based devices using a custom HID protocol on report ID 1.

### Header Byte

`0xA5` (165) -- magic marker for XJ flash commands.

### Commands

| Byte sequence              | Description |
|---------------------------|-------------|
| `[0xA5, 0x0C, 0, 1, 2, 0, 0, 4, 0, 0, 0, 0, 0, 1, 0]` | Erase chip |
| `writeMem` / `configMem`  | Write memory regions |
| `sendFile`                | Write firmware data (508-byte chunks) |
| `sendAck`                 | Acknowledge / sync |
| `reset`                   | Reset device |

### Response Filtering

Responses are identified by `data[0] == 0xA1` (161).

### Data Transfer

Firmware data is sent in 508-byte chunks per HID report.

### Vendor ID

XJ devices use vendor ID `0x34B7` (13495), separate from Keychron's main
`0x3434`.

---

## 5. XJ New Protocol

An updated version of the XJ protocol using HID report ID 0 instead of 1.

### Header

`[0x99, 0x55, 0x80, ...]` -- different magic bytes from the original XJ.

### Data Transfer

Firmware data is sent in 64-byte chunks (matching the HID report size).

---

## Bluetooth Module DFU

Updates the Bluetooth/2.4 GHz wireless module (e.g., LKBT51) firmware.
This is sent *over the keyboard's USB HID connection* -- the main MCU acts
as a passthrough to the wireless module.

### Packet Format

All BLE DFU packets use a framing envelope:

```
Byte  Field               Description
----  -----               -----------
 0    0xAA (170)          PacketType_Header marker
 1    type                0x55 (no-ack) or 0x56 (ack-required)
 2    length              Payload length
 3    byteNor(length)     Bitwise NOT of length (validation)
 4    sequence            Packet sequence number
 5    command             DFU command byte
 6..  data                Command-specific payload
 N    checksum            Additive checksum (varies by command)
```

### Packet Types

| Value | Hex    | Name                   | Description |
|-------|--------|------------------------|-------------|
| 170   | `0xAA` | `PacketType_Header`    | Start-of-packet marker (all packets) |
| 85    | `0x55` | `PacketType_Send_NoAck`| Fire-and-forget command |
| 86    | `0x56` | `PacketType_Send_Ack`  | Command requiring acknowledgment |
| 87    | `0x57` | *(response marker)*    | Multi-packet response continuation |

### DFU Commands

| Command | Hex    | Name                    | Type   | Description |
|---------|--------|-------------------------|--------|-------------|
| 96      | `0x60` | `getBluetoothDfuVersion`| NoAck  | Get BT module model, FW version, HW version |
| 97      | `0x61` | `getDFUVersion`         | NoAck  | Get DFU protocol version |
| 98      | `0x62` | `bluetoothUpdate`       | NoAck  | Prepare for firmware update |
| 99      | `0x63` | `bluetoothStartUpdate`  | Ack    | Start firmware transfer |
| 100     | `0x64` | `bluetoothWriteFile`    | Ack    | Write firmware data chunk |
| 101     | `0x65` | `bluetoothVerify`       | NoAck  | Verify firmware CRC32 |
| 102     | `0x66` | `bluetoothSwitch`       | NoAck  | Switch to new firmware |
| 103     | `0x67` | `bootloaderSwitch`      | NoAck  | Enter bootloader mode |
| 110     | `0x6E` | `getPatchVersion`       | *(special)* | Get Nordic patch/softdevice version |

### DFU Sequence

1. **Get BT DFU Version** (0x60) -- query module info (multi-packet response
   with state machine: Init → Pending, reassembled from multiple reports)
2. **Get DFU Version** (0x61) -- store DFU protocol version
3. **Bluetooth Update** (0x62) -- signal update intent
4. **Start Update** (0x63) -- begin transfer (includes DFU version in payload)
5. **Write File** (0x64) -- send firmware in 16-byte chunks, sequential
   packet numbering, up to 5 retries on failure (1500ms timeout)
6. **Verify** (0x65) -- send CRC32 of entire firmware (4 bytes, sent twice,
   big-endian byte order from hex string)
7. **Switch** (0x66) -- boot into new firmware

### Write File Packet (0x64) -- Version 0

```
Byte  Field           Description
----  -----           -----------
 0    0xAA            Header
 1    0x56            Ack required
 2    length          data_length + 1 + 2 + 2
 3    byteNor(len)    NOT of length
 4    sequence        Packet sequence number (incrementing)
 5    100 (0x64)      Write command
 6    0x00            Padding
 7    sequence        Sequence echo
 8..  data            Firmware data (16 bytes)
 N    checksum_lo     Additive checksum LSB
 N+1  checksum_hi     Additive checksum MSB
```

Checksum = `(100 + sequence + sum_of_data_bytes) & 0xFFFF`, stored LE.

### Write File Packet (0x64) -- Version 1

When `upgradeDfuVersion == 1`, the packet has a slightly different layout:
- Byte 6 = `0x00`
- Byte 7 = sequence
- Data starts at byte 8

### Multi-Packet Response Reassembly (0x60)

The `getBluetoothDfuVersion` response spans multiple HID reports:

1. **First packet**: Check `data[0] == 0xAA && data[1] == 0x57`. Extract
   total length from `data[2]`. Read payload from `data[5..]`.
2. **Subsequent packets**: Append raw bytes until `remaining == 0`.
3. **Parse assembled buffer**:
   - `moduleModel` (ASCII): bytes `4` to `13`
   - `fwVersion` (ASCII): bytes `16` to `24`
   - `hwVersion` (ASCII): bytes `26` to `35`

### Nordic Patch Version (0x6E)

A special command that uses raw bytes instead of the enum-based packet
builder:

```
Request:  [0xAA, 0x56, 3, 0xFC, 1, 110, 110]
Response: Filter for data[0]==0xAA && data[1]==0x55 && data[7]==110
          Parse bytes [14,13,12,11] as binary version string
```

### CRC32

Standard CRC32 with polynomial `0xEDB88320`:

```
for each byte:
    lookup = crc XOR byte
    for 8 iterations:
        if lookup & 1: lookup = (lookup >>> 1) XOR 0xEDB88320
        else:          lookup = lookup >>> 1
    crc = (crc >>> 8) XOR lookup
return crc XOR 0xFFFFFFFF
```

### `byteNor(n)`

Simple bitwise complement for packet length validation:

```
byteNor(n, bits=8):
    return bitwise NOT of n, masked to `bits` width
```

---

## HID Report IDs by DFU Path

| Report ID | DFU Path               |
|-----------|------------------------|
| 0         | XJ New, BLE DFU (keyboard), VIA tunneled |
| 1         | XJ (original)          |
| 178       | BLE DFU (via HID)      |
| 192       | Nordic nRF SLIP/HCI    |

---

## Related Documents

- [Firmware: Wireless DFU & Factory Test](../firmware/wireless-dfu-factory.md) -- firmware-side 0xAA/0xAB
- [Bridge / Dongle Protocol](bridge-dongle-protocol.md) -- FR_DFU_OVER_VIA
- [Launcher Overview](overview.md) -- checksums, buffer construction
