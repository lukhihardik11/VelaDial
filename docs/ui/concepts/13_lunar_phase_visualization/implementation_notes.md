# Concept 13: Lunar Phase Visualization — Implementation Notes

## LVGL Implementation Strategy

### Moon Phase Rendering (Arc-Based Approximation)

The v1 locked implementation avoids pre-rendered PNG images to conserve flash storage. Instead, it uses overlapping LVGL objects to approximate the lunar terminator:

1. **Base layer:** 200×200px amber-filled `obj` (circular via `radius: 100`) — represents the full lit surface
2. **Shadow layer:** 200×200px black-filled `obj` (circular) — positioned with dynamic `x` offset to create the terminator
3. **Limb layer:** 200×200px transparent `obj` with 2px gray border — always visible, defines the moon's edge

The shadow offset is calculated from brightness:
```
shadow_x_offset = (100 - brightness_pct) * 200 / 100
```
- At brightness 100%: offset = 0 → shadow is directly behind lit surface (full moon)
- At brightness 50%: offset = 100 → shadow covers right half (first quarter)
- At brightness 0%: offset = 200 → shadow fully covers (new moon)

### Limitation of Arc-Based Approach

The overlapping-circles method produces a **linear terminator** rather than the true curved terminator of a real moon. This is acceptable for v1 as the visual metaphor is still clear. The v1-expanded version would use pre-rendered images for accurate crescent shapes.

### LVGL Widget Structure

```
Page 1 (Power):
  - obj: moon_lit_power (200px amber circle)
  - obj: moon_shadow_power (200px black circle, offset by state)
  - obj: moon_limb_power (200px border-only circle)
  - label: power_state ("ON"/"OFF", 20pt, semi-transparent)

Page 2 (Brightness):
  - obj: moon_lit_brightness (200px amber circle)
  - obj: moon_shadow_brightness (200px black circle, dynamic offset)
  - obj: moon_limb_brightness (200px border-only circle)
  - label: brightness_pct ("75%", 32pt, semi-transparent)

Page 3 (Presets):
  - label: preset_warm (16pt, "Warm White 80%")
  - label: preset_amber (16pt, "Soft Amber 60%")
  - label: preset_neutral (16pt, "Neutral White 90%")
  - label: preset_night (16pt, "Low Nightlight 15%")
```

Total widget count: ~15 objects + labels

### Shadow Offset Update Mechanism

The shadow position is updated via `lvgl.obj.update` action with computed `x` coordinate. Since LVGL does not support mathematical expressions directly in the `x` property, the offset is computed in a C++ lambda and applied via `lv_obj_set_x()`.

However, based on compile-proven patterns from earlier concepts, we use `lvgl.obj.update` with pre-computed values from the script layer. The script calculates the offset and calls `lvgl.obj.update` with the new position.

### ESPHome Script Architecture

```yaml
scripts:
  - id: update_moon_phase
    # Called whenever brightness changes
    # Computes shadow offset and updates all moon shadow objects

  - id: guarded_knob_cw
    # Wake-only-first, then brightness +5% on page 2

  - id: guarded_knob_ccw
    # Wake-only-first, then brightness -5% on page 2

  - id: guarded_power_toggle
    # Wake-only-first, then toggle power

  - id: guarded_preset_apply
    # Wake-only-first, then apply preset by index

  - id: sync_from_ha
    # Import HA state and update moon phase accordingly
```

### Font Requirements

Single font file with two sizes:
- 32pt: digits 0-9, %, space (for brightness percentage)
- 20pt: O, N, F, space (for ON/OFF label)
- 16pt: Full alphabet for preset names

To avoid duplicate glyph errors, use a single font at 32pt with all needed characters, and use LVGL label `text_font` overrides for smaller sizes.

### Performance Considerations

- Shadow offset update on every knob tick (20ms debounce)
- Only 2 objects move per update (shadow on current page)
- No image decode, no canvas operations
- Expected render time: < 5ms per frame
- Smooth 60fps possible even on ESP32-S3

### Flash Storage

- Zero image assets (arc-based approach)
- Single Roboto font (~15KB for 3 sizes)
- Total concept overhead: < 20KB beyond base firmware

### Known Limitations

1. Linear terminator (not curved like real moon) — acceptable for v1
2. No surface texture or craters — v1-expanded feature
3. No moonrise/moonset animation — simplified to opacity fade
4. Presets page uses text labels, not mini-moons — v1-expanded feature

## Compile-Proven Patterns Used

From Concepts 10-12:
- `ili9xxx` display with `model: GC9A01A`
- `cst816` touchscreen with `i2c_id` disambiguation
- `rotary_encoder` with 2-tick resolution
- `lvgl.page.show` with animation
- `lvgl.label.update` for text changes
- `lvgl.obj.update` for position/style changes
- Wake-only-first via `is_awake` global boolean
- `light.bedroom_group` HA import via `homeassistant` sensor
