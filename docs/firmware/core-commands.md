# Core Commands (0xA0 -- 0xA3)

These are the top-level commands used during the initial handshake with a
Keychron keyboard. They are simple request/response pairs with no sub-commands.

## 0xA0 -- Get Protocol Version

Returns the Keychron protocol version number.

### Request

```
Byte  Value
----  -----
 0    0xA0
 1..31  (unused, zero-filled)
```

### Response

```
Byte  Field               Description
----  -----               -----------
 0    0xFF or 0xA0        0xFF = not a Keychron keyboard; 0xA0 = valid
 1    protocol_version    Protocol version (currently 0x02)
```

### Notes

- This is the first command sent during `reload_keychron()`.
- If the response byte 0 is `0xFF`, the GUI aborts Keychron initialization
  and treats the keyboard as a standard VIA/Vial device.
- The GUI stores the version in `keychron_protocol_version`.

---

## 0xA1 -- Get Firmware Version

Returns the firmware version as a null-terminated ASCII string.

### Request

```
Byte  Value
----  -----
 0    0xA1
 1..31  (unused)
```

### Response

```
Byte  Field               Description
----  -----               -----------
 0    0xFF or 0xA1        0xFF = unsupported
 1..  version_string      Null-terminated ASCII (e.g. "2.0.3\0")
```

### Notes

- The string is decoded with `utf-8` on the GUI side, ignoring errors.
- Stored in `keychron_firmware_version`.
- The version string typically matches the firmware release tag.

---

## 0xA2 -- Get Support Feature

Returns a 2-byte bitmask of supported Keychron features.

### Request

```
Byte  Value
----  -----
 0    0xA2
 1..31  (unused)
```

### Response

```
Byte  Field              Description
----  -----              -----------
 0    0xFF or 0xA2       0xFF = unsupported
 1    (padding)          Unused
 2    features_low       Feature flags byte 0 (bits 0-7)
 3    features_high      Feature flags byte 1 (bits 8-15)
```

### Feature Flag Definitions

| Bit  | Mask     | Constant                 | Description             |
|------|----------|--------------------------|-------------------------|
| 0    | `0x0001` | `FEATURE_DEFAULT_LAYER`  | DIP switch default layer query |
| 1    | `0x0002` | `FEATURE_BLUETOOTH`      | Bluetooth support       |
| 2    | `0x0004` | `FEATURE_P24G`           | 2.4 GHz wireless support |
| 3    | `0x0008` | `FEATURE_ANALOG_MATRIX`  | Hall Effect analog switches |
| 4    | `0x0010` | `FEATURE_STATE_NOTIFY`   | State change notifications |
| 5    | `0x0020` | `FEATURE_DYNAMIC_DEBOUNCE` | Configurable debounce |
| 6    | `0x0040` | `FEATURE_SNAP_CLICK`     | Snap Click (SOCD for mechanical) |
| 7    | `0x0080` | `FEATURE_KEYCHRON_RGB`   | Per-key / Mixed RGB     |
| 8    | `0x0100` | `FEATURE_QUICK_START`    | Quick start mode        |
| 9    | `0x0200` | `FEATURE_NKRO`           | N-Key Rollover toggle   |

### Notes

- The combined 16-bit value is computed as `data[2] | (data[3] << 8)`.
- Stored in `keychron_features`.
- Individual feature support is checked via helper methods like
  `has_keychron_debounce()`, `has_keychron_rgb()`, etc.

---

## 0xA3 -- Get Default Layer

Queries the current default layer, typically set by the physical DIP switch on
the back of the keyboard (e.g., Mac vs Windows layout).

### Request

```
Byte  Value
----  -----
 0    0xA3
 1..31  (unused)
```

### Response

```
Byte  Field              Description
----  -----              -----------
 0    0xA3               Command echo
 1    default_layer      Layer index (0, 1, 2, ...)
```

### Notes

- Only available when `FEATURE_DEFAULT_LAYER` (bit 0) is set.
- Returns `-1` on the GUI side if the response is invalid.
- The default layer determines which base keymap is active (e.g., layer 0 =
  Mac, layer 2 = Windows on many Keychron boards).
