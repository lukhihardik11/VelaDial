# VelaDial — UI Concept Shortlist & Recommendation

**Companion to:** [`ui_concept_direction_matrix.md`](ui_concept_direction_matrix.md), [`ui_concept_selection_criteria.md`](ui_concept_selection_criteria.md), [`ui_concept_notes.md`](ui_concept_notes.md).
**Status:** Planning / direction. **No firmware in this PR.** **No physical PASS claimed.**

This document is the recommendation layer on top of the 20-concept matrix. Concepts are not re-described here — see the matrix for full per-concept analysis. The job here is to choose.

---

## 1. Best concepts for *immediate* v1 adaptation

These are concepts that fit the locked v1 scope as-is, require no asset cost worth speaking of, and either are already in the YAML or are a one-tweak away from being in the YAML.

| # | Concept | Why it qualifies for immediate adoption |
| ---: | --- | --- |
| 2 | SmartKnob-Inspired Arc | Already implemented on Brightness page. Direct-touch fix landed in PR #31. Idiomatic to the rotary hardware. |
| 3 | Large Center Power Button | Already implemented as `power_btn` on Power page. Honest, big, hard to misread. |
| 7 | Text-First Utility | Already used for labels (`50%`, `On` / `Off`, preset names). Lowest visual ambition, highest legibility floor. |
| 10 | Three-Screen Tab Carousel | Already implemented as the navigation layer with the 3-dot indicator. Literally the v1 spec. |
| 6 | Night Mode Ultra-Minimal | Compatible as the *night state* of any page; UI brief §"Day/dim/night" already names this. |
| 12 | Door Switch Replacement | Compatible as the conservative *fallback* visual style if a more ambitious concept fails performance review. |

These six together describe roughly what `esphome/door_side_rotary.yaml` already does today (after PRs #30 and #31). They are the **baseline**.

## 2. Best concepts for visually distinctive but still feasible v1

These are concepts that go beyond the baseline without breaking v1 scope. They preserve 3 pages, 4 presets, knob-first, wake-only-first, and the no-Environment-page rule. They differentiate the device visually from a generic Material thermostat.

| # | Concept | Distinctive contribution |
| ---: | --- | --- |
| 5 | Preset Ring UI | Replaces the 2×2 preset grid with a circular ring. Native to the round display. Same data, same actions, same scope. |
| 9 | LED-Ring Status-First | A *layer* on top of any screen concept: state colour on the 5-LED WS2812 ring already on the board. Bedroom-safe constraints respected (no constantly-bright LEDs). |
| 11 | Brightness-First UI | Re-orders pages so Brightness is the landing page. Optimises for the most-common bedroom action. One-line YAML change; requires Hardik's blessing. |

These three are the **easy upgrades** — small in surface, large in visual impact.

## 3. Best experimental / high-novelty concepts for later

These are the 9/10-novelty concepts that *could* fit v1 if asset and performance budgets allow, but are better treated as a deliberate stretch.

| # | Concept | Why it is exciting | Why "later, not now" |
| ---: | --- | --- | --- |
| 17 | Iris Aperture Mechanism | Perfect brightness metaphor; round-native; the most coherent novelty option. | Animation cost on ESP32-S3 unknown; needs either a sprite atlas (~30 frames) or a C++ canvas component. |
| 20 | Eclipse Corona | Round-native; bedroom-appropriate (mostly dark with a glow ring); rotationally symmetric so asset cost is lower than #17. | Still needs a gradient atlas per preset colour. Worth a quick proof-of-concept. |
| 13 | Lunar Phase Visualization | Strong narrative — brightness = moon phase, presets = named lunar states. Lyrical. | ~16–32 phase frames in a sprite atlas; legibility risk at low brightness. |
| 14 | Sundial Shadow UI | Quiet, contemplative; pairs well with bedroom aesthetic. | Legibility at 3 AM is the open question — does the user *read* brightness from a shadow length? |
| 15 | Tree Ring Growth Pattern | Organic, unique, calm. | Same legibility-at-a-glance question as #14. |
| 19 | Vinyl DJ Crossfader | Tactile, musical; pairs well with the rotary encoder. | Tonally bedroom-loud unless rendered very quietly. |

## 4. Concepts that are visually strong but risky for ESPHome LVGL

These should be designed with a *plan B* in mind because the LVGL / ESP32-S3 pipeline may not carry them.

| # | Concept | Specific LVGL / hardware risk |
| ---: | --- | --- |
| 17 | Iris Aperture | Continuous N-blade iris animation can become the dominant CPU cost; LVGL has no native iris widget. Custom canvas drawing required. |
| 18 | Radar Sweep | Continuous-motion sweep is a frame-rate trap. Better as a *transient* state visualization (loading / connecting). |
| 16 | Topographic Contour Map | Best rendered as contour-line vector art; LVGL is a raster compositor and contour generation in C++ is not trivial. |
| 15 | Tree Ring | Organic ring drawing without sprite atlas needs C++ generative code; with sprite atlas balloons asset size. |

## 5. Recommended top 3

In ranked order:

### Top 1 — Primary recommendation

**Concept combo:** #2 (SmartKnob Arc) + #5 (Preset Ring UI) + #10 (Three-Screen Tab Carousel) — "Smart Rotary Triptych".

**What it is.** Keep the Three-Screen Tab Carousel as the navigation layer (already in YAML). Keep SmartKnob Arc on the Brightness page (already in YAML, with arc-touch landed in PR #31). **Upgrade the Presets page** from the current 2×2 grid to a circular ring layout (the only new visual work).

**Mapping to Power / Brightness / Presets.**
- *Power:* unchanged — large centre power button (#3), amber status ring when on, knob press toggles. Identical to current YAML.
- *Brightness:* unchanged — SmartKnob Arc (#2) with the centre `%` label, knob drives in 5% steps, direct touch syncs in real time, on_release pushes one HA call. Identical to current YAML.
- *Presets:* **upgraded** — 4 preset tiles arranged in a ring around the bezel instead of a 2×2 grid. Knob rotation cycles the highlight around the ring (already supported by `highlighted_preset` global); knob press applies; touch on a tile applies directly. Same 4 presets, same actions.

**What remains functionally unchanged.** Wake-only-first guards (already enforced). Page navigation via swipe with 3-dot indicator. Knob press behaviour per page (toggle / return-to-Power / apply). Adaptive backlight from TSL2591. HA target entity (`light.bedroom_group`). All four preset HA calls.

**Assets needed.** None strictly required — labels alone work for the preset ring. Optional: four small monochrome preset glyphs (sun / candle / cloud / crescent), one font character each. Zero new fonts beyond Roboto.

**Performance risks.** Negligible. The ring layout is just four `obj` widgets in a circular arrangement; redraw cost is the same as the current 2×2 grid. No animation cost beyond the highlight border.

**Why this wins.** Smallest delta from the current YAML; ships the most v1 polish per line of changed YAML; preserves every locked rule; matches the round-display geometry better than the 2×2 grid. Compatible with the new compile CI.

### Top 2 — Visually distinctive stretch

**Concept combo:** Top 1 + **Concept #17 (Iris Aperture)** for the Brightness page background.

**What it adds.** Behind the SmartKnob Arc, draw a stylised camera iris that opens with brightness and closes when low. The arc remains the *interactive* element; the iris is the *atmospheric* element. Static at idle; animates only when the user is actively dragging or rotating.

**Performance budget gate.** Implement only after a proof-of-concept compile + flash demonstrates ≥15 FPS during drag with the iris animation enabled. If the POC fails, fall back to Top 1.

**Assets needed.** Either (a) a ~30-frame sprite atlas of iris states baked at build time, or (b) a custom LVGL `canvas` draw routine in YAML lambdas. Option (a) trades flash size for CPU; option (b) trades development effort for flash size. PSRAM is available on this board, so option (a) is more attractive.

**Why this is the stretch.** All upside is visual; downside is real (animation cost, asset cost, time). Worth a separate dedicated POC PR.

### Top 3 — Conservative fallback

**Concept combo:** #1 (Minimal Thermostat) + #7 (Text-First) + #10 (Tab Carousel).

**What it is.** Essentially the current YAML, refined: tighten the Power page to a single large central glyph with a thin amber ring; keep the brightness arc as-is; keep the 2×2 preset grid (no ring).

**Why fall back to this.** If #5 (Preset Ring) proves harder than expected, or if Hardik prefers to ship absolutely-minimum-risk first and add the ring as a second iteration, this is the path of least resistance.

## 6. Recommended single primary direction

**Adopt Top 1 — "Smart Rotary Triptych" (#2 + #5 + #10) — as the v1 visual direction.**

Rationale:
- It is the **smallest delta** from the current YAML (only the Presets page changes).
- It preserves **every** v1 lock without exception.
- It is round-display-native — the Preset Ring is the kind of layout the GC9A01A geometry was designed for.
- It is **compile-safe**: no new LVGL widgets, no custom canvas drawing, no new fonts, no new lambdas beyond the existing patterns.
- It is **testable** under the existing CI workflow with no infrastructure changes.
- The change can be packaged into a single, small, reviewable later PR.

## 7. Fallback direction

If the Preset Ring layout (#5) proves harder than expected — e.g. LVGL's flex layout cannot place four tiles on a circle without manual `x`/`y` positioning that ends up cropping by the round bezel — fall back to **Top 3 (Minimal Thermostat + Text-First + Tab Carousel)**, which is essentially the current YAML with light polish. Keep the 2×2 preset grid; tighten the typography and spacing.

The choice point between Top 1 and Top 3 can be made cheaply: write a one-page POC YAML for the Preset Ring, compile under CI, flash on hardware, look at it. If it looks right, Top 1; if it crops or feels off, Top 3.

## 8. Wild-card v2 / future direction (recorded, not chosen)

For a future v2 build (after VL53L4CD lands, after a possible "ambient mode" toggle, after sensor fusion), the best high-novelty option is **#20 (Eclipse Corona)** layered over the Top-1 baseline. It is bedroom-appropriate, round-native, asset cost is lower than full Iris (#17) because the corona is rotationally symmetric, and it gives the device a distinctive at-a-distance silhouette in a dark bedroom.

This is **not chosen for v1** — recorded here so the team does not lose the idea.

## 9. What this PR explicitly does **not** do

- ❌ Does **not** implement any UI change. Even Top 1's small Presets-page change is deferred to a separate later PR.
- ❌ Does **not** modify any ESPHome YAML.
- ❌ Does **not** change firmware behaviour.
- ❌ Does **not** claim physical PASS. All `hardware/validation_results.md` rows remain `NOT TESTED`.
- ❌ Does **not** lift the Step 15B physical validation gate.
- ❌ Does **not** authorise any v1 scope change. Concepts marked "Reject unless scope changes" in the matrix remain rejected.
