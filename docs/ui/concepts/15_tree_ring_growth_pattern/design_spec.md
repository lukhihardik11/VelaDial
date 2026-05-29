# UI Concept 15: Tree Ring Growth Pattern — Design Specification

## 1. Concept Classification

| Attribute | Value |
|-----------|-------|
| **Concept Number** | 15 |
| **Name** | Tree Ring Growth Pattern |
| **Novelty Level** | 9/10 (Premium Organic Concept) |
| **Primary Metaphor** | Dendrochronology (tree rings) mapping to brightness |
| **Hardware Target** | ELECROW CrowPanel 1.28" ESP32-S3 Rotary Touch Display |
| **HA Target** | `light.bedroom_group` |

## 2. Design Philosophy

The Tree Ring Growth Pattern concept maps brightness to the number of visible concentric rings on the display, like the cross-section of a tree trunk. At 0% brightness, the display shows a single dark heartwood circle at the center. At 100%, the display is filled with concentric rings extending to the bezel — a mature tree with many growth rings.

This concept connects electric light to natural growth cycles. The round display is the perfect canvas — it IS a tree cross-section. The concentric rings exploit the circular geometry in a way that no rectangular display could achieve. It aims for a premium, biomorphic, "calm technology" aesthetic, avoiding rustic or childish clip-art visuals.

## 3. Visual Identity & Color Palette

The visual palette is derived from natural wood tones, shifting from dark heartwood to lighter sapwood as rings grow outward.

| State | Background/Heartwood | Ring Colors | Text/Accent |
|-------|----------------------|-------------|-------------|
| **100% (Mature)** | Deep Brown (`0x2A1A0A`) | Amber/Gold gradients | White/Light Amber |
| **50% (Growing)** | Dark Brown (`0x1A0F05`) | Medium Amber | Soft Amber |
| **10% (Seedling)** | Very Dark Brown (`0x0A0500`) | Dim Amber | Dim Amber |
| **OFF (Dormant)** | Pure Black (`0x000000`) | N/A | Dim Gray |

## 4. Screen Architecture

The UI adheres to the locked v1 three-page structure.

### Page 1: Power
- **Visualization:** Full rings (ON) vs. single dark heartwood (OFF).
- **Center Element:** Small power icon or text at center.
- **Interaction:** Tap center to toggle power. Knob press toggles power.

### Page 2: Brightness (Hero Page)
- **Visualization:** Ring count proportional to brightness. Rings grow outward from the center.
- **Center Element:** `%` value overlaid on the heartwood.
- **Interaction:** Rotate knob to add/remove rings (adjust brightness). Press knob to return to Power page.

### Page 3: Presets
- **Visualization:** Four quadrants or distinct ring patterns representing seasonal growth states.
- **Center Element:** Active preset highlighted.
- **Interaction:** Tap to select. Press knob to apply highlighted preset.

## 5. Interaction Model

| Action | Power Page | Brightness Page | Presets Page |
|--------|------------|-----------------|--------------|
| **Knob Rotate CW** | No action | Increase brightness (add rings) | Next preset |
| **Knob Rotate CCW** | No action | Decrease brightness (remove rings) | Previous preset |
| **Knob Press** | Toggle Power | Return to Power | Apply Preset |
| **Screen Tap** | Toggle Power | No action | Select Preset |
| **Screen Swipe** | Next/Prev Page | Next/Prev Page | Next/Prev Page |

## 6. Sleep/Wake Behavior

- **Wake-only-first enforced:** The first interaction (touch or knob turn) only wakes the screen; it does not trigger an action.
- **Wake Animation (Growth):** Rings grow outward from the heartwood to the current brightness level over 500ms.
- **Sleep Animation (Retraction):** Rings retract inward to the heartwood, then fade to black over 500ms.

## 7. LED Ring Behavior

The 5-LED WS2812 ring acts as the "outermost growth ring" or sun halo.

| Brightness | LED Color | LED Intensity |
|------------|-----------|---------------|
| **100%** | Warm Amber | 100% |
| **50%** | Medium Amber | 50% |
| **10%** | Dim Amber | 10% |
| **OFF** | OFF | 0% |

## 8. Dark-Room & Daylight Usability

- **Dark-Room:** The dark brown/amber palette is non-stimulating. The heartwood at low brightness is a small, dim circle, minimizing light pollution.
- **Daylight:** The concentric rings have clear contrast against each other. The `%` overlay provides fallback readability.

## 9. Implementation Strategy (LVGL)

Due to the performance constraints of the ESP32-S3 and LVGL 9.x, rendering 20+ simultaneous full-circle arcs with complex gradients is too CPU-intensive.
- **Approach:** We will use a set of 8-10 concentric `lv_arc` widgets (or `lv_obj` with radius and thick borders) to represent the rings. The number of visible rings will be mapped to the brightness percentage.
- **Fallback:** If multiple arcs are too slow, we will use a single expanding thick arc or pre-rendered images (though images are harder to manage in ESPHome YAML). We will attempt the multi-arc approach first.

## 10. v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | Simplified concentric rings for Brightness, simple power state, standard presets. |
| **v1-expanded** | Smooth ring growth animation, quadrant presets. |
| **v2** | Seasonal color variation (spring green, autumn gold), real growth tracking over time. |
