# VelaDial — UI Implementation Candidates

**Status:** Planning / concept direction. No firmware implementation in this PR.
**Depends on:** Hardik's concept selection from [`ui_concept_shortlist.md`](ui_concept_shortlist.md).
**Depends on:** Visual batch approval from [`ui_visual_batch_plan.md`](ui_visual_batch_plan.md).

---

## 1. Purpose

This document translates the selected UI concept direction into practical ESPHome YAML implementation guidance. It describes what LVGL widgets, styles, animations, and assets each candidate combination requires, estimates the implementation effort, and identifies the specific YAML sections that would need to change. No YAML is written in this document — it is a planning reference for the implementation PR that comes after visual approval.

---

## 2. Implementation Effort by Combination

The shortlist defines five recommended combinations (A through E). Each has a different implementation cost. The table below estimates the effort in terms of YAML lines changed, new assets required, and estimated implementation time.

| Combination | Primary | Creative Overlay | Est. YAML Changes | New Assets | Est. Time | Risk Level |
|:-----------:|---------|-----------------|:-----------------:|:----------:|:---------:|:----------:|
| A | SmartKnob Arc (#2) | Iris Aperture (#17) | ~200 lines | 8-12 images | 8-12 hours | Yellow |
| B | Minimal Thermostat (#1) | Eclipse Corona (#20) | ~250 lines | 8-12 images | 10-14 hours | Yellow |
| C | Preset Ring (#5) | Tree Ring Growth (#15) | ~300 lines | 8-12 images | 12-16 hours | Yellow |
| D | SmartKnob Arc (#2) | None | ~80 lines | 0 | 3-5 hours | Green |
| E | Minimal Thermostat (#1) | Lunar Phase (#13) | ~220 lines | 8-12 images | 8-12 hours | Yellow |

Combination D is the fastest and safest. It refines the existing SmartKnob Arc implementation (already in the current YAML) with Night Mode, LED Ring, and Brightness-First layers. No new assets. No new LVGL widgets. Mostly style changes and page reordering.

---

## 3. LVGL Widget Requirements by Concept

### 3.1 Primary Concepts

**SmartKnob Arc (#2) — Already Partially Implemented**

The current `door_side_rotary.yaml` already uses `lv_arc` for the brightness control. The implementation refinement would involve adjusting arc styling (indicator color, background arc color, width), adding the Night Mode style variant, and potentially adding a subtle arc animation on value change. The LVGL widgets needed are `arc`, `label`, `obj` (for page indicator dots), and `button` (for preset buttons). All are standard ESPHome LVGL widgets.

**Minimal Thermostat (#1) — New Implementation**

This concept replaces the arc with a large center label showing the brightness percentage, surrounded by a thin circular progress indicator at the edge of the display. The LVGL widgets needed are `arc` (thin, edge-hugging, for the progress ring), `label` (large center text), `obj` (page indicator), and `button` (presets). The arc would be styled with a very thin width (3-5px) and positioned at the outer edge of the display. The center label would be 48-64pt font.

**Preset Ring (#5) — New Implementation**

This concept arranges the four presets as arc segments around the outer edge of the display, with the selected preset highlighted. The LVGL widgets needed are four `arc` segments (each spanning approximately 80 degrees with 10-degree gaps), `label` (preset names inside or near each arc), and `obj` (center content area). This requires careful LVGL positioning to place four arcs at specific angles. The implementation is more complex than the other primaries because ESPHome's LVGL does not have a native "segmented ring" widget — it must be composed from individual arcs.

### 3.2 Creative Overlay Concepts

All Creative overlay concepts use pre-rendered images displayed via the LVGL `image` widget. The implementation pattern is the same for all of them.

The Brightness page would contain an `lv_image` widget sized to fill the display (240x240). The image source would be selected based on the current brightness level (mapped to one of 8-12 pre-rendered images). On knob rotation, the image source changes to the next/previous image in the sequence. A `label` widget overlays the image to show the numeric brightness percentage.

The key YAML pattern for all Creative overlays is:

1. Define 8-12 image assets in the ESPHome `image:` component
2. Create a script that maps `brightness_pct` (0-100) to an image index (0-11)
3. On knob rotation, update the image widget source via `lvgl.image.update`
4. The label overlay shows the percentage text

The differences between Creative concepts are only in the image assets themselves, not in the YAML structure. This means that switching between Eclipse Corona, Iris Aperture, Lunar Phase, or Tree Rings is a matter of swapping image files, not rewriting YAML logic.

### 3.3 Layer Concepts

**Night Mode (#6)**

Night Mode is implemented as an LVGL style variant applied to all widgets when the TSL2591 ambient light reading drops below a threshold. The implementation adds a `night_mode` boolean global, a script that toggles it based on the light sensor, and style overrides that reduce opacity and change colors. The existing TSL2591 adaptive backlight code already handles the backlight dimming; Night Mode adds the on-screen visual changes.

The specific style changes for Night Mode are: arc indicator color changes from amber (#F5A623) to deep amber (#7A5312) at 30% opacity, label text color changes from warm white (#FFF8E7) to dim amber (#8B6914), page indicator dots become nearly invisible (#1A1A1A), and any Creative overlay images would use a separate set of "night" variants (or the same images with an overlay `obj` at 70% opacity black).

**LED Ring (#9)**

The LED ring implementation is already partially present in the current YAML (WS2812 on GPIO48, 5 LEDs). The enhancement adds state-aware color mapping: all LEDs amber when lights are on, all LEDs off when lights are off, and a brightness-proportional LED count (1 LED at 20%, 2 at 40%, etc.) as an optional mode. The YAML changes are in the `light:` and `script:` sections, not in the LVGL section.

**Three-Screen Carousel (#10)**

Already implemented in the current YAML as three LVGL pages with swipe navigation and a 3-dot page indicator. No changes needed unless the page indicator styling is refined for Night Mode.

**Brightness-First (#11)**

A single-line change: reorder the LVGL pages so the Brightness page is `page_index: 0` instead of `page_index: 1`. The Power page becomes `page_index: 1`. The page indicator dots update accordingly. This requires Hardik's explicit approval because it changes the default landing page.

---

## 4. Flash Storage Plan

The ESP32-S3 has 16MB flash. The current firmware binary is approximately 1.2MB. LVGL and fonts consume approximately 200-400KB. This leaves approximately 2-4MB for image assets.

| Asset Type | Per-Image Size | Count | Total Size |
|-----------|:--------------:|:-----:|:----------:|
| Creative overlay (240x240 PNG, 8-bit) | 15-25KB | 8-12 | 120-300KB |
| Night Mode variants (if separate) | 10-20KB | 8-12 | 80-240KB |
| Power page icons | 2-5KB | 2-4 | 4-20KB |
| Preset icons (if used) | 2-5KB | 4-8 | 8-40KB |
| **Total maximum** | | | **~600KB** |

This is well within the available flash budget. Even the most asset-heavy combination (B: Eclipse Corona with Night Mode variants) uses less than 15% of available flash.

---

## 5. YAML Section Map

This table maps each implementation task to the specific YAML section that would be modified. It serves as a reference for the implementation PR to ensure no section is accidentally changed that should not be.

| Task | YAML Section | Current State | Change Type |
|------|-------------|:------------:|:-----------:|
| Primary concept styling | `lvgl: > pages: > widgets:` | SmartKnob Arc | Style update or widget replacement |
| Creative overlay images | `image:` (new section) | Not present | New section |
| Creative overlay display | `lvgl: > pages: > brightness_page:` | Arc widget | Add image widget |
| Night Mode styles | `lvgl: > style_definitions:` | Not present | New section |
| Night Mode trigger | `script:` | Not present | New script |
| LED Ring enhancement | `light: > led_ring:` | Basic amber | Script-driven color mapping |
| Brightness-First reorder | `lvgl: > pages:` | Power=0, Brightness=1 | Swap page_index |
| Page indicator Night Mode | `lvgl: > pages: > widgets: > indicator_dots:` | Fixed amber | Night Mode style variant |

---

## 6. Implementation Sequence

Regardless of which combination is selected, the implementation should follow this sequence to minimize risk and maximize reviewability.

**Step 1: Night Mode layer.** Add the Night Mode boolean, TSL2591 threshold trigger, and style definitions. Compile. This changes no visual behavior in day mode and adds the dark-room state. Lowest risk, highest value.

**Step 2: LED Ring enhancement.** Add state-aware LED color mapping. Compile. This is independent of the screen UI and can be tested separately.

**Step 3: Primary concept styling.** If the Primary concept is different from the current SmartKnob Arc, replace the page widgets. If the Primary is SmartKnob Arc, refine the existing styles. Compile.

**Step 4: Brightness-First reorder (if approved).** Swap page indices. Compile. One-line change but test navigation thoroughly.

**Step 5: Creative overlay (if approved).** Add image assets, image component, and brightness-to-image mapping script. Compile. This is the highest-risk step and should be the last.

Each step is a separate commit (or even a separate PR if Hardik prefers). No step depends on a later step. If any step fails compile or visual review, it can be reverted without affecting the others.

---

## 7. Known Implementation Risks

| Risk | Mitigation |
|------|-----------|
| LVGL image widget may not support runtime source switching in ESPHome | Verify with a minimal POC before full implementation. If unsupported, use multiple overlapping image widgets with visibility toggling. |
| Night Mode style variant application may cause LVGL redraw flicker | Apply style changes during a display-off transition (backlight off, apply styles, backlight on). |
| Preset Ring arc positioning may not be pixel-perfect at 240x240 | Use absolute positioning with calculated coordinates. Test on physical hardware. |
| Creative overlay images may look different on GC9A01A than on a computer monitor | The GC9A01A has limited color gamut and viewing angles. Generate images with high contrast and test on physical hardware. |
| Flash storage may be tighter than estimated if future firmware features grow | Monitor flash usage after each compile. Set a hard limit of 1MB for image assets. |

---

## 8. Decision Dependencies

No implementation can begin until all of the following are true:

| Dependency | Status | Owner |
|-----------|:------:|:-----:|
| Hardik selects a concept combination | PENDING | Hardik |
| Visual batch generated and approved | PENDING | AI + Hardik |
| Step 15B physical hardware validation passed | PENDING | Hardik |
| Current door-side YAML confirmed working on physical board | PENDING | Hardik |

The implementation PR will be opened only after all four dependencies are satisfied.

---

## 9. What This Document Does Not Do

This document does not write any YAML. It does not generate any images. It does not implement any UI change. It does not modify any firmware. It does not claim physical PASS. It is an implementation planning reference for the future PR that will follow concept selection and visual approval.

