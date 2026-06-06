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

5.  **Confirm Automation Exists & Clean Duplicates**
    *   Go to Settings → Automations & Scenes.
    *   Search for `VelaDial Bridge - Execute Bedroom Group Request`.
    *   If there are duplicate automations (e.g. an unavailable/restored one), **delete or disable** the stale ones. Keep only the active one.

6.  **Test by Tapping VelaDial (Do NOT use "Run Actions")**
    *   Do NOT manually "Run Actions" from the HA UI. This bypasses the trigger context and is an invalid test.
    *   Test by physically tapping or pressing the VelaDial screen/knob.

7.  **Confirm Request Counter Increments**
    *   In Developer Tools → States, verify that `sensor.veladial_door_rotary_request_counter` increased by 1.
    *   Verify `text_sensor.veladial_last_bridge_request` shows the correct action and counter (e.g. `TURN_ON #10`).

8.  **Open Trace and Check Execution**
    *   Go back to the automation and open **Traces**.
    *   Select the trace that exactly matches the timestamp of your physical tap.
    *   Confirm the trigger entity was `text_sensor.veladial_last_bridge_request`.
    *   Confirm the raw value was parsed correctly (e.g., `TURN_ON #10`).
    *   Confirm the choose branch executed the correct action and didn't fall back to `default`.
    *   Check for a Persistent Notification in HA saying `Executed request #X -> TURN_ON` (or `TURN_OFF`).

9.  **Confirm Light Group Changed**
    *   Confirm `light.bedroom_group` physically turned ON/OFF.
    *   If the trace ran but the light didn't change, the automation's `requested_action` variable read the wrong entity state. Fix the entity ID in the YAML variable section.

10. **Test Physical Controls (Only after bridge buttons work)**
    *   Test knob short press. Verify the light toggles and state updates.
    *   Test touch tap. Verify the light toggles and state updates.

## Fallback: Manual UI Automation

If packages refuse to load, create this automation manually in the UI:

*   **Trigger:** State -> Entity: `text_sensor.veladial_last_bridge_request`
*   **Condition:** (None required)
*   **Action:** Choose (Add 4 options):
    *   Option 1: Template condition `{{ states('text_sensor.veladial_last_bridge_request').split('#')[0] | trim | upper == 'TURN_ON' }}` -> Call Service `light.turn_on` on `light.bedroom_group`
    *   Option 2: Template condition `{{ states('text_sensor.veladial_last_bridge_request').split('#')[0] | trim | upper == 'TURN_OFF' }}` -> Call Service `light.turn_off` on `light.bedroom_group`
    *   Option 3: Template condition `{{ states('text_sensor.veladial_last_bridge_request').split('#')[0] | trim | upper == 'TOGGLE' }}` -> Call Service `light.toggle` on `light.bedroom_group`
    *   Option 4: Template condition `{{ states('text_sensor.veladial_last_bridge_request').split('#')[0] | trim | upper == 'SET_BRIGHTNESS' }}` -> Call Service `light.turn_on` on `light.bedroom_group` with Brightness template `{{ states('sensor.veladial_door_rotary_requested_brightness') | int(50) }}`
