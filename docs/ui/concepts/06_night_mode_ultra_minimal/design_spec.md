# Concept 06: Night Mode Ultra-Minimal — Design Specification

## Gate Status

> **PASSES all gates (G1–G8).** This concept is classified as a "Layer" — it overlays on top of any primary concept.

---

## 1. Concept Overview

Night Mode Ultra-Minimal is not a standalone UI theme — it is the **sleep/dark-room state** that overlays any active concept. When the TSL2591 ambient light sensor reads below approximately 5 lux, the display enters Night Mode: backlight drops to minimum PWM, all UI elements disappear except a single small dot or dim number, and the LED ring either turns off entirely or glows at absolute minimum (3/255).

The design philosophy: at 3 AM, the last thing anyone wants is a glowing rectangle on the wall. Night Mode makes the device nearly invisible while remaining instantly accessible.

**This is the concept that remembers VelaDial lives in a bedroom.**

---

## 2. Visual Identity

| Property | Value |
|----------|-------|
| Background | Pure black (`0x000000`) |
| Primary element | Single 8px amber dot OR dim percentage value |
| Dot color | Amber (`0xFFA500`) at 30% opacity |
| Text color (dim value) | Amber (`0xFFA500`) at 30% opacity |
| Text font | Roboto 24pt |
| Backlight PWM | 10% (minimum visible) |
| Design language | Absence, stillness, near-invisibility |

---

## 3. State Matrix

Night Mode has exactly one visual state (not pages), with content determined by light state:

| Light State | Display Content | LED Ring | Backlight |
|-------------|----------------|----------|-----------|
| Lights OFF, room dark (<5 lux) | Single 8px amber dot at center | All off (0/255) | 10% PWM |
| Lights ON, room dark (<5 lux) | Dim `%` value at center, 24pt, amber | 1-2 LEDs at minimum (3/255) | 10% PWM |

### Transition Thresholds

| Transition | Trigger | Duration |
|------------|---------|----------|
| Normal UI → Night Mode | TSL2591 reads < 5 lux for 10 seconds | 2s fade |
| Night Mode → Normal UI | TSL2591 reads > 15 lux for 5 seconds | 500ms fade-in |
| Night Mode → Wake (user input) | Any touch, knob rotation, or knob press | 500ms fade-in |

Note: Hysteresis band (5 lux entry, 15 lux exit) prevents oscillation at threshold.

---

## 4. Layout Architecture

```
┌─────────────────────────┐
│                         │
│                         │
│                         │
│                         │
│           ●             │  ← Single 8px amber dot (lights OFF)
│                         │     OR dim "50%" (lights ON)
│                         │
│                         │
│                         │
│                         │
└─────────────────────────┘
   Backlight at 10% PWM
   Everything else: invisible
```

### Widget Positioning

| Widget | Position | Size |
|--------|----------|------|
| Night dot | align: CENTER | 8x8px, radius: 4 (circle) |
| Night value label | align: CENTER | auto-width, 24pt |

Only ONE of these is visible at any time (dot when lights off, value when lights on).

---

## 5. Widget Inventory (Minimal)

| Widget | Type | ID | Visibility |
|--------|------|----|-----------|
| Night dot | `obj` | `night_dot` | Visible when lights OFF in Night Mode |
| Night value | `label` | `night_value` | Visible when lights ON in Night Mode |
| Night Mode page | `page` | `night_page` | Shown when Night Mode active |

**Total: 2 visible widgets maximum** (only 1 at a time). This is the most minimal LVGL page possible.

---

## 6. Interaction Model

### Night Mode Inputs

| Input | Action |
|-------|--------|
| Touch (anywhere) | Wake to full UI |
| Knob CW | Wake to full UI |
| Knob CCW | Wake to full UI |
| Knob press | Wake to full UI |

**There is NO interaction within Night Mode itself.** No page navigation, no brightness adjustment, no preset selection. The user MUST wake the display first. This is deliberate: in a dark bedroom, any accidental touch should produce minimum visual disruption.

### Wake Behavior

1. User provides input → Night Mode fades out (500ms)
2. Full UI of active concept fades in (500ms)
3. Normal 60-second inactivity timer starts
4. After timeout → if still dark → return to Night Mode

---

## 7. Ambient Light Sensor Integration

### TSL2591 Configuration

| Parameter | Value |
|-----------|-------|
| I2C address | 0x29 (default) |
| Integration time | 100ms |
| Gain | Medium (25x) |
| Update interval | 5s |
| Dark threshold (enter Night Mode) | < 5 lux |
| Light threshold (exit Night Mode) | > 15 lux |
| Debounce (enter) | 10 seconds below threshold |
| Debounce (exit) | 5 seconds above threshold |

### Hysteresis Logic

```
if (current_lux < 5 && time_below > 10s && !night_mode_active):
    enter_night_mode()
elif (current_lux > 15 && time_above > 5s && night_mode_active):
    exit_night_mode()
```

---

## 8. Backlight Control

| State | Backlight PWM | Perceived Brightness |
|-------|---------------|---------------------|
| Full UI (normal) | 80% | Comfortable reading |
| Night Mode | 10% | Barely visible |
| Full sleep (no Night Mode) | 0% | Completely off |

The 10% PWM level is chosen as the minimum at which the GC9A01A display shows any visible content. Below this, the display appears completely black regardless of pixel values.

---

## 9. LED Ring Behavior

| State | LED Configuration |
|-------|-------------------|
| Night Mode, lights OFF | All 5 LEDs off (0, 0, 0) |
| Night Mode, lights ON | LED 0 only at (3, 2, 0) — barely visible warm amber |
| Wake transition | LEDs remain off until full UI renders |

---

## 10. Animation Specification

| Animation | Duration | Easing | Description |
|-----------|----------|--------|-------------|
| Enter Night Mode | 2000ms | linear | All widgets fade to opa 0; dot/value fades to opa 77 (30%) |
| Exit Night Mode (sensor) | 500ms | ease-out | Dot/value fades out; full UI fades in |
| Exit Night Mode (user wake) | 500ms | ease-out | Same as sensor exit |
| Backlight dim (enter) | 2000ms | linear | PWM ramps from current to 10% |
| Backlight brighten (exit) | 500ms | ease-out | PWM ramps from 10% to 80% |

**No pulsing, no breathing, no animation within Night Mode.** Stillness is the design intent.

---

## 11. Prototype Scope (v1 Locked)

For the prototype YAML, the following v1 scope is implemented:

| Feature | Included | Notes |
|---------|----------|-------|
| TSL2591 sensor reading | Yes | Simulated via template sensor for compile |
| Backlight reduction | Yes | LEDC PWM to 10% |
| Minimal center dot | Yes | 8px amber obj |
| Minimal center value | Yes | 24pt amber label |
| Wake-only-first on all inputs | Yes | Inherent to Night Mode |
| LED ring off/minimum | Yes | |
| Slow fade transition | Partial | Script-based, not LVGL animation |
| HA state import | Yes | For lights on/off detection |

### v1-Expanded (NOT in prototype)

- Smooth LVGL opacity animation (requires `lv_anim` which may not be exposed in ESPHome)
- Dot vs. value distinction driven by real-time HA state
- LED ring minimum mode with single LED

### v2 (NOT in prototype)

- User-configurable Night Mode thresholds
- Scheduled Night Mode override
- Per-room ambient sensor calibration

---

## 12. Home Assistant Integration

| Entity | Purpose |
|--------|---------|
| `light.bedroom_group` | Detect ON/OFF state for dot vs. value display |
| Attribute: `brightness` | Show as dim percentage in Night Mode |
| TSL2591 sensor | Published to HA as `sensor.veladial_ambient_lux` |

---

## 13. Differentiation from Other Concepts

| Aspect | Concept 06 (This) | All Other Concepts |
|--------|-------------------|-------------------|
| Purpose | Sleep/dark-room overlay | Active interaction UI |
| Widget count | 1-2 | 10-20+ |
| User interaction | Wake only | Full control |
| Trigger | Automatic (ambient sensor) | User-initiated |
| Backlight | 10% minimum | 80% normal |
| Design philosophy | Absence, invisibility | Presence, clarity |

---

## 14. Unique Value Proposition

Night Mode Ultra-Minimal is the feature that separates a bedroom product from a generic smart-home gadget. It shows that the designers thought about the full 24-hour lifecycle of the device, not just the "demo at a trade show" moment. The single dot is a design signature — recognizable and memorable.
