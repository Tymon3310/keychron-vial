# Screen Protocol (0xAC)

The Screen protocol (`KC_SCREEN = 0xAC`) manages keyboards with built-in
displays. It provides raw flash memory access, a simple filesystem, and
RTC (real-time clock) control. This command is **only found in the Launcher**
and does not exist in our firmware fork.

> **Source:** Reverse-engineered from the Keychron Launcher JavaScript bundle.
> Command names are from the `GUIF_CTL_OPCODE` enum.

## Sub-Command Map

| Sub-cmd | Value  | Hex    | Direction | Description                |
|---------|--------|--------|-----------|----------------------------|
| `GUIF_CTL_OPCODE_FLASH_WRITE`       | 1   | `0x01` | Write | Write raw flash by address |
| `GUIF_CTL_OPCODE_FLASH_READ`        | 2   | `0x02` | Read  | Read raw flash by address  |
| `GUIF_CTL_OPCODE_FLASH_ERASE`       | 3   | `0x03` | Write | Erase flash block          |
| `GUIF_CTL_OPCODE_FS_FILE_OPEN`      | 4   | `0x04` | Write | Open file on device FS     |
| `GUIF_CTL_OPCODE_FS_FILE_WRITE`     | 5   | `0x05` | Write | Write to open file         |
| `GUIF_CTL_OPCODE_FS_FILE_READ`      | 6   | `0x06` | Read  | Read from open file        |
| `GUIF_CTL_OPCODE_FS_FILE_CLOSE`     | 7   | `0x07` | Write | Close open file            |
| `GUIF_CTL_OPCODE_FS_FILE_REMOVE`    | 8   | `0x08` | Write | Delete file from FS        |
| `GUIF_CTL_OPCODE_FS_FORMAT`         | 9   | `0x09` | Write | Format entire filesystem   |
| `GUIF_CTL_OPCODE_EFLASH_EDIT_ENABLE`| 11  | `0x0B` | Write | Enable external flash editing |
| `GUIF_CTRL_SET_COUNT`               | 12  | `0x0C` | Write | Set count (purpose unclear) |
| `GUIF_CTL_OPCODE_GETTIME`           | 15  | `0x0F` | Read  | Get RTC time               |
| `GUIF_CTL_OPCODE_SETTIME`           | 16  | `0x10` | Write | Set RTC time               |
| `GUIF_CTL_OPCODE_RESPONSE`          | 119 | `0x77` | Read  | Response acknowledgment    |

---

## Packet Format

Screen protocol packets use 32-byte or 64-byte HID reports (depending on
the command). The last 2 bytes of the buffer carry an additive checksum.

### General Layout

```
Byte  Field           Description
----  -----           -----------
 0    0xAC            KC_SCREEN command ID
 1    sub_command     GUIF_CTL_OPCODE_* value
 2..  payload         Sub-command-specific data
30-31 checksum        Additive checksum (LE16) -- for 32-byte packets
62-63 checksum        Additive checksum (LE16) -- for 64-byte packets
```

### Checksum Calculation

The checksum is the sum of all relevant payload bytes, stored as a 16-bit
little-endian value in the last 2 bytes of the packet.

---

## Flash Memory Operations

### Flash Read (0x02)

```
Request:
  Byte 0    = 0xAC
  Byte 1    = 0x02 (FLASH_READ)
  Byte 2-4  = address (24-bit, 3 bytes)
  Byte 5    = length (max 24 bytes per read)
  Byte 30-31 = checksum (LE16)

Response:
  Byte 0    = 0xAC
  Byte 1    = 0x02
  Byte 2..  = read data
```

### Flash Write (0x01)

```
Request:
  Byte 0    = 0xAC
  Byte 1    = 0x01 (FLASH_WRITE)
  Byte 2-4  = address (24-bit, 3 bytes)
  Byte 5    = data_length
  Byte 6..  = data bytes
  Byte 30-31 = checksum (LE16)
```

### Flash Erase (0x03)

Erase operations typically operate on 4096-byte blocks (standard flash
sector size).

---

## Filesystem Operations

The device implements a simple flat filesystem on external flash. Files are
identified by name (ASCII string).

### File Open (0x04)

```
Request (64-byte buffer):
  Byte 0    = 0xAC
  Byte 1    = 0x04 (FS_FILE_OPEN)
  Byte 2    = filename_length
  Byte 3..  = filename (ASCII characters)
  Byte 62-63 = checksum (LE16)
```

### File Write (0x05)

```
Request (64-byte buffer):
  Byte 0    = 0xAC
  Byte 1    = 0x05 (FS_FILE_WRITE)
  Byte 2-4  = offset (24-bit LE, byte offset into file)
  Byte 5    = chunk_length
  Byte 6..  = data bytes
  Byte 62-63 = checksum (LE16)
```

### File Read (0x06)

Similar structure to File Write but reads data from the specified offset.

### File Close (0x07)

Closes the currently open file handle.

### File Remove (0x08)

Deletes a file by name from the filesystem.

### Format (0x09)

Formats the entire filesystem, erasing all files.

---

## RTC Time Operations

### Set Time (0x10)

```
Request:
  Byte 0  = 0xAC
  Byte 1  = 0x10 (SETTIME)
  Byte 2  = year - 2000  (e.g., 25 for 2025)
  Byte 3  = month (1-12)
  Byte 4  = day (1-31)
  Byte 5  = hour (0-23)
  Byte 6  = minute (0-59)
  Byte 7  = second (0-59)
```

### Get Time (0x0F)

```
Response:
  Byte 3  = year - 2000
  Byte 4  = month
  Byte 5  = day
  Byte 6  = hour
  Byte 7  = minute
  Byte 8  = second
```

---

## Notes

- The `GUIF_CTL_OPCODE_RESPONSE` (119 / `0x77`) appears to be a generic
  response marker rather than a command the host sends.
- The `EFLASH_EDIT_ENABLE` (0x0B) must be sent before raw flash write/erase
  operations are permitted -- it acts as a safety interlock.
- `GUIF_CTRL_SET_COUNT` (0x0C) purpose is unclear from the minified code;
  it may set a counter for batch operations.
- This protocol is likely used for keyboards with OLED/LCD screens that
  display custom images, animations, or clock widgets.
