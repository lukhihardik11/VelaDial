# UI Concept 14: Sundial Shadow UI — Implementation Notes

## 1. Technical Constraints & LVGL Reality

The Sundial Shadow UI concept relies heavily on soft shadows, gradients, and dynamic geometry. However, the target hardware (ESP32-S3) and software (LVGL 9.5.0 via ESPHome) impose strict limitations:
- **Real-time Blur/Shadows:** LVGL 9.5 supports software-rendered drop shadows, but animating them dynamically (changing size/opacity every frame) causes severe CPU spikes and frame drops on the ESP32-S3.
- **Complex Gradients:** While LVGL 9.2+ supports radial and conical gradients, rendering them dynamically on a 240x240 canvas can be slow.
- **Arc Shadows:** `lv_arc` does not natively support shadows.

## 2. The "Compile-Safe" Approximation Strategy

To ensure a `COMPILE STATUS: PASS` and maintain a responsive UI, we must abstract the sundial shadow into LVGL-safe primitives.

### The Visual Metaphor Translation
Instead of a photorealistic shadow, we will use a **geometric shadow abstraction**:
1. **The Gnomon:** A small, static central circle (`lv_obj` with radius).
2. **The Shadow:** An `lv_arc` that acts as a "shadow wedge." 
   - At 100% brightness (Noon), the arc is very short (e.g., 10 degrees).
   - At 10% brightness (Twilight), the arc is very wide (e.g., 180 degrees).
   - The arc's color will be a dark, semi-transparent charcoal/amber to simulate a shadow cast on the background.
3. **The Background:** The main screen background color will shift based on brightness (bright amber for high, dark charcoal for low).

### Widget Inventory

#### Page 1: Power
- `lv_obj` (Screen background)
- `lv_label` (Large Power Icon/Text)
- `lv_obj` (Central gnomon representation)

#### Page 2: Brightness (Hero)
- `lv_obj` (Screen background, color shifts with brightness)
- `lv_arc` (The "Shadow Wedge" - width inversely proportional to brightness)
- `lv_obj` (Central gnomon)
- `lv_label` (Brightness percentage, centered, high contrast)

#### Page 3: Presets
- 4x `lv_btn` (Arranged in a 2x2 grid or quadrants)
- 4x `lv_label` (Preset names: Warm White, Soft Amber, Neutral White, Low Nightlight)

## 3. Script Architecture

We will reuse the proven script architecture from Concept 13, modifying only the visual update logic.

### `update_ui` Script
This script handles the translation of the HA brightness value (0-255) to the UI elements.
- **Background Color:** Map 0-255 to a color range (e.g., Dark Charcoal to Bright Amber).
- **Shadow Arc Width:** Map 0-255 inversely to arc degrees. 
  - 255 (100%) -> 10 degree arc.
  - 25 (10%) -> 180 degree arc.
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

- **Risk:** The `lv_arc` shadow abstraction looks too much like a standard progress bar.
- **Mitigation:** We will invert the logic (arc grows as brightness decreases) and use a dark, semi-transparent color for the arc against a bright background, reinforcing the "shadow" metaphor rather than a "fill" metaphor.
- **Risk:** LVGL component download timeout.
- **Mitigation:** If the Espressif registry fails, we will manually download the component, though incremental builds usually bypass this issue.
