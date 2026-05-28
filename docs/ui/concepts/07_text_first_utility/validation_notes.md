# Concept 07: Text-First Utility — Validation Notes

## Gate Status

> **PASSES all gates (G1–G8).** Standard primary concept, v1 candidate.

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

### Typography Readability Tests

| Test | Expected | Status |
|------|----------|--------|
| 56pt "ON" readable at 2m distance | Clear text | NOT TESTED |
| 56pt "OFF" readable at 2m (dim gray) | Visible but subdued | NOT TESTED |
| 56pt "75%" readable at arm's length | Clear percentage | NOT TESTED |
| 16pt "Power" label readable at 0.5m | Legible secondary text | NOT TESTED |
| 12pt step indicator readable at 0.3m | Legible tertiary text | NOT TESTED |
| 24pt preset names readable at 0.5m | All 4 names legible | NOT TESTED |
| Amber highlight distinguishable from gray | Clear active state | NOT TESTED |

### Round Bezel Cropping Tests

| Test | Expected | Status |
|------|----------|--------|
| 56pt text fully visible (no bezel clip) | No character clipping | NOT TESTED |
| 24pt preset list within safe area | All 4 items visible | NOT TESTED |
| Page dots visible at y:210 | Dots within circular area | NOT TESTED |
| "Brightness" header at y:50 not clipped | Full text visible | NOT TESTED |

### Dark-Room Tests

| Test | Expected | Status |
|------|----------|--------|
| 56pt white text at 10% backlight | Readable close-up | NOT TESTED |
| No eye strain from high contrast | Comfortable viewing | NOT TESTED |
| Dim gray "OFF" visible at minimum backlight | Barely visible | NOT TESTED |
| Page dots visible at minimum backlight | Dots discernible | NOT TESTED |

### Daylight Tests

| Test | Expected | Status |
|------|----------|--------|
| 56pt text readable in direct sunlight | Maximum contrast helps | NOT TESTED |
| 16pt labels readable in bright room | Sufficient contrast | NOT TESTED |
| Amber vs gray distinguishable in daylight | Color difference clear | NOT TESTED |

### Interaction Tests

| Test | Expected | Status |
|------|----------|--------|
| Knob CW on Power → Brightness page | Page 2 shown | NOT TESTED |
| Knob CCW on Power → Presets page | Page 3 shown | NOT TESTED |
| Knob press on Power → toggle | ON/OFF changes | NOT TESTED |
| Knob CW on Brightness → value increases | +5% per click | NOT TESTED |
| Knob CCW on Brightness → value decreases | -5% per click | NOT TESTED |
| Knob press on Brightness → return to Power | Page 1 shown | NOT TESTED |
| Knob CW on Presets → highlight moves down | Next preset amber | NOT TESTED |
| Knob CCW on Presets → highlight moves up | Previous preset amber | NOT TESTED |
| Knob press on Presets → apply + return | Preset applied, Page 1 | NOT TESTED |
| Wake-only-first on all inputs | No action on wake | NOT TESTED |

### LED Ring Tests

| Test | Expected | Status |
|------|----------|--------|
| Lights ON → all LEDs amber 30% | Subtle amber glow | NOT TESTED |
| Lights OFF → all LEDs off | No light | NOT TESTED |
| LED brightness not distracting | Secondary to display | NOT TESTED |

### Home Assistant Integration Tests

| Test | Expected | Status |
|------|----------|--------|
| HA light ON → "ON" text white | State synced | NOT TESTED |
| HA light OFF → "OFF" text gray | State synced | NOT TESTED |
| HA brightness change → percentage updates | Real-time | NOT TESTED |
| Toggle from device → HA state updates | Bidirectional | NOT TESTED |
| Preset apply → HA service called | Correct effect/scene | NOT TESTED |

---

## 3. Premium vs. Cheap Assessment (Physical Only)

| Question | Status |
|----------|--------|
| Does the typography feel "premium" or "cheap LCD clock"? | NOT TESTED |
| Is the white space confident or empty? | NOT TESTED |
| Does the amber accent feel intentional or random? | NOT TESTED |
| Is the font size hierarchy clear at a glance? | NOT TESTED |
| Does the round bezel cropping feel intentional? | NOT TESTED |

---

## 4. Sign-Off

| Role | Name | Date | Verdict |
|------|------|------|---------|
| Tester | — | — | NOT TESTED |
| Owner | — | — | PENDING |
