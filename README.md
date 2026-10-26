# Keychron Vial

Stock Keychron firmware locks you into VIA and their web launcher (`launcher.keychron.com`) for anything interesting — SOCD, per-key RGB, Hall Effect tuning. Fine if that's all you want, but it means no real Vial: no tap dance, no combos, no dynamic macros, no offline desktop GUI.

Most stock Keychron keyboards can't run Vial natively — the exception is wired-only models, which can actually run upstream `vial-qmk` as-is. Everything else (wireless keyboards, and anyone who wants the Keychron-specific tabs — RGB, SOCD, Hall Effect, the wireless bridge) needs this fork.

This project gets you actual Vial on Keychron and Lemokey keyboards — full remapping, macros, tap dance, combos, 2.4 GHz wireless bridge support, RGB, and Hall Effect switch tuning, all together.

## What to expect

- **Time:** 15–30 minutes if your keyboard's already on the supported list and you're comfortable with a terminal. Longer if you're setting up QMK/WSL for the first time.
- **Reversible:** yes. You can flash back to stock Keychron firmware later if you want the official launcher back — nothing here is one-way.
- **Risk:** low, if you follow the flashing steps as written. The main way to actually cause trouble is interrupting a flash mid-write — don't unplug during that step.
- **Not required:** you don't need to know QMK or C. Compiling firmware is copy-paste; everyday use is all GUI.

> **Disclaimer:** This is a community project, not affiliated with or endorsed by Keychron. Flashing is straightforward and reversible, but as with any firmware change, it's done at your own risk — I'm not responsible for any damage to your keyboard. If in doubt, ask in an issue before flashing.

If that sounds fine, here's where to go:

| If you want to... | Use this |
|---|---|
| **Just get flashing** | Skip to [Getting Started](#getting-started) |
| **Check if your keyboard is supported** | [Supported Keyboards](#supported-keyboards) |
| **See what each configurator does** | [Configurators](#configurators) |
| **Something's not working** | [Troubleshooting](#troubleshooting) / [FAQ](#faq) |

---

## Firmware (`vial-qmk`)

A hybrid fork — [vial-kb/vial-qmk](https://github.com/vial-kb/vial-qmk) merged with [Keychron/qmk_firmware](https://github.com/Keychron/qmk_firmware). It combines:

- **Vial's runtime engine**: live remapping, unlock flow, macros, tap dance, combos.
- **Keychron's hardware layer**: matrix drivers, wireless subsystems, RGB, Hall Effect sensing.
- **A custom wireless protocol patch**: tunnels full Vial configuration over the 2.4 GHz Keychron Link dongle, which upstream Vial has no idea how to talk to.

| Branch | Base | Status | Use it if... |
|---|---|---|---|
| [`vial-updated-keychron`](https://github.com/tymon3310/vial-qmk/tree/vial-updated-keychron) | [2025q3](https://github.com/Keychron/qmk_firmware/tree/2025q3) | **Active, default** | Your keyboard is on the list below under this branch |
| [`vial-keychron`](https://github.com/tymon3310/vial-qmk/tree/vial-keychron) | [wls_2025q1](https://github.com/keychron/qmk_firmware/tree/wls_2025q1) | Legacy | Your keyboard hasn't been ported to the newer base yet |

Keyboards migrate from the legacy branch to the updated one as Keychron pushes changes upstream — keeps things from turning into a merge-conflict nightmare. That said, the legacy branch itself isn't kept in sync with ongoing upstream wls_2025q1 changes after its initial fork point. The only time it gets updated is with community pull request.

---

## Configurators

All of these talk the same protocol and support the same Keychron-specific features (RGB, SOCD, Hall Effect tuning, wireless bridge, built-in flashing — details on the [full feature list](docs/features.md)). Pick based on how you like to work, not which one has more buttons:

| Configurator | Platform | Stack | Notes |
|---|---|---|---|
| [**Pipette Desktop**](https://github.com/tymon3310/pipette-desktop) | Linux, Windows, macOS | TypeScript/Electron | Recommended — what I actually use day-to-day. Has typing analytics and Pipette Hub for sharing layouts |
| [**Pipette Web**](https://pipette.tymon3310.dev) | Chromium browsers | TypeScript/React/WebHID | Zero install, does everything Desktop does |
| [**vial-gui**](https://github.com/tymon3310/vial-gui) | Linux, Windows, macOS | Python/Qt | The OG — rock solid, same feature parity |
| [**Vial Web**](https://vial.tymon3310.dev) | Chromium browsers | WebAssembly/WebHID | Classic browser version, based on vial-gui ([source](https://github.com/tymon3310/vial-web)) |

---

## Supported Keyboards

Every keyboard below works with Vial once it's flashed. The Keychron-specific tabs (wireless bridge, RGB, SOCD, Hall Effect) only show up if you're using Pipette or the custom `vial-gui` — upstream Vial won't have them.

Covers Q / Q Pro / Q Max / Q HE, V / V Max / V 8K, K Pro / K Max / K HE / K Version, C Pro / C Pro V2 / C Pro 8K, R Series, S / X, and Lemokey HE.

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

## Getting Started

### Step 1: Flash the custom firmware

This is the part you can't skip — stock firmware just doesn't speak Vial.

> **Windows users:** you'll need [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) with a Linux distro (Ubuntu's the easy choice) to build QMK firmware. The steps below assume a Linux/macOS shell either way.

Set up QMK and pull the firmware in one go:

```bash
# Install the qmk cli (via your package manager, or pipx)
pipx install qmk
qmk setup https://github.com/tymon3310/vial-qmk.git -b vial-updated-keychron # swap the branch for vial-keychron if needed
```

Then compile and flash for your keyboard (example: V5 Max, ANSI Encoder) — this one command does both:

```bash
qmk flash -kb keychron/v5_max/ansi_encoder -km vial
```

If you just want the `.bin` without flashing yet, swap `qmk flash` for `qmk compile`.

**Flashing options, if `qmk flash` doesn't work for you:**

- **QMK Toolbox** — download it [here](https://github.com/tzarc/qmk_toolbox/releases/tag/beta-xplat), open your compiled `.bin`, put the keyboard into bootloader mode (hold `Esc` while plugging in, or hit the reset button under the spacebar), then hit Flash.
- **`dfu-util` directly:**
  ```bash
  dfu-util -a 0 -d 0483:DF11 -s 0x8000000:leave -D <firmware.bin>
  ```

### Step 2: Pick a configurator

**Pipette Desktop (recommended)** — pre-built releases on [GitHub](https://github.com/tymon3310/pipette-desktop/releases/latest) for Linux (AppImage), Windows, and macOS.

On Arch:
```bash
yay -S pipette-desktop-keychron-bin   # prebuilt
# or
yay -S pipette-desktop-keychron-git   # build from source
```

Building it yourself:
```bash
git clone --branch vial-keychron https://github.com/tymon3310/pipette-desktop.git
cd pipette-desktop
pnpm install
pnpm dev
```

**vial-gui** — same deal, Python/Qt instead. Releases [here](https://github.com/tymon3310/vial-gui/releases/latest).

```bash
yay -S vial-keychron-bin   # prebuilt (Arch)
# or
git clone --branch vial-keychron https://github.com/tymon3310/vial-gui.git
cd vial-gui
uv venv .venv --python 3.12
source .venv/bin/activate
uv pip install -r requirements.txt
python src/main/python/main.py
```

**Pipette Web** — go to [pipette.tymon3310.dev](https://pipette.tymon3310.dev) in a Chromium browser, click Connect, pick your keyboard from the WebHID prompt. That's it.

**Vial Web** — same idea, at [vial.tymon3310.dev](https://vial.tymon3310.dev), click Start Vial.

---

## Updating firmware later

Once you're on Vial firmware, updates are painless — no need to redo Step 1 from scratch.

**Pipette (Desktop or Web):** System tab → Keychron flasher (bottom) → pick your new `.bin` → Flash. It backs up and restores your layout automatically.

**vial-gui / vial-web:** DFU Firmware updater tab → pick your `.bin` → Flash. Same auto-backup behavior.

You'll need `dfu-util` installed:
- Linux: `sudo pacman -S dfu-util` (or `apt install dfu-util`)
- macOS: `brew install dfu-util`
- Windows: `scoop install dfu-util`, or install the STM32 DFU drivers from [QMK Toolbox](https://github.com/qmk/qmk_toolbox/blob/master/windows/QMK%20Toolbox/Resources/qmk_driver_installer.exe) if you're flashing via browser

---

## Troubleshooting

**Keyboard not showing up in Vial?** 90% of the time it's one of these:

1. It's still running stock firmware. Unless you're on a wired-only keyboard using pure upstream `vial-qmk`, stock only works with VIA / the Keychron web launcher — Vial won't see it at all until you flash the firmware from this fork.
2. The physical switch on the keyboard isn't set to **Cable**, or you're not connected via the Keychron Link **2.4 GHz** dongle. Bluetooth doesn't support raw HID, so it won't work for configuration either way.
3. On Linux, you probably need udev permissions:
   ```bash
   export USER_GID=`id -g`; sudo --preserve-env=USER_GID sh -c 'echo "KERNEL==\"hidraw*\", SUBSYSTEM==\"hidraw\", ATTRS{serial}==\"*vial:f64c2b3c*\", MODE=\"0660\", GROUP=\"$USER_GID\", TAG+=\"uaccess\", TAG+=\"udev-acl\"" > /etc/udev/rules.d/59-vial.rules && udevadm control --reload && udevadm trigger'
   ```

**Keychron tabs missing from the GUI?** You need all three of: a custom configurator (not upstream Vial), firmware built with Keychron extensions on, and — for the Analog Matrix tab specifically — an actual Hall Effect keyboard.

---

## FAQ

**Can I go back to stock firmware afterward?**
Yes. Flash the stock `.bin` from Keychron's site the same way you'd flash any other firmware (QMK Toolbox or `dfu-util`). You'll lose Vial-specific settings (keymaps, macros) in the process, but nothing about the hardware changes.

**Will I lose my current keymap when I flash?**
Your existing (stock/VIA) keymap doesn't carry over automatically — you'll be setting it up fresh in Vial. If you're updating an already-Vial-flashed keyboard, though, the built-in flasher backs up and restores your Vial layout automatically.

**Does this void my warranty?**
No — Keychron publishes their firmware source themselves, so flashing custom firmware built from it is normal use of that source, not some unsupported hack. This project just isn't Keychron-endorsed or officially supported by them.

**Do I need to know how to code?**
No. Compiling is copy-paste commands; day-to-day configuration is entirely GUI-based once you're set up.

**My keyboard isn't in the supported list — can I still use this?**
Not out of the box, but ports are welcome — see [Contributing](#contributing).

---

## Contributing

Want to port a new keyboard, or just fix something that's broken? Pull requests welcome.

1. Fork whichever repo you're changing ([vial-qmk](https://github.com/tymon3310/vial-qmk), [pipette-desktop](https://github.com/tymon3310/pipette-desktop), or [vial-gui](https://github.com/tymon3310/vial-gui)).
2. Branch, make your changes.
3. Open a PR.

**Adding a new keyboard port:**
1. New directory under `keyboards/keychron/<model>/` in `vial-qmk`.
2. Add a `keymaps/vial/` folder with `keymap.c`, `config.h` (needs a unique `VIAL_KEYBOARD_UID`), `vial.json`, and `rules.mk`.
3. Generate the UID: `python3 util/vial_generate_keyboard_uid.py`.
4. Compile and actually test the matrix mapping on real hardware before opening the PR.

---

## License

- `vial-qmk`: GPL-2.0 (inherited from QMK)
- `vial-gui`: GPL-2.0 (inherited from Vial)
- `pipette-desktop`: GPL-3.0 (inherited from Pipette)
- Documentation: MIT

## Credits

[QMK](https://qmk.fm/) · [Vial](https://get.vial.today/) · [Keychron](https://github.com/Keychron/qmk_firmware) · [Pipette](https://github.com/darakuneko/pipette-desktop) · everyone who's tested a keyboard or filed an issue

## A note on testing

Daily driver here is a **Keychron V5 Max (ANSI Encoder)** — that's what gets tested first on every change. Everything else in the tree is maintained against Keychron's upstream source and community PRs, so if something's off on your specific keyboard, open an issue (or ping me on the [Keychron Discord](https://discord.com/invite/HAYbRrTsjN)) rather than assuming it's just broken.

