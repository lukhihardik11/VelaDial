# UI Concept Prototypes

This directory contains the research, design specifications, and implementation notes for the 20 UI concept prototypes being explored for VelaDial.

## Workflow

For each of the 20 concepts, the following sequential workflow is strictly followed:

1. **Wide Research:** Extensive research is conducted to gather visual inspiration, related smart home UI examples, LVGL feasibility references, and interaction models. This is documented in `research.md`.
2. **Design Specification:** The visual and interaction design for the concept is detailed in `design_spec.md`.
3. **Implementation Notes:** Technical details, LVGL constraints, and implementation risks are documented in `implementation_notes.md`.
4. **Validation Notes:** Any notes regarding physical validation requirements or limitations are documented in `validation_notes.md`.
5. **Concept Prototype:** An actual ESPHome YAML prototype is created in `esphome/concepts/`.

## Concept List

All 20 concepts are mandatory and will be explored sequentially:

1. Minimal Thermostat
2. SmartKnob-Inspired Arc
3. Large Center Power Button
4. Single-Page Simple Mode
5. Preset Ring UI
6. Night Mode Ultra-Minimal
7. Text-First Utility
8. Apple Watch Complications
9. LED-Ring Status-First
10. Three-Screen Tab Carousel
11. Brightness-First UI
12. Door Switch Replacement
13. Lunar Phase Visualization
14. Sundial Shadow UI
15. Tree Ring Growth Pattern
16. Topographic Contour Map
17. Iris Aperture Mechanism
18. Radar Sweep Animation
19. Vinyl DJ Crossfader
20. Eclipse Corona

## Production Baseline Rule

The current production-ish YAML remains the baseline: `esphome/door_side_rotary.yaml`.

Concept prototypes are allowed to explore expanded pages/screens, visual styles, animations, and interaction models. They are **not** automatically approved for production. After all 20 concepts are researched and coded, a final production direction will be selected.
selected.
