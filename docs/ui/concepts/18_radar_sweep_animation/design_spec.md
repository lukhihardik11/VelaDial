# Concept 18: Radar Sweep Animation — Design Specification

## 1. Visual Identity & Core Metaphor

**The Metaphor:** The round display IS a futuristic scanning instrument or radar scope. A bright amber sweep line rotates around the center, scanning the environment. "Blips" appear on the scope to represent system states or presets. The speed of the sweep line maps to the brightness of the light.

**Visual Language:**
- **Sweep Line:** A bright amber radial line extending from the center to the edge.
- **Afterglow Trail:** A fading amber gradient trailing behind the sweep line, simulating phosphor decay.
- **Range Rings:** Faint, thin concentric circles providing structural depth to the scope.
- **Blips:** Solid amber circles or dots representing data points (power state, presets).
- **Background:** Deep charcoal (`0x111111`) to deep metallic gray (`0x222222`), creating a sense of depth and contrast.
- **Typography:** `Roboto Mono` for a technical, precise, instrument-like feel.

**Why Radar/Sweep Makes Sense for Lighting Control:**
While not a direct physical mapping like the iris aperture (Concept 17), the radar sweep metaphor taps into the idea of "environmental awareness" and "ambient status." Lighting is an ambient environmental factor. A radar sweep constantly scanning the room creates a feeling of active monitoring and presence. Mapping brightness to sweep speed (or intensity) provides a dynamic, mesmerizing visual feedback loop that feels highly advanced and premium.

**Making it Calm and Residential:**
To avoid looking like a military radar or an aggressive video game HUD:
- **Color Palette:** Strictly adhere to the warm amber palette (`0xFFB000`). Avoid military greens or aggressive reds.
- **Animation Speed:** Keep the sweep speed relatively slow and smooth. Even at 100% brightness, it should feel like a deliberate, calm scan, not a frantic spinning blade.
- **Minimalism:** Use negative space generously. Keep range rings faint and unobtrusive. Avoid unnecessary grid lines, crosshairs, or dense data overlays.
- **Softness:** Use gradients and opacity for the afterglow trail to create a soft, ethereal glow rather than harsh, sharp edges.

## 2. Screen Architecture (3-Page Layout)

The UI follows the locked v1 3-page structure, utilizing horizontal swipes for navigation.

### Page 0: Power (Scanner State)
- **Visual:** The radar scope is either active (scanning) or inactive (dark/frozen).
- **ON State:** The sweep line is visible and rotating (or positioned to indicate active state). A large central "blip" or text indicates "ON".
- **OFF State:** The sweep line is hidden or frozen at 12 o'clock. The scope is dark. A dim central indicator shows "OFF".
- **Interaction:** Pressing the knob toggles power.

### Page 1: Brightness Hero (Sweep Speed/Intensity)
- **Visual:** The core radar experience. The sweep line rotates around the center.
- **Mapping:** Brightness percentage maps to the *speed* of the sweep rotation (or the *intensity/length* of the sweep trail if continuous animation is too costly).
  - 100% = Fast sweep (e.g., 1 rev / 2 sec) or full 360° bright trail.
  - 50% = Medium sweep (e.g., 1 rev / 5 sec) or 180° trail.
  - 5% = Slow sweep (e.g., 1 rev / 10 sec) or tiny 15° trail.
- **Overlay:** A large, crisp percentage value (`Roboto Mono`) sits at the center of the scope.
- **Interaction:** Rotating the knob adjusts brightness (and thus sweep speed/intensity). Pressing the knob returns to the Power page.

### Page 2: Presets (Scan Modes / Targets)
- **Visual:** The radar scope displays four distinct "blips" or target zones, typically arranged in quadrants or a grid.
- **Mapping:** Each blip represents a preset.
  - Top Left: Warm White (Wide Scan)
  - Top Right: Soft Amber (Focused Scan)
  - Bottom Left: Neutral White (Daylight Scan)
  - Bottom Right: Low Nightlight (Stealth Scan)
- **Active State:** The currently selected preset blip pulses or is highlighted brightly, while others remain dim.
- **Interaction:** Pressing the corresponding on-screen button applies the preset.

## 3. Hardware Integration

### Knob Behavior
- **Rotate (Page 1):** Adjusts brightness (5% increments).
- **Press (Any Page):** Toggles power (ON/OFF).
- **Wake:** Rotating or pressing the knob while asleep wakes the display without executing a command (wake-only-first).

### Touch Behavior
- **Swipe Left/Right:** Navigates between the 3 pages.
- **Tap (Page 2):** Selects a preset.
- **Wake:** Touching the screen while asleep wakes the display.

### LED Ring Behavior (Lens Rim / Scanner Glow)
- **V1 Locked:** The 5-LED ring acts as a proportional ambient glow, matching the brightness of the main light (warm amber).
- **V1 Expanded (Concept):** The LEDs could illuminate sequentially in a circle, synchronized with the on-screen radar sweep line, creating a physical extension of the scanning animation.

### Backlight Behavior
- Wakes to 80% brightness for crisp visibility.
- Sleeps after 45 seconds of inactivity (fades to 0%).

## 4. Environmental Considerations

### Dark-Room Behavior
- The deep charcoal background (`0x111111`) minimizes light bleed.
- The amber sweep line provides a warm, non-disruptive glow.
- **Risk:** Continuous animation in a dark room can be distracting. The sweep intensity must scale down significantly at low brightness levels to maintain a calm environment.

### Daylight Readability
- The high-contrast amber on black ensures the sweep line and percentage text remain legible in bright ambient light.
- Faint range rings may wash out, but the core information (sweep line, text) will survive.

## 5. Why It Feels Premium & Unique

- **Uncommon Metaphor:** Radar/scanning interfaces are rarely used for simple lighting control. It elevates the device from a "switch" to an "instrument."
- **Mesmerizing Motion:** The continuous (or simulated continuous) sweep provides a hypnotic, ambient visual that draws the eye without demanding interaction.
- **Technical Elegance:** The use of `Roboto Mono`, precise geometry, and high-contrast amber creates a sophisticated, engineered aesthetic reminiscent of high-end automotive HUDs or luxury watch complications.

## 6. V1 Compatibility & Approvals

- **V1 Compatible:** Yes, the 3-page structure and locked presets adhere to v1 constraints.
- **Concept-Only:** The continuous 30 FPS sweep animation is highly experimental and likely too demanding for the ESP32-S3 in a production environment.
- **Requires Hardik Approval:** The specific implementation of the sweep (continuous rotation vs. discrete stepped positions vs. static arc length) depends heavily on hardware performance testing.
- **Hardware Validation Needed:** The performance impact of the sweep animation (frame drops, CPU load, responsiveness to knob input during animation) must be rigorously tested on the physical ELECROW display.
