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

## Current Status (Updated 2026-06-06)

All 20 concepts have been researched, documented, and prototyped. The current firmware exposes a ThemeOS foundation, not a final production UI:

**`esphome/door_side_rotary.yaml` contains 20 selectable skins / 5 planned layout families.** Themes are switchable on-device via a hidden Theme Selector (long-press >1.5s). The current door-side display, touch, knob, and HA bridge ON/OFF path have been physically tested by Hardik, but the UI is still cramped/generic and not production-quality.

The concept prototype YAMLs in `esphome/concepts/` remain as reference implementations and are not intended for production use.
