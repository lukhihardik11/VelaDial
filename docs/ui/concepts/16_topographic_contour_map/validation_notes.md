# UI Concept 16: Topographic Contour Map — Validation Notes

## 1. Validation Status

| Validation Type | Status | Notes |
|-----------------|--------|-------|
| **YAML Compile** | `PASS` | ESPHome 2026.5.0, ESP-IDF 5.5.4, 0 errors, 159s fresh build. |
| **Physical Hardware** | `NOT TESTED` | No physical ESP32-S3 device attached. |
| **HA Integration** | `NOT TESTED` | `light.bedroom_group` target configured but not live-tested. |
| **LVGL Rendering** | `PENDING` | Contour line abstraction needs visual confirmation. |

## 2. Compile Validation Checklist

Before marking this concept as `COMPILE STATUS: PASS`, the following must be verified:
- [ ] `esphome compile` completes with 0 errors.
- [ ] No missing font glyphs.
- [ ] No invalid LVGL widget properties.
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

### Test 2: The Topographic Metaphor
1. Navigate to the Brightness page.
2. Rotate knob clockwise from 0% to 100%.
3. **Expected:** Thin contour lines appear one by one as brightness increases, emerging from the center outward, simulating a rising terrain.
4. Rotate knob counter-clockwise to 10%.
5. **Expected:** Contour lines disappear from the outside in, simulating a sinking terrain.

### Test 3: LED Ring Sync
1. Adjust brightness from 10% to 100%.
2. **Expected:** The 5-LED ring smoothly increases in intensity, acting as the outermost "horizon line".

### Test 4: Presets
1. Navigate to the Presets page.
2. Select "Ridge".
3. **Expected:** The UI updates to reflect the preset, and the HA light group changes state.

## 4. Known Limitations (v1 Prototype)

- The contour lines are perfect geometric circles (`lv_obj` borders) rather than true organic, wavy topographic lines, due to ESP32-S3 performance constraints and the difficulty of managing image assets in pure YAML.
- The terrain emergence is discrete (12 steps) rather than continuous, to maintain performance.
- The visual distinction between this concept and Concept 15 (Tree Rings) relies heavily on line thickness, color palette, and the central summit marker. On a real display, this distinction must be carefully evaluated.
