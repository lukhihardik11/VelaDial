# UI Concept 14: Sundial Shadow UI — Validation Notes

## 1. Validation Status

| Validation Type | Status | Notes |
|-----------------|--------|-------|
| **YAML Compile** | `PASS` | ESPHome 2026.5.0, ESP-IDF 5.5.4, 0 errors, 106s fresh build. |
| **Physical Hardware** | `NOT TESTED` | No physical ESP32-S3 device attached. |
| **HA Integration** | `NOT TESTED` | `light.bedroom_group` target configured but not live-tested. |
| **LVGL Rendering** | `PENDING` | Shadow arc abstraction needs visual confirmation. |

## 2. Compile Validation Checklist

Before marking this concept as `COMPILE STATUS: PASS`, the following must be verified:
- [ ] `esphome compile` completes with 0 errors.
- [ ] No missing font glyphs.
- [ ] No invalid LVGL widget properties (e.g., using `btn` instead of `button`).
- [ ] `shadow_opa` and similar properties use correct formatting (e.g., percentages).
- [ ] I2C and SPI buses are correctly configured without conflicts.

## 3. Physical Validation Test Plan (For Hardik)

When physical hardware is available, perform the following tests:

### Test 1: Wake-Only-First
1. Let the device go to sleep (screen off).
2. Rotate the knob 1 click.
3. **Expected:** Screen wakes up. Brightness does NOT change.
4. Let it sleep again.
5. Tap the screen.
6. **Expected:** Screen wakes up. Power does NOT toggle.

### Test 2: The Sundial Shadow Metaphor
1. Navigate to the Brightness page.
2. Rotate knob clockwise to 100%.
3. **Expected:** The dark "shadow" arc shrinks to a minimal sliver. The background color shifts to bright amber.
4. Rotate knob counter-clockwise to 10%.
5. **Expected:** The dark "shadow" arc grows to cover half the screen. The background color shifts to dark charcoal.

### Test 3: LED Ring Sync
1. Adjust brightness from 10% to 100%.
2. **Expected:** The 5-LED ring smoothly increases in intensity, maintaining a warm amber color.

### Test 4: Presets
1. Navigate to the Presets page.
2. Select "Soft Amber".
3. **Expected:** The UI updates to reflect the preset, and the HA light group changes state.

## 4. Known Limitations (v1 Prototype)

- The shadow is an abstraction (an `lv_arc`) rather than a true ray-traced shadow, due to ESP32-S3 performance constraints.
- The shadow angle does not track real-world time of day (this is a v2 feature).
- The background color shift is discrete or simplified, rather than a continuous 24-hour gradient.
