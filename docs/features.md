# Feature Reference

> **NOTE**: Most of the features below only work on the `vial-updated-keychron` branch of the firmware. If your keyboard is on the legacy `vial-keychron` branch, you won't see these features in the configurator.

Full details on everything the Keychron-specific configurators support. For the short version, see the [README](../README.md).

## Standard Vial stuff you'd expect

- Live keymap and layer remapping (4+ layers)
- Macros and key combos
- Tap dance and key overrides
- Core QMK settings

## Wireless 2.4GHz bridge

Configure your keyboard through the Keychron Link dongle, no cable needed. Everything works the same as wired: keymap edits, Keychron settings, all of it.

- Works with both the USB-A (`3434:D030`) and USB-C (`3434:D031`) dongles.
- The GUI finds the paired keyboard automatically.
- Plugged in via USB *and* the dongle's active? USB wins, no fighting for control.
- Needs the wireless XOR patch in the firmware — it works around some quirky buffer handling in the LKBT51 module.

## Keychron settings

> Most of features below should work almost the same as they do in the Keychron Launcher.
- **Debounce** — 1–20ms.
- **NKRO** — toggle on/off/adaptive.
- **Report rate** — 125Hz up to 8000Hz on supported boards.
- **Wireless low-power tuning** — sleep timers and power-saving behavior.

## Snap Click (SOCD)

Set up opposing key pairs (A+D, W+S, whatever) and pick how conflicts resolve: last input wins, first key wins, second key wins, or neutral.

## RGB

- 40+ onboard lighting effects.
- Per-key custom colors.
- Mixed overlays on top of active animations.
- Separate indicator colors for Caps/Num/Scroll Lock.

## Analog Matrix (Hall Effect boards only)

The deep end. Multiple saved profiles you can swap per game or workflow.

- **Global** — inherit default profile actuation.
- **Regular** — fixed actuation point, 0.1–4.0mm.
- **Rapid Trigger** — independent press/release thresholds (0.1–4.0mm) with bottom deadzone filtering.
- **Dynamic Keystroke (DKS)** — two travel zones (shallow/deep), each mapped to up to 4 actions (press, release, tap, re-press).
- **Gamepad/XInput** — assign switches to HID joystick axes or Xbox 360 inputs, with custom 4-point response curves.
- **Toggle** — key toggles held state on each press.
- **HE SOCD** — Hall Effect-specific resolution (deeper travel wins, last input wins, or neutral) across up to 20 pairs.
- **Calibration & live monitor** — real-time per-key travel depth, manual or auto-calibration on power-on.

## Firmware flashing, built in

Backs up your layout automatically, flashes the new `.bin`, restores everything. One click in Pipette Desktop or vial-gui, or via WebUSB straight from the browser.
