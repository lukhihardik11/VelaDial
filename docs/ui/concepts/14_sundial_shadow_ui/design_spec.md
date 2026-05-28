# UI Concept 14: Sundial Shadow UI — Design Specification

## 1. Concept Classification

| Attribute | Value |
|-----------|-------|
| **Concept Number** | 14 |
| **Name** | Sundial Shadow UI |
| **Novelty Level** | 9/10 (Premium Concept) |
| **Primary Metaphor** | Sundial shadow length and angle mapping to brightness |
| **Hardware Target** | ELECROW CrowPanel 1.28" ESP32-S3 Rotary Touch Display |
| **HA Target** | `light.bedroom_group` |

## 2. Design Philosophy

The Sundial Shadow UI maps brightness to the position and length of a shadow cast by a virtual gnomon at the center of the round display. At 100% brightness, the shadow is short and the display is bright (high noon). At 0% brightness, the shadow is long and the display is dark (sunset/twilight). The round display becomes a sundial face, and the brightness knob controls the "time of day."

This concept connects electric light to natural light—a philosophical statement embedded in a UI. The color palette shifts from noon to twilight, creating a rich, warm visual experience that changes with every brightness adjustment.

## 3. Visual Identity & Color Palette

The visual palette shifts with the shadow: high brightness uses warm white and amber (midday sun), medium brightness uses golden amber and soft orange (golden hour), low brightness uses deep amber and charcoal (twilight). The shadow itself is a soft gradient, not a hard edge, creating a naturalistic light-and-shadow effect.

| State | Background | Shadow Color | Text/Accent |
|-------|------------|--------------|-------------|
| **100% (Noon)** | Warm White/Amber | Soft Gray/Brown | Dark Charcoal |
| **50% (Golden Hour)** | Golden Amber | Deep Orange/Brown | White/Light Amber |
| **10% (Twilight)** | Charcoal/Black | Deep Amber | Soft Amber |
| **OFF (Night)** | Pure Black | N/A | Dim Amber |

## 4. Screen Architecture

The UI adheres to the locked v1 three-page structure.

### Page 1: Power
- **Visualization:** Sundial at noon (ON) vs. sundial at night (OFF).
- **Center Element:** Small gnomon icon or simple power symbol.
- **Interaction:** Tap center to toggle power. Knob press toggles power.

### Page 2: Brightness (Hero Page)
- **Visualization:** Shadow length proportional to brightness (inverted: bright = short shadow, dim = long shadow). The shadow rotates around the center.
- **Center Element:** `%` value overlaid on the sundial face.
- **Interaction:** Rotate knob to move shadow and adjust brightness. Press knob to return to Power page.

### Page 3: Presets
- **Visualization:** Four sundial quadrants, each showing the preset's color temperature as the shadow/background color.
- **Center Element:** Active quadrant highlighted.
- **Interaction:** Tap quadrant to select. Press knob to apply highlighted preset.

## 5. Interaction Model

| Action | Power Page | Brightness Page | Presets Page |
|--------|------------|-----------------|--------------|
| **Knob Rotate CW** | No action | Increase brightness (shorter shadow) | Next preset |
| **Knob Rotate CCW** | No action | Decrease brightness (longer shadow) | Previous preset |
| **Knob Press** | Toggle Power | Return to Power | Apply Preset |
| **Screen Tap** | Toggle Power | No action | Select Preset |
| **Screen Swipe** | Next/Prev Page | Next/Prev Page | Next/Prev Page |

## 6. Sleep/Wake Behavior

- **Wake-only-first enforced:** The first interaction (touch or knob turn) only wakes the screen; it does not trigger an action.
- **Wake Animation (Sunrise):** Shadow sweeps from long (edge of display) to the current brightness position over 500ms.
- **Sleep Animation (Sunset):** Shadow extends to fill the display, then fades to black over 500ms.

## 7. LED Ring Behavior

The 5-LED WS2812 ring mirrors the sundial's current "time of day" palette.

| Brightness | LED Color | LED Intensity |
|------------|-----------|---------------|
| **100%** | Warm Amber | 100% |
| **50%** | Deep Gold | 50% |
| **10%** | Very Dim Amber | 10% |
| **OFF** | OFF | 0% |

## 8. Dark-Room & Daylight Usability

- **Dark-Room:** The twilight/sunset palette at low brightness is warm and non-stimulating. The shadow metaphor naturally produces dark visuals at low brightness levels.
- **Daylight:** The bright "noon" state at high brightness has good contrast. The `%` overlay provides fallback readability.

## 9. Implementation Strategy (LVGL)

Due to the performance constraints of the ESP32-S3 and LVGL 9.x, rendering real-time soft shadows with complex blurs is too CPU-intensive. 
- **Approach:** We will use LVGL-safe approximations. A central circular object will act as the gnomon. The shadow will be approximated using an `lv_arc` or a rotated `lv_line` with a gradient, or a semi-transparent polygon. 
- **Fallback:** If dynamic shadow drawing is too slow, we will use a simplified geometric representation (e.g., a dark wedge that grows/shrinks).

## 10. v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | Simplified geometric shadow for Brightness, simple power state, standard presets. |
| **v1-expanded** | Smooth shadow gradients, color palette shift, sunrise/sunset wake/sleep animations. |
| **v2** | Real time-of-day shadow tracking, seasonal shadow angle, ambient page. |
