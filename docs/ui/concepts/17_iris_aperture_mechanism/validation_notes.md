# UI Concept 17: Iris Aperture Mechanism — Validation Notes

## 1. Validation Status

| Validation Type | Status | Notes |
|-----------------|--------|-------|
| **YAML Compile** | `PASS` | ESPHome 2026.5.0, ESP-IDF 5.5.4, 0 errors, 169.83s fresh build. |
| **Physical Hardware** | `NOT TESTED` | No physical ESP32-S3 device attached. |
| **HA Integration** | `NOT TESTED` | `light.bedroom_group` target configured but not live-tested. |
| **LVGL Rendering** | `PENDING` | Iris blade abstraction needs visual confirmation. |

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

### Test 2: The Iris Metaphor
1. Navigate to the Brightness page.
2. Rotate knob clockwise from 0% to 100%.
3. **Expected:** The central amber aperture expands, and the surrounding dark "blades" retract outward, simulating a camera iris opening.
4. Rotate knob counter-clockwise to 10%.
5. **Expected:** The aperture contracts, and the blades converge inward, simulating the iris stopping down to a pinhole.

### Test 3: LED Ring Sync
1. Adjust brightness from 10% to 100%.
2. **Expected:** The 5-LED ring smoothly increases in intensity, acting as the "light leak" around the lens housing.

### Test 4: Presets
1. Navigate to the Presets page.
2. Select "Portrait".
3. **Expected:** The UI updates to reflect the preset (60% brightness), and the HA light group changes state.

## 4. Known Limitations (v1 Prototype)

- True rotating, overlapping polygon blades are not feasible in pure LVGL YAML without severe performance penalties.
- The iris is abstracted using a central expanding circle and overlapping thick arcs to simulate the octagonal blade edges.
- The animation is a simple scaling of these elements rather than a true mechanical rotation.
- The visual success of this concept relies heavily on the color contrast (dark metallic gray vs. bright amber) and the octagonal shape of the aperture opening. On a real display, this abstraction must be carefully evaluated to ensure it feels like a mechanical aperture and not just a resizing circle.
