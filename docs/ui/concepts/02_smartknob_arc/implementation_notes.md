# UI Concept 02: SmartKnob-Inspired Arc — Implementation Notes

## 1. ESPHome LVGL Arc Constraints

The primary challenge with implementing a SmartKnob-style arc in ESPHome using LVGL 9 is the opaque struct issue.

### The Opaque Struct Problem
In LVGL 9, widget structures (like `lv_arc_t`) are opaque. You cannot directly access their internal fields (e.g., `arc->value`) from an ESPHome lambda. Attempting to use `lv_arc_get_value(id(my_arc).obj)` results in a compilation error because the compiler doesn't know the size or layout of the struct.

### The Workaround: Visual-Only Arc
To bypass this, the arc in this concept is configured as **visual-only**.
- It does not accept touch input (`on_value` and `on_release` are not used for the arc).
- The brightness value is tracked in an ESPHome global variable (`brightness_pct`).
- When the physical rotary knob is turned, the global variable is updated.
- An ESPHome script or lambda then updates the arc's visual value using the safe ESPHome action `lvgl.arc.update`.

This perfectly aligns with the "knob-first" design philosophy of this concept.

## 2. Arc Styling in ESPHome

ESPHome's LVGL component uses specific keys for styling that differ slightly from raw LVGL C code.

To achieve the SmartKnob look:
- **Background Arc:** Styled using the `main` part of the arc widget.
  - `arc_color`: Dark gray or muted amber.
  - `arc_width`: Thick (e.g., 15-20px).
- **Foreground Arc (Fill):** Styled using the `indicator` part.
  - `arc_color`: Bright amber or white.
  - `arc_width`: Same as the background.
- **The "Knob" (Indicator):** Styled using the `knob` part.
  - `bg_color`: High contrast (e.g., white).
  - `pad_all`: Used to make the knob slightly larger than the arc width, giving it that distinct mechanical dial feel.

## 3. Wake-Only-First Implementation

The wake-only-first logic requires a dual-layer approach to prevent accidental triggers when the display is asleep.

1. **Global State:** A global boolean `display_awake` tracks the sleep state.
2. **Knob Handlers:** The `on_clockwise` and `on_anticlockwise` handlers first check `display_awake`. If false, they wake the display and return immediately. If true, they adjust the brightness.
3. **Touch Handlers:** The center label's `on_click` handler checks `display_awake`. However, because LVGL might process the touch event *before* the display fully wakes, a secondary latch (`touch_woke_this_cycle`) is used. If the touch was the event that woke the display, the click action is ignored.

## 4. Performance Considerations

- **Continuous Updates:** Updating the arc and text label on every click of the rotary encoder can cause visual stutter if the ESP32-S3 is busy handling Home Assistant API calls.
- **Debouncing:** To mitigate this, the Home Assistant `light.turn_on` call (which sends the new brightness) should be debounced or delayed slightly, while the local LVGL visual update happens instantly. This concept uses a short delay in the update script to prevent flooding the HA API.

## 5. Home Assistant Integration

The concept targets `light.bedroom_group`. It imports the state (ON/OFF) and brightness (0-255, mapped to 0-100%) from HA to keep the local display synchronized with the actual light state.
