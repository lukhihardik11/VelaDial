# Concept 06: Night Mode Ultra-Minimal — Validation Notes

## Gate Status

> **PASSES all gates (G1–G8).** Layer concept.

---

## 1. Compile Validation

| Check | Status | Notes |
|-------|--------|-------|
| ESPHome config validation | PENDING | No YAML errors expected |
| ESP-IDF framework compile | PENDING | Full firmware binary generation |
| Zero errors | PENDING | Must be 0 errors for PASS |
| Zero warnings (critical) | PENDING | Strapping pin warnings acceptable |

---

## 2. Physical Validation Checklist

All items below require the physical ELECROW CrowPanel 1.28" ESP32-S3 board AND a TSL2591 ambient light sensor.

### Ambient Light Sensor Tests

| Test | Expected | Status |
|------|----------|--------|
| TSL2591 reads lux values | Valid readings (0-88000 lux range) | NOT TESTED |
| Lux < 5 for 10s → Night Mode activates | Page switches to night_page | NOT TESTED |
| Lux > 15 for 5s → Night Mode deactivates | Returns to last active page | NOT TESTED |
| Hysteresis prevents oscillation | No flicker at threshold boundary | NOT TESTED |
| Sensor polling every 5s | Consistent readings | NOT TESTED |

### Display Tests (Night Mode Active)

| Test | Expected | Status |
|------|----------|--------|
| Backlight at 10% PWM | Barely visible in dark room | NOT TESTED |
| Single amber dot visible (lights OFF) | 8px dot at center, very dim | NOT TESTED |
| Dim percentage visible (lights ON) | 24pt amber text at center | NOT TESTED |
| Dot invisible from across room (3m+) | Cannot be seen from distance | NOT TESTED |
| Dot visible at arm's length (0.5m) | Readable close up | NOT TESTED |
| No other UI elements visible | Pure black except dot/value | NOT TESTED |

### Wake Tests

| Test | Expected | Status |
|------|----------|--------|
| Touch in Night Mode → wake | Full UI appears in 500ms | NOT TESTED |
| Knob CW in Night Mode → wake | Full UI appears in 500ms | NOT TESTED |
| Knob CCW in Night Mode → wake | Full UI appears in 500ms | NOT TESTED |
| Knob press in Night Mode → wake | Full UI appears in 500ms | NOT TESTED |
| Wake returns to last active page | Correct page shown | NOT TESTED |
| No action executed during wake | Only display wakes | NOT TESTED |
| 60s inactivity after wake → Night Mode returns | If still dark | NOT TESTED |

### LED Ring Tests (Night Mode)

| Test | Expected | Status |
|------|----------|--------|
| Lights OFF → all LEDs off | Complete darkness | NOT TESTED |
| Lights ON → 1 LED at minimum | Barely visible warm amber | NOT TESTED |
| LED not visible from across room | Cannot disturb sleep | NOT TESTED |

### Transition Tests

| Test | Expected | Status |
|------|----------|--------|
| Normal → Night Mode backlight fade | 2s smooth dim | NOT TESTED |
| Night Mode → Normal backlight fade | 500ms smooth brighten | NOT TESTED |
| No visual glitch during page switch | Clean transition | NOT TESTED |

### Home Assistant Integration Tests

| Test | Expected | Status |
|------|----------|--------|
| HA light ON → night value shows percentage | Correct value | NOT TESTED |
| HA light OFF → night dot shows | Dot visible | NOT TESTED |
| HA brightness change → value updates in Night Mode | Real-time | NOT TESTED |
| Ambient lux published to HA | sensor.veladial_ambient_lux | NOT TESTED |

---

## 3. Dark-Room Usability Observations (Physical Only)

| Observation | Status |
|-------------|--------|
| Is 10% backlight truly the minimum visible? | NOT TESTED |
| Does the dot disturb a sleeping partner? | NOT TESTED |
| Is the wake response fast enough for 3 AM use? | NOT TESTED |
| Does the 5 lux threshold match real bedroom darkness? | NOT TESTED |
| Is the hysteresis band (5-15 lux) appropriate? | NOT TESTED |

---

## 4. Sign-Off

| Role | Name | Date | Verdict |
|------|------|------|---------|
| Tester | — | — | NOT TESTED |
| Owner | — | — | PENDING |
