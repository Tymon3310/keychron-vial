# Keychron Vial

Vial firmware and GUI support for Keychron keyboards.

This project brings full [Vial](https://get.vial.today/) support to Keychron keyboards, including advanced features like Keychron-specific settings, RGB control, and Hall Effect (HE) analog configuration.

## Quick Links

| Resource | Description |
|----------|-------------|
| [Vial Web (Keychron Edition)](https://vial.tymon3310.dev) | Browser-based configurator |
| [Pipette Desktop (Alternative)](https://github.com/tymon3310/pipette-desktop) | Modern Electron-based configurator (Recommended) |
| [vial-gui Desktop App](https://github.com/tymon3310/vial-gui) | Legacy Python-based desktop configurator |
| [vial-qmk Firmware](https://github.com/tymon3310/vial-qmk) | QMK firmware with Vial + Keychron support |
| [Keyboard Definitions](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron) | Vial keymaps |

---

## Repositories

### vial-qmk (Firmware)

Fork of [vial-kb/vial-qmk](https://github.com/vial-kb/vial-qmk) with Keychron keyboard ports.

| Branch | Base | Status | Description |
|--------|------|--------|-------------|
| [`vial-keychron`](https://github.com/tymon3310/vial-qmk/tree/vial-keychron) | [wls_2025q1](https://github.com/keychron/qmk_firmware/tree/wls_2025q1) | Legacy | Older QMK base. Does not fully support the custom GUI. |
| [`vial-updated-keychron`](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron) | [2025q3](https://github.com/Keychron/qmk_firmware/tree/2025q3) | **Active** | Newer QMK base. Fully supports the custom GUI. |

> **Note:** Not all keyboards have been ported to `vial-updated-keychron` yet. Keyboards still only available on `vial-keychron` are marked accordingly.

### pipette-desktop (Alternative Desktop App)

Fork of [darakuneko/pipette-desktop](https://github.com/darakuneko/pipette-desktop) with a refined UI and built-in Pipette Hub for sharing layouts.

| Branch | Description |
|--------|-------------|
| [`vial-keychron`](https://github.com/tymon3310/pipette-desktop/tree/vial-keychron) | **Active** — Main repository with Keychron support |

### vial-gui (Legacy Desktop App)

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

### Q Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
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
|----------|---------|--------|
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
|----------|---------|--------|
| [Q0 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q0_max) | encoder | vial-updated-keychron |
| [Q1 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q1_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [Q2 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q2_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q3 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q3_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q5 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q5_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q6 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q6_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q8 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q8_max) | ansi_encoder | vial-updated-keychron |
| [Q10 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q10_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [Q12 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q12_max) | ansi_encoder | vial-updated-keychron |
| [Q13 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q13_max) | ansi_encoder | vial-updated-keychron |
| [Q14 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q14_max) | ansi_encoder | vial-updated-keychron |
| [Q15 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q15_max) | ansi_encoder | vial-updated-keychron |
| [Q60 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q60_max) | ansi | vial-updated-keychron |
| [Q65 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q65_max) | ansi_encoder | vial-updated-keychron |

### Q HE Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [Q1 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q1_he) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [Q2 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q2_he) | ansi_encoder | vial-updated-keychron |
| [Q3 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q3_he) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [Q4 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q4_he) | ansi | vial-updated-keychron |
| [Q5 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q5_he) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [Q6 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q6_he) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [Q12 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q12_he) | ansi_encoder, iso_encoder | vial-updated-keychron |

### V Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [V1](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v1) | ansi, ansi_encoder, iso, iso_encoder, jis, jis_encoder | vial-updated-keychron |
| [V2](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v2) | ansi, ansi_encoder, iso, iso_encoder, jis, jis_encoder | vial-updated-keychron |
| [V3](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v3) | ansi, ansi_encoder, iso, iso_encoder, jis, jis_encoder | vial-updated-keychron |
| [V4](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v4) | ansi, iso | vial-updated-keychron |
| [V5](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v5) | ansi, ansi_encoder, iso, iso_encoder | vial-updated-keychron |
| [V6](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v6) | ansi, ansi_encoder, iso, iso_encoder | vial-updated-keychron |
| [V6 ver 2](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v6_version_2) | iso_encoder | vial-updated-keychron |
| [V7](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v7) | ansi, iso | vial-updated-keychron |
| [V8](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v8) | ansi, ansi_encoder, iso, iso_encoder | vial-updated-keychron |
| [V10](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v10) | ansi_encoder, iso_encoder | vial-updated-keychron |

### V Max Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [V1 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v1_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [V2 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v2_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [V3 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v3_max) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [V4 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v4_max) | ansi | vial-updated-keychron |
| [V5 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v5_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [V6 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v6_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [V8 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v8_max) | ansi_encoder | vial-updated-keychron |
| [V10 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v10_max) | ansi_encoder | vial-updated-keychron |

### K Pro Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
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
|----------|---------|--------|
| [K1 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k1_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K2 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k2_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K3 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k3_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K4 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k4_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K5 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k5_max) | ansi/rgb, ansi/white, iso/rgb, iso/white | vial-updated-keychron |
| [K7 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k7_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K8 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k8_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K9 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k9_max) | ansi/rgb, ansi/white | vial-updated-keychron |
| [K10 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k10_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K11 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k11_max) | ansi_encoder/rgb, ansi_encoder/white, iso_encoder/rgb, iso_encoder/white | vial-updated-keychron |
| [K13 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k13_max) | ansi/rgb, ansi/white, iso/rgb, iso/white | vial-updated-keychron |
| [K15 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k15_max) | ansi_encoder/rgb, ansi_encoder/white, iso_encoder/rgb, iso_encoder/white | vial-updated-keychron |
| [K17 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k17_max) | ansi_encoder/rgb, ansi_encoder/white, iso_encoder/rgb, iso_encoder/white, jis_encoder/rgb, jis_encoder/white | vial-keychron |

### K HE Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [K2 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k2_he) | ansi, iso, jis | vial-updated-keychron |
| [K4 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k4_he) | ansi, iso, jis | vial-updated-keychron |
| [K6 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k6_he) | ansi | vial-updated-keychron |
| [K8 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k8_he) | ansi, iso, jis | vial-updated-keychron |
| [K10 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k10_he) | ansi, iso | vial-updated-keychron |

### K Version Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [K3 VERSION 3](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k3_version_3) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |

### C Pro Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [C1 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c1_pro) | ansi/rgb, ansi/white | vial-updated-keychron |
| [C2 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c2_pro) | ansi/rgb, ansi/white | vial-updated-keychron |
| [C3 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c3_pro) | ansi/red, ansi/rgb | vial-updated-keychron |

### C Pro V2 Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [C1 PRO V2](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c1_pro_v2) | ansi/non_light, ansi/rgb, ansi/white | vial-updated-keychron |
| [C2 PRO V2](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c2_pro_v2) | ansi/rgb, ansi/white | vial-updated-keychron |

### C Pro 8K Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [C1 PRO 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c1_pro_8k) | ansi, iso, jis | vial-updated-keychron |
| [C2 PRO 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c2_pro_8k) | ansi, iso | vial-updated-keychron |
| [C3 PRO 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c3_pro_8k) | ansi, iso, jis | vial-updated-keychron |

### Other

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [S1](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/s1) | ansi/rgb, ansi/white | vial-updated-keychron |
| [X0](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/x0) | red | vial-updated-keychron |

### Lemokey (Keychron Sub-brand)

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [L1 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/lemokey/l1_he) | ansi | vial-updated-keychron |
| [P1 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/lemokey/p1_he) | ansi, iso, jis | vial-updated-keychron |
| [P2 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/lemokey/p2_he) | ansi | vial-updated-keychron |

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

#### Wireless Configuration (2.4 GHz Bridge)
Configure your wireless Keychron keyboard through the 2.4 GHz USB dongle (Keychron Link) — no USB cable needed:
- **Transparent VIA/Vial tunneling**: All standard and Keychron-specific features work wirelessly, exactly as they do over USB
- **Supported dongles**: Keychron Link USB-A (`3434:D030`) and USB-C (`3434:D031`)
- **Automatic detection**: The GUI detects the bridge dongle and wirelessly-connected keyboard automatically
- **USB+dongle deduplication**: When the same keyboard is connected via both USB and dongle, the USB connection is preferred
- **Non-blocking connection**: The GUI stays responsive while establishing the wireless link
- **Requires firmware support**: The keyboard must be running the Vial firmware from the `vial-updated-keychron` branch with the wireless XOR encoding patch (required to work around a hardware limitation in the LKBT51 wireless module)

#### Keychron Settings Tab
- **Debounce**: Adjust key debounce time (1-20ms)
- **NKRO**: Toggle N-Key Rollover
- **Report Rate**: 125Hz / 250Hz / 500Hz / 1000Hz (8000Hz on supported models)
- **Wireless Low Power Mode**: Battery optimization for wireless keyboards

#### Snap Click (SOCD)
Configure Simultaneous Opposite Cardinal Direction handling for non-HE keyboards:
- Define key pairs (e.g., A+D, W+S)
- Choose resolution mode: Last Input, First Key, Second Key, or Neutral

#### Keychron RGB
- **Global RGB Mode**: Select from 40+ RGB effects
- **Per-Key RGB**: Set individual key colors
- **Mixed RGB**: Combine effects with per-key overrides
- **OS Indicators**: Customize Caps Lock, Num Lock, Scroll Lock LED colors

#### Analog Matrix (Hall Effect keyboards only)

##### Profiles
- Multiple named profiles, each storing a full independent configuration
- Switch between profiles on the fly; save or reset per profile

##### Actuation
- **Multiple Modes** per key:
  - **Global** — inherit the profile's default settings
  - **Regular** — fixed actuation point
  - **Rapid Trigger** — dynamic actuation; re-triggers on direction change
  - **Dynamic Keystroke (DKS)** — two travel zones (shallow/deep), each triggering up to 4 keycodes with configurable actions (Press, Release, Tap, Re-press)
  - **Gamepad** — assign key to a joystick axis or button (see below)
  - **Toggle** — key toggles its held state on each press
- **Actuation Point**: Per-key or global, 0.1mm–4.0mm
- **Rapid Trigger**: Separate press and release sensitivity (0.1mm–4.0mm); bottom dead zone prevents false re-triggers near bottom-out

##### SOCD (HE keyboards)
Configure Simultaneous Opposite Cardinal Direction resolution per key pair:
- **Deeper Travel Wins** — whichever key is pressed further takes priority
- **Deeper Travel Wins (Single)** — as above, but holds last state if both fully bottomed out
- **Last Input Wins** — most recently pressed key takes priority
- **Key 1 Priority / Key 2 Priority** — one key always wins
- **Neutral** — both keys cancel each other out
- Up to 20 configurable pairs per profile

##### Gamepad Mode
Keys in Gamepad mode can be assigned to standard HID joystick or XInput (Xbox controller) inputs:
- **HID Joystick**: 6 axes (X, Y, Z, RX, RY, RZ); up to 32 buttons
- **XInput**: Full Xbox 360 controller emulation — left/right sticks, analog triggers (LT/RT), 16 buttons (A, B, X, Y, LB, RB, Back, Start, L3, R3, D-pad)
- **Typing + Gaming**: Optional mode where keyboard input stays active alongside gamepad
- **Circular normalization**: Diagonal stick directions are scaled to stay within the circular input boundary
- **Response Curve**: 4-point piecewise linear curve maps key travel to analog axis value

##### Calibration
- **Auto-calibration**: Runs at power-on and monitors for sensor drift
- **Manual calibration**: Two-phase (zero travel → full travel → save)
- **Real-time key travel monitor**: Live travel depth readout per key for diagnostics
- **Read calibrated values**: Inspect per-key zero/full travel and scale factor

#### Firmware Flasher (STM32 DFU)
Built-in firmware flasher in the GUI — no external tools needed on desktop:
- **Automatic layout backup**: Saves current keymap, macros and Keychron settings before flashing
- **One-click flash**: Reboots keyboard into DFU mode, waits for DFU device, flashes `.bin` with `dfu-util`, then restores layout automatically
- **Progress bar**: Live erase and download progress from `dfu-util`
- **MCU/firmware info**: Displays detected MCU type and current firmware version before flashing
- **Web support**: Also available in the browser via WebUSB DfuSe
---

## Getting Started

### Option 1: Use Vial Web (Easiest)

1. Visit [vial.tymon3310.dev](https://vial.tymon3310.dev)
2. Click "Start Vial"
3. Select your keyboard from the WebHID prompt
4. Configure your keyboard

> **Note:** Keychron-specific tabs may not be fully functional in the web version.

### Option 2: Arch Linux (AUR)

If you are using Arch Linux, you can install the GUI via the AUR.

**Recommended (Pipette):**
- [`pipette-desktop-keychron-bin`](https://aur.archlinux.org/packages/pipette-desktop-keychron-bin) (Pre-compiled)
- [`pipette-desktop-keychron-git`](https://aur.archlinux.org/packages/pipette-desktop-keychron-git) (Build from source)

**Legacy (vial-gui):**
- [`vial-keychron-bin`](https://aur.archlinux.org/packages/vial-keychron-bin) (Pre-compiled)
- [`vial-keychron-git`](https://aur.archlinux.org/packages/vial-keychron-git) (Build from source)

```bash
# Example using yay
# Recommended (Pipette)
yay -S pipette-desktop-keychron-bin

# Legacy (vial-gui)
yay -S vial-keychron-bin
```

### Option 3: Pipette Desktop (Recommended)

Pipette is the modern, Electron-based alternative to the original Vial GUI. It features a cleaner UI and faster performance.

1. Download the latest release from the [Pipette GitHub Releases](https://github.com/tymon3310/pipette-desktop/releases/latest).
2. For Linux, download the `.AppImage`, make it executable, and run:
   ```bash
   chmod +x Pipette-linux-x86_64.AppImage
   ./Pipette-linux-x86_64.AppImage
   ```
3. For Windows/macOS, use the provided installers.

#### Build from source

1. Clone the repository:
   ```bash
   git clone --branch vial-keychron https://github.com/tymon3310/pipette-desktop.git
   cd pipette-desktop
   ```

2. Install dependencies (requires [pnpm](https://pnpm.io/)):
   ```bash
   pnpm install
   ```

3. Run the app:
   ```bash
   pnpm dev
   ```

   OR build the production app:
   ```bash
   pnpm build
   pnpm dist:linux  # or dist:win, dist:mac
   ```

### Option 4: Legacy Desktop App (vial-gui)

Use this if you prefer the original Python-based interface or encounter issues with Pipette.

1. Download the latest release from the [vial-gui GitHub Releases](https://github.com/tymon3310/vial-gui/releases/latest).
2. For Linux, download the `.AppImage`, make it executable, and run:
   ```bash
   chmod +x Vial-*.AppImage
   ./Vial-*.AppImage
   ```
3. For Windows/macOS, use the provided installers.

#### Build from source

1. Clone the vial-gui repository:
   ```bash
   git clone --branch vial-keychron https://github.com/tymon3310/vial-gui.git
   cd vial-gui
   ```

2. Install dependencies:
   ```bash
   uv venv .venv --python 3.12
   source .venv/bin/activate  # Linux/macOS
   # or: .venv\Scripts\activate  # Windows
   uv pip install -r requirements.txt
   ```

3. Run the app:
   ```bash
   python src/main/python/main.py
   ```
   OR
   Compile the app:
   ```bash
   pyinstaller misc/Vial.spec --noconfirm
   ./dist/Vial/Vial
   ```

### Option 5: Build and Flash Custom Firmware

1. Clone the vial-qmk repository:
   ```bash
   # Active branch (recommended)
   git clone --branch vial-updated-keychron https://github.com/tymon3310/vial-qmk.git
   # Legacy branch (for keyboards not yet ported)
   git clone --branch vial-keychron https://github.com/tymon3310/vial-qmk.git
   cd vial-qmk
   ```

2. Set up QMK:
   ```bash
   qmk git submodule
   ```

3. Build and flash firmware for your keyboard:
   ```bash
   # Compile only
   qmk compile -kb keychron/v5_max/ansi_encoder -km vial

   # Or compile and flash in one step (puts keyboard in bootloader mode automatically)
   qmk flash -kb keychron/v5_max/ansi_encoder -km vial
   ```

   > `qmk flash` will prompt you to put the keyboard in bootloader mode (hold Esc while plugging in, or press the reset button). For subsequent updates once Vial is running, the GUI flasher is more convenient as it can restore settings — see [Flashing Instructions](#flashing-instructions).

---

## Flashing Instructions

> **Note:** The GUI Flasher tab requires the keyboard to already be running Vial firmware. For a first-time flash from stock firmware, use the manual method.

### Option 1: GUI Flasher (Recommended for updates)

The desktop and web apps have a built-in **DFU Firmware updater** tab — use this to update firmware on a keyboard that is already running Vial:

1. Open the app and connect your keyboard
2. Go to the **DFU Firmware updater** tab
3. Select your compiled `.bin` file
4. Click **Flash** — the GUI will back up your layout, reboot the keyboard into DFU mode, flash the firmware, and if you selected to restore current layout (on by default), restore your layout automatically

> The GUI flasher on desktop calls `dfu-util` under the hood and works on Windows, Linux, and macOS — as long as `dfu-util` is installed and on PATH.
> - **Linux:** use your's system package manager (ex, `sudo pacman -S dfu-util`)
> - **Windows:** `scoop install dfu-util` (via [scoop](https://scoop.sh/)), or download from [dfu-util.sourceforge.net](https://dfu-util.sourceforge.net/) and add to PATH
> - **macOS:** `brew install dfu-util`
>
> The web app flasher does not require `dfu-util`, but on Windows you need to install additional drivers. You can download the driver installer from [QMK Toolbox](https://github.com/qmk/qmk_toolbox/blob/master/windows/QMK%20Toolbox/Resources/qmk_driver_installer.exe) repo.
> 

### Option 2: Manual Flash (dfu-util)

Use this for a first-time flash from stock firmware, or any time the GUI flasher is not available:

1. Enter bootloader mode:
   - Hold **Esc** while plugging in the USB cable, OR
   - Press the **reset button** under the spacebar

2. Flash with dfu-util:
   ```bash
   # STM32 (most Keychron keyboards)
   dfu-util -a 0 -d 0483:DF11 -s 0x8000000:leave -D <firmware.bin>
   ```

3. The keyboard reboots automatically after flashing (`:leave` flag).

---

## Troubleshooting

### Keyboard not detected in Vial

1. Ensure you're using the Vial firmware (not stock QMK or VIA)
2. Check that the keyboard is in wired mode (not Bluetooth)
3. On Linux, add udev rules:
   ```bash
   export USER_GID=`id -g`; sudo --preserve-env=USER_GID sh -c 'echo "KERNEL==\"hidraw*\", SUBSYSTEM==\"hidraw\", ATTRS{serial}==\"*vial:f64c2b3c*\", MODE=\"0660\", GROUP=\"$USER_GID\", TAG+=\"uaccess\", TAG+=\"udev-acl\"" > /etc/udev/rules.d/59-vial.rules && udevadm control --reload && udevadm trigger'
   ```

### Keychron tabs not showing

The Keychron-specific tabs only appear when:
1. Using a compatible GUI (Pipette or custom vial-gui fork)
2. Keyboard firmware has Keychron features enabled
3. Keyboard supports the specific feature (e.g., Analog Matrix only on HE keyboards)

### RGB not working correctly

1. Ensure VialRGB is enabled in firmware
2. Check that the keyboard declares `"lighting": "vialrgb"` in vial.json
3. Use a compatible GUI (Pipette or custom vial-gui fork) - stock Vial GUI may not support all effects

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
- [Pipette](https://github.com/darakuneko/pipette-desktop) - Modern Electron-based GUI
- Community contributors
- Claude Opus for fixing some annoying stuff that for some reason just stopped working on it's own

---
