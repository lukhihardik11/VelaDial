# VelaDial — UI Visual Batch Plan

**Status:** Planning / concept direction. No firmware implementation in this PR.
**Depends on:** Hardik's concept selection from [`ui_concept_shortlist.md`](ui_concept_shortlist.md).
**Produces:** AI-generated mockup images for final visual approval before implementation.

---

## 1. Purpose

This document defines the plan for generating AI-rendered visual mockups of the selected UI concept direction. The visual batch is the last approval gate before any YAML implementation begins. Its purpose is to show Hardik exactly what the device will look like before a single line of firmware code is changed.

No images are generated in this PR. This document only defines what images will be generated, how they will be prompted, and how they will be reviewed. The actual generation happens in a separate task after Hardik selects a concept direction.

---

## 2. Batch Structure

The visual batch is organized into three tiers. Tier 1 is mandatory for any concept direction. Tier 2 is mandatory if a Creative overlay is selected. Tier 3 is optional and produced only if Hardik requests it.

### Tier 1 — Core Page Mockups (Mandatory)

These images show each of the three pages in their primary state, rendered as they would appear on the physical 240x240 round display.

| Image ID | Description | Display State | Notes |
|:--------:|------------|:------------:|-------|
| T1-01 | Power Page — lights ON | Day mode | Shows power icon, on/off state, page indicator dots |
| T1-02 | Power Page — lights OFF | Day mode | Shows power icon in off state |
| T1-03 | Brightness Page — 75% | Day mode | Shows the primary brightness control at a representative value |
| T1-04 | Brightness Page — 25% | Day mode | Shows the brightness control at a low value |
| T1-05 | Presets Page — Warm White selected | Day mode | Shows all 4 presets with one highlighted |
| T1-06 | Presets Page — Low Nightlight selected | Day mode | Shows the nightlight preset highlighted |
| T1-07 | Power Page — lights ON | Night mode | Same as T1-01 but with Night Mode palette |
| T1-08 | Brightness Page — 25% | Night mode | Same as T1-04 but with Night Mode palette |
| T1-09 | Sleep State | Display off | Shows the device with display completely dark, LED ring off |
| T1-10 | Wake Animation — frame 1 | Transition | Shows the first visible frame of the wake-up fade-in |

### Tier 2 — Creative Overlay Mockups (If Creative Selected)

These images show the Creative overlay concept at multiple brightness levels, demonstrating how the visual metaphor maps to the brightness value.

| Image ID | Description | Brightness Level | Notes |
|:--------:|------------|:----------------:|-------|
| T2-01 | Creative overlay at 5% | Minimum | Barely visible — tests the metaphor at its lowest |
| T2-02 | Creative overlay at 25% | Low | Quarter intensity |
| T2-03 | Creative overlay at 50% | Mid | Half intensity |
| T2-04 | Creative overlay at 75% | High | Three-quarter intensity |
| T2-05 | Creative overlay at 100% | Maximum | Full intensity — the hero image |
| T2-06 | Creative overlay at 50% — Night Mode | Mid, dark room | Tests the overlay under Night Mode backlight |
| T2-07 | Creative overlay transition — 25% to 75% | Animated | 3-frame strip showing the knob rotation effect |
| T2-08 | Creative overlay with center percentage text | Mid | Tests whether the numeric overlay is readable |

### Tier 3 — Optional Detail Mockups

| Image ID | Description | Purpose |
|:--------:|------------|---------|
| T3-01 | LED ring glow — lights ON | Shows the physical LED ring amber glow on a wall/surface |
| T3-02 | LED ring glow — Night Mode | Shows the LED ring at minimum or off |
| T3-03 | Page transition animation | 3-frame strip showing swipe between pages |
| T3-04 | Device in bedroom context | Product shot showing VelaDial mounted on a wall or on a nightstand |
| T3-05 | Comparison: day mode vs night mode side-by-side | Split image for quick visual comparison |

---

## 3. Prompting Guidelines

All AI-generated mockups must follow these prompting rules to ensure visual consistency and accuracy.

### 3.1 Global Style Rules

Every prompt must include the following constraints. The display is a 240x240 pixel round display with a GC9A01A panel behind a circular bezel. The background is always pure black (#000000). The primary accent color is warm amber (#F5A623 or similar). The secondary accent is deep amber (#C47F17). Text color is warm white (#FFF8E7). The font style is clean sans-serif (similar to Roboto or Inter). The page indicator is three small dots at the bottom, with the active dot in amber and inactive dots in charcoal (#333333). The overall aesthetic is "luxury bedroom product, not IoT gadget."

### 3.2 Concept-Specific Prompt Templates

Each Creative concept has a specific prompt template that will be filled in with the brightness level and mode.

**Eclipse Corona Template:**
> A 240x240 round display showing a solar eclipse corona effect. The center is a pure black disc (the moon). Around it, amber corona streamers radiate outward toward the bezel. At {brightness}% brightness, the corona extends {coverage_description}. The streamers are soft and organic, not geometric. Background is pure black. No text except a small "{brightness}%" in warm white at the center. Three small dots at the bottom (amber active, charcoal inactive). {night_mode_modifier}. Photorealistic rendering, luxury product aesthetic.

**Iris Aperture Template:**
> A 240x240 round display showing a camera iris aperture mechanism. {blade_count} metallic amber blades form a circular opening in the center. At {brightness}% brightness, the aperture is {opening_description}. The blades have a subtle metallic sheen. Behind the aperture, warm amber light glows through the opening. Background is pure black. Small "{brightness}%" text in warm white at center. Three dots at bottom. {night_mode_modifier}. Precision mechanical aesthetic, luxury product.

**Lunar Phase Template:**
> A 240x240 round display showing a moon phase. At {brightness}% brightness, the moon is {phase_description}. The illuminated portion glows warm amber. The dark portion is barely visible charcoal. The moon surface has subtle crater texture. Background is pure black. Small "{brightness}%" in warm white. Three dots at bottom. {night_mode_modifier}. Contemplative, bedroom-appropriate, poetic.

**Tree Ring Growth Template:**
> A 240x240 round display showing concentric tree rings growing from the center. At {brightness}% brightness, {ring_count} rings are visible, extending {coverage_description} from center. Rings are warm amber lines on black background. The outermost ring is brightest; inner rings are slightly dimmer. Small "{brightness}%" in warm white at center. Three dots at bottom. {night_mode_modifier}. Organic, natural, warm.

### 3.3 Night Mode Modifier

When generating Night Mode variants, append this modifier to the prompt: "Night Mode: all amber elements are reduced to 30% opacity. Backlight is at minimum. The overall image should be very dark, barely visible, calming. No bright elements."

---

## 4. Review Process

After the visual batch is generated, Hardik reviews each image against these criteria.

| Criterion | Pass | Fail |
|-----------|------|------|
| Readable at arm's length? | Text and state are clear | Must squint or guess |
| Bedroom-appropriate? | Calming, not distracting | Too bright, too busy, or too cold |
| Round-display native? | Looks designed for a circle | Looks like a cropped rectangle |
| Premium feel? | Would not be embarrassed to show guests | Looks like a hobby project |
| Knob mapping clear? | Can imagine the knob rotation effect | Unclear how knob relates to visual |
| Night Mode adequate? | Dark enough for 3 AM | Would wake a sleeping partner |

If any Tier 1 image fails review, the concept direction is revised before implementation. If any Tier 2 image fails, the Creative overlay is dropped and the Primary concept ships alone (Combination D from the shortlist).

---

## 5. Asset Generation (If Creative Overlay Approved)

If the visual batch passes review and a Creative overlay is approved, the next step is generating the actual 240x240 PNG assets that will be embedded in the ESP32-S3 firmware. These are different from the mockup images — they must be:

- Exactly 240x240 pixels
- PNG format with alpha transparency where needed
- Optimized for 8-bit color depth (to minimize flash usage)
- Palette-matched to the VelaDial amber palette
- Manually reviewed for quality before embedding

The asset generation is a separate task that happens after visual batch approval and before the implementation PR. It is estimated at 8-12 images per Creative concept, totaling 120-350KB of flash storage.

---

## 6. Timeline

| Step | Trigger | Estimated Duration |
|------|---------|-------------------|
| Hardik selects concept direction | After reviewing shortlist | Hardik's timeline |
| Tier 1 visual batch generated | After concept selection | 1-2 hours |
| Hardik reviews Tier 1 | After generation | Hardik's timeline |
| Tier 2 visual batch generated (if Creative selected) | After Tier 1 approval | 1-2 hours |
| Hardik reviews Tier 2 | After generation | Hardik's timeline |
| Asset generation (if Creative approved) | After Tier 2 approval | 2-4 hours |
| Implementation PR opened | After asset approval | Separate PR |

---

## 7. What This Document Does Not Do

This document does not generate any images. It does not implement any UI change. It does not modify any ESPHome YAML. It does not claim physical PASS. It defines the plan for visual generation that will happen in a future task.

