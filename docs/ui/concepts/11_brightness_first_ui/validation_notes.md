# UI Concept 11: Brightness-First UI — Validation Notes

## Validation Status

| Check | Status |
| --- | --- |
| YAML syntax | PENDING |
| ESPHome config validation | PENDING |
| Full compile (ESP-IDF) | PENDING |
| Flash to hardware | NOT TESTED |
| Physical interaction test | NOT TESTED |
| HA integration test | NOT TESTED |
| Dark-room usability test | NOT TESTED |

## Compile Environment

| Parameter | Value |
| --- | --- |
| ESPHome version | 2026.5.0 |
| Framework | ESP-IDF 5.5.4 |
| Platform | ESP32-S3 |
| Board | esp32-s3-devkitc-1 |
| PSRAM | Quad, 8MB |

## Validation Criteria

### Compile Validation (Automated)

1. Zero errors from `esphome compile`
2. Binary generated successfully
3. No warnings related to LVGL configuration
4. Font glyph deduplication verified

### Functional Validation (Hardware Required)

1. **Default page test:** Device boots to Brightness page (not Power)
2. **Knob rotation test:** CW increases brightness, CCW decreases, immediate arc update
3. **Knob press shortcut test:** Press on Brightness page toggles power
4. **Page navigation test:** Swipe left → Power → Presets, swipe right reverses
5. **Page boundary test:** Swipe right on Brightness (page 0) does nothing
6. **Dot indicator test:** Dots correctly show Brightness/Power/Presets order
7. **Wake target test:** After sleep, wake always returns to Brightness page
8. **3AM flow test:** Wake → rotate brightness → press off (zero swipes, <3 seconds)
9. **HA sync test:** Brightness changed in HA app reflects on display within 1 second
10. **Preset application test:** Selecting preset updates brightness arc on return to page 0

### UX Validation (User Testing Required)

1. **Mental model test:** Does user find brightness on first interaction without instruction?
2. **Power discovery test:** Can user find power toggle within 5 seconds?
3. **Knob press discovery test:** Does user discover knob press = power shortcut?
4. **Adaptation time:** How many interactions before user is comfortable with new order?

## Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| User confusion from non-standard page order | Medium | Clear onboarding, consistent dot indicators |
| Knob press shortcut not discoverable | Low | Physical label on device, or brief animation hint |
| Hardik rejects page reorder | High | Concept is simply not selected; no code waste |
| Arc not updating smoothly | Low | Same implementation as Concept 10 (proven) |

## Hardware Dependencies (All NOT TESTED)

- GC9A01A display renders arc correctly at 200px diameter
- CST816S touch detects swipe gestures reliably
- Rotary encoder debouncing works at ±5% step rate
- Backlight PWM ramp is smooth (not stepped)
- WS2812 LED ring responds to brightness changes

## Production Readiness

**NOT PRODUCTION READY.** This is a concept prototype only. Production readiness requires:
1. Hardik approval of brightness-first page order
2. Physical hardware validation (all checks above)
3. User testing of the non-standard mental model
4. Decision on knob press shortcut behavior
5. Integration with Night Mode layer (Concept 06)

## Notes

- This concept is the lowest-risk implementation change in the entire 20-concept matrix
- It reuses 95% of Concept 10's code with only page order swapped
- The risk is entirely in UX (user mental model), not in technical implementation
- If rejected, the code is trivially reverted to Concept 10's page order
