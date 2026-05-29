# UI Concept 15: Tree Ring Growth Pattern — Implementation Notes

## 1. Technical Constraints & LVGL Reality

The Tree Ring Growth Pattern concept requires rendering multiple concentric circles. On the ESP32-S3 with LVGL 9.5.0 via ESPHome:
- **Multiple Arcs:** Rendering 10-20 simultaneous `lv_arc` widgets can cause performance issues (frame drops) during animations.
- **Organic Textures:** True organic wood grain textures require image assets, which are difficult to manage purely in ESPHome YAML without external file hosting or complex C-array conversions.
- **The "Compile-Safe" Abstraction:** We must abstract the tree rings into LVGL-safe geometric primitives (concentric circles/arcs) that evoke the *feeling* of growth rings without requiring photorealistic textures.

## 2. The Abstraction Strategy

We will use a series of concentric `lv_arc` widgets (or `lv_obj` with borders) to represent the growth rings.
- **Ring Count:** We will use 10 rings. Each ring represents 10% brightness.
- **Ring Visibility:** As brightness increases, more rings become visible (opacity changes from 0 to 255).
- **Ring Colors:** The rings will use a gradient of colors from dark brown (inner) to light amber (outer).

### Widget Inventory

#### Page 1: Power
- `lv_obj` (Screen background)
- `lv_obj` (Central heartwood circle)
- `lv_label` (Power Icon/Text)

#### Page 2: Brightness (Hero)
- `lv_obj` (Screen background)
- 10x `lv_arc` (Concentric rings, radii from 20 to 110)
- `lv_obj` (Central heartwood)
- `lv_label` (Brightness percentage)

#### Page 3: Presets
- 4x `lv_btn` (Arranged in a 2x2 grid)
- 4x `lv_label` (Preset names: Noon, Golden, Morning, Dusk)

## 3. Script Architecture

### `update_ui` Script
This script handles the translation of the HA brightness value (0-255) to the UI elements.
- **Ring Visibility:** Map 0-100% brightness to the visibility of the 10 rings.
  - If brightness >= 10%, ring 1 is visible.
  - If brightness >= 20%, ring 2 is visible, etc.
- **Label Update:** Update the `%` text.

### `update_led_ring` Script
Maps the brightness to the 5 WS2812 LEDs on GPIO48.
- Color: Amber (`r: 1.0, g: 0.6, b: 0.0`)
- Brightness: Proportional to HA brightness.

## 4. Font and Glyph Strategy

To avoid compile errors with duplicate characters in the `glyphs` string, we will use a strictly deduplicated set of characters for the Roboto font.
- Required characters: `0123456789% ONF WarmSftNeulLwIigh` (deduplicated).

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

- **Risk:** 10 simultaneous arcs cause rendering lag.
- **Mitigation:** We will use `lv_obj` with `radius` and `border_width` instead of `lv_arc` if possible, as simple bordered circles are often faster to render than arcs. We will set `bg_opa: TRANSP` for the rings so only the borders are drawn.
- **Risk:** The UI looks like a target/bullseye instead of tree rings.
- **Mitigation:** We will use a carefully selected palette of browns and ambers, and vary the border widths slightly if possible, to make it feel more organic.
