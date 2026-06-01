# VelaDial Door-Side — Single-Source Test Guide

## Hardware

- **Device**: ELECROW CrowPanel 1.28" ESP32-S3 Rotary Touch Display (240×240)
- **Firmware**: `esphome/door_side_rotary.yaml`
- **Target entity**: `light.bedroom_group` (5 Surplife RGBCW bulbs)
- **Branch**: `firmware/fix-ha-toggle-theme-selector-v2`

---

## Prerequisites

1. Home Assistant running and accessible at `homeassistant.local:8123`
2. `light.bedroom_group` exists (Settings → Helpers → Light Group → "Bedroom Group" with all 5 bulbs)
3. ESPHome Add-on or CLI installed
4. USB-C cable connected to CrowPanel

---

## Flash Command

```bash
esphome run esphome/door_side_rotary.yaml
```

---

## 11-Step Test Sequence

### Step 1: Verify `light.bedroom_group` exists in HA

- Go to **Developer Tools → States**
- Search: `light.bedroom_group`
- Confirm it shows state `on` or `off` with 5 member entities

### Step 2: Manually toggle `light.bedroom_group` in HA

- Go to **Developer Tools → Actions**
- Action: `light.toggle`
- Target: `light.bedroom_group`
- Click **Perform action**
- Confirm bulbs physically turn on/off

### Step 3: Flash firmware

```bash
esphome run esphome/door_side_rotary.yaml
```

- Wait for boot splash ("VELADIAL") to appear
- Wait 3 seconds for splash to hide and Power page to show

### Step 4: Confirm Power page says "Bedroom Group"

- Power page must show:
  - **"Bedroom Group"** near top (target label)
  - **"OFF"** or **"ON"** in center (state)
  - **"T01 Minimal"** at bottom (theme label)
  - Ice-blue decorative ring

### Step 5: Press HA diagnostic button

- In HA, go to the VelaDial device page or Developer Tools → Actions
- Press **"VelaDial Test Toggle Bedroom Group"** button
- Confirm bulbs toggle
- Check ESPHome logs for: `HA BUTTON: VelaDial Test Toggle Bedroom Group pressed`
- Check logs for: `TOGGLE BEDROOM GROUP: command sent to light.bedroom_group`

**If this fails**: The bug is HA action syntax. Check API connection.
**If this works but Step 6/7 fail**: The bug is input handling (knob/touch).

### Step 6: Short-press knob on Power page

- Give a quick press (<600ms) on the physical knob
- Confirm bulbs toggle
- Check logs for: `KNOB SHORT PRESS POWER: toggle bedroom group`
- Check logs for: `TOGGLE BEDROOM GROUP: command sent to light.bedroom_group`
- Screen should briefly show "Toggling..." then update to ON/OFF

### Step 7: Tap Power page center (touch)

- Tap the center circular area of the screen
- Confirm bulbs toggle
- Check logs for: `TOUCH POWER TAP: toggle bedroom group`
- Check logs for: `TOGGLE BEDROOM GROUP: command sent to light.bedroom_group`

### Step 8: Long-press knob to open Theme Selector

- Press and hold knob for >1.5 seconds
- Theme Selector page should appear showing:
  - **"THEME SELECT"** header
  - **"01/20"** (or current theme number)
  - Theme name (e.g., "Minimal")
  - Large color preview circle
  - **"Rotate Browse | Press Apply"** instruction

### Step 9: Rotate knob and confirm visible changes

- Rotate knob clockwise/counterclockwise
- Confirm these change visibly with each click:
  - Theme number (01/20 → 02/20 → etc.)
  - Theme name (Minimal → SmartKnob → Power → etc.)
  - Preview circle color
  - Outer ring color

### Step 10: Press knob to apply theme — confirm NO light toggle

- Press knob while in Theme Selector
- Confirm:
  - Screen shows **"APPLIED"** in green
  - Shows applied theme name in green
  - Returns to Power page after ~800ms
  - Power page shows new theme label (e.g., "T04 Simple")
  - **Lights did NOT toggle** (critical — no fall-through)
- Check logs for: `SHORT PRESS: applying theme from selector (no fall-through)`
- Check logs do NOT show: `KNOB SHORT PRESS POWER: toggle bedroom group`

### Step 11: Reboot and confirm theme persists

- Power cycle the device (unplug USB, replug)
- After boot, confirm Power page shows the theme you selected (not T01)
- Theme index is stored with `restore_value: yes`

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| "WAIT HA" on screen | API not connected yet | Wait 5-10s after boot for WiFi+API |
| Knob press does nothing | Screen asleep | First press wakes, second press toggles |
| Touch logs but no toggle | `touch_woke_this_cycle` guard | First touch wakes, second touch toggles |
| Theme apply also toggles lights | Fall-through bug (should be fixed) | Check logs for both messages |
| "TOGGLE BEDROOM GROUP: requested" but no bulb change | API connected but entity missing | Verify `light.bedroom_group` in HA |

---

## Design Notes

- This is a **20-theme selectable foundation**
- Visual differentiation is **color/name/ring/LED** based for now
- Full per-theme rich layouts can be refined after hardware validation
- Every theme visibly changes: color, name, preview circle, LED ring color
- All themes control the same target: `light.bedroom_group`
