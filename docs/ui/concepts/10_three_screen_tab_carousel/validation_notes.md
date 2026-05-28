# Concept 10: Three-Screen Tab Carousel — Validation Notes

**Status:** CONCEPT PROTOTYPE — NOT PRODUCTION
**Physical Validation:** NOT TESTED

---

## 1. Compile Validation

| Field | Value |
|-------|-------|
| ESPHome version | 2026.5.0 |
| Framework | ESP-IDF |
| Target | ELECROW CrowPanel 1.28" ESP32-S3 |
| Compile result | PENDING (to be updated after compile) |
| Errors | PENDING |

---

## 2. Gate Compliance

| Gate | Requirement | Status |
|------|-------------|--------|
| G1 | Three-page lock | **PASS** — exactly 3 pages (Power, Brightness, Presets) |
| G2 | Four locked presets | **PASS** — Warm White, Soft Amber, Neutral White, Low Nightlight |
| G3 | Wake-only-first | **PASS** — all inputs wake-only when asleep |
| G4 | Rotary encoder integration | **PASS** — knob CW/CCW for brightness, press for power/apply |
| G5 | LED ring integration | **PASS** — amber proportional when on, off when off |
| G6 | Home Assistant target | **PASS** — targets `light.bedroom_group` |
| G7 | Round display optimization | **PARTIAL** — safe area respected, but does not exploit round form |
| G8 | Dark-room usability | **PASS** — large elements, high contrast, amber accent |

---

## 3. Hardware Tests Required

| Test | Description | Status |
|------|-------------|--------|
| HW-01 | Swipe gesture recognition on CST816D | NOT TESTED |
| HW-02 | Page transition smoothness at 200ms | NOT TESTED |
| HW-03 | 2x2 grid visibility within bezel | NOT TESTED |
| HW-04 | Arc rendering performance | NOT TESTED |
| HW-05 | Page dot visibility at 8px | NOT TESTED |
| HW-06 | Swipe vs tap disambiguation (30px threshold) | NOT TESTED |
| HW-07 | LED ring brightness at 30% ceiling | NOT TESTED |
| HW-08 | Wake-to-Page-0 transition appearance | NOT TESTED |

---

## 4. Comparison to Production YAML

This concept is architecturally closest to the existing `door_side_rotary.yaml`. Key differences:

| Aspect | Production | Concept 10 |
|--------|-----------|------------|
| Page navigation | Swipe + knob | Swipe only (knob for brightness) |
| Preset selection | Touch tiles | Touch tiles (same) |
| Arc interaction | Touch + knob | Knob only (arc display-only) |
| Page indicator | 3 dots | 3 dots (same, refined spacing) |
| Wake behavior | Last page | Always Page 0 (Power) |
| LED ring | Basic | Same basic (amber on/off) |

---

## 5. Risk Items for Physical Testing

1. **Swipe gesture sensitivity:** CST816D may require calibration for reliable left/right detection
2. **Page dot size:** 8px dots may be too small on 190 PPI — may need 10px
3. **2x2 grid touch targets:** 85x45px tiles may be too small for reliable finger taps
4. **Transition jank:** 200ms animation at ESP32-S3 clock speed — needs performance verification

---

## 6. Acceptance Criteria (Physical)

For this concept to be considered physically validated:

- [ ] All 3 pages render correctly within circular bezel
- [ ] Swipe left/right navigates pages reliably (>90% success rate)
- [ ] Page dots update correctly on every page change
- [ ] Brightness arc tracks knob rotation smoothly
- [ ] All 4 preset tiles are tappable and respond
- [ ] Wake-to-Page-0 works from any sleep state
- [ ] LED ring reflects power state correctly
- [ ] No visual artifacts at page boundaries
- [ ] 48pt text readable at 1m distance in dark room
