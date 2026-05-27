# Concept 04: Single-Page Simple Mode — Validation Notes

## Gate Status

> **FAILS Gate G1 (Three-Page Lock).** Prototyped for exploration only. Cannot be selected for production without explicit owner waiver.

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
| All three zones visible within round bezel | No clipping | NOT TESTED |
| 48pt brightness value readable at center | Clear, large | NOT TESTED |
| 24pt power label readable in top zone | Clear | NOT TESTED |
| 20pt preset name readable in bottom zone | Clear | NOT TESTED |
| Arc indicator visible around center value | Thin amber line | NOT TESTED |
| Black background fills entire round display | No light bleed | NOT TESTED |

### Touch Zone Tests

| Test | Expected | Status |
|------|----------|--------|
| Tap top zone → power toggles | ON↔OFF | NOT TESTED |
| Tap bottom zone → preset cycles | Name changes | NOT TESTED |
| Tap center zone → no action | Nothing happens | NOT TESTED |
| Touch while asleep → wake only | No toggle/cycle | NOT TESTED |
| Second touch while awake → action executes | Toggle/cycle works | NOT TESTED |

### Rotary Encoder Tests

| Test | Expected | Status |
|------|----------|--------|
| CW rotation → brightness increases | +5% per detent | NOT TESTED |
| CCW rotation → brightness decreases | -5% per detent | NOT TESTED |
| Knob press → power toggles | ON↔OFF | NOT TESTED |
| Rotate while asleep → wake only | No brightness change | NOT TESTED |
| Press while asleep → wake only | No toggle | NOT TESTED |

### LED Ring Tests

| Test | Expected | Status |
|------|----------|--------|
| Power ON → amber LEDs | Visible glow | NOT TESTED |
| Power OFF → LEDs off | Dark | NOT TESTED |
| Wake → LEDs fade in | Smooth 300ms | NOT TESTED |
| Sleep → LEDs fade out | Smooth 300ms | NOT TESTED |

### Home Assistant Integration Tests

| Test | Expected | Status |
|------|----------|--------|
| HA brightness change → display updates | Value + arc sync | NOT TESTED |
| HA power toggle → display updates | Label changes | NOT TESTED |
| HA preset applied → display updates | Name changes | NOT TESTED |
| Device power toggle → HA state syncs | Bidirectional | NOT TESTED |
| Device brightness change → HA syncs | Bidirectional | NOT TESTED |

---

## 3. Usability Observations (Physical Only)

| Observation | Status |
|-------------|--------|
| Can user read all three zones simultaneously? | NOT TESTED |
| Is the bottom zone tap discoverable? | NOT TESTED |
| Does the single-page feel cramped on 240x240? | NOT TESTED |
| Is dark-room readability acceptable? | NOT TESTED |
| Does the concept feel "too simple" for premium device? | NOT TESTED |

---

## 4. Gate G1 Waiver Decision

| Question | Answer |
|----------|--------|
| Does this concept pass Gate G1? | **NO** — uses 1 page, not 3 |
| Can it be selected for production? | Only with explicit owner waiver |
| Is the prototype still valuable? | Yes — explores radical simplification |
| Owner decision recorded? | PENDING |

---

## 5. Sign-Off

| Role | Name | Date | Verdict |
|------|------|------|---------|
| Tester | — | — | NOT TESTED |
| Owner | — | — | PENDING |
