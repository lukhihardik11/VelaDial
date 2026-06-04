# VelaDial Test Guide

## Required Test Order

**CRITICAL: Do NOT test the UI or the physical knob until the lights actually turn ON and OFF using the diagnostics paths.**

1.  **HA Manual Test**
    *   In HA, manually turn `light.bedroom_group` ON.
    *   In HA, manually turn `light.bedroom_group` OFF.
    *   Confirm the real bulbs turn on and off.

2.  **Confirm HA Bridge Package is Installed**
    *   Copy package file to `/config/packages/veladial_control_bridge.yaml`.
    *   Ensure packages are enabled in `configuration.yaml` (e.g. `packages: !include_dir_named packages`).
    *   Restart Home Assistant.

3.  **Confirm Automation Exists**
    *   In Home Assistant, verify automation `VelaDial Bridge - Execute Bedroom Group Request` exists and is enabled.

4.  **Test Bridge Buttons**
    *   Press HA button: `VelaDial Bridge Request Turn ON`. Confirm bulbs turn ON.
    *   Press HA button: `VelaDial Bridge Request Turn OFF`. Confirm bulbs turn OFF.
    *   Press HA button: `VelaDial Bridge Request Brightness 50`. Confirm bulbs turn ON at 50%.
    *   If bridge buttons fail:
        *   do not test knob/touch
        *   inspect automation trace
        *   inspect request counter entity
        *   inspect requested action entity

5.  **Physical Tests (Only after bridge buttons work)**
    *   Test knob short press. Verify the light toggles and state updates.
    *   Test touch tap. Verify the light toggles and state updates.

6.  **Theme Selector/UI (Only after control works)**
    *   Long-press the knob to enter the selector.
    *   Rotate to browse the 5 core layout families.
    *   Short press to apply.
