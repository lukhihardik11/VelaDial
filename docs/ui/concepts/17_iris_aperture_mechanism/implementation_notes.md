# UI Concept 17: Iris Aperture Mechanism — Implementation Notes

## 1. Technical Constraints & LVGL Reality

The Iris Aperture Mechanism concept requires rendering overlapping, rotating polygonal blades. On the ESP32-S3 with LVGL 9.5.0 via ESPHome:
- **Rotating Polygons:** LVGL does not natively support rotating complex polygons or lines easily from pure YAML without custom C++ components or `lv_canvas`.
- **Pre-rendered Images:** Using 12-20 full-screen images for an animation sequence consumes significant flash memory and can cause loading delays.
- **The "Compile-Safe" Abstraction:** We must abstract the iris into LVGL-safe geometric primitives. We will simulate the aperture using a central expanding circle (the light) and a series of thick, overlapping arcs or lines to represent the blades.

## 2. The Abstraction Strategy (The "Octagonal Iris")

We will simulate an 8-blade iris (octagon) using a combination of a central expanding circle and 8 overlapping lines/arcs that form the inner edge of the blades.

### Widget Inventory

#### Page 1: Power
- `lv_obj` (Screen background - dark metallic gray)
- `lv_obj` (Central aperture - small when OFF, larger when ON)
- `lv_label` (Power Icon/Text)

#### Page 2: Brightness (Hero)
- `lv_obj` (Screen background - dark metallic gray representing the blade material)
- `lv_obj` (Central aperture - warm amber circle, size proportional to brightness)
- 8x `lv_line` or `lv_arc` (Blade edges - arranged in an octagon around the central aperture, moving inward/outward as brightness changes)
- `lv_label` (Brightness percentage, centered inside the aperture)

#### Page 3: Presets
- 4x `lv_btn` (Arranged in a 2x2 grid, representing aperture modes)
- 4x `lv_label` (Preset names: Wide Open, Portrait, Landscape, Pinhole)

## 3. Script Architecture

### `update_ui` Script
This script handles the translation of the HA brightness value (0-255) to the UI elements.
- **Aperture Size:** Map 0-100% brightness to the radius of the central amber circle.
  - 100% = Radius 110 (nearly full screen)
  - 5% = Radius 20 (pinhole)
- **Blade Edges:** The 8 lines/arcs representing the blade edges must be repositioned or resized to match the radius of the central aperture, maintaining the octagonal shape.
- **Label Update:** Update the `%` text.

### `update_led_ring` Script
Maps the brightness to the 5 WS2812 LEDs on GPIO48.
- Color: Amber (`r: 1.0, g: 0.6, b: 0.0`)
- Brightness: Proportional to HA brightness.

## 4. Font and Glyph Strategy

To avoid compile errors with duplicate characters in the `glyphs` string, we will use a strictly deduplicated set of characters for the Roboto font.
- Required characters: `0123456789% ONFWidepPrtatLnscPhl` (deduplicated).

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

- **Risk:** The abstracted iris looks like a simple expanding circle rather than a mechanical aperture.
- **Mitigation:** We will use 8 overlapping `lv_arc` widgets with flat ends to create an octagonal opening. By carefully setting the start/end angles and thickness of these arcs, we can simulate the overlapping blades of an iris. The background will be dark gray, and the central circle will be amber, creating the illusion of light shining through a mechanical opening.
- **Risk:** Performance lag when updating 8 arcs simultaneously during knob rotation.
- **Mitigation:** We will optimize the `update_ui` script to only update the necessary properties (radius/thickness) and avoid full page redraws. If lag is severe, we will reduce the blade count to 6 (hexagon).
