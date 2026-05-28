# Concept 12: Door Switch Replacement — Validation Notes

## Compile Validation

| Field | Value |
|-------|-------|
| Target | Compile PASS on ESPHome 2026.5.0 with ESP-IDF framework |
| YAML file | `esphome/concepts/door_side_concept_12_door_switch_replacement.yaml` |
| Status | PENDING |

## Gate Compliance

| Gate | Requirement | Status |
|------|-------------|--------|
| G1 | Three-page lock (Power, Brightness, Presets) | **PASS** — 3 pages implemented |
| G2 | Rotary encoder as primary input | **PASS** — knob press = toggle, knob rotate = brightness on page 1 |
| G3 | Wake-only-first on all inputs | **PASS** — all scripts check `is_asleep` |
| G4 | Home Assistant integration | **PASS** — targets `light.bedroom_group` |
| G5 | 4 locked presets | **PASS** — Warm White, Soft Amber, Neutral White, Low Nightlight |
| G6 | Round display safe area | **PASS** — full-screen obj uses radius: 120 for circular clip |
| G7 | LED ring integration | **PASS** — 5 LEDs, amber on/off with 50% brightness |
| G8 | No production YAML modification | **PASS** — standalone concept file |

## Physical Validation Checklist (NOT TESTED)

### Power Page Tests
- [ ] Full-screen touch target responds to tap anywhere within circle
- [ ] Tap vs swipe discrimination works (30px movement threshold)
- [ ] Amber fill is visible from 3m (doorway distance)
- [ ] OFF state border circle is visible in dark room at 15% backlight
- [ ] Toggle latency < 100ms (switch-like responsiveness)
- [ ] 64pt "ON" text is readable at 2m distance

### Animation Tests
- [ ] Background color transition is smooth (300ms)
- [ ] No flicker during ON→OFF transition
- [ ] No visible frame tearing during color change

### Sleep/Wake Tests
- [ ] Power page sleeps after 30 seconds of inactivity
- [ ] Brightness/Presets pages sleep after 60 seconds
- [ ] First touch on sleeping display ONLY wakes (does not toggle)
- [ ] Wake animation is immediate (< 200ms to full brightness)

### LED Ring Tests
- [ ] All 5 LEDs amber when lights ON
- [ ] All 5 LEDs off when lights OFF
- [ ] 50% brightness is appropriate for door-side (not too bright for room)
- [ ] LED state visible from 3m+ (across room)

### Guest Usability Tests
- [ ] First-time user can toggle lights without instruction
- [ ] State is obvious from doorway (amber = on, dark = off)
- [ ] No accidental toggles during swipe navigation
- [ ] Preset page is discoverable via swipe

### Dark-Room Tests
- [ ] OFF state border circle visible but not disturbing
- [ ] Backlight at 15% is sufficient for touch guidance
- [ ] LED ring does not disturb sleeping occupant (door-side location)

### Daylight Tests
- [ ] Full amber fill visible in direct sunlight
- [ ] "ON" text readable in bright room
- [ ] Page indicator dots visible in all lighting

## Hardware Dependencies

| Component | Status | Notes |
|-----------|--------|-------|
| GC9A01A display | Compile-verified | Physical test pending |
| CST816S touch | Compile-verified | Tap/swipe discrimination needs physical test |
| EC11 rotary | Compile-verified | Knob press toggle needs physical test |
| WS2812 LED ring | Compile-verified | Color order and brightness need physical test |
| Backlight PWM | Compile-verified | 15% dark-room level needs physical test |

## Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|-----------|
| Tap/swipe confusion on full-screen target | Medium | LVGL handles internally; may need tuning |
| 64pt font memory usage | Low | Only 2 glyphs ("O", "N") |
| Amber fill too bright in dark room | Medium | Backlight PWM can be reduced |
| Style transition not supported in ESPHome LVGL | Low | Fallback to immediate color change |
| 30s sleep too short for some users | Low | Configurable via HA |

## Success Criteria

1. **Primary:** Any person can toggle lights on first interaction without instruction
2. **Secondary:** Light state visible from doorway (3m) in all lighting conditions
3. **Tertiary:** Toggle latency feels instantaneous (< 100ms perceived)
4. **Quaternary:** No accidental toggles during page navigation

## NOT Claimed

- Production readiness
- Physical hardware validation
- Performance benchmarks
- User testing results
- LED color order verification
- Touch calibration
