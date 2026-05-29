# Concept 19: Vinyl DJ Crossfader — Design Specification

## 1. Visual Identity & Core Metaphor

**The Metaphor:** The round display IS a premium vinyl turntable platter viewed from above. The interface borrows the tactile, analog warmth of high-end audio equipment (like Bang & Olufsen or a classic Technics SL-1200) to control light. 

**Visual Language:**
- **The Platter:** A deep charcoal/black background representing the vinyl record.
- **The Grooves:** Concentric, faint gray arcs that simulate the texture of a record.
- **The Tonearm/Needle:** A distinct, warm amber indicator that moves across the grooves.
- **The Center Label:** A central circular area containing the primary data readout (percentage or state).
- **Typography:** `Roboto` for clean, modern legibility, avoiding overly stylized "DJ" fonts to maintain a residential feel.
- **Color Palette:** Deep blacks (`0x111111`), subtle grays (`0x333333` for grooves), and warm amber (`0xFFB000`) for active elements.

**Why Vinyl/Audio Control Makes Sense for Lighting:**
Both audio and lighting are ambient environmental factors that require smooth, continuous adjustment. The physical act of turning a knob to adjust volume is universally understood; mapping this to brightness leverages the same muscle memory. The turntable metaphor adds a layer of tactile satisfaction and premium nostalgia.

**Making it Calm and Residential:**
To avoid looking like a nightclub toy or a generic media player:
- **No Play/Pause Icons:** We strictly avoid standard media controls. This is an instrument, not a Spotify player.
- **Subtle Textures:** The vinyl grooves are faint and elegant, not high-contrast zebra stripes.
- **Warmth over Flash:** We use amber instead of neon colors or RGB rainbows. The aesthetic is "late-night listening room," not "EDM festival."

## 2. Screen Architecture (3-Page Layout)

The UI follows the locked v1 3-page structure, utilizing horizontal swipes for navigation.

### Page 0: Power (The Platter State)
- **Visual:** The center label displays the power state.
- **ON State:** The center label shows "SPINNING" or "ACTIVE" in amber. The needle is positioned on the record.
- **OFF State:** The center label shows "STOPPED" in dim gray. The needle is lifted or hidden.
- **Interaction:** Pressing the knob toggles power (starts/stops the platter).

### Page 1: Brightness Hero (The Groove Intensity)
- **Visual:** The core turntable experience. The screen displays concentric vinyl grooves.
- **Mapping:** Brightness percentage maps to the position of the "tonearm" (an amber indicator) across the grooves.
  - 100% = Needle at the outermost groove (full volume/brightness).
  - 5% = Needle at the innermost groove near the label.
- **Overlay:** A large, crisp percentage value sits on the center label.
- **Interaction:** Rotating the knob moves the needle across the grooves, adjusting brightness. Pressing the knob returns to the Power page.

### Page 2: Presets (The Track Selection)
- **Visual:** The record is divided into four distinct "tracks" (bands of grooves separated by slight gaps, mimicking a real LP).
- **Mapping:** Each track represents a preset.
  - Track 1 (Outer): Warm White
  - Track 2: Soft Amber
  - Track 3: Neutral White
  - Track 4 (Inner): Low Nightlight
- **Active State:** The selected track is highlighted in amber, while others remain dim gray.
- **Interaction:** Pressing the corresponding on-screen button "drops the needle" on that track, applying the preset.

## 3. Hardware Integration

### Knob Behavior
- **Rotate (Page 1):** Moves the needle across the grooves (adjusts brightness in 5% increments).
- **Press (Any Page):** Toggles power (starts/stops the platter).
- **Wake:** Rotating or pressing the knob while asleep wakes the display without executing a command (wake-only-first).

### Touch Behavior
- **Swipe Left/Right:** Navigates between the 3 pages.
- **Tap (Page 2):** Selects a preset (drops the needle on a track).
- **Wake:** Touching the screen while asleep wakes the display.

### LED Ring Behavior (The Platter Rim)
- **V1 Locked:** The 5-LED ring acts as a proportional ambient glow, matching the brightness of the main light (warm amber).
- **V1 Expanded (Concept):** The LEDs could simulate the strobe dots on a Technics platter, providing a subtle visual indication of the "spin speed" or simply glowing warmly like a tube amplifier.

### Backlight Behavior
- Wakes to 80% brightness for crisp visibility.
- Sleeps after 45 seconds of inactivity (fades to 0%).

## 4. Environmental Considerations

### Dark-Room Behavior
- The deep charcoal background (`0x111111`) minimizes light bleed.
- The amber needle and center label provide a warm, non-disruptive glow.
- The faint gray grooves (`0x333333`) fade into the background in low light, leaving only the essential controls visible.

### Daylight Readability
- The high-contrast amber on black ensures the needle and percentage text remain legible in bright ambient light.
- The groove texture provides visual interest without compromising the readability of the center label.

## 5. Why It Feels Premium & Unique

- **Audiophile Aesthetic:** It taps into the luxury and precision associated with high-end analog audio gear.
- **Tactile Metaphor:** The visual of a needle moving across grooves perfectly complements the physical sensation of turning a rotary encoder.
- **Not a Media Player:** By avoiding standard play/pause iconography, it remains a dedicated environmental controller, avoiding confusion with music apps.

## 6. V1 Compatibility & Approvals

- **V1 Compatible:** Yes, the 3-page structure and locked presets adhere to v1 constraints.
- **Concept-Only:** True rotation animation of the vinyl grooves (to simulate spinning) is highly experimental and likely too demanding for the ESP32-S3 in a production environment.
- **Requires Hardik Approval:** The specific implementation of the "needle" (an arc segment vs. a rotating line) depends heavily on hardware performance testing.
- **Hardware Validation Needed:** The visual fidelity of the concentric grooves (aliasing issues on a 240x240 screen) must be rigorously tested on the physical ELECROW display.
