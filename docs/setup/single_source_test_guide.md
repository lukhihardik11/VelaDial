# VelaDial Single-Source Test Guide

This guide is the current source of truth for the first real working end-to-end control path: door-side VelaDial input to Home Assistant bridge automation to `light.bedroom_group`.

## Current Confirmed Status

Hardik has physically confirmed:

- Door-side display works.
- Door-side touch works.
- Door-side knob works.
- Door-side Home Assistant bridge ON/OFF control works against `light.bedroom_group`.
- VelaDial receives Home Assistant state back and verifies ON/OFF correctly.
- Duplicate touch/knob events are ignored by the current lockout behavior.

Do not extend that result beyond what was tested. Bedside is not part of this fix. The UI/ThemeOS work is not production-quality yet.

## Working Home Assistant Entity IDs

Hardik's working Home Assistant instance generated doubled/prefixed ESPHome entity IDs:

| Purpose | Entity ID |
| --- | --- |
| Last bridge request trigger | `sensor.veladial_door_rotary_veladial_last_bridge_request` |
| Request counter | `sensor.veladial_door_rotary_veladial_door_rotary_request_counter` |
| Requested action | `sensor.veladial_door_rotary_veladial_door_rotary_requested_action` |
| Requested brightness | `sensor.veladial_door_rotary_veladial_door_rotary_requested_brightness` |
| Requested preset | `sensor.veladial_door_rotary_veladial_door_rotary_requested_preset` |
| Light target | `light.bedroom_group` |

These IDs are instance-specific. Before testing, go to Home Assistant **Developer Tools -> States**, search for `veladial` and `last_bridge`, and confirm the exact entity IDs in your Home Assistant instance. If your generated IDs differ, update `/config/packages/veladial_control_bridge.yaml` to match your instance.

## Required Test Order

1. **Confirm the light group works in Home Assistant**

   In Home Assistant, manually turn `light.bedroom_group` ON and OFF. Confirm the real bulbs respond before testing VelaDial.

2. **Install the bridge package**

   Put `homeassistant/packages/veladial_control_bridge.yaml` at `/config/packages/veladial_control_bridge.yaml` in Home Assistant.

   Ensure packages are enabled in `configuration.yaml`, for example:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

3. **Restart Home Assistant**

   Fully restart Home Assistant after adding or editing the package. Reloading only automations may not load package changes reliably.

4. **Confirm the automation exists**

   Go to **Settings -> Automations & Scenes** and search for `VelaDial Bridge - Execute Bedroom Group Request`.

   If stale restored/duplicate bridge automations exist, disable or delete the stale copy and keep only the active package automation.

5. **Do not use Run Actions as the test**

   Home Assistant's manual **Run Actions** button is not a valid test for this automation. It does not prove the real ESPHome state-triggered bridge path.

   Test only by physically tapping/pressing VelaDial.

6. **Tap or press VelaDial**

   Physically tap the door-side display or press the knob while the UI is awake and on the expected control page.

7. **Check the last bridge request**

   In **Developer Tools -> States**, confirm:

   - `sensor.veladial_door_rotary_veladial_last_bridge_request` changed.
   - The value looks like `TURN_ON #1` or `TURN_OFF #2`.
   - The request counter changed consistently.

8. **Check the automation trace**

   Open the bridge automation trace and select the trace whose timestamp exactly matches the physical tap/press. Ignore older traces and traces created by manual Run Actions.

   Confirm:

   - Trigger entity is `sensor.veladial_door_rotary_veladial_last_bridge_request`.
   - Raw value is parsed as the expected action.
   - The correct choose branch runs.
   - The trace calls `light.turn_on`, `light.turn_off`, or `light.toggle` against `light.bedroom_group`.

9. **Confirm the light state and VelaDial verification**

   Confirm the real bulbs changed state and Home Assistant reports the new `light.bedroom_group` state. VelaDial logs should show verification matching the expected state, for example:

   ```text
   VelaDial Last Bridge Request >> 'TURN_ON #1'
   HA state changed to on
   VERIFY seq=1: state_now=1 expected=1
   VelaDial Last Bridge Request >> 'TURN_OFF #2'
   HA state changed to off
   VERIFY seq=2: state_now=0 expected=0
   ```

## If the Bridge Fails

Check these in order:

1. `light.bedroom_group` works directly from Home Assistant.
2. `sensor.veladial_door_rotary_veladial_last_bridge_request` exists and changes after a physical tap/press.
3. The bridge package trigger entity matches the actual entity ID in Developer Tools -> States.
4. The automation trace timestamp matches the physical tap/press.
5. The trace parsed the action correctly and did not enter the default error branch.
6. The target entity is still `light.bedroom_group`.

Wrong entity IDs are the most likely failure mode. In Hardik's instance, the working IDs are doubled/prefixed `sensor.veladial_door_rotary_veladial...` IDs, not clean `text_sensor.veladial_last_bridge_request` IDs.

## Fast Tap Behavior

Fast repeated taps may appear to do nothing because the current firmware/control path intentionally avoids duplicate command spam:

- Input lockout ignores duplicate events.
- The deterministic power script can still be running.
- Home Assistant state verification may still be in progress.

Do not change the control logic during this bridge stabilization pass. The recommended follow-up is to show visible feedback such as `Busy`, `Wait`, `Bridge Sent`, or `Verified` for 1-2 seconds, then tune lockout only after soak testing.

## Current Next PR Sequence

1. **Stability / UX feedback:** show Busy, Bridge Sent, and Verified states; reduce confusing HA failure messages; keep the bridge control path unchanged.
2. **UI cleanup:** make the 240 x 240 round screen readable, remove cramped labels, and ship one polished production layout first.
3. **ThemeOS visual work:** do not implement 20 full themes at once. Start with 5 real layout families: Minimal Thermostat, SmartKnob Arc, Large Center Power, Preset Ring, and Eclipse Corona. Describe the current state as 20 selectable skins / 5 planned layout families.
