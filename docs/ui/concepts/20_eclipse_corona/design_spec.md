# Concept 20: Eclipse Corona — Design Specification

## 1. Visual Identity & Core Metaphor

**The Metaphor:** The round display IS the eclipsed sun. A dark moon disk sits at the center, providing a canvas for minimal text. The corona (glowing plasma streamers) radiates outward from behind the dark disk. 
- **Power ON** = Eclipse in progress (corona visible).
- **Power OFF** = No eclipse (dark sky, spindle/moon only).
- **Brightness** = Corona intensity and radius. At 100%, the corona is massive, bright, and warm. At 5%, it is a faint, thin sliver of light.
- **Presets** = Different eclipse phases or corona color temperatures (e.g., Solar Maximum vs. Annular Ring).

**Visual Identity:**
- **Aesthetic:** Captured cosmic wonder. Cinematic, premium, and calm.
- **Palette:** Deep space black (`0x000000`), warm amber (`0xFFB000`), pearl white (`0xF0E6D3`), and dark charcoal (`0x111111`).
- **Typography:** Roboto (clean, modern, unobtrusive).
- **Texture:** Soft radial gradients, glowing halos, and abstract streamer structures. No hard geometric lines (unlike Iris Aperture).

## 2. Why Eclipse Corona?

**Why it makes sense for lighting control:**
An eclipse is the ultimate natural manipulation of light. The corona is literally light escaping from behind an obstruction. By mapping the room's brightness to the corona's intensity, the UI perfectly mirrors the physical reality of the room.

**How to make it calm and residential:**
We avoid sci-fi tropes (no stars, no planets, no HUD elements). The UI is treated as an abstract art piece—a luminous halo on a dark background. The warm amber palette evokes a tube amplifier or a candle, not a spaceship.

**Differentiation:**
- **vs. Lunar Phase (Concept 13):** Lunar Phase shows the illuminated surface of a sphere (crescent to full). Eclipse Corona shows light radiating from *behind* a dark sphere.
- **vs. Iris Aperture (Concept 17):** Iris Aperture is mechanical, metallic, and geometric. Eclipse Corona is organic, soft, and luminous.

## 3. Screen Architecture (3-Page Layout)

### Page 0: Power
- **Visual:** A dark moon disk. When OFF, the screen is mostly black with a faint, barely-there ring. When ON, a dramatic "diamond ring" flash occurs, settling into the active corona.
- **Text:** "ECLIPSE ACTIVE" (ON) vs "TOTALITY" (OFF).

### Page 1: Brightness Hero
- **Visual:** The main interactive corona. As the knob turns, the corona expands and brightens.
- **Implementation:** 6-8 concentric `lv_obj` circles with decreasing opacity and increasing size, creating a smooth radial glow. The innermost ring is brightest (amber/white), fading to dark amber at the edges.
- **Text:** Large percentage readout in the dark center.

### Page 2: Presets
- **Visual:** Four smaller "flare" zones or miniature eclipses arranged radially.
- **Mapping:**
  - **Warm White (80%):** Solar Maximum (large, warm amber glow)
  - **Soft Amber (60%):** Golden Corona (medium, rich amber)
  - **Neutral White (90%):** Full Totality (large, cool white)
  - **Low Nightlight (15%):** Partial Eclipse (tiny, faint glow)
- **Interaction:** Tapping a zone triggers the preset.

## 4. Interaction & Hardware Mapping

- **Knob Rotation:** Expands/contracts the corona radius and opacity. 5% per detent.
- **Knob Press:** Toggles power. Triggers the "diamond ring" ignition flash.
- **Touch:** Swipe left/right to navigate pages. Tap preset zones.
- **LED Ring:** Acts as the outer corona halo, projecting warm amber light onto the physical wall, extending the eclipse beyond the screen.
- **Wake/Sleep:** Wake-only-first enforced. Wake animation is the corona "igniting" from nothing. Sleep is a slow fade to black.

## 5. LVGL Compromises & Feasibility

True volumetric plasma rendering is impossible on the ESP32-S3 at 240x240.
**The Compromise:** We use concentric `lv_obj` circles with stepped opacity (e.g., 100%, 80%, 60%, 40%, 20%, 10%) to simulate a radial gradient. We use `lv_arc` widgets with varying start/end angles and opacities to simulate asymmetric coronal streamers. This provides the *illusion* of a complex glowing corona while remaining 100% compile-safe and performant.

## 6. Classification
- **v1-compatible:** Yes, using the concentric circle/arc approximation.
- **Concept-only (v2):** Real-time generative plasma shaders, animated streamer undulation.
- **Hardware Validation Needed:** Testing the concentric circle glow effect for banding/aliasing on the physical GC9A01 display. Testing the LED ring wall-projection effect.
