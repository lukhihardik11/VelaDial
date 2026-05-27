# Concept 03: Large Center Power Button — Implementation Notes

## LVGL Widget Strategy

The prototype uses standard ESPHome LVGL widgets without custom drawing or external assets. The implementation centers on the `btn` widget for the power button and `arc` widget for the brightness page, with `label` widgets for text and `obj` widgets for the page indicator dots and preset circles.

### Power Page Widgets

The power button is implemented as an `lv_btn` widget with `radius: 70` (half of 140px width) to create a perfect circle. The glow effect is approximated using the LVGL `shadow_width` and `shadow_opa` style properties, which create a soft halo around the button without requiring custom rendering. The button contains a centered label showing "ON" or "OFF".

In ESPHome LVGL YAML, the button is defined with:
- `width: 140`, `height: 140`, `align: CENTER`
- `radius: 70` (creates perfect circle)
- `bg_color: 0xFFA500` (amber when on) or `0x1A1A1A` (dark when off)
- `shadow_width: 20`, `shadow_color: 0xFFA500`, `shadow_opa: 100` (glow when on)
- `border_width: 2`, `border_color: 0x555555` (ghost outline when off)

### Brightness Page Widgets

The brightness page uses a smaller center circle (100px `obj` or `btn`) displaying the percentage value, surrounded by a standard `arc` widget. The arc is visual-only (not touch-adjustable) — brightness is controlled exclusively via the rotary encoder.

### Presets Page Widgets

Four `btn` widgets (55px diameter each) positioned manually in a diamond pattern:
- Top: `align: CENTER`, `y: -60`
- Left: `align: CENTER`, `x: -60`
- Right: `align: CENTER`, `x: 60`
- Bottom: `align: CENTER`, `y: 60`

Each button has `radius: 28` (half of 55px, rounded) for a circular shape.

## Wake-Only-First Implementation

The dual-layer wake guard is implemented identically to the production YAML and previous concepts:

1. **Layer 1 (Global `display_awake`):** All knob handlers (CW, CCW, press) check `display_awake` first. If false, they wake the display and return without acting.

2. **Layer 2 (Touch latch `touch_woke_this_cycle`):** The `touchscreen: on_touch` handler sets both `display_awake = true` and `touch_woke_this_cycle = true` when waking. All LVGL widget event handlers (`on_click`, `on_value`) check this latch and refuse to act if it is set. The latch is cleared after a 500ms delay.

3. **LVGL `resume_on_input: false`:** Prevents LVGL from auto-resuming the display and simultaneously dispatching widget events on the same input that wakes.

## Page Navigation

Page navigation uses the ESPHome LVGL swipe gesture system. The `on_swipe` events on each page trigger page transitions:
- Left swipe: advance to next page
- Right swipe: return to previous page

The 3-dot page indicator is implemented as three small `obj` widgets (8px diameter) positioned at the bottom of each page, with the active dot colored amber and inactive dots colored dark gray. The dots are updated via `lvgl.widget.update` when the page changes.

**Note:** For the prototype, page navigation is simplified. The production YAML's full swipe implementation with page indicator is the reference. The prototype focuses on demonstrating the visual concept (large button, glow, diamond presets) rather than reimplementing the full navigation system.

## State Synchronization with Home Assistant

The prototype imports the `light.bedroom_group` state via `text_sensor` with `platform: homeassistant`. When the HA entity state changes externally (e.g., from another controller), the UI updates to reflect the new state:
- State "on" → button turns amber with glow, label shows "ON"
- State "off" → button turns dark with ghost outline, label shows "OFF"
- Brightness attribute → arc and percentage label update

Outbound commands use `homeassistant.service` calls for `light.toggle`, `light.turn_on` (with `brightness_pct`), and preset application.

## Performance Considerations

The shadow/glow effect (`shadow_width: 20`) adds a small rendering cost when the button is first drawn or when its style changes. However, since the shadow is static (not animated continuously), it only costs rendering time during the toggle transition. LVGL caches the rendered widget after the style change completes, so there is no ongoing performance impact.

The diamond preset layout with four 55px buttons is well within LVGL's rendering budget for the ESP32-S3 with PSRAM. No performance concerns are expected.

## Prototype Simplifications

For compile verification, the prototype makes the following simplifications compared to the full design spec:

1. **Single page only:** The prototype implements only the Power page to demonstrate the hero element. The Brightness and Presets pages are described in the design spec but not implemented in the prototype YAML (to keep compile scope manageable and focused).

2. **No animation timing:** The toggle transition is instantaneous (style update) rather than animated over 200ms. LVGL animations require `lv_anim` which is complex to configure in ESPHome YAML.

3. **Shadow approximation:** The glow effect uses `shadow_width` and `shadow_opa` which may render differently on the actual GC9A01A panel. Physical validation is required to tune the values.

## Key ESPHome LVGL Syntax Notes

Based on compile experience from Concepts 01 and 02:
- Use `displays:` (plural) in the `lvgl:` config block
- Use `resume_on_input: false` for wake-only-first
- Arc uses `start_angle`/`end_angle` (not `bg_angle_start`/`bg_angle_end`)
- Button `on_click` handlers can use lambdas for wake-guard checks
- `lvgl.widget.update` can change `bg_color`, `shadow_width`, `shadow_opa`, `border_width` dynamically
- Font references use the `id` defined in the `font:` section
- `to_string()` is available in lambdas for int-to-string conversion
