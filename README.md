# Keychron Vial

Vial firmware and GUI support for Keychron keyboards.

This project brings full [Vial](https://get.vial.today/) support to Keychron keyboards, including advanced features like Keychron-specific settings, RGB control, and Hall Effect (HE) analog configuration.

## Quick Links

| Resource | Description |
|----------|-------------|
| [Vial Web (Keychron Edition)](https://vial.tymon3310.dev) | Browser-based configurator |
| [vial-qmk Firmware](https://github.com/tymon3310/vial-qmk) | QMK firmware with Vial + Keychron support |
| [vial-gui Desktop App](https://github.com/tymon3310/vial-gui) | Desktop configurator with Keychron tabs |
| [Keyboard Definitions](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron) | Vial JSON files and keymaps |

---

## Repositories

### vial-qmk (Firmware)

Fork of [vial-kb/vial-qmk](https://github.com/vial-kb/vial-qmk) with Keychron keyboard ports.

| Branch | Base | Status | Description |
|--------|------|--------|-------------|
| [`vial-keychron`](https://github.com/tymon3310/vial-qmk/tree/vial-keychron) | [wls_2025q1](https://github.com/keychron/qmk_firmware/tree/wls_2025q1) | **Active** | Current stable branch with all keyboards |
| [`vial-updated-keychron`](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron) | [2025q3](https://github.com/Keychron/qmk_firmware/tree/2025q3) | WIP | Will become main once Keychron updates upstream |

### vial-gui (Desktop App)

Fork of [vial-kb/vial-gui](https://github.com/vial-kb/vial-gui) with Keychron-specific configuration tabs.

| Branch | Description |
|--------|-------------|
| [`vial-keychron`](https://github.com/tymon3310/vial-gui/tree/vial-keychron) | Adds Keychron settings tabs |

### vial-web (Browser App)

Fork of [vial-kb/vial-web](https://github.com/vial-kb/vial-web) configured to use the Keychron-enabled GUI.

| URL | Description |
|-----|-------------|
| [vial.tymon3310.dev](https://vial.tymon3310.dev) | Live deployment |

---

## Supported Keyboards

All keyboards below have Vial support with full keymap editing. Keychron-specific features (RGB, settings) require the custom GUI.

### Q Series (Aluminum, Gasket Mount)

| Keyboard | Layouts | Link |
|----------|---------|------|
| Q0 | Numpad | [keyboards/keychron/q0](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q0) |
| Q1 v1 | ANSI/ISO | [keyboards/keychron/q1v1](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q1v1) |
| Q1 v2 | ANSI/ISO, Encoder | [keyboards/keychron/q1v2](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q1v2) |
| Q2 | ANSI/ISO, Encoder | [keyboards/keychron/q2](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q2) |
| Q3 | ANSI/ISO, Encoder | [keyboards/keychron/q3](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q3) |
| Q4 | ANSI | [keyboards/keychron/q4](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q4) |
| Q5 | ANSI/ISO, Encoder | [keyboards/keychron/q5](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q5) |
| Q6 | ANSI/ISO, Encoder | [keyboards/keychron/q6](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q6) |
| Q7 | ANSI | [keyboards/keychron/q7](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q7) |
| Q8 | ANSI/ISO, Encoder | [keyboards/keychron/q8](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q8) |
| Q9 | ANSI, Encoder | [keyboards/keychron/q9](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q9) |
| Q9 Plus | ANSI, Encoder | [keyboards/keychron/q9_plus](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q9_plus) |
| Q10 | ANSI/ISO, Encoder | [keyboards/keychron/q10](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q10) |
| Q11 | ANSI/ISO, Encoder | [keyboards/keychron/q11](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q11) |
| Q12 | ANSI/ISO, Encoder | [keyboards/keychron/q12](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q12) |
| Q60 | ANSI | [keyboards/keychron/q60](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q60) |
| Q65 | ANSI, Encoder | [keyboards/keychron/q65](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q65) |

### Q HE Series (Hall Effect)

| Keyboard | Layouts | Link |
|----------|---------|------|
| Q1 HE | ANSI, Encoder | [keyboards/keychron/q1_he](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q1_he) |
| Q2 HE | ANSI, Encoder | [keyboards/keychron/q2_he](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q2_he) |
| Q3 HE | ANSI, Encoder | [keyboards/keychron/q3_he](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q3_he) |
| Q4 HE | ANSI, Encoder | [keyboards/keychron/q4_he](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q4_he) |
| Q5 HE | ANSI, Encoder | [keyboards/keychron/q5_he](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q5_he) |
| Q6 HE | ANSI, Encoder | [keyboards/keychron/q6_he](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q6_he) |
| Q12 HE | ANSI, Encoder | [keyboards/keychron/q12_he](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q12_he) |

### V Series (Plastic, Budget-Friendly)

| Keyboard | Layouts | Link |
|----------|---------|------|
| V1 | ANSI/ISO, Encoder | [keyboards/keychron/v1](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v1) |
| V2 | ANSI/ISO, Encoder | [keyboards/keychron/v2](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v2) |
| V3 | ANSI/ISO, Encoder | [keyboards/keychron/v3](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v3) |
| V4 | ANSI | [keyboards/keychron/v4](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v4) |
| V5 | ANSI, Encoder | [keyboards/keychron/v5](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v5) |
| V6 | ANSI/ISO, Encoder | [keyboards/keychron/v6](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v6) |
| V7 | ANSI | [keyboards/keychron/v7](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v7) |
| V8 | ANSI/ISO, Encoder | [keyboards/keychron/v8](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v8) |
| V10 | ANSI/ISO, Encoder | [keyboards/keychron/v10](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v10) |

### V Max Series (Wireless)

| Keyboard | Layouts | Link |
|----------|---------|------|
| V1 Max | ANSI, Encoder | [keyboards/keychron/v1_max](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v1_max) |
| V5 Max | ANSI, Encoder | [keyboards/keychron/v5_max](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v5_max) |
| V6 Max | ANSI, Encoder | [keyboards/keychron/v6_max](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v6_max) |

### K Series (Wireless, Low Profile Available)

| Keyboard | Layouts | Link |
|----------|---------|------|
| K1 Pro | ANSI/ISO | [keyboards/keychron/k1_pro](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k1_pro) |
| K8 Pro | ANSI/ISO, Encoder | [keyboards/keychron/k8_pro](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k8_pro) |
| K9 Max | ANSI, Encoder | [keyboards/keychron/k9_max](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k9_max) |

### K HE Series (Hall Effect)

| Keyboard | Layouts | Link |
|----------|---------|------|
| K2 HE | ANSI, Encoder | [keyboards/keychron/k2_he](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k2_he) |
| K4 HE | ANSI, Encoder | [keyboards/keychron/k4_he](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k4_he) |
| K6 HE | ANSI, Encoder | [keyboards/keychron/k6_he](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k6_he) |
| K8 HE | ANSI, Encoder | [keyboards/keychron/k8_he](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k8_he) |
| K10 HE | ANSI, Encoder | [keyboards/keychron/k10_he](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k10_he) |

### C Series (Wired, Budget)

| Keyboard | Layouts | Link |
|----------|---------|------|
| C1 Pro | ANSI | [keyboards/keychron/c1_pro](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/c1_pro) |
| C1 Pro v2 | ANSI | [keyboards/keychron/c1_pro_v2](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/c1_pro_v2) |
| C1 Pro 8K | ANSI | [keyboards/keychron/c1_pro_8k](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/c1_pro_8k) |
| C2 Pro | ANSI | [keyboards/keychron/c2_pro](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/c2_pro) |
| C2 Pro v2 | ANSI | [keyboards/keychron/c2_pro_v2](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/c2_pro_v2) |
| C2 Pro 8K | ANSI | [keyboards/keychron/c2_pro_8k](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/c2_pro_8k) |
| C3 Pro | ANSI | [keyboards/keychron/c3_pro](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/c3_pro) |
| C3 Pro 8K | ANSI | [keyboards/keychron/c3_pro_8k](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/c3_pro_8k) |

### Other

| Keyboard | Layouts | Link |
|----------|---------|------|
| S1 | ANSI | [keyboards/keychron/s1](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/s1) |
| X0 | Numpad | [keyboards/keychron/x0](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/x0) |

---

## Features

### Standard Vial Features
- Real-time keymap editing
- Layer management
- Macro programming
- Tap dance configuration
- Combo keys
- Key overrides
- QMK Settings

### Keychron-Specific Features (Custom GUI Required)

#### Keychron Settings Tab
- **Debounce**: Adjust key debounce time (1-20ms)
- **NKRO**: Toggle N-Key Rollover
- **Report Rate**: 125Hz / 250Hz / 500Hz / 1000Hz (8000Hz on supported models)
- **Wireless Low Power Mode**: Battery optimization for wireless keyboards

#### Snap Click (SOCD)
Configure Simultaneous Opposite Cardinal Direction handling for gaming:
- Define key pairs (e.g., A+D, W+S)
- Choose resolution mode (Last input wins, Neutral, etc.)

#### Keychron RGB
- **Global RGB Mode**: Select from 40+ RGB effects
- **Per-Key RGB**: Set individual key colors
- **Mixed RGB**: Combine effects with per-key overrides
- **OS Indicators**: Customize Caps Lock, Num Lock, Scroll Lock LED colors

#### Analog Matrix (Hall Effect keyboards only)
- **Multiple Profiles**: Store different actuation configurations
- **Actuation Point**: Adjust per-key actuation distance (0.1mm - 4.0mm)
- **Rapid Trigger**: Enable/disable with configurable sensitivity
- **Response Curves**: Linear, Fast, Slow, or Custom curves

---

## Getting Started

### Option 1: Use Vial Web (Easiest)

1. Visit [vial.tymon3310.dev](https://vial.tymon3310.dev)
2. Click "Start Vial"
3. Select your keyboard from the WebHID prompt
4. Configure your keyboard

> **Note:** Keychron-specific tabs may not be fully functional in web version.

### Option 2: Desktop App

1. Clone the vial-gui repository:
   ```bash
   git clone --branch vial-keychron https://github.com/tymon3310/vial-gui.git
   cd vial-gui
   ```

2. Install dependencies:
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Linux/macOS
   # or: .venv\Scripts\activate  # Windows
   pip install -r requirements.txt
   ```

3. Run the app:
   ```bash
   python src/main/python/main.py
   ```

### Option 3: Build Custom Firmware

1. Clone the vial-qmk repository:
   ```bash
   git clone --branch vial-keychron https://github.com/tymon3310/vial-qmk.git
   cd vial-qmk
   ```

2. Set up QMK:
   ```bash
   qmk setup -H .
   ```

3. Build firmware for your keyboard:
   ```bash
   # Example: Keychron V5 Max ANSI with encoder
   qmk compile keychron/v5_max/ansi_encoder -km vial
   
   # Example: Keychron Q5 HE ANSI with encoder
   qmk compile keychron/q5_he_max/ansi_encoder -km vial
   ```

4. Flash the firmware using QMK Toolbox or:
   ```bash
   # Put keyboard in bootloader mode first (usually Esc when repluing usb cable or reset button)
   qmk flash -kb keychron/v5_max/ansi_encoder -km vial
   ```

---

## Flashing Instructions

### Entering Bootloader Mode

Most Keychron keyboards enter bootloader mode by:

1. **Esc** while plugging in, OR
2. **Physical reset button** bellow space key

### Flashing Tools

- **QMK Toolbox** (Windows/macOS): [Download](https://github.com/qmk/qmk_toolbox/releases)
- **dfu-util** (Linux): `sudo apt install dfu-util`

### Flash Command Example
```bash
# STM32 (most Keychron keyboards)
dfu-util -a 0 -d 0483:df11 -s 0x08000000:leave -D keychron_v5_max_ansi_encoder_vial.bin
```

---

## Troubleshooting

### Keyboard not detected in Vial

1. Ensure you're using the Vial firmware (not stock QMK or VIA)
2. Check that the keyboard is in wired mode (not Bluetooth)
3. On Linux, add udev rules:
   ```bash
   echo 'KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="3434", MODE="0666"' | sudo tee /etc/udev/rules.d/99-keychron.rules
   sudo udevadm control --reload-rules
   ```

### Keychron tabs not showing

The Keychron-specific tabs only appear when:
1. Using the custom vial-gui fork
2. Keyboard firmware has Keychron features enabled
3. Keyboard supports the specific feature (e.g., Analog Matrix only on HE keyboards)

### RGB not working correctly

1. Ensure VialRGB is enabled in firmware
2. Check that the keyboard declares `"lighting": "vialrgb"` in vial.json
3. Use the custom GUI - stock Vial GUI may not support all effects

---

## Contributing

Contributions are welcome! Please:

1. Fork the relevant repository
2. Create a feature branch
3. Submit a pull request

### Adding a New Keyboard

1. Create the keyboard folder in `keyboards/keychron/<keyboard_name>/`
2. Add `keymaps/vial/` with:
   - `keymap.c`
   - `config.h` (with unique `VIAL_KEYBOARD_UID`)
   - `vial.json` (keyboard layout definition)
   - `rules.mk`
3. Generate a unique UID: `python3 util/vial_generate_keyboard_uid.py`
4. Test the build and flash

---

## License

- vial-qmk: GPL-2.0 (inherited from QMK)
- vial-gui: GPL-2.0 (inherited from Vial)
- This documentation: MIT

---

## Credits

- [QMK Firmware](https://qmk.fm/) - The foundation
- [Vial](https://get.vial.today/) - Real-time keyboard configuration
- [Keychron](https://github.com/Keychron/qmk_firmware) - Original keyboard firmware
- Community contributors
- Claude Opus for fixing some anoying stuff that for some reson just stopped working on it's own

---
