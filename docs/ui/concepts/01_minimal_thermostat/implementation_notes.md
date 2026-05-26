# Concept 01: Minimal Thermostat — Implementation Notes

## LVGL Feasibility
The Minimal Thermostat concept is highly feasible within ESPHome's LVGL implementation. The core visual elements—a central label and a surrounding arc—are native LVGL widgets (`lv_label` and `lv_arc`).

## Technical Approach

### 1-Page Architecture
Implementing a single page simplifies the LVGL setup. We only need one `page` component. This avoids the complexity of swipe gestures and page transitions, reducing memory overhead and potential rendering glitches on the ESP32-S3.

### The Arc Widget
- **Widget:** `arc`
- **Styling:** The arc will be styled to be relatively thin (e.g., 8-12 pixels) to maintain a minimal aesthetic. The background arc will be a dark gray, and the indicator arc will be a solid color (e.g., amber).
- **Interaction:** The arc will be set to read-only (`adjustable: false`) in LVGL. Direct touch manipulation of the arc is disabled to prevent accidental changes and to enforce the knob as the primary input for brightness.

### The Central Label
- **Widget:** `label`
- **Styling:** A large, bold font (e.g., Roboto 48) will be used. The label will be perfectly centered within the arc.
- **Dynamic Updates:** The label text will dynamically update based on the `brightness_pct` global variable.

### Knob Integration
The physical rotary encoder will directly update the `brightness_pct` global variable. An `on_value` trigger on the encoder will update the LVGL arc value and the central label text simultaneously, ensuring a responsive feel.

### Wake-Only-First Logic
The dual-layer wake guard from the production YAML will be retained.
1. The `touchscreen` component's `on_touch` trigger will wake the display.
2. The central label's `on_click` trigger will check the `touch_woke_this_cycle` latch before executing the power toggle.

## Risks and Constraints

### Font Rendering
Large fonts (size 48+) consume significant flash memory. We must ensure the font file only includes the necessary glyphs (numbers 0-9 and the % symbol) to minimize the binary size.

### Arc Value Synchronization
As discovered in the production YAML, directly reading the arc value via C++ lambdas (`lv_arc_get_value`) is problematic in LVGL 9 due to opaque structs. Therefore, the arc will act purely as a visual indicator, driven by the `brightness_pct` global variable, rather than acting as the source of truth.

### Long-Press Detection
ESPHome's `rotary_encoder` component does not natively support long-press detection on the switch pin. We will need to implement a custom script using `on_press` and `on_release` with a delay to detect a long press for cycling presets.
