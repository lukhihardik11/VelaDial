# Concept 06: Night Mode Ultra-Minimal — Implementation Notes

## Gate Status

> **PASSES all gates (G1–G8).** Layer concept — overlays on any primary concept.

---

## 1. Architecture Decision

Night Mode is implemented as a **dedicated LVGL page** that the system shows when the ambient light sensor triggers the dark threshold. This approach is simpler than opacity overlays because:

1. ESPHome LVGL does not expose `lv_anim` for smooth per-widget opacity transitions
2. A page switch is atomic and compile-proven (used in Concepts 01-05)
3. The Night Mode page contains only 2 widgets — minimal rendering overhead

### Page Structure

```
Page 0: night_page (Night Mode — shown when dark)
Page 1: power_page (Normal UI — Power)
Page 2: brightness_page (Normal UI — Brightness)
Page 3: presets_page (Normal UI — Presets)
```

When Night Mode activates, the system shows `night_page`. When the user wakes or ambient light rises, the system shows the last active normal page.

---

## 2. TSL2591 Ambient Light Sensor

### Hardware Connection

| Pin | ESP32-S3 GPIO | Notes |
|-----|---------------|-------|
| SDA | GPIO6 | Shared I2C bus |
| SCL | GPIO7 | Shared I2C bus |
| INT | Not connected | Polling mode |

### ESPHome Configuration

```yaml
sensor:
  - platform: tsl2591
    id: ambient_light
    name: "Ambient Lux"
    address: 0x29
    update_interval: 5s
    integration_time: 100ms
    gain: medium
    visible:
      name: "Visible Light"
    infrared:
      name: "Infrared Light"
    calculated_lux:
      name: "Calculated Lux"
      on_value:
        then:
          - script.execute: check_night_mode
```

**Note:** For the prototype YAML, the TSL2591 is simulated via a `template` sensor since the physical sensor may not be present during compile testing.

---

## 3. Night Mode State Machine

```
States:
  NORMAL    — Full UI active, backlight at 80%
  ENTERING  — Transitioning to Night Mode (2s fade)
  NIGHT     — Night Mode active, backlight at 10%
  WAKING    — Transitioning from Night Mode (500ms fade-in)

Transitions:
  NORMAL → ENTERING:  lux < 5 for 10 seconds
  ENTERING → NIGHT:   fade complete
  NIGHT → WAKING:     user input OR lux > 15 for 5 seconds
  WAKING → NORMAL:    fade complete
```

### Implementation via Globals

```cpp
globals:
  - id: night_mode_state
    type: int
    initial_value: '0'  // 0=NORMAL, 1=ENTERING, 2=NIGHT, 3=WAKING
  - id: last_active_page
    type: int
    initial_value: '1'  // Track which normal page to return to
  - id: dark_start_time
    type: unsigned long
    initial_value: '0'
  - id: light_start_time
    type: unsigned long
    initial_value: '0'
```

---

## 4. Backlight Dimming Strategy

The LEDC PWM output controls backlight brightness. Night Mode uses a script-based ramp:

```yaml
script:
  - id: dim_to_night
    then:
      - light.turn_on:
          id: back_light
          brightness: 10%
          transition_length: 2s
      - delay: 2s
      - lambda: 'id(night_mode_state) = 2;'  // NIGHT

  - id: wake_from_night
    then:
      - light.turn_on:
          id: back_light
          brightness: 80%
          transition_length: 500ms
      - delay: 500ms
      - lambda: 'id(night_mode_state) = 0;'  // NORMAL
```

The `transition_length` parameter on the monochromatic light component provides smooth PWM ramping without manual stepping.

---

## 5. Night Mode Page Widgets

Only 2 widgets on the Night Mode page:

### Night Dot (lights OFF)

```yaml
- obj:
    id: night_dot
    align: CENTER
    width: 8
    height: 8
    radius: 4
    bg_color: 0xFFA500
    bg_opa: 30%
    border_width: 0
```

### Night Value (lights ON)

```yaml
- label:
    id: night_value
    align: CENTER
    text_font: roboto24
    text_color: 0xFFA500
    text_opa: 30%
    text: "50%"
```

Visibility is toggled based on `light_is_on` state:
- Lights OFF → show dot, hide value (value text set to empty)
- Lights ON → hide dot (opa 0), show value with current brightness

---

## 6. Wake-Only-First (Inherent)

In Night Mode, ALL inputs trigger wake — there is no "action" to execute within Night Mode. The wake logic:

```cpp
// In all input handlers:
if (id(night_mode_state) == 2) {  // NIGHT
    id(night_mode_state) = 3;  // WAKING
    id(wake_from_night).execute();
    // Show last active page
    if (id(last_active_page) == 1) id(show_power_page).execute();
    else if (id(last_active_page) == 2) id(show_brightness_page).execute();
    else id(show_presets_page).execute();
    return;  // Do NOT execute the normal action
}
```

---

## 7. Hysteresis Implementation

```cpp
// In check_night_mode script:
float lux = id(ambient_light).state;
unsigned long now = millis();

if (lux < 5.0f) {
    id(light_start_time) = 0;
    if (id(dark_start_time) == 0) {
        id(dark_start_time) = now;
    } else if (now - id(dark_start_time) > 10000 && id(night_mode_state) == 0) {
        // Enter Night Mode
        id(night_mode_state) = 1;  // ENTERING
        id(enter_night_mode).execute();
    }
} else if (lux > 15.0f) {
    id(dark_start_time) = 0;
    if (id(light_start_time) == 0) {
        id(light_start_time) = now;
    } else if (now - id(light_start_time) > 5000 && id(night_mode_state) == 2) {
        // Exit Night Mode
        id(night_mode_state) = 3;  // WAKING
        id(wake_from_night).execute();
    }
} else {
    // In hysteresis band — no action
    id(dark_start_time) = 0;
    id(light_start_time) = 0;
}
```

---

## 8. Performance Considerations

| Concern | Mitigation |
|---------|-----------|
| TSL2591 polling every 5s | Negligible CPU; I2C read is <1ms |
| Night Mode page rendering | 2 widgets only — fastest possible render |
| Backlight transition | Hardware PWM ramp — zero CPU |
| State machine overhead | Single integer comparison per input |

---

## 9. Known Limitations

1. **No smooth per-widget opacity animation** — ESPHome LVGL doesn't expose `lv_anim`; page switch is instant
2. **TSL2591 may not be present** — prototype uses template sensor for compile
3. **No "entering" visual feedback** — transition is backlight-only; widgets switch instantly
4. **Single threshold** — no per-room calibration in v1
5. **No scheduled override** — Night Mode is purely sensor-driven in v1

---

## 10. Files

| File | Purpose |
|------|---------|
| `esphome/concepts/door_side_concept_06_night_mode_ultra_minimal.yaml` | Prototype YAML |
| `docs/ui/concepts/06_night_mode_ultra_minimal/research.md` | Research findings |
| `docs/ui/concepts/06_night_mode_ultra_minimal/design_spec.md` | Visual specification |
| `docs/ui/concepts/06_night_mode_ultra_minimal/implementation_notes.md` | This file |
| `docs/ui/concepts/06_night_mode_ultra_minimal/validation_notes.md` | Physical test plan |
