# ESPHome Concept Prototypes

This directory contains the actual ESPHome YAML prototypes for the 20 UI concepts being explored for VelaDial.

## Prototype Rules

1. **Compile Verification:** Each prototype must compile through ESPHome CI or report the exact compile blocker.
2. **No Physical PASS Claims:** Prototypes must not claim physical PASS.
3. **No Production Readiness Claims:** Prototypes must not claim production readiness.
4. **Hardware Validation:** Hardware validation remains marked NOT TESTED.
5. **Exploration Allowed:** Prototypes may explore more pages/screens than v1 production if clearly marked as concept exploration.
6. **No Secrets:** Prototypes must not introduce secrets.
7. **Status Clarity:** Each prototype must clearly state whether it is a production candidate, concept prototype, v2/future concept, or visual-only experiment.
8. **Validation Requirements:** Each prototype must explain what would need physical hardware validation.
9. **Home Assistant Target:** Prototypes must preserve the Home Assistant target concept: `light.bedroom_group`.

## Production Baseline

Do **not** directly overwrite the production `esphome/door_side_rotary.yaml` for every concept. The production firmware stays stable while these concepts are explored.
