# VelaDial Door-Side Rotary — Single Source Test Guide

**Firmware branch:** `firmware/fix-real-ha-control-and-themeos-visuals`
**Primary file:** `esphome/door_side_rotary.yaml`
**Target entity:** `light.bedroom_group` (5 Surplife RGBCW bulbs)

---

## Prerequisites

Before flashing, ensure:

1. `light.bedroom_group` exists in Home Assistant (Settings → Helpers → Light Group → "Bedroom Group" with all 5 bulbs)
2. All 5 Surplife bulbs are members of the group
3. Manual toggle works: Developer Tools → Actions → `light.toggle` → `light.bedroom_group`
4. **CRITICAL**: ESPHome device has "Allow the device to perform Home Assistant actions" enabled:
   - Settings → Devices & Services → ESPHome → CONFIGURE (next to veladial-door-rotary) → Check the box → Submit

---

## Flash Command

```bash
esphome run esphome/door_side_rotary.yaml
```

---

## 12-Step Test Sequence

### Step 1: Manual HA Toggle (baseline)

In Home Assistant Developer Tools → Actions:

```yaml
action: light.toggle
target:
  entity_id: light.bedroom_group
```

**Expected:** Bulbs physically toggle ON/OFF.
**If this fails:** Problem is HA/bulb integration, not VelaDial firmware.

---

### Step 2: HA Diagnostic Button — Turn ON

In Home Assistant → VelaDial Door Rotary device → press **"VelaDial Turn ON Bedroom Group"** button.

**Expected:**
- ESPHome log: `TURN ON: HA confirmed SUCCESS`
- Screen shows: `ON OK` in green
- Bulbs physically turn ON

**If bulbs don't turn on but log says SUCCESS:** HA permission issue. Go back to Prerequisites step 4.
**If log says ERROR:** HA denied the action. Enable "Allow device to perform Home Assistant actions."

---

### Step 3: Confirm Bulbs ON

Visually verify all 5 bedroom bulbs are illuminated.

---

### Step 4: HA Diagnostic Button — Turn OFF

Press **"VelaDial Turn OFF Bedroom Group"** button.

**Expected:**
- ESPHome log: `TURN OFF: HA confirmed SUCCESS`
- Screen shows: `OFF OK` in green
- Bulbs physically turn OFF

---

### Step 5: Confirm Bulbs OFF

Visually verify all 5 bedroom bulbs are off.

---

### Step 6: HA Diagnostic Button — Toggle

Press **"VelaDial Test Toggle Bedroom Group"** button.

**Expected:**
- ESPHome log: `TOGGLE BEDROOM GROUP: HA confirmed SUCCESS`
- Screen shows: `OK` in green, then state updates
- Bulbs physically toggle

---

### Step 7: Confirm Toggle Works

Verify bulbs changed state from Step 5.

---

### Step 8: Knob Short Press (Power Page)

Only test this AFTER Steps 2-7 pass. On the VelaDial device:

1. Ensure you're on the Power page (page 0, shows "OFF" or "ON")
2. Short-press the rotary knob (<600ms)

**Expected:**
- ESPHome log: `KNOB SHORT PRESS POWER: toggle bedroom group`
- ESPHome log: `TOGGLE BEDROOM GROUP: HA confirmed SUCCESS`
- Screen shows: `Sending...` → `OK` → state updates
- Bulbs physically toggle

**If this fails but Step 6 passed:** Input routing issue (report logs).

---

### Step 9: Touch Tap (Power Page)

Tap the center of the round display (the power button area).

**Expected:**
- ESPHome log: `TOUCH POWER TAP: toggle bedroom group`
- Same toggle behavior as Step 8
- Bulbs physically toggle

**If this fails but Step 8 passed:** Touch target area issue (report touch coordinates from log).

---

### Step 10: Long-Press Theme Selector

Long-press the rotary knob (>1.5 seconds).

**Expected:**
- ESPHome log: `THEME SELECTOR: activated`
- Screen transitions to full-screen Theme Selector page:
  - "THEME SELECT" header
  - "01/20" index
  - Large color-filled circle
  - Theme name inside circle
  - Color ring around circle
  - "Rotate Browse | Press Apply" instruction

---

### Step 11: Rotate Through Themes

Rotate the knob clockwise/counterclockwise while in Theme Selector.

**Expected:**
- Index changes: "02/20", "03/20", etc.
- Circle color changes per theme
- Ring color changes per theme
- Theme name updates: "SmartKnob", "Power", "Simple", etc.
- Each rotation is immediately visible

---

### Step 12: Apply Theme and Verify Power Page

Press the knob to apply a theme (e.g., select "02 SmartKnob").

**Expected:**
- "APPLIED" shows briefly in green
- Screen transitions back to Power page
- Power page now shows:
  - Theme-specific arc/ring style (e.g., thick partial arc for SmartKnob)
  - Theme-specific color (orange for SmartKnob)
  - Motif text (e.g., "~ ARC ~" for SmartKnob)
  - "T02 SmartKnob" at bottom in theme color
  - "Bedroom Group" target label
  - "Press/Tap Toggle" control hint

---

## Troubleshooting

### "HA DENIED" on screen / "HA returned ERROR" in logs

The device doesn't have permission to call HA actions.

**Fix:** Settings → Devices & Services → ESPHome → CONFIGURE (next to veladial-door-rotary) → Enable "Allow the device to perform Home Assistant actions" → Submit

### "WAIT HA" on screen

API not connected. Check:
- Wi-Fi connection (device should show IP in logs)
- ESPHome integration in HA (device should be "Online")
- Encryption key matches between firmware and HA

### "Sending..." stays forever (no OK/ERROR)

The `capture_response` callback didn't fire. This may indicate:
- ESPHome version mismatch (needs 2024.6+ for capture_response)
- Network timeout between device and HA

### Knob press toggles but touch doesn't

Touch coordinates may be offset. Check logs for `TOUCH: x=, y=` values.
The power button is a 120x120px circle at center (120,120). Touch must land within ~60px of center.

### Theme changes color but no structural difference

Only 5 theme groups have structural layout changes:
- Group A (T01,T04,T07,T10): Minimal — thin ring, small button
- Group B (T02,T08,T17,T18): SmartKnob — thick partial arc + secondary arc visible
- Group C (T03,T11,T12): Power — no outer arc, inner glow ring, thick button border
- Group D (T06): Night — very thin border, deep red
- Group E (T05,T13-T16,T19,T20): Eclipse — both secondary arc + inner glow ring visible

All 20 themes have unique: color, motif text, theme name, arc color, border color.

---

## What Each Diagnostic Button Tests

| Button | Tests | Isolates |
|--------|-------|----------|
| VelaDial Turn ON Bedroom Group | `light.turn_on` syntax | HA permission + entity existence |
| VelaDial Turn OFF Bedroom Group | `light.turn_off` syntax | Same as above |
| VelaDial Test Toggle Bedroom Group | `light.toggle` syntax | Toggle specifically |
| VelaDial Brightness 50 Bedroom Group | `light.turn_on` + `brightness_pct` | Data parameter passing |

If ALL 4 buttons fail → HA permission not enabled.
If buttons work but knob/touch don't → Input routing bug in firmware.
If buttons work AND knob/touch work → Full success.
