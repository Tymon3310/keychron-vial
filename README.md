# Keychron Vial

Vial support for Keychron and Lemokey keyboards, bringing real-time Vial configuration, 2.4 GHz wireless bridge access, Hall Effect analog switch tuning, RGB customization, and hardware settings to supported models.

> **Important Prerequisite:** Stock Keychron factory firmware only supports VIA and Keychron's proprietary web app (`launcher.keychron.com`) for advanced features like SOCD, per-key RGB, and Hall Effect tuning. To unlock open-source Vial configurability—including real-time remapping, tap dance, combos, dynamic macros, and offline desktop GUI support alongside all Keychron hardware features—you **must flash the custom firmware from `vial-qmk` onto your keyboard**.

---

## Start Here

| If you want to... | Use this | Notes |
|---|---|---|
| **Build or flash firmware** | [`vial-qmk`](https://github.com/tymon3310/vial-qmk) | Required: keyboard must run this custom firmware for Vial support |
| **Check keyboard support** | [Supported Keyboards](#supported-keyboards) | Full list of supported boards, layouts, and branches |
| **Use modern desktop GUI** | [Pipette Desktop](https://github.com/tymon3310/pipette-desktop) | Recommended: TypeScript/Electron GUI with full Keychron tab support |
| **Use Python desktop GUI** | [vial-gui](https://github.com/tymon3310/vial-gui) | Python/Qt GUI with identical Keychron protocol feature parity |
| **Configure in browser** | [Vial Web](https://vial.tymon3310.dev) | Instant WebHID access without installing software |

---

## Ecosystem Overview

### Firmware (`vial-qmk`)

A hybrid fork combining [vial-kb/vial-qmk](https://github.com/vial-kb/vial-qmk) and [Keychron/qmk_firmware](https://github.com/Keychron/qmk_firmware).

- **Vial runtime engine**: Real-time layout remapping, unlock flow, macros, tap dance, and combo configuration.
- **Keychron hardware layer**: Matrix drivers, wireless subsystems, RGB controls, and Hall Effect analog sensing.
- **Wireless XOR protocol**: Custom tunneling patch allowing full VIA/Vial configuration over the 2.4 GHz Keychron Link dongle.

| Branch | Base | Status | Description |
|---|---|---|---|
| [`vial-updated-keychron`](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron) | [2025q3](https://github.com/Keychron/qmk_firmware/tree/2025q3) | **Active (Default)** | Newer QMK base. Full feature parity with custom GUIs. |
| [`vial-keychron`](https://github.com/tymon3310/vial-qmk/tree/vial-keychron) | [wls_2025q1](https://github.com/keychron/qmk_firmware/tree/wls_2025q1) | Legacy | Older QMK base for boards not yet updated in Keychron upstream. |

> **Note on Branch Migration:** Keyboards remaining on the legacy branch (`vial-keychron`) are ported to `vial-updated-keychron` as Keychron updates their official upstream repository, avoiding upstream merge conflicts and keeping the codebase clean.

### Desktop and Web Configurators

Both desktop GUIs ([Pipette Desktop](https://github.com/tymon3310/pipette-desktop) and [vial-gui](https://github.com/tymon3310/vial-gui)) share identical Keychron feature support and protocol capabilities:
- Keychron Settings (Debounce, NKRO, 8K report rates, Wireless Low Power Mode)
- Snap Click (SOCD for standard mechanical switches)
- Keychron RGB (Effects, per-key RGB, mixed RGB, OS indicator colors)
- Analog Matrix (Hall Effect profiles, Rapid Trigger, DKS, Gamepad/XInput mode, calibration, and live travel monitor)
- 2.4 GHz Wireless Bridge support (automatic discovery via Keychron Link USB dongles)
- Built-in STM32 DFU Firmware flasher with layout backup and restore

| Configurator | Platform | Tech Stack | Notes |
|---|---|---|---|
| [**Pipette Desktop**](https://github.com/tymon3310/pipette-desktop) | Linux, Windows, macOS | TypeScript / Electron | **Recommended GUI.** Modern interface with Pipette Hub layout sharing. |
| [**vial-gui**](https://github.com/tymon3310/vial-gui) | Linux, Windows, macOS | Python / Qt | Classic Vial interface with full Keychron protocol support. |
| [**Vial Web**](https://vial.tymon3310.dev) | Chromium browsers | WebAssembly / WebHID | Quick browser configurator ([Source](https://github.com/tymon3310/vial-web)). |

---

## Supported Keyboards

All keyboards below support Vial keymap editing once flashed with our custom firmware. Keychron-specific tabs (wireless dongle bridge, custom RGB, SOCD, Hall Effect tuning) require using either Pipette Desktop or the custom `vial-gui` fork.

Most active boards live on the `vial-updated-keychron` branch. Boards still on `vial-keychron` are indicated in the branch column.

Support covers Keychron and Lemokey models across:
- Q / Q Pro / Q Max / Q HE
- V / V Max / V 8K
- K Pro / K Max / K HE / K Version
- C Pro / C Pro V2 / C Pro 8K
- R Series
- S / X / Lemokey HE

### Quick Navigation

Q Family: [Q Series](#q-series), [Q Pro](#q-pro-series), [Q Max](#q-max-series), [Q HE](#q-he-series)

V Family: [V Series](#v-series), [V Max](#v-max-series), [8K Series (V 8K)](#8k-series)

K Family: [K Pro](#k-pro-series), [K Max](#k-max-series), [K HE](#k-he-series), [K Version](#k-version-series)

Other: [C Pro](#c-pro-series), [C Pro V2](#c-pro-v2-series), [C Pro 8K](#c-pro-8k-series), [R Series](#r-series), [Other (S/X)](#other), [Lemokey](#lemokey-keychron-sub-brand)

### Q Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [Q0](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q0) | base, plus | vial-updated-keychron |
| [Q1v1](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q1v1) | ansi, ansi_encoder, iso, iso_encoder | vial-updated-keychron |
| [Q1v2](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q1v2) | ansi, ansi_encoder, iso, iso_encoder, jis, jis_encoder | vial-updated-keychron |
| [Q2](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q2) | ansi, ansi_encoder, iso, iso_encoder, jis, jis_encoder | vial-updated-keychron |
| [Q3](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q3) | ansi, ansi_encoder, iso, iso_encoder, jis, jis_encoder | vial-updated-keychron |
| [Q4](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q4) | ansi, iso | vial-updated-keychron |
| [Q5](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q5) | ansi, ansi_encoder, iso, iso_encoder | vial-updated-keychron |
| [Q6](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q6) | ansi, ansi_encoder, iso, iso_encoder | vial-updated-keychron |
| [Q7](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q7) | ansi, iso | vial-updated-keychron |
| [Q8](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q8) | ansi, ansi_encoder, iso, iso_encoder | vial-updated-keychron |
| [Q9](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q9) | ansi, ansi_encoder, iso, iso_encoder | vial-updated-keychron |
| [Q9 PLUS](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q9_plus) | ansi_encoder | vial-updated-keychron |
| [Q10](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q10) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q11](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q11) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q12](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q12) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q60](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q60) | ansi | vial-updated-keychron |
| [Q65](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q65) | ansi_encoder | vial-updated-keychron |

### Q Pro Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [Q1 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q1_pro) | ansi_knob, iso_knob, jis_encoder | vial-keychron |
| [Q2 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q2_pro) | ansi_encoder, iso_encoder | vial-keychron |
| [Q3 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q3_pro) | ansi_encoder, ansi_encoder_se, iso_encoder, iso_encoder_se, jis_encoder_se | vial-keychron |
| [Q4 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q4_pro) | ansi, iso | vial-keychron |
| [Q5 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q5_pro) | ansi_encoder, iso_encoder | vial-keychron |
| [Q6 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q6_pro) | ansi_encoder, iso_encoder | vial-keychron |
| [Q8 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q8_pro) | ansi_encoder, iso_encoder | vial-keychron |
| [Q10 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q10_pro) | ansi_encoder, iso_encoder | vial-keychron |
| [Q13 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q13_pro) | ansi_encoder, iso_encoder | vial-keychron |
| [Q14 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q14_pro) | ansi_encoder, iso_encoder | vial-keychron |

### Q Max Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [Q0 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q0_max) | encoder | vial-updated-keychron |
| [Q1 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q1_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [Q2 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q2_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q3 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q3_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q5 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q5_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [Q6 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q6_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q8 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q8_max) | ansi_encoder | vial-updated-keychron |
| [Q10 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q10_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q12 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q12_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q13 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q13_max) | ansi_encoder, jis_encoder | vial-updated-keychron |
| [Q14 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q14_max) | ansi_encoder | vial-updated-keychron |
| [Q15 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q15_max) | ansi_encoder | vial-updated-keychron |
| [Q60 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q60_max) | ansi | vial-updated-keychron |
| [Q65 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q65_max) | ansi_encoder | vial-updated-keychron |

### Q HE Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [Q1 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q1_he) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [Q2 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q2_he) | ansi_encoder | vial-updated-keychron |
| [Q3 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q3_he) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [Q4 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q4_he) | ansi | vial-updated-keychron |
| [Q5 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q5_he) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [Q6 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q6_he) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [Q12 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q12_he) | ansi_encoder, iso_encoder | vial-updated-keychron |

### V Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [V1](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v1) | abnt2_encoder, ansi, ansi_encoder, iso, iso_encoder, jis, jis_encoder | vial-updated-keychron |
| [V2](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v2) | abnt2_encoder, ansi, ansi_encoder, iso, iso_encoder, jis, jis_encoder | vial-updated-keychron |
| [V3](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v3) | abnt2_encoder, ansi, ansi_encoder, iso, iso_encoder, jis, jis_encoder | vial-updated-keychron |
| [V4](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v4) | ansi, iso | vial-updated-keychron |
| [V5](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v5) | ansi, ansi_encoder, iso, iso_encoder | vial-updated-keychron |
| [V6](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v6) | abnt2_encoder, ansi, ansi_encoder, iso, iso_encoder | vial-updated-keychron |
| [V6 ver 2](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v6_version_2) | iso_encoder | vial-updated-keychron |
| [V7](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v7) | ansi, iso | vial-updated-keychron |
| [V8](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v8) | ansi, ansi_encoder, iso, iso_encoder | vial-updated-keychron |
| [V10](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v10) | ansi_encoder, iso_encoder | vial-updated-keychron |

### V Max Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [V1 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v1_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [V2 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v2_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [V3 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v3_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [V4 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v4_max) | ansi, iso | vial-updated-keychron |
| [V5 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v5_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [V6 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v6_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [V8 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v8_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [V10 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v10_max) | ansi_encoder, iso_encoder | vial-updated-keychron |

### 8K Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [V1 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v1_8k) | ansi_encoder | vial-updated-keychron |
| [V3 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v3_8k) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [V6 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v6_8k) | ansi_encoder, iso_encoder | vial-updated-keychron |

### K Pro Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [K1 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k1_pro) | iso/rgb | vial-updated-keychron |
| [K2 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k2_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K3 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k3_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K4 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k4_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white | vial-keychron |
| [K5 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k5_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K6 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k6_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb | vial-keychron |
| [K7 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k7_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white | vial-keychron |
| [K8 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k8_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K9 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k9_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K10 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k10_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K11 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k11_pro) | ansi/rgb, ansi/white, ansi_encoder/rgb, ansi_encoder/white | vial-keychron |
| [K12 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k12_pro) | ansi/rgb, ansi/white | vial-keychron |
| [K13 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k13_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white | vial-keychron |
| [K14 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k14_pro) | ansi/rgb, ansi/white | vial-keychron |
| [K15 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k15_pro) | ansi_encoder/rgb, ansi_encoder/white | vial-keychron |
| [K17 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k17_pro) | ansi_encoder/rgb, ansi_encoder/white, iso_encoder/rgb, iso_encoder/white | vial-keychron |

### K Max Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [K1 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k1_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K2 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k2_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K3 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k3_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K4 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k4_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K5 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k5_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white, jis_v2/rgb, jis_v2/white | vial-updated-keychron |
| [K7 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k7_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white, jis_v2/rgb, jis_v2/white | vial-updated-keychron |
| [K8 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k8_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K9 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k9_max) | ansi/rgb, ansi/white | vial-updated-keychron |
| [K10 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k10_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K11 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k11_max) | ansi_encoder/rgb, ansi_encoder/white, iso_encoder/rgb, iso_encoder/white, jis_encoder/rgb, jis_encoder/white | vial-updated-keychron |
| [K13 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k13_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K15 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k15_max) | ansi_encoder/rgb, ansi_encoder/white, iso_encoder/rgb, iso_encoder/white | vial-updated-keychron |
| [K17 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k17_max) | ansi_encoder/rgb, ansi_encoder/white, iso_encoder/rgb, iso_encoder/white, jis_encoder/rgb, jis_encoder/white | vial-updated-keychron |

### K HE Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [K2 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k2_he) | ansi, iso, jis | vial-updated-keychron |
| [K4 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k4_he) | ansi, iso, jis | vial-updated-keychron |
| [K6 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k6_he) | ansi | vial-updated-keychron |
| [K8 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k8_he) | ansi, iso, jis | vial-updated-keychron |
| [K10 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k10_he) | ansi, iso | vial-updated-keychron |

### K Version Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [K3 VERSION 3](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k3_version_3) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |

### C Pro Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [C1 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c1_pro) | ansi/rgb, ansi/white | vial-updated-keychron |
| [C2 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c2_pro) | ansi/rgb, ansi/white | vial-updated-keychron |
| [C3 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c3_pro) | ansi/red, ansi/rgb | vial-updated-keychron |

### C Pro V2 Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [C1 PRO V2](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c1_pro_v2) | ansi/non_light, ansi/rgb, ansi/white | vial-updated-keychron |
| [C2 PRO V2](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c2_pro_v2) | ansi/rgb, ansi/white | vial-updated-keychron |

### C Pro 8K Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [C1 PRO 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c1_pro_8k) | ansi, iso, jis | vial-updated-keychron |
| [C2 PRO 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c2_pro_8k) | ansi, iso | vial-updated-keychron |
| [C3 PRO 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c3_pro_8k) | ansi, iso, jis | vial-updated-keychron |

### R Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [R3](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/r3) | ansi, jis, th | vial-updated-keychron |
| [R5](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/r5) | ansi | vial-updated-keychron |
| [R6](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/r6) | ansi, th | vial-updated-keychron |

### Other

| Keyboard | Layouts | Branch |
|---|---|---|
| [S1](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/s1) | ansi/rgb, ansi/white | vial-updated-keychron |
| [X0](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/x0) | red | vial-updated-keychron |

### Lemokey (Keychron Sub-brand)

| Keyboard | Layouts | Branch |
|---|---|---|
| [L1 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/lemokey/l1_he) | ansi | vial-updated-keychron |
| [P1 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/lemokey/p1_he) | ansi, iso, jis | vial-updated-keychron |
| [P2 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/lemokey/p2_he) | ansi | vial-updated-keychron |
| [P3 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/lemokey/p3_he) | ansi | vial-updated-keychron |

---

## Features

### Standard Vial Capabilities
- Real-time on-the-fly keymap and layer remapping (up to 4+ layers)
- Macro programming and key combinations
- Tap dance actions and combo keys
- Key overrides and core QMK settings

### Keychron-Specific Capabilities

#### 1. Wireless 2.4 GHz Bridge Configuration
Configure supported wireless Keychron keyboards through the Keychron Link 2.4 GHz USB dongle without plugging in a physical cable:
- **Transparent VIA/Vial tunneling**: All standard keymap edits and Keychron-specific configuration commands work identically over wireless 2.4 GHz as they do over wired USB.
- **Supported dongles**: Keychron Link USB-A (`3434:D030`) and USB-C (`3434:D031`).
- **Automatic device detection**: The GUI detects the bridge dongle and paired keyboard automatically.
- **Smart connection deduplication**: If a keyboard is connected simultaneously via USB and 2.4 GHz dongle, the wired USB connection is cleanly prioritized.
- **Requires custom firmware**: The keyboard must run firmware from `vial-updated-keychron` with the wireless XOR encoding patch (working around internal buffer handling in the LKBT51 module).

#### 2. Keychron Settings
- **Debounce**: Adjust hardware key debounce filter (1ms to 20ms).
- **NKRO**: Toggle N-Key Rollover mode on or off.
- **Report Rate**: Switch polling rates between 125Hz, 250Hz, 500Hz, 1000Hz (and 8000Hz on 8K supported models).
- **Wireless Low Power Mode**: Fine-tune sleep timers and power-saving behavior for wireless operation.

#### 3. Snap Click (SOCD)
Configure Simultaneous Opposite Cardinal Direction handling for mechanical keyboards:
- Define opposing key pairs (e.g. A+D for strafing, W+S for pitch).
- Select resolution modes: Last Input Priority, First Key Priority, Second Key Priority, or Neutral (cancellation).

#### 4. Keychron RGB Lighting
- **Global RGB Modes**: Select and customize 40+ onboard lighting animations.
- **Per-Key RGB**: Set distinct custom colors for individual keys.
- **Mixed RGB**: Overlay per-key color overrides on top of active background animations.
- **OS Indicator Colors**: Customize LED indicator colors for Caps Lock, Num Lock, and Scroll Lock states.

#### 5. Analog Matrix (Hall Effect Keyboards Only)
- **Profiles**: Store multiple independent analog profiles; switch, save, and reset configurations per game or workflow.
- **Actuation & Travel Modes**:
  - **Global**: Inherit default profile actuation parameters.
  - **Regular**: Custom static actuation point from 0.1mm to 4.0mm.
  - **Rapid Trigger**: Dynamic press and release thresholds (0.1mm to 4.0mm) with bottom deadzone filtering.
  - **Dynamic Keystroke (DKS)**: Two distinct travel zones (shallow and deep), each triggering up to 4 key actions (Press, Release, Tap, Re-press).
  - **Gamepad / XInput**: Assign analog switches to HID joystick axes or Xbox 360 controller inputs (triggers, analog sticks, and buttons) with custom 4-point response curves.
  - **Toggle**: Key toggles held state upon each press.
- **HE SOCD**: Hall Effect-specific resolution including Deeper Travel Wins, Last Input Wins, or Neutral cancellation across up to 20 pairs.
- **Calibration & Live Monitor**: Real-time per-key travel depth monitor with manual and power-on auto-calibration.

#### 6. STM32 DFU Firmware Flasher
Integrated directly into Pipette Desktop and vial-gui:
- **Automatic Layout Backup**: Backs up your current keymaps, macros, and Keychron settings before flashing.
- **One-Click Flash**: Reboots keyboard into DFU mode, flashes compiled `.bin` firmware using `dfu-util`, and restores your layout automatically.
- **Live Progress & MCU Detection**: Reports real-time erase/write progress and detected MCU type.
- **WebUSB Flashing**: Also available directly in the browser via WebUSB DfuSe.

---

## Getting Started

### Step 1: Flash Custom Firmware (Prerequisite)

While stock factory firmware supports features like SOCD and Hall Effect tuning exclusively through Keychron's web launcher (`launcher.keychron.com`), it does not support Vial. To configure your keyboard with Vial (desktop or web) while retaining full control over Keychron hardware features, you must first flash our custom `vial-qmk` firmware.

1. **Build from source**:
   ```bash
   # Clone the active repository
   git clone --branch vial-updated-keychron https://github.com/tymon3310/vial-qmk.git
   cd vial-qmk

   # Set up QMK CLI environment
   pipx install qmk
   qmk setup -H .
   qmk git submodule

   # Compile firmware for your model and layout (example: V5 Max ANSI Encoder)
   qmk compile -kb keychron/v5_max/ansi_encoder -km vial
   ```

2. **Flash to keyboard**:
   - **Using QMK CLI**:
     ```bash
     qmk flash -kb keychron/v5_max/ansi_encoder -km vial
     ```
   - **Using `dfu-util` directly**:
     Put the keyboard into bootloader mode (hold `Esc` while plugging in the USB cable, or press the reset button under the spacebar):
     ```bash
     dfu-util -a 0 -d 0483:DF11 -s 0x8000000:leave -D <firmware.bin>
     ```

### Step 2: Configure Your Keyboard

Once custom firmware is running on your board:

#### Option A: Pipette Desktop (Recommended)
1. Download the latest installer or package from [Pipette GitHub Releases](https://github.com/tymon3310/pipette-desktop/releases/latest).
   - **Linux AppImage**:
     ```bash
     chmod +x Pipette-linux-x86_64.AppImage
     ./Pipette-linux-x86_64.AppImage
     ```
   - **Arch Linux (AUR)**:
     ```bash
     yay -S pipette-desktop-keychron-bin
     ```
   - **Windows / macOS**: Download and run the release installers.

#### Option B: vial-gui (Python Desktop)
1. Download from [vial-gui GitHub Releases](https://github.com/tymon3310/vial-gui/releases/latest).
   - **Arch Linux (AUR)**:
     ```bash
     yay -S vial-keychron-bin
     ```
   - **Linux AppImage / Windows / macOS**: Use the release packages or run from source:
     ```bash
     git clone --branch vial-keychron https://github.com/tymon3310/vial-gui.git
     cd vial-gui
     uv venv .venv --python 3.12
     source .venv/bin/activate
     uv pip install -r requirements.txt
     python src/main/python/main.py
     ```

#### Option C: Vial Web (Browser)
1. Open [vial.tymon3310.dev](https://vial.tymon3310.dev) in any Chromium-based browser.
2. Click **Start Vial** and select your keyboard from the WebHID prompt.

---

## Subsequent Firmware Updates

Once your keyboard is running Vial firmware, routine updates can be flashed directly from the **DFU Firmware updater** tab in Pipette Desktop or vial-gui:
1. Open Pipette Desktop or vial-gui.
2. Navigate to the **DFU Firmware updater** tab.
3. Select your updated `.bin` file.
4. Click **Flash**. The application will automatically back up your keymap and settings, reboot into DFU mode, flash the update, and restore your configuration.

> **Flashing Requirements:**
> - **Linux:** Install `dfu-util` via package manager (e.g. `sudo pacman -S dfu-util` or `sudo apt install dfu-util`).
> - **Windows:** Install `dfu-util` via [Scoop](https://scoop.sh/) (`scoop install dfu-util`) or ensure it is on your `PATH`. For browser WebUSB flashing, install the STM32 DFU drivers from [QMK Toolbox](https://github.com/qmk/qmk_toolbox/blob/master/windows/QMK%20Toolbox/Resources/qmk_driver_installer.exe).
> - **macOS:** Install `dfu-util` via Homebrew (`brew install dfu-util`).

---

## Troubleshooting

### Keyboard Not Detected in Vial
1. Ensure the keyboard has been flashed with custom `vial-qmk` firmware (stock Keychron firmware only works with VIA / `launcher.keychron.com` and will not be detected by Vial).
2. Ensure the physical mode switch is set to **Cable** (or connect via the Keychron Link 2.4 GHz wireless dongle). Bluetooth connection does not support raw HID configuration.
3. On Linux, configure udev permissions:
   ```bash
   export USER_GID=`id -g`; sudo --preserve-env=USER_GID sh -c 'echo "KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{serial}=="*vial:f64c2b3c*", MODE="0660", GROUP="$USER_GID", TAG+="uaccess", TAG+="udev-acl"" > /etc/udev/rules.d/59-vial.rules && udevadm control --reload && udevadm trigger'
   ```

### Keychron Tabs Not Appearing in GUI
Keychron-specific tabs appear when:
1. You are using our custom configurator ([Pipette Desktop](https://github.com/tymon3310/pipette-desktop), custom [vial-gui](https://github.com/tymon3310/vial-gui), or [vial-web](https://vial.tymon3310.dev)). Stock upstream Vial GUI does not include Keychron tabs.
2. The keyboard is running custom firmware built with Keychron extensions enabled.
3. The keyboard model supports the specific hardware feature (for instance, the Analog Matrix tab requires a Hall Effect model).

---

## Contributing

Contributions and ports for additional keyboard models are welcome:
1. Fork the target repository ([vial-qmk](https://github.com/tymon3310/vial-qmk), [pipette-desktop](https://github.com/tymon3310/pipette-desktop), or [vial-gui](https://github.com/tymon3310/vial-gui)).
2. Create a feature branch.
3. Submit a pull request.

### Adding a New Keyboard Port
1. Create the keyboard directory under `keyboards/keychron/<model>/` in `vial-qmk`.
2. Add the `keymaps/vial/` folder containing `keymap.c`, `config.h` (with unique `VIAL_KEYBOARD_UID`), `vial.json`, and `rules.mk`.
3. Generate a unique UID: `python3 util/vial_generate_keyboard_uid.py`.
4. Compile and verify matrix mapping on hardware.

---

## License

- **vial-qmk**: GPL-2.0 (inherited from QMK)
- **vial-gui**: GPL-2.0 (inherited from Vial)
- **pipette-desktop**: GPL-3.0 (inherited from Pipette)
- **Documentation**: MIT

---

## Credits

- [QMK Firmware](https://qmk.fm/)
- [Vial](https://get.vial.today/)
- [Keychron](https://github.com/Keychron/qmk_firmware)
- [Pipette](https://github.com/darakuneko/pipette-desktop)
- Community contributors and testers

---

## Hardware Testing Note

Daily development and hardware verification are conducted on a **Keychron V5 Max (ANSI Encoder)**. Other targets in the tree are maintained against Keychron upstream firmware source definitions and community pull requests. If you notice any model-specific anomalies, please open an issue or pull request.
