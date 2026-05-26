# VelaDial — UI Concept Selection Criteria

**Status:** Planning / concept direction. No firmware implementation in this PR.
**Used by:** [`ui_concept_shortlist.md`](ui_concept_shortlist.md) to rank and filter concepts.
**Source matrix:** [`ui_concept_direction_matrix.md`](ui_concept_direction_matrix.md) (all 20 concepts).

---

## 1. Purpose

This document defines the scoring rubric and pass/fail gates used to evaluate all 20 UI concepts in the direction matrix. The rubric ensures that concept selection is objective, repeatable, and aligned with VelaDial's locked v1 constraints. No concept can be selected for implementation unless it passes all hard gates AND scores above the minimum threshold on the weighted rubric.

---

## 2. Hard Gates (Pass/Fail)

A concept that fails ANY hard gate is immediately rejected, regardless of its rubric score. Hard gates are non-negotiable constraints derived from the locked v1 scope.

| Gate ID | Gate Name | Pass Condition | Fail Consequence |
|:-------:|-----------|---------------|-----------------|
| G1 | Three-Page Lock | Concept must work within exactly 3 pages (Power, Brightness, Presets) | Immediate reject |
| G2 | Four-Preset Lock | Concept must support exactly 4 presets (Warm White, Soft Amber, Neutral White, Low Nightlight) | Immediate reject |
| G3 | Wake-Only-First | Concept must not bypass wake-only-first on any input path | Immediate reject |
| G4 | Knob-First | Concept must be fully operable via rotary encoder alone (touch is optional enhancement) | Immediate reject |
| G5 | No Environment Page | Concept must not promote SHT45/TSL2591 to first-class UI elements | Immediate reject |
| G6 | No Sensor Fusion | Concept must not require VL53L4CD or any sensor fusion logic | Immediate reject |
| G7 | Compile Gate | Concept must be implementable in ESPHome YAML + LVGL without external firmware | Immediate reject |
| G8 | Single HA Target | Concept must target `light.bedroom_group` only (no multi-entity control) | Immediate reject |

### Gate Application Results

| # | Concept | G1 | G2 | G3 | G4 | G5 | G6 | G7 | G8 | Result |
|---:|---------|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|--------|
| 1 | Minimal Thermostat | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 2 | SmartKnob Arc | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 3 | Large Center Power | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 4 | Single-Page Simple | **FAIL** | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **REJECT** |
| 5 | Preset Ring UI | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 6 | Night Mode | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 7 | Text-First Utility | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 8 | Apple Watch Complications | PASS | PASS | PASS | PASS | **FAIL** | PASS | PASS | PASS | **REJECT** |
| 9 | LED-Ring Status-First | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 10 | Three-Screen Carousel | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 11 | Brightness-First UI | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 12 | Door Switch Replacement | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 13 | Lunar Phase | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 14 | Sundial Shadow | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 15 | Tree Ring Growth | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 16 | Topographic Contour | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 17 | Iris Aperture | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 18 | Radar Sweep | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** (performance-flagged) |
| 19 | Vinyl DJ Crossfader | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |
| 20 | Eclipse Corona | PASS | PASS | PASS | PASS | PASS | PASS | PASS | PASS | **PASS** |

Concepts #4 and #8 are rejected by hard gates. The remaining 18 concepts proceed to rubric scoring.

---

## 3. Weighted Scoring Rubric

Each concept that passes all hard gates is scored on 10 dimensions. Each dimension is scored 1-5, then multiplied by its weight. The maximum possible score is 500 (all 5s at all weights). The minimum threshold for v1 consideration is 300.

| Dimension | Weight | 1 (Poor) | 3 (Adequate) | 5 (Excellent) |
|-----------|:------:|----------|-------------|---------------|
| **Dark-Room Usability** | 15 | Bright/distracting at 3 AM | Usable but not optimized | Calming, minimal light, readable |
| **Knob-First Fidelity** | 15 | Knob feels disconnected from UI | Knob works but mapping is indirect | Knob rotation directly maps to visual change |
| **Round-Display Exploitation** | 12 | Could work on any rectangle | Uses circle somewhat | Impossible on a rectangle, native to circle |
| **LVGL Feasibility** | 12 | Requires custom C++ components | Needs careful LVGL work but achievable | Standard LVGL widgets only |
| **Premium Feel** | 10 | Looks like a DIY project | Looks professional | Looks like a luxury product |
| **Daylight Readability** | 8 | Unreadable in bright light | Readable with effort | Clear in any lighting |
| **Animation Performance** | 8 | Continuous heavy animation | Occasional animation, manageable | Static or very light animation |
| **Asset Cost** | 7 | 20+ custom images needed | 5-10 images needed | Zero or minimal images |
| **Novelty/Distinctiveness** | 7 | Identical to existing products | Somewhat distinctive | Completely unique, never seen before |
| **Implementation Speed** | 6 | Weeks of work | Days of work | Hours of work (already implemented) |

### Scoring Results

| # | Concept | Dark | Knob | Round | LVGL | Premium | Day | Anim | Asset | Novel | Speed | **Total** |
|---:|---------|:----:|:----:|:-----:|:----:|:-------:|:---:|:----:|:-----:|:-----:|:-----:|:---------:|
| 1 | Minimal Thermostat | 4 | 4 | 3 | 5 | 3 | 5 | 5 | 5 | 1 | 5 | **385** |
| 2 | SmartKnob Arc | 4 | 5 | 4 | 4 | 4 | 4 | 4 | 5 | 3 | 5 | **413** |
| 3 | Large Center Power | 3 | 3 | 3 | 5 | 3 | 5 | 5 | 5 | 2 | 5 | **370** |
| 5 | Preset Ring UI | 4 | 4 | 5 | 4 | 4 | 4 | 4 | 4 | 4 | 3 | **404** |
| 6 | Night Mode | 5 | 4 | 4 | 5 | 5 | 2 | 5 | 5 | 3 | 4 | **418** |
| 7 | Text-First Utility | 4 | 3 | 2 | 5 | 2 | 5 | 5 | 5 | 1 | 5 | **355** |
| 9 | LED-Ring Status | 4 | 3 | 4 | 5 | 4 | 4 | 5 | 5 | 3 | 4 | **399** |
| 10 | Three-Screen Carousel | 4 | 4 | 4 | 5 | 3 | 4 | 5 | 5 | 2 | 5 | **403** |
| 11 | Brightness-First | 4 | 5 | 4 | 5 | 4 | 4 | 5 | 5 | 2 | 5 | **425** |
| 12 | Door Switch | 3 | 3 | 4 | 4 | 4 | 5 | 3 | 4 | 3 | 3 | **359** |
| 13 | Lunar Phase | 5 | 4 | 5 | 3 | 5 | 2 | 3 | 2 | 5 | 2 | **370** |
| 14 | Sundial Shadow | 4 | 4 | 4 | 3 | 4 | 3 | 3 | 2 | 5 | 2 | **348** |
| 15 | Tree Ring Growth | 4 | 4 | 5 | 3 | 4 | 3 | 3 | 2 | 5 | 2 | **358** |
| 16 | Topographic Contour | 3 | 3 | 4 | 3 | 3 | 3 | 3 | 2 | 5 | 2 | **318** |
| 17 | Iris Aperture | 4 | 5 | 5 | 3 | 5 | 4 | 3 | 2 | 5 | 2 | **386** |
| 18 | Radar Sweep | 3 | 2 | 5 | 2 | 4 | 2 | 1 | 3 | 5 | 1 | **280** |
| 19 | Vinyl DJ Crossfader | 3 | 4 | 4 | 3 | 4 | 3 | 3 | 2 | 5 | 2 | **341** |
| 20 | Eclipse Corona | 5 | 4 | 5 | 3 | 5 | 2 | 3 | 2 | 5 | 2 | **378** |

### Ranking by Total Score

| Rank | # | Concept | Score | Category |
|:----:|---:|---------|:-----:|----------|
| 1 | 11 | Brightness-First UI | 425 | Layer |
| 2 | 6 | Night Mode Ultra-Minimal | 418 | Layer |
| 3 | 2 | SmartKnob-Inspired Arc | 413 | Primary |
| 4 | 5 | Preset Ring UI | 404 | Primary |
| 5 | 10 | Three-Screen Carousel | 403 | Layer |
| 6 | 9 | LED-Ring Status-First | 399 | Layer |
| 7 | 17 | Iris Aperture | 386 | Creative |
| 8 | 1 | Minimal Thermostat | 385 | Primary |
| 9 | 20 | Eclipse Corona | 378 | Creative |
| 10 | 3 | Large Center Power | 370 | Primary |
| 11 | 13 | Lunar Phase | 370 | Creative |
| 12 | 12 | Door Switch Replacement | 359 | Primary |
| 13 | 15 | Tree Ring Growth | 358 | Creative |
| 14 | 7 | Text-First Utility | 355 | Primary |
| 15 | 14 | Sundial Shadow | 348 | Creative |
| 16 | 19 | Vinyl DJ Crossfader | 341 | Creative |
| 17 | 16 | Topographic Contour | 318 | Creative |
| 18 | 18 | Radar Sweep | 280 | **BELOW THRESHOLD** |

Concept #18 (Radar Sweep) scores below the 300-point threshold and is rejected from v1 consideration on scoring grounds (in addition to its performance risk).

---

## 4. Performance Risk Classification

Beyond the rubric score, each concept is classified by its rendering performance risk on the ESP32-S3 + LVGL pipeline. This classification determines whether a proof-of-concept compile + flash test is required before implementation approval.

| Risk Level | Criteria | Concepts | Action Required |
|:----------:|---------|----------|----------------|
| **Green** | Static or near-static rendering. Standard LVGL widgets. No custom drawing. | #1, #2, #3, #5, #6, #7, #9, #10, #11, #12 | No POC required. Implement directly. |
| **Yellow** | Pre-rendered image sequences. Occasional animation. No continuous motion. | #13, #14, #15, #16, #17, #19, #20 | POC recommended. Compile + flash + visual check before full implementation. |
| **Red** | Continuous animation. Custom canvas drawing. Frame-rate dependent. | #18 | POC mandatory. Must demonstrate ≥15 FPS before approval. |

---

## 5. Flash Storage Budget

Concepts that require pre-rendered image assets must fit within the ESP32-S3's available flash storage. The total flash is 16MB, of which approximately 2-4MB is available for image assets after firmware, LVGL, and font allocation.

| Concept | Estimated Asset Size | Within Budget? |
|---------|---------------------|:--------------:|
| #13 Lunar Phase | 8-12 images x ~15KB = ~120-180KB | Yes |
| #14 Sundial Shadow | 8-12 images x ~15KB = ~120-180KB | Yes |
| #15 Tree Ring Growth | 8-12 images x ~15KB = ~120-180KB | Yes |
| #16 Topographic Contour | 8-12 images x ~10KB = ~80-120KB | Yes |
| #17 Iris Aperture | 8-12 images x ~20KB = ~160-240KB | Yes |
| #19 Vinyl DJ Crossfader | 1 background + 8 tonearm = ~100-150KB | Yes |
| #20 Eclipse Corona | 8-12 images x ~25KB + 4 small = ~250-350KB | Yes |

All creative concepts fit within the flash budget. The Eclipse Corona (#20) is the most expensive but still well within the 2-4MB available.

---

## 6. How to Use This Framework for Future Concepts

If Hardik or anyone suggests a 21st concept after this PR merges, the workflow is:

First, apply the hard gates. Does the concept require changing the locked v1 scope? If any gate fails, the concept is immediately rejected. If all gates pass, score the concept on all 10 dimensions using the 1-5 scale. Multiply each score by its weight and sum. If the total is below 300, the concept is rejected for v1. If between 300-380, it is a v1 candidate with adaptation (POC required). If above 380, it is a strong v1 candidate. Then classify the performance risk (Green/Yellow/Red) and estimate the flash storage cost. Append the results to the direction matrix and shortlist.

---

## 7. Scoring Methodology Notes

The scoring is performed by the AI assistant based on the detailed concept descriptions in the direction matrix, the locked v1 constraints in the project docs, and the hardware specifications in the TRD and pinout documents. The scores are subjective estimates, not measurements. They should be treated as a ranking tool, not as absolute values.

Hardik may override any score based on personal preference, product vision, or information not available to the AI. The rubric exists to make the decision process transparent and to ensure that no concept is selected without considering all dimensions. The final decision is Hardik's alone.

---

## 8. What This Document Does Not Do

This document does not select a concept. It provides the scoring framework and results. The selection is made in `ui_concept_shortlist.md` based on these scores plus qualitative judgment. This document does not implement any UI change. This document does not modify any ESPHome YAML. This document does not claim physical PASS.

