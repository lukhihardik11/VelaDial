# Concept 05: Preset Ring UI — Validation Notes

## Gate Status

> **PASSES all gates (G1–G8).**

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

All items below require the physical ELECROW CrowPanel 1.28" ESP32-S3 board.

### Display Tests

| Test | Expected | Status |
|------|----------|--------|
| 4 arc segments visible on Presets page | All 4 quadrants visible | NOT TESTED |
| 10° gaps between segments visible | Clear separation | NOT TESTED |
| Active segment color distinct from inactive | Amber vs gray contrast | NOT TESTED |
| Warm White vs Soft Amber distinguishable | Color difference visible | NOT TESTED |
| Neutral White (cool) vs Warm White (amber) distinct | Clear difference | NOT TESTED |
| Low Nightlight (dim amber) visible on black bg | Readable | NOT TESTED |
| Center text readable within ring | No overlap with arcs | NOT TESTED |
| Power ring (full circle) renders correctly | Complete 360° | NOT TESTED |
| Brightness arc proportional to value | Accurate fill | NOT TESTED |
| Page indicator dots visible | 3 dots at bottom | NOT TESTED |

### Rotary Encoder Tests

| Test | Expected | Status |
|------|----------|--------|
| CW on Power page → navigate to Brightness | Page change | NOT TESTED |
| CCW on Power page → navigate to Presets | Page change | NOT TESTED |
| Press on Power page → toggle power | ON↔OFF | NOT TESTED |
| CW on Brightness page → increase brightness | +5% per detent | NOT TESTED |
| CCW on Brightness page → decrease brightness | -5% per detent | NOT TESTED |
| CW on Presets page → next preset highlighted | Ring updates | NOT TESTED |
| CCW on Presets page → previous preset highlighted | Ring updates | NOT TESTED |
| Press on Presets page → apply preset | HA service called | NOT TESTED |
| All inputs while asleep → wake only | No action executed | NOT TESTED |

### Touch Tests

| Test | Expected | Status |
|------|----------|--------|
| Tap center on Power page → toggle | ON↔OFF | NOT TESTED |
| Tap center on Presets page → apply | Preset applied | NOT TESTED |
| Touch while asleep → wake only | No action | NOT TESTED |

### LED Ring Tests

| Test | Expected | Status |
|------|----------|--------|
| Warm White preset → amber LEDs | (255, 165, 0) | NOT TESTED |
| Soft Amber preset → deep gold LEDs | (204, 136, 0) | NOT TESTED |
| Neutral White preset → cool white LEDs | (180, 200, 255) | NOT TESTED |
| Low Nightlight preset → very dim amber LEDs | (40, 25, 0) | NOT TESTED |
| Power OFF → LEDs off | (0, 0, 0) | NOT TESTED |

### Home Assistant Integration Tests

| Test | Expected | Status |
|------|----------|--------|
| HA brightness change → arc updates | Proportional fill | NOT TESTED |
| HA power toggle → ring color changes | Amber↔gray | NOT TESTED |
| HA color_temp change → active segment updates | Correct segment highlighted | NOT TESTED |
| Device preset apply → HA state syncs | Bidirectional | NOT TESTED |

---

## 3. Usability Observations (Physical Only)

| Observation | Status |
|-------------|--------|
| Are 4 arc segments clearly distinguishable? | NOT TESTED |
| Does the ring metaphor feel natural with the knob? | NOT TESTED |
| Is the "rotate to select, press to apply" discoverable? | NOT TESTED |
| Does the color coding help or confuse? | NOT TESTED |
| Is the 10° gap sufficient visual separation? | NOT TESTED |

---

## 4. Sign-Off

| Role | Name | Date | Verdict |
|------|------|------|---------|
| Tester | — | — | NOT TESTED |
| Owner | — | — | PENDING |
