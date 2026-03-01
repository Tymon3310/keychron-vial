# Keychron RGB Protocol (0xA8)

The RGB sub-protocol (`KC_KEYCHRON_RGB = 0xA8`) manages Keychron's per-key
RGB system and the Mixed RGB layering system. This is separate from the
standard QMK/Vial RGB Matrix -- Keychron's system adds per-key color
assignment, region-based effect layering, and OS indicator LED control.

All commands follow the standard response format:

```
Byte  Field           Description
----  -----           -----------
 0    0xA8            Command echo
 1    sub_command     Sub-command echo
 2    status          0 = success, 1 = fail
 3..  payload         Sub-command-specific data
```

## Sub-Command Map

| Sub-cmd | Value  | Direction | Description                     |
|---------|--------|-----------|---------------------------------|
| `RGB_GET_PROTOCOL_VER`         | `0x01` | Read  | Protocol version          |
| `RGB_SAVE`                     | `0x02` | Write | Save all RGB to EEPROM    |
| `GET_INDICATORS_CONFIG`        | `0x03` | Read  | OS indicator LED config   |
| `SET_INDICATORS_CONFIG`        | `0x04` | Write | Set OS indicator config   |
| `RGB_GET_LED_COUNT`            | `0x05` | Read  | Total LED count           |
| `RGB_GET_LED_IDX`              | `0x06` | Read  | LED matrix index mapping  |
| `PER_KEY_RGB_GET_TYPE`         | `0x07` | Read  | Per-key effect type       |
| `PER_KEY_RGB_SET_TYPE`         | `0x08` | Write | Set per-key effect type   |
| `PER_KEY_RGB_GET_COLOR`        | `0x09` | Read  | Per-key colors (batched)  |
| `PER_KEY_RGB_SET_COLOR`        | `0x0A` | Write | Set per-key color         |
| `MIXED_EFFECT_RGB_GET_INFO`    | `0x0B` | Read  | Mixed RGB layer info      |
| `MIXED_EFFECT_RGB_GET_REGIONS` | `0x0C` | Read  | LED region assignments    |
| `MIXED_EFFECT_RGB_SET_REGIONS` | `0x0D` | Write | Set LED region assignments|
| `MIXED_EFFECT_RGB_GET_EFFECT_LIST` | `0x0E` | Read  | Region effect list  |
| `MIXED_EFFECT_RGB_SET_EFFECT_LIST` | `0x0F` | Write | Set region effects  |

---

## 0x01 -- Get Protocol Version

```
Request:  [0xA8] [0x01]
Response: data[3..4] = version (LE16)
```

Current version: `0x0001` (`PER_KEY_RGB_VER`).

---

## 0x02 -- Save RGB

Persists all RGB settings (per-key colors, effect type, mixed RGB regions and
effects, OS indicator config) to EEPROM.

```
Request:  [0xA8] [0x02]
Response: data[2] = status
```

This must be called after making changes to commit them to non-volatile storage.

---

## 0x03 / 0x04 -- OS Indicator Config

OS indicators are special LEDs (Caps Lock, Num Lock, Scroll Lock) that can be
configured with a custom color and individually enabled/disabled.

### Get (0x03)

```
Request:  [0xA8] [0x03]
Response:
  data[3] = available_mask   (which indicators the hardware supports)
  data[4] = disable_mask     (which indicators are disabled by the user)
  data[5] = hue              (H in HSV, 0-255)
  data[6] = saturation       (S in HSV, 0-255)
  data[7] = value            (V in HSV, 0-255)
```

### Set (0x04)

```
Request:  [0xA8] [0x04] [disable_mask] [hue] [sat] [val]
```

### Notes

- `available_mask` is read-only (determined by hardware).
- `disable_mask` bits: bit 0 = Caps Lock, bit 1 = Num Lock, bit 2 = Scroll Lock.
  Setting a bit disables that indicator.
- The HSV color applies to all enabled OS indicators uniformly.

---

## 0x05 -- Get LED Count

```
Request:  [0xA8] [0x05]
Response: data[3] = led_count
```

Returns the total number of addressable LEDs on the keyboard.

---

## 0x06 -- Get LED Index Mapping

Maps matrix positions (row, col) to LED indices. This is essential for the GUI
to correlate key positions with LED addresses.

### Request

```
[0xA8] [0x06] [row] [col_mask_b0] [col_mask_b1] [col_mask_b2] [0x00]
```

- `row`: Matrix row number
- `col_mask`: 24-bit little-endian bitmask indicating which columns to query

### Response

```
data[3..26] = LED indices for each column (0xFF = no LED at that position)
```

### Notes

- The GUI calls this for each row during `reload_led_matrix_mapping()` to build
  a complete `(row, col) -> led_index` dictionary.
- Entries with value `0xFF` indicate matrix positions that have no physical LED.

---

## 0x07 / 0x08 -- Per-Key RGB Effect Type

### Get (0x07)

```
Request:  [0xA8] [0x07]
Response: data[3] = effect_type
```

### Set (0x08)

```
Request:  [0xA8] [0x08] [effect_type]
```

### Per-Key Effect Types

| Value | Constant                         | Name                |
|-------|----------------------------------|---------------------|
| 0     | `PER_KEY_RGB_SOLID`             | Solid               |
| 1     | `PER_KEY_RGB_BREATHING`         | Breathing           |
| 2     | `PER_KEY_RGB_REACTIVE_SIMPLE`   | Reactive Simple     |
| 3     | `PER_KEY_RGB_REACTIVE_MULTI_WIDE` | Reactive Multi Wide |
| 4     | `PER_KEY_RGB_REACTIVE_SPLASH`   | Reactive Splash     |

### Notes

- **Solid**: Each key displays its assigned HSV color constantly.
- **Breathing**: Keys pulse between their color and off.
- **Reactive**: Keys light up in response to keypresses, then fade.
- The effect type applies globally to all per-key LEDs. Individual key colors
  are set separately via the color commands.

---

## 0x09 / 0x0A -- Per-Key Colors

### Get Colors (0x09)

Reads colors in batches (up to 9 LEDs per packet).

```
Request:  [0xA8] [0x09] [start_index] [count]
Response: data[3..] = H, S, V triplets (3 bytes per LED)
```

- Maximum `count` per request: 9 (fits 27 bytes in the 29-byte payload area)
- Colors are in HSV format, each component 0-255.

### Set Color (0x0A)

```
Request:  [0xA8] [0x0A] [index] [1] [hue] [sat] [val]
```

### Notes

- After setting colors, call `RGB_SAVE` (`0x02`) to persist to EEPROM.
- The GUI reads all colors during initialization and caches them in
  `keychron_per_key_colors` as a list of `(H, S, V)` tuples.

---

## 0x0B -- Mixed RGB Get Info

Returns metadata about the Mixed RGB layering system.

```
Request:  [0xA8] [0x0B]
Response:
  data[3] = effect_layers    (number of regions/layers)
  data[4] = effects_per_layer (max effects per region)
```

### Notes

- If `effect_layers == 0`, Mixed RGB is not supported on this keyboard.
- Each LED is assigned to one region (layer). Each region has a list of effects
  that cycle in sequence.

---

## 0x0C / 0x0D -- Mixed RGB Regions

Region assignments determine which Mixed RGB effect layer each LED belongs to.

### Get Regions (0x0C)

```
Request:  [0xA8] [0x0C] [start_index] [count]
Response: data[3..] = region assignments (1 byte per LED, max 29 per packet)
```

### Set Regions (0x0D)

```
Request:  [0xA8] [0x0D] [start_index] [count] [region0] [region1] ...
```

Maximum 28 regions per packet (32 - 4 header bytes).

---

## 0x0E / 0x0F -- Mixed RGB Effect Lists

Each Mixed RGB region has an ordered list of effects that play in sequence.

### Get Effect List (0x0E)

```
Request:  [0xA8] [0x0E] [region] [start_index] [count]
Response: data[3..] = effect entries (8 bytes each, max 3 per packet)
```

### Set Effect List (0x0F)

```
Request:  [0xA8] [0x0F] [region] [start_index] [count] [entries...]
```

### Effect Entry Format (8 bytes each)

```
Byte  Field    Description
----  -----    -----------
 0    effect   Effect type index (from firmware effect list)
 1    hue      HSV hue (0-255)
 2    sat      HSV saturation (0-255)
 3    speed    Animation speed (0-255)
 4-7  time     Display duration in milliseconds (LE32)
```

### Notes

- Effects within a region play sequentially. When the `time` duration for one
  effect expires, the next effect in the list begins.
- Maximum 3 effects fit per HID packet (3 x 8 = 24 bytes).
- The firmware defines effect types in `keychron_rgb_type.h`. Common types
  include solid, breathing, rainbow, and reactive variants.
