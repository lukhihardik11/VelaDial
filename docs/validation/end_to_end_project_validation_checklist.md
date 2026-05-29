# End-to-End Project Validation Checklist

> **Status:** Validation package created. Hardware validation **NOT TESTED**.
> **Target:** Full VelaDial System (Raspberry Pi, Home Assistant, Door-side ELECROW, Bedside ESP32-C6)

This document provides the complete end-to-end validation checklist for the VelaDial project, from zero setup to final PASS/FAIL sign-off.

## Stage 0 — Required Hardware & Materials

| Item | Status |
| :--- | :--- |
| Raspberry Pi | PENDING |
| microSD card | PENDING |
| Power supply | PENDING |
| Network access | PENDING |
| ELECROW CrowPanel 1.28" ESP32-S3 rotary display | PENDING |
| Adafruit ESP32-C6 Feather bedside controller | PENDING |
| APDS-9960 gesture sensor | PENDING |
| USB cables | PENDING |
| Bedroom smart bulbs / Tuya bulbs | PENDING |
| Computer/phone for Home Assistant access | PENDING |
| Required credentials (no secrets stored in repo) | PENDING |

## Stage 1 — Raspberry Pi Home Assistant Setup

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **Flash OS** | Flash Home Assistant OS using Raspberry Pi Imager. | PENDING |
| **Boot Pi** | Boot Raspberry Pi successfully. | PENDING |
| **Find IP** | Find HA URL/IP on the local network. | PENDING |
| **Create Account** | Create HA account. | PENDING |
| **Dashboard Loads** | Confirm HA dashboard loads. | PENDING |
| **Checkpoints** | After-each-step checkpoints completed. | PENDING |
| **Failure Check** | Failure checklist for boot/network/IP issues cleared. | PENDING |

## Stage 2 — Home Assistant Base Configuration

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **ESPHome Add-on** | Install/configure ESPHome add-on. | PENDING |
| **ESPHome Dashboard** | Confirm ESPHome dashboard opens. | PENDING |
| **Secrets Config** | Configure safe `secrets.yaml` placeholders. | PENDING |
| **No Secrets in Repo** | Confirm no secrets committed to repo. | PENDING |
| **LocalTuya Setup** | Install/configure LocalTuya or selected local bulb integration. | PENDING |
| **Add Bulbs** | Add bedroom bulbs to HA. | PENDING |
| **Create Group** | Create/verify `light.bedroom_group`. | PENDING |
| **Test Group** | Test `light.bedroom_group` ON/OFF/brightness/color temperature from HA. | PENDING |
| **Failure Check** | Failure checklist for LocalTuya DP mapping and unavailable bulbs cleared. | PENDING |

## Stage 3 — Door-Side ELECROW Flashing

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **Compile Firmware** | Compile `esphome/door_side_rotary.yaml`. | PENDING |
| **Flash via USB** | Flash via USB first. | PENDING |
| **Boot Logs** | Confirm boot logs show successful startup. | PENDING |
| **Display On** | Confirm display turns on. | PENDING |
| **Network Connection** | Confirm Wi-Fi/API connection in HA. | PENDING |
| **Diagnostic Sensor** | Confirm active theme diagnostic text sensor appears in HA. | PENDING |
| **Post-Flash Tests** | Step-by-step tests after flashing completed. | PENDING |

## Stage 4 — Door-Side Physical Validation

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **Display Rendering** | Screen renders UI correctly. | PENDING |
| **Touch** | Touch input registers. | PENDING |
| **Swipe** | Swiping changes pages. | PENDING |
| **Knob Rotation** | Rotating knob changes values. | PENDING |
| **Short Press** | Short press triggers page action. | PENDING |
| **Long Press** | Long press opens Theme Selector. | PENDING |
| **Wake-Only Touch** | Touching sleeping screen wakes it without action. | PENDING |
| **Wake-Only Rotate** | Rotating sleeping knob wakes screen without action. | PENDING |
| **Wake-Only Press** | Pressing sleeping knob wakes screen without action. | PENDING |
| **Theme Selector Opens** | Long press opens Theme Selector. | PENDING |
| **Long Press Isolation** | Long press does not toggle light. | PENDING |
| **Browse Themes** | Can browse all 20 themes. | PENDING |
| **Apply Themes** | Can apply each theme. | PENDING |
| **Theme Persistence** | Reboot and verify theme persistence. | PENDING |
| **LED Ring** | LED ring behavior matches theme. | PENDING |
| **TSL2591 Backlight** | Adaptive backlight works if sensor is present. | PENDING |
| **SHT45 Diagnostic** | SHT45 remains diagnostic only. | PENDING |
| **API Unavailable** | API unavailable state displays correctly. | PENDING |
| **Locked Presets** | 4 locked presets work. | PENDING |
| **HA Command Path** | HA command path to `light.bedroom_group` works. | PENDING |

## Stage 5 — Bedside ESP32-C6 Flashing

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **Compile Firmware** | Compile `esphome/bedside_gesture.yaml`. | PENDING |
| **Flash via USB** | Flash via USB. | PENDING |
| **Boot Logs** | Confirm boot logs show successful startup. | PENDING |
| **Network Connection** | Confirm Wi-Fi/API connection. | PENDING |
| **Sensor Detected** | Confirm APDS-9960 detected. | PENDING |
| **Diagnostic Sensors** | Confirm diagnostic sensors/entities if present. | PENDING |
| **Failure Checks** | Step-by-step failure checks cleared. | PENDING |

## Stage 6 — Bedside Gesture Validation

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **LEFT Gesture** | LEFT gesture triggers lights off. | PENDING |
| **RIGHT Gesture** | RIGHT gesture triggers lights on. | PENDING |
| **Cooldown** | Gesture cooldown works. | PENDING |
| **No False Triggers** | No repeated accidental triggers. | PENDING |
| **APDS Status** | APDS proximity/diagnostic status if included. | PENDING |
| **No VL53L4CD** | No VL53L4CD v1 behavior unless explicitly implemented later. | PENDING |
| **No Sensor Fusion** | No sensor fusion. | PENDING |
| **Physical Test Table** | Physical test table completed. | PENDING |

## Stage 7 — Full End-to-End System Validation

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **Door-Side Power** | Door-side Power page toggles `light.bedroom_group`. | PENDING |
| **Door-Side Brightness** | Door-side Brightness page changes brightness. | PENDING |
| **Door-Side Presets** | Door-side Presets apply all 4 presets. | PENDING |
| **Door-Side Theme** | Door-side Theme Selector changes visual theme and persists after reboot. | PENDING |
| **Bedside LEFT** | Bedside LEFT turns bedroom group off. | PENDING |
| **Bedside RIGHT** | Bedside RIGHT turns bedroom group on. | PENDING |
| **HA Dashboard** | HA dashboard reflects changes from both devices. | PENDING |
| **LocalTuya Response** | LocalTuya/bulbs respond locally. | PENDING |
| **Network Outage** | Network outage/unavailable behavior checked. | PENDING |
| **Pi Reboot Recovery** | Reboot Raspberry Pi and verify devices reconnect. | PENDING |
| **ESP Reboot Recovery** | Reboot both ESP devices and verify states recover. | PENDING |
| **Soak Test** | 30-minute soak test completed. | PENDING |
| **Failure Log** | Failure log after each subsystem completed. | PENDING |

## Stage 8 — Evidence Collection

| Item | Status |
| :--- | :--- |
| Photo of Raspberry Pi HA dashboard | PENDING |
| Screenshot/photo of ESPHome devices online | PENDING |
| Photo/video of door-side boot | PENDING |
| Video of Theme Selector long-press | PENDING |
| Video cycling all 20 themes | PENDING |
| Video proving long-press does not toggle light | PENDING |
| Video of bedside gesture controlling lights | PENDING |
| Screenshot of `light.bedroom_group` | PENDING |
| Serial logs if failure occurs | PENDING |
| Final PASS/FAIL sign-off | PENDING |

## Stage 9 — Final System Sign-Off

| Subsystem | Result |
| :--- | :--- |
| Raspberry Pi setup | PENDING |
| HA setup | PENDING |
| LocalTuya/group setup | PENDING |
| Door-side flashing | PENDING |
| Door-side hardware validation | PENDING |
| Bedside flashing | PENDING |
| Bedside gesture validation | PENDING |
| Full E2E command path | PENDING |
| **Final Decision** | **[ ] PASS / [ ] FAIL / [ ] CONDITIONAL PASS** |
