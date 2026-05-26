# VelaDial — UI Concept Shortlist

**Status:** Planning / concept direction. No firmware implementation in this PR.
**Depends on:** [`ui_concept_direction_matrix.md`](ui_concept_direction_matrix.md) (all 20 concepts evaluated).
**Scored by:** [`ui_concept_selection_criteria.md`](ui_concept_selection_criteria.md) (rubric and pass/fail gates).

---

## 1. Purpose

This document identifies the top concept directions from the full 20-concept matrix and organizes them into three categories: Primary Candidates (could ship as the v1 visual identity), Layer Candidates (should be adopted regardless of which primary wins), and Creative Candidates (high-novelty concepts that could elevate VelaDial from "good smart switch" to "iconic bedroom product"). The shortlist is Hardik's decision input — not a final choice.

---

## 2. Shortlist Categories

### 2.1 Primary Candidates — "The Base UI"

These concepts define the overall visual identity of VelaDial v1. Exactly one will be chosen as the primary direction. They are ranked by a weighted combination of feasibility, bedroom-appropriateness, knob-first fidelity, and premium feel.

| Rank | # | Concept | Why Shortlisted |
|:----:|---:|---------|----------------|
| 1 | 2 | SmartKnob-Inspired Arc | Best knob-to-screen mapping. The arc IS the knob. Already partially implemented. Medium LVGL effort. Feels like a precision instrument. |
| 2 | 1 | Minimal Thermostat | Safest, most proven pattern. Easiest LVGL. Highest readability. Risks feeling generic but "generic done perfectly" is premium. |
| 3 | 5 | Preset Ring UI | Best exploitation of round form factor for presets. Medium LVGL effort. The ring layout is VelaDial's unique advantage over rectangular displays. |
| 4 | 12 | Door Switch Replacement | Best power-toggle experience. The full-screen amber fill is dramatic and memorable. Risk: other pages feel secondary. |
| 5 | 7 | Text-First Utility | Maximum readability, minimum risk. The fallback if all creative directions fail. Premium only if typography is perfect. |

### 2.2 Layer Candidates — "Always Adopt"

These concepts are not standalone UIs but behavioral layers that should be applied on top of whichever primary concept wins. They are not mutually exclusive — all of them can coexist.

| # | Concept | Layer Function | Why Always Adopt |
|---:|---------|---------------|-----------------|
| 6 | Night Mode Ultra-Minimal | Dark-room visual state | VelaDial lives in a bedroom. Night Mode is not optional — it is the product's most important UX moment. |
| 9 | LED-Ring Status-First | Ambient room awareness | The 5-LED ring is physical hardware that should communicate state. Even a simple on/off echo adds value. |
| 10 | Three-Screen Tab Carousel | Navigation structure | The 3-page swipe with 3-dot indicator is the locked v1 spec. It is the navigation layer under any visual concept. |
| 11 | Brightness-First UI | Page order optimization | Requires Hardik approval, but the argument is strong: brightness adjustment is the most common bedroom action. |

### 2.3 Creative Candidates — "The Iconic Direction"

These high-novelty concepts could make VelaDial visually unforgettable. They require image assets and more LVGL work, but the payoff is a device that people photograph, share, and remember. Exactly one could be chosen as the visual theme for the Brightness page (the hero page), while Power and Presets pages use a simpler primary candidate layout.

| Rank | # | Concept | Why Shortlisted | Key Requirement |
|:----:|---:|---------|----------------|----------------|
| 1 | 20 | Eclipse Corona | Most visually stunning. The corona on a round display is breathtaking. LED ring extends the effect onto the wall. | 8-12 high-quality corona images |
| 2 | 17 | Iris Aperture | Best mechanical metaphor. Aperture literally controls light. Knob-to-iris mapping is deeply satisfying. | 8-12 high-quality iris blade images |
| 3 | 13 | Lunar Phase | Most poetic. Moon phase = brightness is intuitive and beautiful. Perfect round-display metaphor. | 8-12 high-quality moon phase images |
| 4 | 15 | Tree Ring Growth | Most organic/natural. Warm, unique, nature-inspired. | 8-12 high-quality tree ring images |
| 5 | 14 | Sundial Shadow | Most conceptually coherent (light = shadow). Warm color palette. | 8-12 sundial state images |
| 6 | 19 | Vinyl DJ Crossfader | Most playful. Adds personality. "Drop the needle" is delightful. | Vinyl background + tonearm images |

---

## 3. Recommended Combinations

The shortlist is designed to be mixed. A complete VelaDial v1 UI direction is a combination of one Primary + all Layers + optionally one Creative. The recommended combinations are presented below.

### Combination A — "Precision Instrument"

**Primary:** SmartKnob-Inspired Arc (#2)
**Creative overlay:** Iris Aperture (#17) on Brightness page
**Layers:** Night Mode (#6) + LED Ring (#9) + Three-Screen Carousel (#10)

The device feels like a camera lens meets a precision dial. The knob-to-arc mapping on Power and Presets pages uses the SmartKnob language. The Brightness page shows the iris aperture opening and closing with knob rotation. Night Mode strips everything to a minimal dot. The LED ring echoes the iris opening. This combination is the most mechanically coherent — every interaction feels like operating a precision instrument.

### Combination B — "Celestial Bedroom"

**Primary:** Minimal Thermostat (#1)
**Creative overlay:** Eclipse Corona (#20) on Brightness page
**Layers:** Night Mode (#6) + LED Ring (#9) + Three-Screen Carousel (#10)

The device feels like a captured eclipse on the wall. Power and Presets pages use clean Minimal Thermostat layouts for maximum readability. The Brightness page shows the corona intensifying and diminishing with knob rotation. Night Mode reduces the corona to a faint glow around the dark disc. The LED ring extends the corona onto the wall, creating a physical halo effect. This combination is the most visually dramatic.

### Combination C — "Natural Warmth"

**Primary:** Preset Ring UI (#5)
**Creative overlay:** Tree Ring Growth (#15) on Brightness page
**Layers:** Night Mode (#6) + LED Ring (#9) + Three-Screen Carousel (#10)

The device feels organic and warm. The Presets page uses the ring layout (four arc segments around the bezel). The Brightness page shows tree rings growing and retracting with knob rotation. Night Mode reduces to a single heartwood dot. The LED ring glows warm amber. This combination is the most nature-inspired and the best exploitation of the round form factor.

### Combination D — "Pure Function"

**Primary:** SmartKnob-Inspired Arc (#2)
**Creative overlay:** None
**Layers:** Night Mode (#6) + LED Ring (#9) + Three-Screen Carousel (#10) + Brightness-First (#11)

The device is a precision tool with no decorative metaphor. The arc is the UI. Brightness is the default page. Night Mode is the dark-room state. This is the safest, fastest-to-implement direction. It ships the most polish per line of changed YAML. This is the fallback if all creative directions fail performance review.

### Combination E — "Moonlit Bedroom"

**Primary:** Minimal Thermostat (#1)
**Creative overlay:** Lunar Phase (#13) on Brightness page
**Layers:** Night Mode (#6) + LED Ring (#9) + Three-Screen Carousel (#10)

The device shows a moon on the wall. Power and Presets use clean Minimal Thermostat layouts. The Brightness page shows the moon waxing and waning with knob rotation. Night Mode shows a faint crescent. The LED ring glows moonlight-white. This combination is the most poetic and the most bedroom-appropriate creative direction.

---

## 4. Rejected Concepts (with reasons)

| # | Concept | Rejection Reason |
|---:|---------|-----------------|
| 4 | Single-Page Simple Mode | Violates the locked 3-page constraint. Cannot be adopted without Hardik lifting the lock. |
| 8 | Apple Watch Complications | Too complex for 240x240. Promotes SHT45 to first-class UI element (violates v1 scope where SHT45 is diagnostics-only). Poor dark-room usability with dense information. |
| 18 | Radar Sweep | Continuous animation is too expensive for ESP32-S3 + LVGL. Bedroom-inappropriate (distracting, surveillance aesthetic). The sweep speed mapping to brightness is unintuitive. |
| 16 | Topographic Contour | Honorable mention but ranked below other creative candidates. The brightness-to-elevation metaphor is less intuitive than moon phase, iris, or eclipse. Could be reconsidered for v2. |

---

## 5. Decision Required from Hardik

Hardik must choose one option from each row:

| Decision | Options |
|----------|---------|
| Primary concept | One of: #2 SmartKnob Arc, #1 Minimal Thermostat, #5 Preset Ring, #12 Door Switch, #7 Text-First |
| Creative overlay (optional) | One of: #20 Eclipse Corona, #17 Iris Aperture, #13 Lunar Phase, #15 Tree Ring, #14 Sundial, #19 Vinyl DJ, or None |
| Brightness-First page order | Yes (Brightness = page 0) or No (Power = page 0, current default) |
| Recommended combination | One of A/B/C/D/E above, or a custom combination |

No implementation will begin until Hardik makes this selection. The next step after selection is a visual batch (AI-generated mockup images) of the chosen direction for final approval before any YAML changes.

---

## 6. What This PR Explicitly Does Not Do

This PR does not implement any UI change. Even the smallest Presets-page change is deferred to a separate later PR after Hardik's selection. This PR does not modify any ESPHome YAML. This PR does not change firmware behavior. This PR does not claim physical PASS. All `hardware/validation_results.md` rows remain NOT TESTED. This PR does not lift the Step 15B physical validation gate. This PR does not authorize any v1 scope change. Concepts marked "Reject" in the matrix remain rejected.

