# Theme Selector UX

> **Note:** Knob button behavior uses `on_multi_click` exclusively (no `on_press`).
> Short press (<600ms + release) = page-specific action. Long press (>1.5s) = theme selector.
> Long press does NOT trigger short-press action.

## 1. Accessing the Selector
The Theme Selector is a hidden, device-level menu. It should not interfere with daily lighting control.
- **Action:** Long-press the rotary knob (hold for > 1.5 seconds).
- **Feedback:** The screen transitions to the Theme Selector page, and the LED ring pulses briefly to confirm entry.

## 2. Navigating Themes
Once inside the Theme Selector:
- **Action:** Rotate the knob clockwise (CW) or counter-clockwise (CCW).
- **Feedback:** The screen scrolls through the 20 available themes. The display updates immediately to show a preview of the highlighted theme.

## 3. The Selector Interface
The Theme Selector page will display:
- **Header:** "THEME SELECTOR" (small, top center)
- **Index:** "1 / 20" (medium, indicating position)
- **Theme Name:** e.g., "Minimal Thermostat" (large, center)
- **Instruction:** "Press to apply" (small, bottom center)

## 4. Applying a Theme
- **Action:** Short-press the rotary knob while a theme is highlighted.
- **Feedback:** 
  1. The `active_theme` global is updated.
  2. The `render_active_theme` script executes, applying the new visual style.
  3. The screen transitions back to the Power page (Page 0).
  4. The selection is saved to flash memory.

## 5. Exiting Without Changes
- **Action:** Swipe down, or wait for the 60-second idle timeout.
- **Feedback:** The screen returns to the Power page without changing the `active_theme`.

## 6. Home Assistant Integration
The active theme will be exposed to Home Assistant as a diagnostic text sensor (if supported by the ESPHome configuration) or simply managed locally. The primary control target remains `light.bedroom_group` regardless of the active theme.
