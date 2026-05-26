# Concept 01: Minimal Thermostat — Design Specification

## Visual Identity
The Minimal Thermostat concept borrows the clean, radial visual language of premium smart thermostats (like the Nest Learning Thermostat) but adapts it for bedroom light control. The identity relies on extreme reduction: a single, large, central value surrounded by a subtle, continuous arc. The background is pure OLED black to blend seamlessly with the physical bezel, creating the illusion of floating typography.

## Screen/Page Architecture
This concept uses a **1-page architecture**. Instead of swiping between Power, Brightness, and Presets, all primary interactions occur on a single unified screen. This reduces cognitive load and eliminates the need for horizontal navigation, which can be awkward on a small round display.

## Interaction Model

### Power Behavior
- **Action:** A single tap on the center of the screen toggles the `light.bedroom_group` power state.
- **Visual Feedback:** When off, the central value dims significantly and the arc disappears. When on, the value brightens and the arc illuminates.

### Brightness Behavior
- **Action:** Rotating the physical knob directly adjusts the brightness.
- **Visual Feedback:** The central value updates in real-time (e.g., "45%"). The surrounding arc fills proportionally to represent the brightness level.

### Presets Behavior
- **Action:** A long press on the physical knob cycles through the 4 locked presets (Warm White, Soft Amber, Neutral White, Low Nightlight).
- **Visual Feedback:** The central value briefly displays the preset name or an icon, and the arc color shifts to reflect the preset's color temperature.

### Sleep/Wake Behavior
- **Action:** The screen dims to 10% brightness after 60 seconds of inactivity.
- **Wake-Only-First:** Any input (touch, knob rotation, knob press) while asleep *only* wakes the screen. It does not trigger an action. A second deliberate input is required to interact.

### LED Ring Behavior
- The physical WS2812 LED ring mirrors the on-screen arc, providing a soft ambient glow that matches the current brightness and color temperature.

### Dark-Room Behavior
- The pure black background ensures minimal light bleed. The UI elements use warm, amber tones to avoid disrupting circadian rhythms.

## Why it feels premium
The premium feel comes from restraint. By removing all unnecessary UI chrome (buttons, borders, labels) and focusing entirely on a single, perfectly centered typographic element and a smooth, high-resolution arc, the interface feels intentional and sophisticated. It avoids the "cluttered dashboard" look common in DIY smart home projects.

## What is v1-compatible
- Controls `light.bedroom_group`
- Supports Power, Brightness, and 4 Presets
- Enforces wake-only-first logic
- Uses pure black background

## What is concept-only
- The 1-page architecture (v1 specifies 3 pages)
- Long-press for presets (v1 specifies a dedicated page)
- Arc color shifting (v1 specifies amber only)

## Hardik Approval Required
- Moving from a 3-page swipe model to a 1-page unified model.
- Using long-press for preset cycling instead of a dedicated menu.
