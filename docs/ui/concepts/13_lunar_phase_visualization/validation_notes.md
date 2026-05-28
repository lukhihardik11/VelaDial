# Concept 13: Lunar Phase Visualization — Validation Notes

## Compile Validation

| Item | Status |
|------|--------|
| ESPHome version | 2026.5.0 |
| Framework | ESP-IDF |
| Target | ESP32-S3 |
| Compile result | PENDING |
| Errors | PENDING |
| Warnings | PENDING |

## Validation Criteria

### Must Pass (Compile Gate)

1. YAML parses without error
2. All LVGL widgets resolve correctly
3. All scripts reference valid IDs
4. Font glyph string has no duplicates
5. GPIO pins do not conflict
6. Home Assistant sensor imports are valid
7. Binary compiles with 0 errors

### Must Pass (Design Gate)

1. Three-page architecture maintained (Power / Brightness / Presets)
2. Wake-only-first enforced on all input paths
3. Knob CW/CCW drives brightness on Page 2
4. All 4 locked presets present and functional
5. `light.bedroom_group` is the HA target
6. LED ring behavior defined for all states
7. Page indicators present

### Hardware Test Pending (NOT VALIDATED)

| Test | Expected | Actual | Status |
|------|----------|--------|--------|
| Moon circle renders as true circle on GC9A01A | 200px perfect circle | — | NOT TESTED |
| Shadow overlay creates convincing terminator | Visible phase change | — | NOT TESTED |
| Amber color (0xFFA500) matches brand on display | Warm amber | — | NOT TESTED |
| Semi-transparent text readable over moon | 40% opacity legible | — | NOT TESTED |
| Shadow offset smooth during knob rotation | No stutter | — | NOT TESTED |
| LED ring warm amber matches display amber | Color consistency | — | NOT TESTED |
| Backlight at 80% sufficient for amber visibility | Clear in dark room | — | NOT TESTED |
| Touch on full moon area registers correctly | 200px touch target | — | NOT TESTED |
| New moon limb visible in dark room | 2px gray border | — | NOT TESTED |
| Page swipe animation smooth (200ms) | No frame drops | — | NOT TESTED |

## Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| Linear terminator looks artificial | Medium | Acceptable for v1; v1-expanded uses images |
| Shadow offset calculation overflow | Low | Clamped to 0-200 range |
| Semi-transparent text unreadable in daylight | Medium | Fallback: increase opacity to 60% |
| Moon too small at 200px on 240px display | Low | 200px leaves 20px margin for safe area |
| Amber color washes out in daylight | Medium | Known limitation; dark-room optimized |

## Production Readiness

**NOT PRODUCTION READY.** This is a concept prototype only. Production deployment requires:
- Hardware validation of all pending tests
- Visual quality assessment on actual display
- User testing of lunar metaphor comprehension
- Performance profiling of shadow offset updates
- Decision on v1 (arc-based) vs v1-expanded (image-based)

## Version Scope Boundaries

| Feature | v1 Locked | v1-Expanded | v2 |
|---------|-----------|-------------|-----|
| Arc-based moon phase | Yes | — | — |
| Pre-rendered moon images | — | Yes | — |
| Moonrise/moonset animation | Opacity fade only | Full wax/wane | — |
| Mini-moons on Presets page | No (text only) | Yes | — |
| Real lunar phase from date | — | — | Yes |
| Parallax texture | — | — | Yes |
| Tidal ambient page | — | — | Yes |
