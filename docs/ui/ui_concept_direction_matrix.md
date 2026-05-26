# VelaDial — UI Concept Direction Matrix

**Status:** Planning / concept direction. **No firmware implementation in this PR.**
**Hardware target:** ELECROW CrowPanel 1.28″ ESP32-S3 round rotary touch display, 240×240 GC9A01A, ESPHome LVGL.
**v1 lock (preserved):** 3 pages (Power / Brightness / Presets), 4 presets (Warm White / Soft Amber / Neutral White / Low Nightlight), no Environment page, wake-only-first, knob-first interaction, 3-dot indicator (active = amber), SHT45 = secondary/future diagnostics, APDS-only bedside, no fusion, no VL53L4CD/VL53L0X firmware.

---

## 1. Purpose

This document evaluates **all 20 UI concepts** in the project's concept matrix against the locked VelaDial v1 scope and against the practical constraints of ESPHome LVGL on a 240×240 round display. The output of this document is **not** an implementation — it is the evidence base for the shortlist in [`ui_concept_shortlist.md`](ui_concept_shortlist.md), scored against the framework in [`ui_concept_selection_criteria.md`](ui_concept_selection_criteria.md), with clustering and moodboard notes in [`ui_concept_notes.md`](ui_concept_notes.md).

A separate later PR will implement the chosen direction; this one only chooses it.

## 2. Column definitions

The matrix below carries three columns that originated from the brief:

- **Novelty.** Where the concept sits on the *visual originality* axis. `Standard` = an established smart-home / wearable / thermostat pattern that users have seen before. `9/10` = an unusual, atmospheric, or generative metaphor that almost no smart-home device ships today. Higher novelty trades familiarity for distinctiveness; neither value is "better".
- **Feasibility.** Engineering effort to ship this on ESPHome LVGL on this exact ESP32-S3 hardware. `Easy` = stock LVGL widgets, no custom drawing, fits in normal ESPHome YAML. `Medium` = needs custom canvas drawing, sprite assets, or non-trivial state machines. `Hard` = needs C++ components, large image assets, or interactions LVGL does not natively support cleanly.
- **Visual Batch?** Whether the concept is suited to *image-batch generation* for a moodboard pass later (e.g. a single AI image-gen prompt batch can produce coherent visual references for it). `Yes` = a single visual style brief produces useful mockups. `No` = the concept is not screen-image-driven (e.g. it lives mostly on the WS2812 ring) or is too information-dense to capture in a single still image.

## 3. The 20-concept matrix

| # | Concept | Novelty | Feasibility | Visual Batch? |
| ---: | --- | --- | --- | --- |
| 1 | Minimal Thermostat | Standard | Easy | Yes |
| 2 | SmartKnob-Inspired Arc | Standard | Medium | Yes |
| 3 | Large Center Power Button | Standard | Easy | Yes |
| 4 | Single-Page Simple Mode | Standard | Easy | Yes |
| 5 | Preset Ring UI | Standard | Medium | Yes |
| 6 | Night Mode Ultra-Minimal | Standard | Easy | Yes |
| 7 | Text-First Utility | Standard | Easy | Yes |
| 8 | Apple Watch Complications | Standard | Hard | No |
| 9 | LED-Ring Status-First | Standard | Medium | No |
| 10 | Three-Screen Tab Carousel | Standard | Easy | Yes |
| 11 | Brightness-First UI | Standard | Easy | Yes |
| 12 | Door Switch Replacement | Standard | Easy | Yes |
| 13 | Lunar Phase Visualization | 9/10 | Medium | Yes |
| 14 | Sundial Shadow UI | 9/10 | Medium | Yes |
| 15 | Tree Ring Growth Pattern | 9/10 | Medium | Yes |
| 16 | Topographic Contour Map | 9/10 | Medium | Yes |
| 17 | Iris Aperture Mechanism | 9/10 | Medium | Yes |
| 18 | Radar Sweep Animation | 9/10 | Medium | Yes |
| 19 | Vinyl DJ Crossfader | 9/10 | Medium | Yes |
| 20 | Eclipse Corona | 9/10 | Medium | Yes |

## 4. Verdict legend used below

| Verdict | Meaning |
| --- | --- |
| **v1 candidate** | Fits the locked v1 scope as-is. Could be adopted with minimal effort. |
| **v1 candidate with adaptation** | Fits v1 scope after a small, scope-preserving tweak (e.g. used on a single page rather than as a whole-device metaphor). |
| **v2 / future** | Strong concept but better suited to a future build (after VL53L4CD lands, after sensor fusion, or as an "ambient mode" toggle). Not contradictory to v1, just larger than v1 should carry. |
| **Reject unless scope changes** | Cannot be adopted without breaking a v1 lock (e.g. 3-page rule, knob-first, wake-only-first). Listed for completeness; would require Hardik to formally re-open the locked decision. |

## 5. Per-concept analysis

### 1. Minimal Thermostat

**Description:** Nest-style. A large center value (temperature, in our case brightness percent or ON/OFF), a subtle ambient ring around the bezel, almost no chrome. The screen feels like one big disc of information.
**Why visually interesting:** It has become the universal "smart-home object that takes itself seriously" language. People know how to read it immediately.
**v1 mapping:**
- Power: large ON/OFF text in centre, optional amber ring when lights are on.
- Brightness: large `%` in centre, faint outer ring as arc.
- Presets: 4 large tiles inside the disc — current 2×2 grid is already close.
**Knob-first preserved:** Yes — the knob drives the value; the disc is the readout.
**Wake-only-first preserved:** Yes — same touch / knob / press model.
**240×240 round fit:** Excellent.
**LVGL safe:** Yes — standard `label`, `arc`, `obj` widgets.
**Custom assets needed:** None.
**Verdict:** **v1 candidate.** This is essentially the *refined* version of what the current YAML already does.

### 2. SmartKnob-Inspired Arc

**Description:** The SmartKnob (Scott Bezek) aesthetic: an arc that visually follows the rotary encoder with detents, with a large value in the middle. Arc segment animates smoothly on rotation; the encoder feels mechanically tied to the screen.
**Why visually interesting:** It is the most directly "knob-native" pattern for a round display — touching the knob and watching the arc move feels like one continuous object.
**v1 mapping:**
- Power: not the natural fit (boolean, not analog), but could host the same arc as a *status* ring.
- Brightness: native fit. **Already largely implemented** in `esphome/door_side_rotary.yaml` after the arc direct-touch fix.
- Presets: arc could become a rotary selector around the rim.
**Knob-first preserved:** Yes — the entire concept is knob-first.
**Wake-only-first preserved:** Yes — already enforced in YAML.
**240×240 round fit:** Excellent.
**LVGL safe:** Yes — `arc` widget is well documented; `on_value` / `on_release` triggers already in use.
**Custom assets needed:** None.
**Verdict:** **v1 candidate.** Already the spine of the Brightness page; extending the same pattern to Power and Presets is the cheapest visual unification.

### 3. Large Center Power Button

**Description:** A single, very large, very round touch target. Tap to toggle. The display is essentially one giant button surrounded by a thin status ring.
**Why visually interesting:** It is the most physically honest mapping between intent ("turn the room on") and gesture ("hit the big circle"). No nesting, no menus.
**v1 mapping:**
- Power: native fit. **Already implemented** as `power_btn`.
- Brightness: not directly — but the same shape could host a brightness `%` glyph at idle.
- Presets: not directly — but the active preset could be displayed inside the centre disc.
**Knob-first preserved:** Yes — knob press can still toggle.
**Wake-only-first preserved:** Yes — already enforced.
**240×240 round fit:** Excellent.
**LVGL safe:** Yes — `button` widget.
**Custom assets needed:** None.
**Verdict:** **v1 candidate.** Already in Power page; would be a regression to remove. Adopt as part of the unified visual language.

### 4. Single-Page Simple Mode

**Description:** Everything on one screen. Power glyph in centre, brightness arc around it, four tiny preset chips on the edge. No swipe, no pages.
**Why visually interesting:** Maximum information density without navigation cost. The user sees the entire system state at a glance.
**v1 mapping:**
- Power, Brightness, Presets all on one screen — **directly violates the 3-page lock**.
- Could be a future "summary view" alongside the 3 pages, but not as the only view.
**Knob-first preserved:** Yes.
**Wake-only-first preserved:** Yes.
**240×240 round fit:** Marginal — preset chips at the edge get cut by the round bezel; brightness arc and centre glyph crowd each other.
**LVGL safe:** Yes — but layout is tight.
**Custom assets needed:** Possibly small preset icon glyphs.
**Verdict:** **Reject unless scope changes.** Would require Hardik to lift the "exactly 3 pages" lock. Documented here for completeness.

### 5. Preset Ring UI

**Description:** The four presets arranged around the edge of the round display as a ring of tiles or arcs, instead of the current 2×2 grid. Knob rotation cycles the highlighted preset around the ring; knob press applies.
**Why visually interesting:** Maps the four discrete choices onto the round geometry natively. The 2×2 grid is what you would build on a *square* display; the ring is what you build for *this* hardware.
**v1 mapping:**
- Power: n/a.
- Brightness: n/a.
- Presets: native fit — replaces the current 2×2 grid.
**Knob-first preserved:** Yes — knob cycles the ring.
**Wake-only-first preserved:** Yes — guard pattern reused.
**240×240 round fit:** Excellent — exactly the geometry the round display rewards.
**LVGL safe:** Yes — built from `obj` + `label` widgets in a circular layout. No custom drawing required.
**Custom assets needed:** Four small preset glyphs (could be text labels initially, glyphs later).
**Verdict:** **v1 candidate with adaptation.** Same data, same actions, same scope; only the visual layout of the Presets page changes. Strong candidate for the primary direction.

### 6. Night Mode Ultra-Minimal

**Description:** Single dim dot, tiny number, near-black background. Almost no light pollution. Used when the room is dark.
**Why visually interesting:** Most smart-home UIs forget that they live in bedrooms. This one remembers.
**v1 mapping:**
- Not a page in its own right — it is the **sleep / night state** of every page, gated by the TSL2591 ambient-mode classifier (`dark` / `dim` / `bright`).
**Knob-first preserved:** Yes.
**Wake-only-first preserved:** Yes — in fact this is the *visual aspect* of wake-only-first: when asleep, the screen looks almost off.
**240×240 round fit:** Excellent.
**LVGL safe:** Yes — uses backlight PWM + minimal labels.
**Custom assets needed:** None.
**Verdict:** **v1 candidate.** Should be the night-mode visual layer on top of whichever main concept wins. UI brief §"Day / dim / night UI modes" already aligns.

### 7. Text-First Utility

**Description:** Large readable text labels for everything — `ON`, `OFF`, `70%`, `WARM`, `AMBER`. Minimal graphics, no decorative widgets.
**Why visually interesting:** Honest. Maximum readability per pixel. Ages well.
**v1 mapping:**
- Power: `ON` / `OFF` label fills centre.
- Brightness: `%` value fills centre; arc optional.
- Presets: preset *names* as 2×2 or ring; no icons.
**Knob-first preserved:** Yes.
**Wake-only-first preserved:** Yes.
**240×240 round fit:** Excellent — text scales naturally.
**LVGL safe:** Yes — just `label` widgets.
**Custom assets needed:** None.
**Verdict:** **v1 candidate.** A safe baseline / fallback. Current YAML already leans this way.

### 8. Apple Watch Complications

**Description:** A grid of small data slots ("complications") around the dial — temperature, humidity, brightness, light state, time, comfort, etc.
**Why visually interesting:** Watch-like density. Looks information-rich.
**v1 mapping:**
- Would require multiple data slots on screen simultaneously, which conflicts with the *exactly 3 pages, each focused* lock.
- SHT45 temp/humidity would have to be promoted to a main-page element, which is explicitly out of v1 scope.
**Knob-first preserved:** Marginal — knob can scroll between complications but mapping is unclear.
**Wake-only-first preserved:** Possible but the dense surface makes wake-only-first harder to communicate visually.
**240×240 round fit:** Tight. Apple Watch complications work in part because the watch has a higher-DPI display and an OS-level grid; LVGL on 240×240 cannot match it.
**LVGL safe:** Possible but heavy.
**Custom assets needed:** Many small complication faces.
**Verdict:** **Reject unless scope changes.** Promotes SHT45 to first-class UI and clutters the focused-page philosophy. Documented for completeness.

### 9. LED-Ring Status-First

**Description:** Most state is conveyed via the 5-LED WS2812 ring built into the ELECROW board, not the screen. Screen stays minimal; the ring carries colour for state, brightness, preset.
**Why visually interesting:** Ambient. The room sees the device's state from across the room, not just from arm's length.
**v1 mapping:**
- Power: ring glows amber when lights are on, off when off.
- Brightness: ring brightness scales with light brightness.
- Presets: ring colour reflects the active preset's warm/cool quality.
**Knob-first preserved:** Yes — knob drives the value; the ring reflects it.
**Wake-only-first preserved:** Yes — ring obeys the same sleep timer.
**240×240 round fit:** N/A (hardware ring, not screen).
**LVGL safe:** N/A.
**Custom assets needed:** None on the screen; ring colour map needed.
**Verdict:** **v1 candidate with adaptation.** Strong as a *layer* on top of any screen concept. Five LEDs is a small canvas — pattern animations are limited but state colour works. Note the project's bedroom-safe principle: avoid constantly-lit bright LEDs in a dark bedroom (already documented).

### 10. Three-Screen Tab Carousel

**Description:** Three swipeable pages with a 3-dot indicator. Active dot is amber, others are muted.
**Why visually interesting:** It is the directly idiomatic implementation of the v1 spec.
**v1 mapping:**
- Power / Brightness / Presets across three carousel pages — **already implemented**.
**Knob-first preserved:** Yes — knob press behaviour differs per page (toggle / return / apply).
**Wake-only-first preserved:** Yes — already enforced via `guarded_swipe_*` scripts.
**240×240 round fit:** Excellent.
**LVGL safe:** Yes — `lvgl.page.next` / `.previous` already used.
**Custom assets needed:** None (3-dot indicator is just three small `obj` circles).
**Verdict:** **v1 candidate.** This is the v1 spec made literal. Already in the YAML. Should be the *navigation* layer underneath any visual concept.

### 11. Brightness-First UI

**Description:** The default landing page is Brightness rather than Power. Rotating the knob from sleep gets you straight to adjusting light level — the most common bedroom action.
**Why visually interesting:** Optimises for the *frequency* of user actions, not the conceptual hierarchy. "I'm awake at night and want to nudge the light" is more common than "I want to toggle".
**v1 mapping:**
- Same 3 pages, same 4 presets — but Brightness becomes page 0, Power becomes page 1, Presets stays page 2.
- Knob press on Brightness page currently returns to Power; would change to "toggle" or stay on page.
**Knob-first preserved:** Yes.
**Wake-only-first preserved:** Yes — but the rules about *which page wakes* would need explicit decision.
**240×240 round fit:** Excellent (no layout change).
**LVGL safe:** Yes — page order is a one-line change.
**Custom assets needed:** None.
**Verdict:** **v1 candidate with adaptation.** Single, low-risk tweak; requires Hardik to bless re-ordering. Worth raising as a question, not unilaterally adopting.

### 12. Door Switch Replacement

**Description:** Ultra-conservative. The display looks and behaves like a smart wall switch — a single toggle, no chrome, almost no decoration. The brightness/presets pages exist but are secondary.
**Why visually interesting:** Honest about what the device *replaces*. Lowest cognitive load.
**v1 mapping:**
- Power page is the only "primary" page; Brightness and Presets are demoted to secondary tabs.
- The 3-page structure stays, but Brightness/Presets become discoverable rather than equal.
**Knob-first preserved:** Yes — knob press is the toggle.
**Wake-only-first preserved:** Yes.
**240×240 round fit:** Excellent.
**LVGL safe:** Yes.
**Custom assets needed:** None.
**Verdict:** **v1 candidate.** Lowest visual ambition; safest fallback. Could be the visual style if a more elaborate concept fails performance review.

### 13. Lunar Phase Visualization

**Description:** Brightness is rendered as a moon phase — full moon at 100%, crescent at 5%, dark disc at off. Backlight curve modulates phase brightness too.
**Why visually interesting:** Brightness has a natural physical analogue (light from a celestial body). Ties beautifully into a bedroom / nightlight aesthetic.
**v1 mapping:**
- Power: moon present/absent.
- Brightness: moon phase = brightness percent.
- Presets: each preset is a *named lunar state* (Warm = harvest moon, Amber = blood moon, Neutral = full moon, Nightlight = waning crescent). Strong naming-to-visual mapping.
**Knob-first preserved:** Yes — knob rotates through phases.
**Wake-only-first preserved:** Yes — phase changes are visual only until release.
**240×240 round fit:** Excellent — moon is a circle on a circle.
**LVGL safe:** Medium — needs a custom drawing or a sprite atlas of phases. Possible via LVGL `canvas` widget or pre-rendered PNGs.
**Custom assets needed:** ~16–32 moon-phase frames (sprite atlas).
**Verdict:** **v1 candidate with adaptation** *if* asset cost is manageable; **v2 / future** if not. Strong candidate for the "atmospheric" direction.

### 14. Sundial Shadow UI

**Description:** A circular dial with a virtual gnomon casting a shadow. Shadow angle / length encodes brightness or preset choice. The device feels like a still-life of time and light.
**Why visually interesting:** Slow, contemplative; pairs well with the bedroom-safe principle.
**v1 mapping:**
- Power: sun visible vs. occluded.
- Brightness: shadow length = brightness (longer shadow = lower light, like late afternoon).
- Presets: each preset corresponds to a *time-of-day* (Warm = sunset, Amber = dusk, Neutral = noon, Nightlight = pre-dawn).
**Knob-first preserved:** Yes — knob rotates the sun position.
**Wake-only-first preserved:** Yes.
**240×240 round fit:** Excellent — sundials are natively round.
**LVGL safe:** Medium — shadow drawing via `canvas` or a sprite stack. Animation is light.
**Custom assets needed:** Sun/gnomon sprite plus shadow gradient.
**Verdict:** **v1 candidate with adaptation.** Lovely metaphor; legibility risk (does the user *read* the brightness from a shadow at 3 AM?). Could pair with a discreet text label.

### 15. Tree Ring Growth Pattern

**Description:** Brightness or "age of room state" rendered as concentric rings, like a tree slice. Each ring is a year; brightness = number/saturation of rings.
**Why visually interesting:** Organic, slow, calm. Unique among smart-home UIs.
**v1 mapping:**
- Power: rings absent (off) or present (on).
- Brightness: number / fill of rings.
- Presets: each preset's ring colour palette (cool/warm/neutral grain).
**Knob-first preserved:** Yes — knob adds/removes rings.
**Wake-only-first preserved:** Yes.
**240×240 round fit:** Excellent — concentric rings on a circle.
**LVGL safe:** Medium — needs custom canvas drawing for the organic-look rings, or a sprite atlas.
**Custom assets needed:** Either generative algorithm in C++ or sprite atlas of ring states.
**Verdict:** **v1 candidate with adaptation** if a small, deterministic ring set suffices; **v2 / future** for richer generative variants. Readability at-a-glance is the main risk — the user may need 1 second of attention to parse "how bright".

### 16. Topographic Contour Map

**Description:** Brightness as topographic contour lines — denser lines = higher brightness. The device looks like a map fragment.
**Why visually interesting:** Unusual; turns brightness into a spatial / cartographic visual.
**v1 mapping:**
- Brightness: contour density.
- Power: contour absent / present.
- Presets: contour colour palette per preset.
**Knob-first preserved:** Yes.
**Wake-only-first preserved:** Yes.
**240×240 round fit:** Marginal — contour maps are usually rectangular; cropping to a circle loses information.
**LVGL safe:** Medium — needs canvas drawing or large sprite atlas.
**Custom assets needed:** Significant — many sprite states or a generative renderer.
**Verdict:** **v2 / future.** High novelty but readability and asset cost both unfavourable for v1.

### 17. Iris Aperture Mechanism

**Description:** A camera-iris animation. The iris opens as brightness rises and closes as brightness falls. Closed iris = lights off. Animated transitions on every change.
**Why visually interesting:** Stunning visual metaphor for "how much light is coming out". Native to a round bezel.
**v1 mapping:**
- Power: iris fully open (on) or fully closed (off).
- Brightness: iris aperture = brightness %.
- Presets: each preset has an iris aperture stop + tint colour.
**Knob-first preserved:** Yes — knob rotation moves blades.
**Wake-only-first preserved:** Yes — guard pattern applies; the iris does *not* open until awake.
**240×240 round fit:** Excellent.
**LVGL safe:** Medium — needs custom canvas drawing of N-blade iris, or a sprite atlas of pre-rendered frames. Animation is the cost driver.
**Custom assets needed:** Either a pre-rendered sprite atlas (~30 frames for 1–100% smooth) or a C++ iris-draw component.
**Verdict:** **v1 candidate with adaptation** *if* performance proves OK on ESP32-S3 with PSRAM; **v2 / future** if frame rate is poor. Highest visual reward in the matrix.

### 18. Radar Sweep Animation

**Description:** A rotating sweep line, like a radar PPI display. The sweep can indicate "scanning" (connecting to HA, waiting for confirmation) or "active" (lights on, sweep slow).
**Why visually interesting:** Familiar from sci-fi; communicates motion and aliveness.
**v1 mapping:**
- Power: sweep on/off.
- Brightness: sweep speed or sweep brightness — neither maps as cleanly as Iris/Lunar.
- Presets: hard to map.
**Knob-first preserved:** Knob rotation could move the sweep but the metaphor breaks (radar sweeps autonomously).
**Wake-only-first preserved:** Yes.
**240×240 round fit:** Excellent.
**LVGL safe:** Medium — continuous animation has performance cost.
**Custom assets needed:** Sprite or canvas line drawing.
**Verdict:** **v2 / future.** Better as a *transient* state visualization (loading, connecting, error) than as the always-on display. Could become an "Unavailable" badge animation later.

### 19. Vinyl DJ Crossfader

**Description:** Two overlapping discs that crossfade (like turntable A/B). Power off ↔ Power on is a crossfade slide; brightness is a horizontal fader; presets are a crossfade between preset discs.
**Why visually interesting:** Tactile, musical, gestural. Pairs nicely with the physical rotation of the encoder.
**v1 mapping:**
- Power: crossfade A ↔ B.
- Brightness: fader position.
- Presets: each preset is a disc; knob rotates between discs.
**Knob-first preserved:** Yes — rotation = scratch / scroll.
**Wake-only-first preserved:** Yes.
**240×240 round fit:** Good — discs nest naturally.
**LVGL safe:** Medium — two-layer compositing; opacity transitions.
**Custom assets needed:** Disc art, possibly a small fader sprite.
**Verdict:** **v1 candidate with adaptation.** Fun but tonally bedroom-loud; would need a much quieter visual register than a real DJ deck.

### 20. Eclipse Corona

**Description:** A black central disc surrounded by a glowing corona that intensifies with brightness. The "lights on" state is a sun behind the moon; "lights off" is a black disc on black.
**Why visually interesting:** Dramatic, atmospheric, perfectly bedroom-safe (mostly dark with selective glow). Round-native.
**v1 mapping:**
- Power: corona present / absent.
- Brightness: corona intensity.
- Presets: corona colour per preset (warm gold, deep amber, white-blue, dim red).
**Knob-first preserved:** Yes — knob modulates corona intensity.
**Wake-only-first preserved:** Yes — corona stays dim until awake-and-acted-on.
**240×240 round fit:** Excellent — eclipse imagery is exactly the geometry of the display.
**LVGL safe:** Medium — needs a gradient/glow renderer or a pre-rendered sprite atlas.
**Custom assets needed:** Corona gradient atlas (likely ~10–20 frames per preset colour).
**Verdict:** **v1 candidate with adaptation.** Visually unique and bedroom-appropriate. Lower asset cost than full Iris (#17) because the corona shape is rotational-symmetric.

## 6. Verdict counts

| Verdict | Count | Concepts |
| --- | ---: | --- |
| v1 candidate | 6 | #1, #2, #3, #6, #7, #10, #12 *(7 — #12 listed but described as fallback baseline)* |
| v1 candidate with adaptation | 8 | #5, #9, #11, #13, #14, #15, #17, #19, #20 *(9)* |
| v2 / future | 2 | #16, #18 |
| Reject unless scope changes | 2 | #4, #8 |

*(Two rows above show 7 and 9 because #12 and #20 straddle their categories — #12 is best understood as a v1 fallback baseline; #20 is "v1 with adaptation" if asset budget allows.)*

## 7. What this PR explicitly does **not** do

- ❌ Does **not** implement any UI change.
- ❌ Does **not** change firmware behaviour or touch any ESPHome YAML.
- ❌ Does **not** lift the Step 15B physical validation gate. Every row in `hardware/validation_results.md` remains `NOT TESTED`.
- ❌ Does **not** authorise any v1 scope change — concepts marked "Reject unless scope changes" remain rejected.
- ❌ Does **not** claim any physical PASS.
- ❌ Final concept implementation will come in a **separate later PR** after Hardik picks a direction (using [`ui_concept_shortlist.md`](ui_concept_shortlist.md)) and approves.

## 8. Next steps

1. Hardik reviews this matrix.
2. Hardik reads [`ui_concept_shortlist.md`](ui_concept_shortlist.md) and either approves the recommended primary direction or picks a different shortlisted concept.
3. A later PR (out of scope for this PR) implements the chosen direction inside `esphome/door_side_rotary.yaml`, gated by the ESPHome compile CI from PR #30 and the wake-only-first guards already in place after PR #31.
