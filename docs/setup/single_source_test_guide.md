# VelaDial Test Guide

## Required Test Order

**CRITICAL: Do NOT test the UI or the physical knob until the lights actually turn ON and OFF using the diagnostics paths.**

1.  **HA Manual Test**
    *   In HA, manually turn `light.bedroom_group` ON.
    *   In HA, manually turn `light.bedroom_group` OFF.
    *   Confirm the real bulbs turn on and off.

2.  **Confirm Request Entities Exist**
    *   Go to Home Assistant **Developer Tools → States**.
    *   Search for `veladial_door_rotary_request_counter`. Verify the exact entity ID is `sensor.veladial_door_rotary_request_counter`.
    *   Search for `veladial_door_rotary_requested_action`. **CRITICAL:** Note whether the entity ID is `sensor.veladial_door_rotary_requested_action` OR `text_sensor.veladial_door_rotary_requested_action`.
    *   *If the action entity starts with `text_sensor.`, you MUST manually edit the package file or UI automation to match it.*

3.  **Install HA Bridge Package**
    *   Copy the bridge file to `/config/packages/veladial_control_bridge.yaml`.
    *   Ensure packages are enabled in `configuration.yaml` (e.g. `packages: !include_dir_named packages`).

4.  **Restart Home Assistant**
    *   After adding the package, fully restart HA to load it.

5.  **Confirm Automation Exists & Open Trace**
    *   Go to Settings → Automations & Scenes.
    *   Search for `VelaDial Bridge - Execute Bedroom Group Request`.
    *   Open it and click **Traces** (top right).

6.  **Press VelaDial Bridge Request Button**
    *   Go to the ESPHome device page for VelaDial.
    *   Press the HA diagnostic button: `VelaDial Bridge Request Turn ON` or `Turn OFF`.

7.  **Confirm Request Counter Increments**
    *   In Developer Tools → States, verify that `sensor.veladial_door_rotary_request_counter` increased by 1.
    *   Verify `text_sensor.veladial_last_bridge_request` shows the correct action and counter.

8.  **Confirm Automation Trace Ran**
    *   Look at the automation trace. A new trace should have appeared.
    *   If it did not appear, the trigger entity (`sensor.veladial_door_rotary_request_counter`) is incorrect or the package is not loading.
    *   You should also see a Persistent Notification pop up in HA saying "VelaDial Bridge Executed".

9.  **Confirm Light Group Changed**
    *   Confirm `light.bedroom_group` physically turned ON/OFF.
    *   If the trace ran but the light didn't change, the automation's `requested_action` variable read the wrong entity state. Fix the entity ID in the YAML variable section.

10. **Test Physical Controls (Only after bridge buttons work)**
    *   Test knob short press. Verify the light toggles and state updates.
    *   Test touch tap. Verify the light toggles and state updates.

## Fallback: Manual UI Automation

If packages refuse to load, create this automation manually in the UI:

*   **Trigger:** State -> Entity: `sensor.veladial_door_rotary_request_counter`
*   **Condition:** Template -> `{{ trigger.to_state.state | int(0) > 0 }}`
*   **Action:** Choose (Add 4 options):
    *   Option 1: Template condition `{{ states('sensor.veladial_door_rotary_requested_action') == 'TURN_ON' }}` -> Call Service `light.turn_on` on `light.bedroom_group`
    *   Option 2: Template condition `{{ states('sensor.veladial_door_rotary_requested_action') == 'TURN_OFF' }}` -> Call Service `light.turn_off` on `light.bedroom_group`
    *   Option 3: Template condition `{{ states('sensor.veladial_door_rotary_requested_action') == 'TOGGLE' }}` -> Call Service `light.toggle` on `light.bedroom_group`
    *   Option 4: Template condition `{{ states('sensor.veladial_door_rotary_requested_action') == 'SET_BRIGHTNESS' }}` -> Call Service `light.turn_on` on `light.bedroom_group` with Brightness template `{{ states('sensor.veladial_door_rotary_requested_brightness') | int(50) }}`
