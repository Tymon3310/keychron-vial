# Keychron Vial

Vial firmware and GUI support for Keychron keyboards.

This project brings full [Vial](https://get.vial.today/) support to Keychron keyboards, including advanced features like Keychron-specific settings, RGB control, and Hall Effect (HE) analog configuration.

## Quick Links

| Resource | Description |
|----------|-------------|
| [Vial Web (Keychron Edition)](https://vial.tymon3310.dev) | Browser-based configurator |
| [vial-qmk Firmware](https://github.com/tymon3310/vial-qmk) | QMK firmware with Vial + Keychron support |
| [vial-gui Desktop App](https://github.com/tymon3310/vial-gui) | Desktop configurator with Keychron tabs |
| [Keyboard Definitions](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron) | Vial keymaps |

---

## Repositories

### vial-qmk (Firmware)

Fork of [vial-kb/vial-qmk](https://github.com/vial-kb/vial-qmk) with Keychron keyboard ports.

| Branch | Base | Status | Description |
|--------|------|--------|-------------|
| [`vial-keychron`](https://github.com/tymon3310/vial-qmk/tree/vial-keychron) | [wls_2025q1](https://github.com/keychron/qmk_firmware/tree/wls_2025q1) | Stable | Older QMK base. Note: Does not complitly supports my gui |
| [`vial-updated-keychron`](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron) | [2025q3](https://github.com/Keychron/qmk_firmware/tree/2025q3) | **Active** | Newer QMK base. Note: Complitly supports my gui |


> **Note:** The `vial-updated-keychron` branch now includes full vial keymaps for all Q HE, K HE, K8 Pro, K9 Max, V1 Max, V6 Max, C Pro series, and X0 keyboards.

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
| [Q1 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q1_pro) | ansi_knob, iso_knob, jis_encoder | vial-keychron |
| [Q2 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q2_pro) | ansi_encoder, iso_encoder | vial-keychron |
| [Q3 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q3_pro) | ansi_encoder, ansi_encoder_se, iso_encoder, iso_encoder_se, jis_encoder_se | vial-keychron |
| [Q4 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q4_pro) | ansi, iso | vial-keychron |
| [Q5 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q5_pro) | ansi_encoder, iso_encoder | vial-keychron |
| [Q6 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q6_pro) | ansi_encoder, iso_encoder | vial-keychron |
| [Q8 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q8_pro) | ansi_encoder, iso_encoder | vial-keychron |
| [Q10 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q10_pro) | ansi_encoder, iso_encoder | vial-keychron |
| [Q13 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q13_pro) | ansi_encoder, iso_encoder | vial-keychron |
| [Q14 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q14_pro) | ansi_encoder, iso_encoder | vial-keychron |

### Q Max Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [Q0 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q0_max) | encoder | vial-keychron |
| [Q1 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q1_max) | ansi_encoder, iso_encoder, jis_encoder | vial-keychron |
| [Q2 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q2_max) | ansi_encoder, iso_encoder | vial-keychron |
| [Q3 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q3_max) | ansi_encoder, iso_encoder | vial-keychron |
| [Q5 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q5_max) | ansi_encoder, iso_encoder | vial-keychron |
| [Q6 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q6_max) | ansi_encoder, iso_encoder | vial-keychron |
| [Q8 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q8_max) | ansi_encoder | vial-keychron |
| [Q10 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q10_max) | ansi_encoder, iso_encoder | vial-keychron |
| [Q12 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q12_max) | ansi_encoder | vial-keychron |
| [Q13 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q13_max) | ansi_encoder | vial-keychron |
| [Q14 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q14_max) | ansi_encoder | vial-keychron |
| [Q15 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q15_max) | ansi_encoder | vial-keychron |
| [Q60 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q60_max) | ansi | vial-keychron |
| [Q65 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/q65_max) | ansi_encoder | vial-keychron |

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
| [V7](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v7) | ansi, iso | vial-updated-keychron |
| [V8](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v8) | ansi, ansi_encoder, iso, iso_encoder | vial-updated-keychron |
| [V10](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v10) | ansi_encoder, iso_encoder | vial-updated-keychron |

### V Max Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [V1 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v1_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [V2 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v2_max) | ansi_encoder, iso_encoder | vial-keychron |
| [V3 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v3_max) | ansi_encoder, iso_encoder | vial-keychron |
| [V4 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v4_max) | ansi | vial-keychron |
| [V5 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v5_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [V6 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v6_max) | ansi_encoder, iso_encoder, jis_encoder | vial-updated-keychron |
| [V8 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v8_max) | ansi_encoder | vial-keychron |
| [V10 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/v10_max) | ansi_encoder | vial-keychron |

### K Pro Series

| Keyboard | Layouts | Branch |
|----------|---------|--------|
| [K1 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k1_pro) | iso/rgb | vial-keychron |
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
| [K1 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k1_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K2 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k2_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K3 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k3_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K5 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k5_max) | ansi/rgb, ansi/white, iso/rgb, iso/white | vial-keychron |
| [K7 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k7_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K8 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k8_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K9 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k9_max) | ansi/rgb, ansi/white | vial-updated-keychron |
| [K10 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k10_max) | ansi/rgb, ansi/white, iso/rgb, iso/white | vial-keychron |
| [K11 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k11_max) | ansi_encoder/rgb, ansi_encoder/white, iso_encoder/rgb, iso_encoder/white | vial-keychron |
| [K13 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k13_max) | ansi/rgb, ansi/white, iso/rgb, iso/white | vial-keychron |
| [K15 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k15_max) | ansi_encoder/rgb, ansi_encoder/white, iso_encoder/rgb, iso_encoder/white | vial-keychron |
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
| [K3 VERSION 3](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k3_version_3) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |

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
| [BLUETOOTH](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/bluetooth) | Unknown | vial-keychron |
| [S1](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/s1) | ansi/rgb, ansi/white | vial-updated-keychron |
| [X0](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/x0) | red | vial-updated-keychron |

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

### Option 3: Build Custom Firmware

1. Clone the vial-qmk repository:
   ```bash
   git clone --branch vial-keychron https://github.com/tymon3310/vial-qmk.git OR
   git clone --branch vial-updated-keychron https://github.com/tymon3310/vial-qmk.git
   cd vial-qmk
   ```

2. Set up QMK:
   ```bash
   qmk git submodule
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
- Claude Opus for fixing some annoying stuff that for some reason just stopped working on it's own

---
