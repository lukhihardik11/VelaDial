# VelaDial Test Guide

## Required Test Order

**CRITICAL: Do NOT test the UI or the physical knob until the lights actually turn ON and OFF using the diagnostics paths.**

1.  **HA Manual Test**
    *   Open Home Assistant.
    *   Find `light.bedroom_group`.
    *   Toggle it manually. Confirm the real bulbs turn on and off.

2.  **Direct HA Diagnostic Buttons**
    *   Go to the VelaDial Door Rotary device page in HA (Settings > Devices).
    *   Under "Diagnostic" or "Controls", press **VelaDial Turn ON Bedroom Group**.
    *   Press **VelaDial Turn OFF Bedroom Group**.
    *   **Result:** Do the real bulbs change? If YES, skip to step 5. If NO, continue to step 3.

3.  **Enable HA Action Permission**
    *   If step 2 failed, your ESPHome device might not have permission to run HA actions.
    *   In HA, go to Settings > Devices > ESPHome > VelaDial Door Rotary > Configure.
    *   Check "Allow the device to perform Home Assistant actions".
    *   Repeat Step 2. If it works, skip to step 5. If it still fails, go to step 4.

4.  **Install/Enable HA Bridge Package (Fallback Path)**
    *   If direct actions STILL fail, the fallback bridge must be used.
    *   Copy the package file `homeassistant/packages/veladial_control_bridge.yaml` into your HA configuration.
    *   In `esphome/door_side_rotary.yaml`, find `use_ha_bridge: "false"` in the `substitutions:` block and change it to `"true"`.
    *   Recompile and reflash the firmware.
    *   Test the bridge buttons or knob (which now sends requests through the bridge).
    *   Check HA logs for the automation `veladial_door_rotary_control_bridge` triggering.

5.  **Physical Tests (Only after control works)**
    *   Test knob short press. Verify the light toggles and state updates.
    *   Test touch tap. Verify the light toggles and state updates.

6.  **Theme Selector/UI**
    *   Only after the physical control reliably works, test the Theme Selector.
    *   Long-press the knob to enter the selector.
    *   Rotate to browse the 5 core layout families and 20 color/text variations.
    *   Short press to apply. Verify the layout changes appropriately (e.g. Minimal, SmartKnob Arc, Large Center Power, Preset Ring, Eclipse Corona).
