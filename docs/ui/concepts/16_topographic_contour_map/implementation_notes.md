# UI Concept 16: Topographic Contour Map — Implementation Notes

## 1. Technical Constraints & LVGL Reality

The Topographic Contour Map concept requires rendering multiple thin, closely spaced lines. On the ESP32-S3 with LVGL 9.5.0 via ESPHome:
- **Irregular Lines:** Drawing true, wavy, irregular contour lines dynamically requires `lv_canvas` or complex `lv_line` arrays, which are highly memory-intensive and difficult to manage in pure YAML.
- **Multiple Arcs:** Rendering 15-20 simultaneous `lv_arc` widgets can cause performance issues.
- **The "Compile-Safe" Abstraction:** We must abstract the topographic map into LVGL-safe geometric primitives. We will use thin, concentric `lv_obj` circles with borders. To differentiate this from Concept 15 (Tree Rings), the lines will be thinner, more numerous, and styled with a technical, cartographic aesthetic.

## 2. The Abstraction Strategy

We will use a series of 12 concentric `lv_obj` widgets with thin borders to represent the contour lines.
- **Line Count:** 12 lines. Each line represents an "elevation threshold" (approx. 8.3% brightness per line).
- **Line Style:** Thin borders (1px or 2px), sharp contrast, technical amber colors.
- **The Summit:** The center will feature a distinct "summit marker" (a small triangle or crosshair) to reinforce the map metaphor.

### Widget Inventory

#### Page 1: Power
- `lv_obj` (Screen background - dark charcoal)
- `lv_obj` (Central summit marker)
- `lv_label` (Power Icon/Text)

#### Page 2: Brightness (Hero)
- `lv_obj` (Screen background)
- 12x `lv_obj` (Concentric contour lines, radii from 15 to 125)
- `lv_obj` (Central summit marker)
- `lv_label` (Elevation/Brightness percentage)

#### Page 3: Presets
- 4x `lv_btn` (Arranged in a 2x2 grid, representing terrain zones)
- 4x `lv_label` (Preset names: Summit, Ridge, Valley, Cave)

## 3. Script Architecture

### `update_ui` Script
This script handles the translation of the HA brightness value (0-255) to the UI elements.
- **Contour Visibility:** Map 0-100% brightness to the visibility of the 12 contour lines.
  - If brightness >= 8%, line 1 is visible.
  - If brightness >= 16%, line 2 is visible, etc.
- **Label Update:** Update the `%` text.

### `update_led_ring` Script
Maps the brightness to the 5 WS2812 LEDs on GPIO48.
- Color: Amber (`r: 1.0, g: 0.6, b: 0.0`)
- Brightness: Proportional to HA brightness.

## 4. Font and Glyph Strategy

To avoid compile errors with duplicate characters in the `glyphs` string, we will use a strictly deduplicated set of characters for the Roboto font.
- Required characters: `0123456789% ONF SumitRdgeValyCv` (deduplicated).

## 5. ESPHome YAML Structure

The YAML will follow the standard structure:
1. `substitutions`
2. `esphome`, `esp32`, `logger`, `api`, `ota`
3. `i2c`, `spi`, `display`, `touchscreen`
4. `sensor` (Rotary Encoder)
5. `binary_sensor` (Button)
6. `light` (LED Ring)
7. `globals` (State tracking, wake-only-first logic)
8. `script` (UI updates, HA sync)
9. `lvgl` (Pages, widgets, styles)

## 6. Known Risks & Mitigations

- **Risk:** 12 simultaneous bordered objects cause rendering lag.
- **Mitigation:** We will use `lv_obj` with `radius` and `border_width` instead of `lv_arc`, as simple bordered circles are faster to render. We will set `bg_opa: TRANSP` for the lines so only the borders are drawn.
- **Risk:** The UI looks exactly like Concept 15 (Tree Rings).
- **Mitigation:** We will use thinner lines (1px or 2px), a different color palette (technical amber on charcoal vs. organic browns), and a distinct central "summit marker" instead of a solid heartwood circle. The preset names (Summit, Ridge, Valley, Cave) will also reinforce the terrain metaphor.
