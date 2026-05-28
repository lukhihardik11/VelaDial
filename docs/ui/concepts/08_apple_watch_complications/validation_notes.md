# Concept 08: Apple Watch Complications — Validation Notes

## Gate Status

> **PASSES all gates (G1–G8)** in v1-expanded adaptation.
> **NOT RECOMMENDED for v1** per direction matrix — too complex for bedroom-first philosophy.
> This prototype implements the v1-expanded adaptation: 2 decorative corner complications only.

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

### Complication Readability Tests

| Test | Expected | Status |
|------|----------|--------|
| 14pt WiFi complication readable at 0.3m | Legible text | NOT TESTED |
| 14pt Lux complication readable at 0.3m | Legible text | NOT TESTED |
| Complications visible but not distracting | Secondary to center | NOT TESTED |
| Complication text not clipped by round bezel | Fully visible | NOT TESTED |
| Gray color distinguishable from black background | Subtle but visible | NOT TESTED |

### Center Content Tests

| Test | Expected | Status |
|------|----------|--------|
| 48pt "ON" readable at 2m distance | Clear primary text | NOT TESTED |
| 48pt "OFF" readable at 2m (dim gray) | Visible but subdued | NOT TESTED |
| 48pt "75%" readable at arm's length | Clear percentage | NOT TESTED |
| 24pt preset name readable at 0.5m | Legible text | NOT TESTED |
| Center content clearly dominant over complications | Visual hierarchy clear | NOT TESTED |

### Information Density Assessment

| Test | Expected | Status |
|------|----------|--------|
| Page feels "information-rich" not "cluttered" | Balanced density | NOT TESTED |
| Complications add value without overwhelming | Ambient data useful | NOT TESTED |
| Dark-room readability with complications | Acceptable | NOT TESTED |
| Daylight readability of 14pt complications | Marginal (known risk) | NOT TESTED |

### Interaction Tests

| Test | Expected | Status |
|------|----------|--------|
| Knob CW on Power → Brightness page | Page 2 shown | NOT TESTED |
| Knob CCW on Power → Presets page | Page 3 shown | NOT TESTED |
| Knob press on Power → toggle | ON/OFF changes | NOT TESTED |
| Knob CW on Brightness → value increases | +5% per click | NOT TESTED |
| Knob CCW on Brightness → value decreases | -5% per click | NOT TESTED |
| Knob press on Brightness → return to Power | Page 1 shown | NOT TESTED |
| Knob CW on Presets → cycle forward | Next preset highlighted | NOT TESTED |
| Knob press on Presets → apply + return | Preset applied, Page 1 | NOT TESTED |
| Wake-only-first on all inputs | No action on wake | NOT TESTED |
| Complications NOT interactive (no tap response) | Display-only confirmed | NOT TESTED |

### Sensor Data Tests

| Test | Expected | Status |
|------|----------|--------|
| WiFi RSSI updates every 30s | Value changes | NOT TESTED |
| Lux value updates every 10s | Value changes | NOT TESTED |
| Sensor unavailable → shows "--" placeholder | Graceful fallback | NOT TESTED |
| Sensor data persists across page navigation | Same value on all pages | NOT TESTED |

### LED Ring Tests

| Test | Expected | Status |
|------|----------|--------|
| Lights ON → all LEDs amber 30% | Subtle glow | NOT TESTED |
| Lights OFF → all LEDs off | No light | NOT TESTED |

---

## 3. Bedroom-First Philosophy Assessment

| Question | Status |
|----------|--------|
| Do complications compromise dark-room comfort? | NOT TESTED |
| Is 14pt text visible at minimum backlight? | NOT TESTED |
| Does information density feel appropriate for bedroom? | NOT TESTED |
| Would a partner find the display distracting at night? | NOT TESTED |

> **Known Risk:** The direction matrix warns that "high information density means many small elements compete for attention in low light" and that this concept is "fundamentally at odds with the bedroom-first design philosophy." Physical testing must confirm or deny this concern.

---

## 4. Sign-Off

| Role | Name | Date | Verdict |
|------|------|------|---------|
| Tester | — | — | NOT TESTED |
| Owner | — | — | PENDING |
