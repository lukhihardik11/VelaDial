# Concept 01: Minimal Thermostat — Validation Notes

## Hardware Validation Requirements

This concept prototype requires physical hardware validation (Step 15B) to confirm the following specific behaviors:

### 1. Display Quality and Contrast
- **Black Levels:** The concept relies heavily on a pure black background to create a seamless, floating UI effect. We must validate how deep the black levels are on the GC9A01A IPS panel. If the backlight bleed is too high, the "floating" effect will be ruined, and the screen edges will be visible in a dark room.
- **Viewing Angles:** The central label must remain legible from off-axis viewing angles, typical of a wall-mounted or bedside device.

### 2. Interaction Ergonomics
- **Knob Sensitivity:** The mapping of physical detents to the on-screen arc fill and brightness percentage must feel natural. We need to validate if 1 detent = 5% brightness is too fast or too slow for this specific 1-page design.
- **Long-Press Reliability:** The custom long-press logic for cycling presets must be tested for reliability. Is the timing (e.g., 800ms) intuitive? Does it accidentally trigger when the user meant to do a short press?
- **Touch Target Size:** The central label acts as the power toggle. We must ensure the touch target area is large enough to hit reliably without looking, especially in the dark.

### 3. Performance
- **Arc Rendering:** We must verify that the arc updates smoothly without tearing or lagging when the knob is turned rapidly.
- **Font Rendering:** Ensure the large Roboto 48 font renders crisply without jagged edges on the 240x240 display.

## Home Assistant Validation

- **State Synchronization:** Verify that the 1-page UI correctly reflects state changes initiated from the Home Assistant app (e.g., if the light is turned off via the app, the central label should dim and the arc should disappear on the VelaDial).
