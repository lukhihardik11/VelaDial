# UI Concept 02: SmartKnob-Inspired Arc — Design Specification

## 1. Visual Identity
This concept borrows heavily from the open-source SmartKnob project and premium automotive/audio dials. It moves away from the flat, static look of Concept 01 and introduces a more dynamic, instrument-like feel. The core identity relies on a thick, segmented-looking arc (or a continuous arc with a distinct indicator knob) that visually communicates "rotary control" rather than just "progress bar."

## 2. Screen/Page Architecture
**Single-Page Architecture.**
To maintain focus and reduce navigation friction, this concept uses a single unified page. The entire 240×240 canvas is dedicated to the primary light control task.

## 3. Core Elements
- **Central Value Display:** A large, crisp numeric display showing the current brightness percentage (e.g., "45%"). When off, it displays "OFF".
- **The SmartKnob Arc:** A prominent circular arc tracking the outer edge of the display.
  - **Background Arc:** A subtle, low-opacity track (e.g., dark gray/amber).
  - **Foreground Arc (Indicator):** A bright, high-contrast fill representing the current brightness level.
  - **Knob/Handle:** A distinct visual element at the leading edge of the foreground arc, simulating a physical dial indicator.

## 4. Interaction Behavior

### Power Behavior
- **Knob Press:** Toggles the light power state (ON/OFF).
- **Center Touch:** Toggles the light power state (ON/OFF).

### Brightness Behavior
- **Knob Rotation (CW/CCW):** Adjusts brightness in 5% increments. The arc and central number update immediately to reflect the new value.
- **Arc Touch/Drag:** Explicitly **disabled** in this concept. The arc is a visual output only. This avoids the LVGL 9 opaque struct issue and reinforces the "knob-first" interaction model.

### Presets Behavior
- **Not implemented** in this single-page concept to maintain absolute minimalism. (Could be added via a long-press or swipe in a future iteration).

### Sleep/Wake Behavior (Wake-Only-First)
- **Idle Timeout:** After 60 seconds of no interaction, the display dims to a minimal backlight level (e.g., 10%) to prevent light pollution in a dark bedroom.
- **Wake Action:** Any input (knob turn, knob press, screen touch) while asleep will **only** wake the display to full brightness. It will **not** trigger a power toggle or brightness change.
- **Active State:** Once awake, subsequent inputs perform their normal functions.

### LED Ring Behavior
- The WS2812 LED ring mirrors the power state (Amber glow when ON, off when OFF).

### Backlight Behavior
- Controlled by the sleep/wake logic (100% awake, 10% asleep).

### Unavailable/Error State
- If the Home Assistant connection is lost, the central text changes to "N/A" or "ERR", and the arc color shifts to a muted gray or red to indicate the disconnected state.

## 5. Typography and Spacing
- **Font:** Uses a clean, highly legible sans-serif font (e.g., Roboto or Montserrat, depending on what is compiled into the ESPHome font file).
- **Size:** The central number is massive (e.g., 60-80px) for instant readability from a distance.
- **Spacing:** The arc is pushed to the absolute edge of the 240px display to maximize the central canvas and emphasize the circular nature of the hardware.

## 6. Dark-Room Behavior & Daylight Readability
- **Dark-Room:** The deep black background (OLED-style) ensures the display doesn't glow unnecessarily. The amber/warm-white arc provides enough contrast without blinding the user. The 10% sleep backlight is crucial here.
- **Daylight:** The high-contrast foreground arc and large white text ensure readability even in bright ambient light.

## 7. Motion Concept
- While ESPHome LVGL animation capabilities are limited, the arc fill should feel responsive and smooth when the knob is turned, mimicking the immediate feedback of a physical dial.

## 8. Premium Feel & Differentiation
- **Why it feels premium:** It mimics high-end audio equipment and automotive gauges. The distinct "knob" on the arc gives a sense of physical weight and precision.
- **Differentiation from generic arcs:** The focus on the leading-edge indicator (the "knob" on the arc) rather than just a solid fill makes it feel like a mechanical dial translated to a screen.

## 9. Scope & Constraints
- **v1-Compatible:** Yes. It adheres to the single-entity control (light.bedroom_group) and wake-only-first rules.
- **Concept-Only:** The lack of presets makes it a pure concept exploration. A production version would need a way to access presets.
- **Requires Hardik Approval:** The decision to drop presets for the sake of minimalism, and the specific visual styling of the arc.
