# Keychron Vial

[Vial](https://get.vial.today/) firmware and configurators for Keychron and Lemokey keyboards.

Stock Keychron firmware works with VIA and the [Keychron Launcher](https://launcher.keychron.com). That covers remapping, lighting, SOCD and Hall Effect settings, but not Vial's tap dance, combos, key overrides or desktop apps. Wired-only Keychron boards can run upstream `vial-qmk`. Wireless boards, and anyone who wants the Keychron settings in Vial (RGB, Snap Click, Hall Effect, editing over the 2.4 GHz receiver), need the firmware from this project.

## Before you start

- **Time:** 15 to 30 minutes if your keyboard is on the supported list and you are comfortable with a terminal. Longer if you are setting up QMK or WSL for the first time.
- **Going back:** you can flash Keychron's stock firmware again at any time.
- **Risk:** low. Don't unplug the keyboard while it is flashing.
- **Skills:** you don't need to know QMK or C. Building the firmware is a few commands; everything else is done in a GUI.

> **Disclaimer:** this is a community project, not affiliated with or endorsed by Keychron. You flash custom firmware at your own risk. If you are unsure about something, open an issue before flashing.

| To... | Go to |
|---|---|
| Flash your keyboard | [Getting Started](#getting-started) |
| Check whether your keyboard is supported | [Supported Keyboards](#supported-keyboards) |
| Choose a configurator | [Configurators](#configurators) |
| Fix a problem | [Troubleshooting](#troubleshooting), [FAQ](#faq) |

---

## Firmware (`vial-qmk`)

[tymon3310/vial-qmk](https://github.com/tymon3310/vial-qmk) is [vial-kb/vial-qmk](https://github.com/vial-kb/vial-qmk) merged with [Keychron/qmk_firmware](https://github.com/Keychron/qmk_firmware):

- **From Vial:** live remapping, the unlock flow, macros, tap dance, combos, key overrides.
- **From Keychron:** matrix drivers, Bluetooth and 2.4 GHz, RGB, Hall Effect sensing.
- **Added here:** Vial over the Keychron Link 2.4 GHz receiver, which upstream Vial can't use.

| Branch | Base | Status | Use it if |
|---|---|---|---|
| [`vial-updated-keychron`](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron) | [2025q3](https://github.com/Keychron/qmk_firmware/tree/2025q3) | **Active, default** | Your keyboard is listed under this branch |
| [`vial-keychron`](https://github.com/tymon3310/vial-qmk/tree/vial-keychron) | [wls_2025q1](https://github.com/keychron/qmk_firmware/tree/wls_2025q1) | Legacy | Your keyboard is only listed under this branch |

Keyboards move to `vial-updated-keychron` as they are ported. The legacy branch does not follow Keychron's later `wls_2025q1` changes; it only changes through community pull requests.

---

## Configurators

All four support the Keychron features: RGB, Snap Click (SOCD), Hall Effect, the 2.4 GHz receiver and firmware flashing. See the [feature list](docs/features.md).

| Configurator | Platform | Built with | Notes |
|---|---|---|---|
| [**Pipette Desktop**](https://github.com/tymon3310/pipette-desktop) | Linux, Windows, macOS | Tauri, TypeScript, React | Recommended. Adds typing statistics and Pipette Hub for sharing layouts |
| [**Pipette Web**](https://pipette.tymon3310.dev) | Chromium browsers | React, WebHID | No install. Data stays in the browser; Cloud Sync and Hub uploads need the desktop app |
| [**vial-gui**](https://github.com/tymon3310/vial-gui) | Linux, Windows, macOS | Python, Qt | The classic Vial app |
| [**Vial Web**](https://vial.tymon3310.dev) | Chromium browsers | WebAssembly, WebHID | vial-gui in the browser ([source](https://github.com/tymon3310/vial-web)) |

---

## Supported Keyboards

Every keyboard below works with any Vial app once flashed. The Keychron settings (RGB, Snap Click, Hall Effect, 2.4 GHz receiver) only appear in the configurators above, not in upstream Vial.

**Jump to:** [Q Series](#q-series) · [Q Pro](#q-pro-series) · [Q Max](#q-max-series) · [Q HE](#q-he-series) · [V Series](#v-series) · [V Max](#v-max-series) · [V 8K](#v-8k-series) · [K Pro](#k-pro-series) · [K Max](#k-max-series) · [K HE](#k-he-series) · [K Version](#k-version-series) · [C Pro](#c-pro-series) · [C Pro V2](#c-pro-v2-series) · [C Pro 8K](#c-pro-8k-series) · [R Series](#r-series) · [Other](#other) · [Lemokey](#lemokey-keychron-sub-brand)

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
| [Q0 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/q0_he) | encoder | vial-updated-keychron |
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

### V 8K Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [V1 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v1_8k) | ansi_encoder | vial-updated-keychron |
| [V3 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v3_8k) | ansi_encoder, iso_encoder | vial-updated-keychron |
| [V6 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/v6_8k) | ansi_encoder, iso_encoder | vial-updated-keychron |

### K Pro Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [K1 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k1_pro) | iso/rgb | vial-updated-keychron |
| [K2 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k2_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K3 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k3_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K4 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k4_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white | vial-keychron |
| [K5 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k5_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K6 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k6_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb | vial-keychron |
| [K7 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k7_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white | vial-keychron |
| [K8 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k8_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K9 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k9_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K10 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k10_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-keychron |
| [K11 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k11_pro) | ansi/rgb, ansi/white, ansi_encoder/rgb, ansi_encoder/white | vial-keychron |
| [K12 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k12_pro) | ansi/rgb, ansi/white | vial-keychron |
| [K13 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k13_pro) | ansi/rgb, ansi/white, iso/rgb, iso/white | vial-keychron |
| [K14 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k14_pro) | ansi/rgb, ansi/white | vial-keychron |
| [K15 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k15_pro) | ansi_encoder/rgb, ansi_encoder/white | vial-keychron |
| [K17 PRO](https://github.com/tymon3310/vial-qmk/tree/vial-keychron/keyboards/keychron/k17_pro) | ansi_encoder/rgb, ansi_encoder/white, iso_encoder/rgb, iso_encoder/white | vial-keychron |

### K Max Series

| Keyboard | Layouts | Branch |
|---|---|---|
| [K0 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k0_max) | default | vial-updated-keychron |
| [K1 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k1_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K2 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k2_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K3 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k3_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K4 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k4_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K5 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k5_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white, jis_v2/rgb, jis_v2/white | vial-updated-keychron |
| [K7 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k7_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white, jis_v2/rgb, jis_v2/white | vial-updated-keychron |
| [K8 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k8_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K9 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k9_max) | ansi/rgb, ansi/white | vial-updated-keychron |
| [K10 MAX](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k10_max) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K10 MAX V2](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k10_max_v2) | ansi_rgb | vial-updated-keychron |
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
| [K1 VERSION 6](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k1_version_6) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K2 VERSION 3](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k2_version_3) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K3 VERSION 3](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k3_version_3) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |
| [K4 VERSION 3](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/k4_version_3) | ansi/rgb, ansi/white, iso/rgb, iso/white, jis/rgb, jis/white | vial-updated-keychron |

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
| [J5](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/j5) | ansi, ansi_encoder | vial-updated-keychron |
| [S1](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/s1) | ansi/rgb, ansi/white | vial-updated-keychron |
| [X0](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/x0) | red | vial-updated-keychron |
| [C100 8K](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/keychron/c100_8k) | --- | vial-updated-keychron |

### Lemokey (Keychron Sub-brand)

| Keyboard | Layouts | Branch |
|---|---|---|
| [L1 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/lemokey/l1_he) | ansi | vial-updated-keychron |
| [P1 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/lemokey/p1_he) | ansi, iso, jis | vial-updated-keychron |
| [P2 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/lemokey/p2_he) | ansi | vial-updated-keychron |
| [P3 HE](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron/keyboards/lemokey/p3_he) | ansi | vial-updated-keychron |

---
---

## Getting Started

### Step 1: Flash the firmware

Stock firmware doesn't support Vial, so this step is required.

> **Windows:** building QMK firmware needs [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) with a Linux distribution such as Ubuntu. The commands below are for a Linux or macOS shell.

Install the QMK CLI and download the firmware:

```bash
pipx install qmk   # or install qmk from your package manager
qmk setup https://github.com/tymon3310/vial-qmk.git -b vial-updated-keychron   # use -b vial-keychron for legacy boards
```

Build and flash, for example a V5 Max ANSI Encoder:

```bash
qmk flash -kb keychron/v5_max/ansi_encoder -km vial
```

To only build the `.bin`, use `qmk compile` instead of `qmk flash`.

If `qmk flash` doesn't work:

- **QMK Toolbox:** download it [here](https://github.com/tzarc/qmk_toolbox/releases/tag/beta-xplat) and open your `.bin`. Put the keyboard into bootloader mode (hold `Esc` while plugging it in, or press the reset button under the space bar), then click Flash.
- **`dfu-util`:**
  ```bash
  dfu-util -a 0 -d 0483:DF11 -s 0x8000000:leave -D <firmware.bin>
  ```

### Step 2: Install a configurator

**Pipette Desktop (recommended):** download it from the [releases page](https://github.com/tymon3310/pipette-desktop/releases/latest) (Linux AppImage, Windows, macOS).

On Arch Linux:
```bash
yay -S pipette-desktop-keychron-bin   # release build
yay -S pipette-desktop-keychron-git   # built from source
```

See the [Pipette README](https://github.com/tymon3310/pipette-desktop#development) to build it yourself.

**vial-gui:** download it from the [releases page](https://github.com/tymon3310/vial-gui/releases/latest), or on Arch Linux:

```bash
yay -S vial-keychron-bin
```

To run it from source:

```bash
git clone --branch vial-keychron https://github.com/tymon3310/vial-gui.git
cd vial-gui
uv venv .venv --python 3.12
source .venv/bin/activate
uv pip install -r requirements.txt
python src/main/python/main.py
```

**Pipette Web:** open [pipette.tymon3310.dev](https://pipette.tymon3310.dev) in a Chromium browser, click Connect and pick your keyboard.

**Vial Web:** open [vial.tymon3310.dev](https://vial.tymon3310.dev) and click Start Vial.

---

## Updating the firmware

Once the keyboard runs this firmware, you can update it from the configurator:

- **Pipette (Desktop or Web):** System tab → **Keychron Flasher** → choose the new `.bin` → Flash.
- **vial-gui and Vial Web:** **DFU Firmware Updater** tab → choose the `.bin` → Flash.

Both back up your layout before flashing and restore it afterwards.

The desktop apps need `dfu-util`:
- Linux: `sudo pacman -S dfu-util` or `sudo apt install dfu-util`
- macOS: `brew install dfu-util`
- Windows: `scoop install dfu-util`. To flash from the browser, install the STM32 DFU driver with the [QMK driver installer](https://github.com/qmk/qmk_toolbox/blob/master/windows/QMK%20Toolbox/Resources/qmk_driver_installer.exe).

---

## Troubleshooting

**The keyboard doesn't show up in Vial.** Check these:

1. The keyboard still runs stock firmware. Vial only sees it after you flash this firmware (or upstream `vial-qmk` on a wired-only board).
2. The keyboard's mode switch isn't on **Cable**, and it isn't connected through the Keychron Link **2.4 GHz** receiver. Configuration over Bluetooth isn't supported.
3. On Linux, your user can't access the device. Add a udev rule:
   ```bash
   export USER_GID=`id -g`; sudo --preserve-env=USER_GID sh -c 'echo "KERNEL==\"hidraw*\", SUBSYSTEM==\"hidraw\", ATTRS{serial}==\"*vial:f64c2b3c*\", MODE=\"0660\", GROUP=\"$USER_GID\", TAG+=\"uaccess\", TAG+=\"udev-acl\"" > /etc/udev/rules.d/59-vial.rules && udevadm control --reload && udevadm trigger'
   ```

**The Keychron settings are missing.** You need one of the configurators above (not upstream Vial) and a keyboard flashed with this firmware. The Hall Effect settings only appear on Hall Effect keyboards.

---

## FAQ

**Can I go back to stock firmware?**
Yes. Download the stock `.bin` from Keychron and flash it with QMK Toolbox or `dfu-util`. Your Vial keymaps and macros are erased.

**Will I keep my current keymap?**
No. The stock (VIA) keymap isn't carried over, so you set it up again in Vial. Later updates through the configurator keep your Vial layout.

**Does this void my warranty?**
Keychron publishes its firmware source, and this firmware is built from it, but Keychron doesn't support it. Ask Keychron if you need a definite answer.

**Do I need to know how to code?**
No. You copy a few commands to build the firmware; everything else is done in a GUI.

**My keyboard isn't listed. Can I use this?**
Not yet. Ports are welcome; see [Contributing](#contributing).

---

## Contributing

Pull requests are welcome, both new keyboard ports and fixes.

1. Fork the repo you want to change ([vial-qmk](https://github.com/tymon3310/vial-qmk), [pipette-desktop](https://github.com/tymon3310/pipette-desktop) or [vial-gui](https://github.com/tymon3310/vial-gui)).
2. Make your changes on a branch.
3. Open a pull request.

**Porting a keyboard to `vial-qmk`:**
1. Next to the board's `keymaps/keychron/`, create `keymaps/vial/` with `keymap.c`, `config.h`, `vial.json` and `rules.mk`.
2. Give the board its own `VIAL_KEYBOARD_UID` in `config.h`: `python3 util/vial_generate_keyboard_uid.py`.
3. Build it, and test the key matrix on the keyboard before opening the pull request.

---

## License

- `vial-qmk`: GPL-2.0 (from QMK)
- `vial-gui`: GPL-2.0 (from Vial)
- `pipette-desktop`: GPL-3.0 (from Pipette)
- Documentation: MIT

## Credits

[QMK](https://qmk.fm/) · [Vial](https://get.vial.today/) · [Keychron](https://github.com/Keychron/qmk_firmware) · [Pipette](https://github.com/darakuneko/pipette-desktop) · everyone who has tested a keyboard or reported an issue

## Testing

Changes are tested on a **Keychron V5 Max (ANSI Encoder)**. The other keyboards follow Keychron's source and community pull requests. If something doesn't work on yours, open an issue or ask on the [Keychron Discord](https://discord.com/invite/HAYbRrTsjN).
