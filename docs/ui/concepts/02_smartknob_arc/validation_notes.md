# UI Concept 02: SmartKnob-Inspired Arc — Validation Notes

This document outlines the specific tests required when this concept is deployed to the physical ELECROW CrowPanel 1.28" ESP32-S3 Rotary Touch Display.

## 1. Visual Validation

- **Arc Rendering:** Does the arc render smoothly without jagged edges? Is the thickness appropriate for the 240x240 screen?
- **Knob Indicator:** Is the "knob" part of the arc clearly visible and distinct from the indicator fill? Does it look like a mechanical dial?
- **Typography:** Is the central brightness/power label crisp and readable from a typical bedside distance (e.g., 2-3 feet)?
- **Dark-Room Contrast:** In a pitch-black room, is the 10% sleep backlight dim enough to avoid light pollution? When awake, is the contrast between the amber arc and black background comfortable?

## 2. Interaction Validation

- **Knob Rotation (Responsiveness):** When turning the physical knob, does the arc update instantly? Is there any noticeable lag or stutter?
- **Knob Rotation (Direction):** Does clockwise rotation increase brightness and counter-clockwise decrease it? (The encoder pins may need swapping depending on the specific hardware revision).
- **Knob Press:** Does pressing the knob reliably toggle the power state?
- **Center Touch:** Does tapping the center label toggle the power state? Is the touch target area large enough?
- **Arc Touch:** Verify that touching or dragging the arc does **not** change the brightness (as per the visual-only design spec).

## 3. Wake-Only-First Validation

This is a critical safety feature to prevent accidental light changes in the middle of the night.

- **Test 1 (Knob Turn):** Let the display sleep (dim to 10%). Turn the knob one click. The display should wake to 100% brightness, but the light's actual brightness should **not** change.
- **Test 2 (Knob Press):** Let the display sleep. Press the knob. The display should wake, but the light should **not** toggle power.
- **Test 3 (Touch):** Let the display sleep. Tap the center label. The display should wake, but the light should **not** toggle power.

## 4. Home Assistant Synchronization

- **State Import:** If the light is turned on/off via the Home Assistant app, does the VelaDial display update to reflect the new state within 1-2 seconds?
- **Brightness Sync:** If the brightness is changed via the HA app, does the arc and central label update to match?
- **Command Flooding:** When spinning the knob rapidly, does the ESP32 remain responsive? Check the HA logs to ensure it is not being flooded with hundreds of API calls per second.

## 5. Hardware-Specific Checks

- **LED Ring:** Does the WS2812 LED ring correctly mirror the power state (Amber when ON, off when OFF)? Are the colors correct (GRB vs RGB order)?
- **Backlight PWM:** Does the backlight smoothly transition between 10% and 100%, or does it flicker?
