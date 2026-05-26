# VelaDial — UI Concept Notes (Creative Design Notebook)

**Status:** Planning / concept direction. No firmware implementation in this PR.
**Companion to:** All other `docs/ui/` files.

---

## 1. Purpose

This document is the creative design notebook for VelaDial's UI direction. It captures design thinking, cross-concept observations, aesthetic principles, and implementation insights that do not belong in the structured matrix or shortlist but are essential for making good design decisions. It is written in a more conversational tone than the other documents because design thinking is exploratory, not prescriptive.

---

## 2. The Round Display Advantage

The single most important design insight for VelaDial is that the 240x240 GC9A01A round display is not a limitation — it is the product's defining advantage. Every rectangular smart-home display (Nest, Echo Show, Samsung SmartThings) must fight the bezel to show content. VelaDial's round display IS the content. The circle is the UI.

This means that concepts which exploit circular geometry (Preset Ring, Iris Aperture, Eclipse Corona, Tree Rings, Lunar Phase) have a structural advantage over concepts that happen to work on a circle but were designed for rectangles (Text-First, Apple Watch Complications). The best VelaDial UI is one that could not exist on a rectangular display.

The round bezel also creates a natural "frame" effect. Content that extends to the edges of the display appears to be emerging from the bezel, like looking through a porthole. This is why the Eclipse Corona concept is so powerful — the corona appears to extend beyond the display into the physical LED ring and onto the wall. The display becomes a window, not a screen.

---

## 3. The Knob-First Principle

VelaDial has a rotary encoder. This is not a touchscreen tablet that happens to have a knob — it is a knob that happens to have a screen. The UI must be designed knob-first, with touch as a convenience layer.

The best knob-to-screen mappings are those where the physical rotation of the knob directly corresponds to a visual change on screen. The arc sweep (SmartKnob) gives a direct 1:1 mapping between knob rotation and arc angle change. The iris opening (Iris Aperture) feels like turning a camera lens ring. The moon phase (Lunar Phase) sweeps the terminator poetically but directly. The tree ring growth adds or removes rings organically and satisfyingly.

The worst knob-to-screen mappings are those where the knob drives an abstract value that the screen represents indirectly. The radar sweep speed change has no visual 1:1 correspondence. The sundial shadow position mapping exists but is not immediately obvious.

The design principle is this: **the user should be able to close their eyes, rotate the knob, and predict what the screen looks like when they open their eyes.** If the mapping is direct enough for that, the concept passes the knob-first test.

---

## 4. The Bedroom Constraint

VelaDial lives in a bedroom. This is not a kitchen display, a living room controller, or a hallway switch. The bedroom context imposes specific constraints that cannot be negotiated away.

At 3 AM, the device must not wake the user's partner. This means no bright white fills, ever, not even during transitions. No flashing or pulsing animations. The LED ring must not pulse. The display backlight must be at its lowest setting in Night Mode. The wake animation must be gentle — a fade-in, not a snap-on.

The device must be readable at arm's length in the dark. This means amber on black is the primary palette, since amber is the least sleep-disruptive visible color. Text must be large enough to read without glasses (minimum 24pt for primary values, 16pt for labels). The page indicator dots must be visible but not bright.

The device must not look like a gadget on the nightstand or wall. The sleep state should be completely dark (no always-on elements). The wake state should look intentional and designed, not like a debug screen. The overall aesthetic should be "luxury bedroom accessory," not "IoT device."

These constraints favor concepts with dark palettes, warm colors, and minimal visual elements: Night Mode, Eclipse Corona, Lunar Phase, and Minimal Thermostat. They disfavor concepts with bright fills (Door Switch Replacement at full amber), dense information (Apple Watch Complications), and continuous motion (Radar Sweep).

---

## 5. The "Premium" Question

What makes a smart-home device feel premium? After analyzing all 20 concepts, the answer is a combination of three factors.

**Factor 1: Visual restraint.** Premium products show less, not more. A Leica camera has fewer buttons than a Canon. A Bang & Olufsen speaker has fewer controls than a JBL. VelaDial should show the minimum information needed for the current action, and nothing else. This is why Night Mode (Concept 6) scores highest on premium feel — it shows almost nothing.

**Factor 2: Material honesty.** The UI should look like it belongs on the hardware. The round display should show round things. The rotary encoder should drive rotary visuals. The amber LED ring should be echoed in the on-screen color palette. When the physical and digital languages match, the device feels coherent and intentional. This is why the Iris Aperture (Concept 17) feels premium — the mechanical metaphor matches the physical knob.

**Factor 3: Surprise and delight.** Premium products have moments that make the user smile. The first time the Eclipse Corona ignites on wake. The first time the Lunar Phase waxes with the knob. The first time the Tree Rings grow from the center. These moments are what separate a $20 smart switch from a $200 smart controller. They are also what make guests ask "what is that?" — which is the ultimate premium product test.

---

## 6. Concept Clusters

The 20 concepts naturally fall into six visual clusters. Concepts inside a cluster share aesthetic DNA — same mental category, same likely asset palette, similar performance envelope.

### Cluster A — Smart-Home Conservative

Concepts #1 (Minimal Thermostat), #3 (Large Center Power Button), #4 (Single-Page Simple Mode), #7 (Text-First Utility), and #12 (Door Switch Replacement) share the Nest/Hue/generic-smart-home visual register. They are familiar, safe, and age well. Low novelty by design. All are easy feasibility. None require custom drawing or asset atlases. The risk with this whole cluster is sameness — the device ends up looking like every other smart-home object. The reward is frictionless adoption — every user immediately knows what to do.

### Cluster B — Round-Display Native

Concepts #2 (SmartKnob Arc), #5 (Preset Ring UI), #14 (Sundial Shadow), #17 (Iris Aperture), #19 (Vinyl DJ Crossfader), and #20 (Eclipse Corona) treat the round bezel as a feature rather than a constraint. Circles, arcs, rings, rotational symmetry. These concepts feel "of" the GC9A01A in a way that Cluster A concepts (which would work equally well on a square display) do not.

### Cluster C — Astronomical/Atmospheric

Concepts #13 (Lunar Phase), #14 (Sundial Shadow), #18 (Radar Sweep), and #20 (Eclipse Corona) share sky imagery, slow motion, soft glow, and contemplative pacing. Bedroom-appropriate by construction. These are the concepts that would feel right at 3 AM — they do not shout at you.

### Cluster D — Pattern/Generative

Concepts #15 (Tree Ring Growth) and #16 (Topographic Contour) are organic, generative, and pattern-based. Strong novelty. Both share the same Achilles heel: legibility at a glance. The user has to read the pattern to extract the value. Both score high on novelty but lower overall because of this readability concern.

### Cluster E — Hardware Ambient

Concept #9 (LED-Ring Status-First) is the only member. It carries state via the 5-LED WS2812 ring rather than the screen. Compatible with any other cluster as a layer. Worth treating as a horizontal capability rather than a competing direction.

### Cluster F — Motion/Energy

Concepts #17 (Iris Aperture), #18 (Radar Sweep), and #19 (Vinyl DJ Crossfader) share continuous or near-continuous animation. Highest visual reward in the matrix; highest performance risk. Whatever wins from this cluster must be POC'd before adoption — animation cost on ESP32-S3 is the wildcard.

---

## 7. Element Borrowing

Even if only one concept wins as the primary direction, elements from others can be borrowed and layered on top. The implementation PR can pick up "easy wins" from rejected concepts.

| Element | From Concept | Could Be Borrowed By | Why It Works |
|---------|-------------|---------------------|-------------|
| Ambient outer ring around the disc | #1 Minimal Thermostat | All of Cluster B | A subtle status ring around the bezel works for almost any center-content concept and reinforces on/off/current state at a glance. |
| Center percentage glyph | #7 Text-First | All Brightness-page concepts | Even the most atmospheric Brightness visual benefits from a numeric anchor for unambiguous reading. |
| Phase/state language for presets | #13 Lunar | #2 + #5 baseline | The four presets could be named in lunar terms without changing the actual `light.turn_on` calls. Pure UI labeling. |
| Sun/shadow imagery for Night preset | #14 Sundial | All preset concepts | The "Low Nightlight" preset could visually depict a sun below the horizon, regardless of how the other three presets are rendered. |
| LED ring state color | #9 LED-Ring | Any winning screen concept | The screen and the ring can carry different fidelities — screen shows the number, ring shows the room-from-across-the-room glance. No conflict. |
| Iris-shaped power button | #17 Iris | #3 Large Center Power Button | The big center power button could be an iris that opens on toggle, even if no other iris animation appears elsewhere. Single-event animation, much lower cost than continuous. |

The recommendation is exactly one borrow at a time — too many and the design becomes incoherent.

---

## 8. Visual Similarity Warnings

Three concept pairs are visually similar enough that they should not be combined within a single device — the user would not be able to distinguish them at a glance.

Concepts #1 (Minimal Thermostat) and #3 (Large Center Power Button) are both "big circle with a value in the middle." Use one or the other; combining loses the distinction. Concepts #13 (Lunar Phase) and #20 (Eclipse Corona) are both circular-celestial imagery. Combining them muddies which is which (is that the moon or the sun?). Concepts #14 (Sundial Shadow) and #18 (Radar Sweep) both feature a single rotating line on a circle. Combining looks like the device is undecided about its visual language.

These pairs can alternate between modes (e.g., lunar at night, sundial by day) but cannot share the same screen at the same time without confusion.

---

## 9. The Animation Budget

The ESP32-S3 running ESPHome + LVGL has a limited animation budget. Based on the existing door-side YAML compile and the LVGL documentation, the practical limits are as follows. Static rendering has no cost and applies to all Green-risk concepts. Triggered animation (on knob rotation or page change) is low cost, runs for 200-500ms then stops, and all Yellow-risk concepts can use this approach. Continuous animation (always running while display is awake) is high cost, and only Concept 18 (Radar Sweep) requires this. The ESP32-S3 may not sustain 30 FPS with a full sweep plus trail.

The design principle is: **animate on interaction, not on idle.** The device should be visually still when the user is not touching it. Motion should be a response to input, not a background decoration.

---

## 10. The Night Mode Layer

Night Mode (Concept 6) is not a concept — it is a requirement. Every concept must have a Night Mode state that is darker, simpler, and calmer than its day state. The Night Mode layer should reduce backlight to minimum (TSL2591 adaptive, already implemented), reduce all amber elements to their dimmest visible level, hide or minimize non-essential UI elements (page indicator dots become barely visible), disable or slow down any animations, and ensure the LED ring is off or at its absolute minimum.

Night Mode is triggered by the TSL2591 ambient light sensor reading below a threshold. It is not a user-selectable mode — it is automatic. The user should never have to think about Night Mode; it should just happen.

---

## 11. Design Philosophy Summary

VelaDial's UI should be **Round** (exploit the circular display, reject rectangular thinking), **Knob-first** (the physical rotation drives the visual change, touch is a shortcut), **Dark** (amber on black, no white fills, the device lives in a bedroom), **Quiet** (animate on interaction not on idle, visually still when untouched), **Honest** (do not claim features that are not implemented), and **Premium** (show less not more, match physical and digital languages, create moments of surprise and delight).

---

## 12. Open Questions for Hardik

| # | Question | Impact |
|---:|---------|--------|
| 1 | Should Brightness be the default landing page (Concept 11)? | Changes page order in YAML. One-line change but affects user mental model. |
| 2 | Is a Creative overlay worth the asset generation effort? | If yes, need 8-12 high-quality images. If no, ship Pure Function (Combination D). |
| 3 | Which Creative concept resonates most with the VelaDial brand? | Eclipse Corona (dramatic), Iris Aperture (mechanical), Lunar Phase (poetic), Tree Rings (organic)? |
| 4 | Should the Presets page use a ring layout (Concept 5) or stay as a 2x2 grid? | Ring layout is more round-native but requires LVGL positioning work. |
| 5 | What is the acceptable LED ring brightness at night? | Determines whether LED ring can echo the Creative overlay or must be off. |
| 6 | Is the "drop the needle" interaction (Concept 19) too playful for VelaDial's brand? | Determines whether Vinyl DJ Crossfader stays in the shortlist. |

---

## 13. What This Document Does Not Do

This document does not select a concept. It does not implement any UI change. It does not modify any ESPHome YAML. It does not claim physical PASS. It is a thinking document, not a specification.

