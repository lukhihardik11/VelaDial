# End-to-End Project Validation Checklist

> **Status:** Door-side display, touch, knob, and ON/OFF through the Home Assistant bridge have been physically tested by Hardik. Full-system validation is still incomplete.
> **Target:** Full VelaDial system: Raspberry Pi + Home Assistant + LocalTuya/bedroom lights + door-side ELECROW ESP32-S3 + bedside ESP32-C6/APDS-9960.  
> **Rule:** Do not mark any row PASS until Hardik has physically performed the step and captured evidence.

This is the technician-style field runbook for validating VelaDial from zero setup to final PASS/FAIL sign-off. It intentionally separates **setup**, **subsystem validation**, **full end-to-end validation**, **failure triage**, and **evidence capture**.

## How to use this checklist

1. Complete stages in order. Do not skip ahead.
2. Mark each row as one of: `PENDING`, `PASS`, `FAIL`, `N/A`, or `RETEST REQUIRED`.
3. For every `FAIL`, record the failure in `end_to_end_test_results_template.md` before changing firmware or configuration.
4. Do not edit production firmware during validation unless a failure is confirmed and a new fix PR is opened.
5. Hardware PASS requires every required stage to pass on physical hardware. Compile success alone is not hardware PASS.

## Required result labels

| Label | Meaning |
| :--- | :--- |
| `PENDING` | Not tested yet. Default state. |
| `PASS` | Tested on real hardware and met expected behavior. |
| `FAIL` | Tested and did not meet expected behavior. Must include evidence and reproduction steps. |
| `N/A` | Not applicable to this hardware setup. Explain why. |
| `RETEST REQUIRED` | Previously failed or changed; must be tested again. |

## Stop/go gates

| Gate | Must pass before continuing | If failed |
| :--- | :--- | :--- |
| Gate A — HA reachable | Home Assistant dashboard loads from phone/computer | Stop. Fix Raspberry Pi/network first. |
| Gate B — lights controllable in HA | `light.bedroom_group` works from HA UI before ESP devices are tested | Stop. Fix bulbs/LocalTuya/group mapping first. |
| Gate C — door-side boots | ELECROW boots, display renders, ESPHome API connects | Stop. Fix flash/pin/display/network before UI testing. |
| Gate D — long-press safe | Long press opens Theme Selector and does **not** toggle lights | Stop. Do not proceed until fixed. This is a safety gate. |
| Gate E — bedside boots | ESP32-C6 boots, APDS-9960 is detected, ESPHome API connects | Stop. Fix wiring/flash/I2C before gesture testing. |
| Gate F — full E2E path | Door-side and bedside both control `light.bedroom_group` through HA/local integration | Only then consider conditional full-system hardware PASS. Door-side ON/OFF bridge PASS alone is not bedside or full-system PASS. |

---

# Stage 0 — Required hardware and materials

| Item | Required detail | Status | Evidence / notes |
| :--- | :--- | :--- | :--- |
| Raspberry Pi | Model and power supply known; stable 5V supply. | PENDING | |
| microSD card | HA OS target card available; capacity recorded. | PENDING | |
| Network access | Same LAN for Pi, ESP devices, phone/computer, and bulbs. | PENDING | |
| Computer or phone | Can access Home Assistant web UI. | PENDING | |
| ELECROW CrowPanel 1.28" ESP32-S3 rotary display | Door-side hardware present. | PENDING | |
| Adafruit ESP32-C6 Feather | Bedside controller present. | PENDING | |
| APDS-9960 gesture sensor | Connected via STEMMA/Qwiic or wired I2C. | PENDING | |
| USB cables | Data-capable cables for both boards. | PENDING | |
| Bedroom smart bulbs / Tuya bulbs | Installed, powered, reset/onboarded as needed. | PENDING | |
| Required credentials | Wi-Fi, HA account, Tuya/local keys available privately. | PENDING | No secrets committed to repo. |
| Repo checkout | Latest `main` pulled after PR #57 merge. | PENDING | Commit SHA: |

# Stage 1 — Raspberry Pi Home Assistant setup

## 1.1 Flash and boot Home Assistant OS

| Step | Action | Expected result | Status | Evidence / notes |
| :--- | :--- | :--- | :--- | :--- |
| 1.1.1 | Use Raspberry Pi Imager to flash Home Assistant OS to microSD. | Flash completes without error. | PENDING | |
| 1.1.2 | Insert microSD, connect Ethernet/Wi-Fi as applicable, power Pi. | Pi powers on and boots. | PENDING | |
| 1.1.3 | Wait for first boot and onboarding. | HA becomes reachable after initial setup time. | PENDING | |
| 1.1.4 | Open `http://homeassistant.local:8123` or Pi IP address. | HA onboarding page loads. | PENDING | |
| 1.1.5 | Create Home Assistant owner account. | Dashboard loads after sign-in. | PENDING | |

## 1.2 Failure triage — Raspberry Pi / HA access

| Symptom | Check first | Next action | Status |
| :--- | :--- | :--- | :--- |
| No HA URL | Router client list / IP scan | Try IP:8123 directly. | PENDING |
| Pi not booting | Power supply, SD card, HDMI/LED behavior | Reflash SD card if needed. | PENDING |
| HA slow/unreachable | Wait for initial boot, check network | Use Ethernet for first setup if possible. | PENDING |
| Browser cannot connect | Same network/VPN off | Confirm phone/computer is on same LAN. | PENDING |

# Stage 2 — Home Assistant base configuration

## 2.1 ESPHome add-on setup

| Step | Action | Expected result | Status | Evidence / notes |
| :--- | :--- | :--- | :--- | :--- |
| 2.1.1 | Install ESPHome add-on in HA. | Add-on installs successfully. | PENDING | |
| 2.1.2 | Start ESPHome add-on. | ESPHome dashboard opens. | PENDING | |
| 2.1.3 | Confirm ESPHome can access/import project YAMLs. | Door-side and bedside YAMLs are visible or uploadable. | PENDING | |
| 2.1.4 | Confirm no real secrets are in repo files. | Credentials exist only in local `secrets.yaml` or HA secrets. | PENDING | |

## 2.2 LocalTuya / bedroom light setup

| Step | Action | Expected result | Status | Evidence / notes |
| :--- | :--- | :--- | :--- | :--- |
| 2.2.1 | Install/configure LocalTuya or chosen local bulb integration. | Integration loads without errors. | PENDING | |
| 2.2.2 | Add each bedroom bulb. | Bulbs appear in HA as controllable light entities. | PENDING | |
| 2.2.3 | Validate each bulb individually. | ON/OFF, brightness, and color temperature work from HA UI. | PENDING | |
| 2.2.4 | Create `light.bedroom_group`. | Group entity appears in HA. | PENDING | |
| 2.2.5 | Test `light.bedroom_group` ON/OFF. | All intended bedroom bulbs respond. | PENDING | |
| 2.2.6 | Test `light.bedroom_group` brightness. | All bulbs dim/brighten consistently. | PENDING | |
| 2.2.7 | Test `light.bedroom_group` color temperature. | Warm/cool changes work as expected. | PENDING | |

## 2.3 LocalTuya / group failure triage

| Symptom | Likely area | Check | Status |
| :--- | :--- | :--- | :--- |
| Bulb unavailable | Network/local key/device ID | Verify bulb IP, local key, Tuya device mapping. | PENDING |
| ON/OFF works but brightness fails | DP mapping | Verify brightness DP and scale. | PENDING |
| Color temperature reversed/wrong | DP mapping/range | Verify CT min/max and DP mapping. | PENDING |
| Group does not respond | Group entity config | Confirm group members and entity ID exactly `light.bedroom_group`. | PENDING |

**Gate B:** Do not test ESP devices until `light.bedroom_group` works from HA UI.

# Stage 3 — Door-side ELECROW flashing

## 3.1 Compile and flash

| Step | Command / action | Expected result | Status | Evidence / notes |
| :--- | :--- | :--- | :--- | :--- |
| 3.1.1 | Pull latest repo main after PR #57 merge. | Local repo is current. | PENDING | Commit SHA: |
| 3.1.2 | Verify local secrets are present but not committed. | `esphome/secrets.yaml` exists locally only. | PENDING | |
| 3.1.3 | Compile door-side firmware. | `esphome compile esphome/door_side_rotary.yaml` passes. | PENDING | |
| 3.1.4 | Connect ELECROW board via USB-C data cable. | Board appears as serial device. | PENDING | Port: |
| 3.1.5 | Flash via USB first. | `esphome run esphome/door_side_rotary.yaml` uploads successfully. | PENDING | |
| 3.1.6 | Keep serial monitor open after reboot. | No bootloop, panic, or repeated reset. | PENDING | |

## 3.2 Door-side first boot checkpoint

| Test item | Expected behavior | Status | Evidence / notes |
| :--- | :--- | :--- | :--- |
| Display backlight | Turns on after boot. | PENDING | |
| UI render | Power page or expected default page appears. | PENDING | |
| Wi-Fi | Device joins configured Wi-Fi. | PENDING | |
| HA API | Device appears online in Home Assistant / ESPHome. | PENDING | |
| Active theme entity | `text_sensor.veladial_doorside_active_theme` appears if implemented. | PENDING | |
| Serial logs | No critical errors for display, touch, sensors, or HA API. | PENDING | |

# Stage 4 — Door-side physical validation

## 4.1 Input and wake-only-first validation

| Test item | Procedure | Expected behavior | Status | Evidence / notes |
| :--- | :--- | :--- | :--- | :--- |
| Screen sleep | Leave device idle until timeout. | Screen dims/sleeps as designed. | PENDING | |
| Touch wake-only | While asleep, tap/touch once. | Wakes only; no toggle, preset, or page action. | PENDING | |
| Knob rotation wake-only | While asleep, rotate one detent. | Wakes only; brightness/preset does not change. | PENDING | |
| Knob press wake-only | While asleep, press knob once. | Wakes only; light does not toggle. | PENDING | |
| Short press | While awake, short press on Power page. | Toggles `light.bedroom_group`. | PENDING | |
| Long press | Hold knob >1.5s. | Opens Theme Selector. | PENDING | |
| Long press isolation | Watch bulbs while long-pressing. | Long press does **not** toggle or apply action. | PENDING | Safety gate. |
| Swipe left/right | Swipe pages while awake. | Navigates Power / Brightness / Presets. | PENDING | |
| Page indicator | Navigate pages. | Indicator matches active page. | PENDING | |

## 4.2 Control validation

| Test item | Procedure | Expected behavior | Status | Evidence / notes |
| :--- | :--- | :--- | :--- | :--- |
| Bridge entity IDs | In HA Developer Tools -> States, verify generated VelaDial bridge entities. | Hardik's working trigger is `sensor.veladial_door_rotary_veladial_last_bridge_request`; adjust package if local IDs differ. | PASS for Hardik's HA; verify per instance | Do not assume clean `text_sensor.*` IDs. |
| Bridge trace method | Physically tap/press VelaDial, then open the automation trace with the matching timestamp. | Trace is from the physical input, not manual Run Actions. | PASS for Hardik's ON/OFF test; required for retest | Home Assistant Run Actions is not a valid bridge test. |
| Power page | Short press or tap Power. | `light.bedroom_group` toggles ON/OFF through the HA bridge. | PASS for Hardik's ON/OFF bridge test only | Evidence reported: `TURN_ON #1` verified on, `TURN_OFF #2` verified off. |
| Brightness page | Rotate knob CW/CCW. | Brightness changes in visible steps. | PENDING | |
| Brightness HA sync | Watch HA light entity during changes. | HA reflects brightness value. | PENDING | |
| Preset selection | Rotate on Presets page. | Highlight moves through 4 presets. | PENDING | |
| Warm White | Apply preset. | Light changes to Warm White target. | PENDING | |
| Soft Amber | Apply preset. | Light changes to Soft Amber target. | PENDING | |
| Neutral White | Apply preset. | Light changes to Neutral White target. | PENDING | |
| Low Nightlight | Apply preset. | Light changes to Low Nightlight target. | PENDING | |

## 4.3 Theme Selector validation

| Test item | Procedure | Expected behavior | Status | Evidence / notes |
| :--- | :--- | :--- | :--- | :--- |
| Open selector | Long press knob. | Theme Selector opens. | PENDING | |
| Browse forward | Rotate CW. | Theme preview index increments. | PENDING | |
| Browse backward | Rotate CCW. | Theme preview index decrements. | PENDING | |
| Wrap behavior | Rotate past first/last. | Selector wraps or stops as documented. | PENDING | |
| Apply theme | Short press selected theme. | Theme applies and returns to main UI. | PENDING | |
| Persist theme | Power-cycle/reboot after applying. | Same theme remains active. | PENDING | |
| HA theme sensor | Check HA entity. | Active theme text updates after apply. | PENDING | |

## 4.4 Theme-by-theme visual validation

| # | Theme | Minimum expected visible difference | Status | Visual issue / notes |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Minimal Thermostat | Minimal thermostat-style palette/labels/motif. | PENDING | |
| 2 | SmartKnob-Inspired Arc | Arc/knob-inspired color and control emphasis. | PENDING | |
| 3 | Large Center Power Button | Large central power emphasis. | PENDING | |
| 4 | Single-Page Simple Mode | Simplified/low-density presentation. | PENDING | |
| 5 | Preset Ring UI | Preset/ring-inspired visual treatment. | PENDING | |
| 6 | Night Mode Ultra-Minimal | Very dim night-safe palette. | PENDING | |
| 7 | Text-First Utility | Typography-first presentation. | PENDING | |
| 8 | Apple Watch Complications | Small status/complication-inspired treatment. | PENDING | |
| 9 | LED-Ring Status-First | LED ring is visually important. | PENDING | |
| 10 | Three-Screen Tab Carousel | Clear tab/page-carousel identity. | PENDING | |
| 11 | Brightness-First UI | Brightness is the default hero. | PENDING | |
| 12 | Door Switch Replacement | Obvious wall-switch mental model. | PENDING | |
| 13 | Lunar Phase Visualization | Moon/lunar visual identity. | PENDING | |
| 14 | Sundial Shadow UI | Light/shadow/sundial identity. | PENDING | |
| 15 | Tree Ring Growth Pattern | Organic/tree-ring identity. | PENDING | |
| 16 | Topographic Contour Map | Contour/elevation identity. | PENDING | |
| 17 | Iris Aperture Mechanism | Optical aperture identity. | PENDING | |
| 18 | Radar Sweep Animation | Scanning/sweep identity. | PENDING | |
| 19 | Vinyl DJ Crossfader | Turntable/audio-instrument identity. | PENDING | |
| 20 | Eclipse Corona | Eclipse/corona glow identity. | PENDING | |

## 4.5 Sensor, LED, and unavailable-state validation

| Test item | Procedure | Expected behavior | Status | Evidence / notes |
| :--- | :--- | :--- | :--- | :--- |
| LED ring ON | Turn lights ON. | LED ring follows active theme behavior. | PENDING | |
| LED ring OFF | Turn lights OFF. | LED ring dims/off as designed. | PENDING | |
| Theme LED color | Apply several themes. | LED color/behavior changes with theme. | PENDING | |
| LED brightness comfort | Observe in dark room. | Not painfully bright. | PENDING | |
| TSL2591 adaptive backlight | Change room light level if sensor present. | Display backlight changes appropriately. | PENDING | |
| SHT45 diagnostic | Check HA diagnostics if sensor present. | Temp/humidity present; not a main UI page. | PENDING | |
| API unavailable | Temporarily disconnect HA/network safely. | UI shows unavailable state without crashing. | PENDING | |

# Stage 5 — Bedside ESP32-C6 flashing

| Step | Command / action | Expected result | Status | Evidence / notes |
| :--- | :--- | :--- | :--- | :--- |
| 5.1 | Confirm APDS-9960 wiring / STEMMA connection. | Sensor physically connected. | PENDING | |
| 5.2 | Compile bedside YAML. | `esphome compile esphome/bedside_gesture.yaml` passes. | PENDING | |
| 5.3 | Connect ESP32-C6 via USB data cable. | Serial device appears. | PENDING | Port: |
| 5.4 | Flash via USB. | `esphome run esphome/bedside_gesture.yaml` uploads successfully. | PENDING | |
| 5.5 | Keep serial monitor open after reboot. | No bootloop, panic, or repeated reset. | PENDING | |
| 5.6 | Confirm Wi-Fi/API connection. | Device appears online in HA / ESPHome. | PENDING | |
| 5.7 | Confirm APDS-9960 detection. | No I2C/sensor initialization errors. | PENDING | |

# Stage 6 — Bedside gesture validation

| Test item | Procedure | Expected behavior | Status | Evidence / notes |
| :--- | :--- | :--- | :--- | :--- |
| LEFT gesture | Swipe left above APDS-9960. | `light.bedroom_group` turns OFF. | PENDING | |
| RIGHT gesture | Swipe right above APDS-9960. | `light.bedroom_group` turns ON. | PENDING | |
| Cooldown | Repeat gestures quickly. | Cooldown prevents rapid repeated triggers. | PENDING | |
| False trigger check | Stand nearby / move normally. | No accidental triggers. | PENDING | |
| HA reflects state | Watch `light.bedroom_group`. | HA state updates after gesture. | PENDING | |
| APDS diagnostics | Check available entities/logs. | Sensor status is stable. | PENDING | |
| VL53L4CD v1 behavior | Try hand-hold if hardware attached. | No v1 action unless explicitly implemented later. | PENDING | |
| Sensor fusion | Review behavior/logs. | No APDS+VL53 fusion behavior in v1. | PENDING | |

# Stage 7 — Full end-to-end system validation

| Test item | Procedure | Expected behavior | Status | Evidence / notes |
| :--- | :--- | :--- | :--- | :--- |
| Door → HA → lights power | Use door Power page. | Bedroom group toggles; HA state matches. | PENDING | |
| Door → HA → lights brightness | Use door Brightness page. | Bulbs change brightness; HA state matches. | PENDING | |
| Door → HA → presets | Apply all 4 presets. | Bulbs match expected preset behavior. | PENDING | |
| Door theme persistence | Apply theme, reboot door device. | Same theme returns. | PENDING | |
| Bedside → HA → lights OFF | LEFT gesture. | Bedroom group turns OFF. | PENDING | |
| Bedside → HA → lights ON | RIGHT gesture. | Bedroom group turns ON. | PENDING | |
| Cross-device state recovery | Change lights from one device, observe other path. | HA remains source of truth; no stuck state. | PENDING | |
| HA dashboard reflects both | Perform door and bedside actions. | HA UI updates for both. | PENDING | |
| Local-only behavior | Disconnect internet but keep LAN if safe. | Local HA/LocalTuya path continues if configured locally. | PENDING | |
| Pi reboot recovery | Reboot Raspberry Pi. | ESP devices reconnect; controls recover. | PENDING | |
| Door reboot recovery | Reboot ELECROW board. | Device reconnects and theme persists. | PENDING | |
| Bedside reboot recovery | Reboot ESP32-C6. | Device reconnects; gestures resume. | PENDING | |
| 30-minute soak | Leave system running and test intermittently. | No crashes, disconnect loops, overheating, or UI lockups. | PENDING | |

# Stage 8 — Evidence collection

| Evidence item | Required for PASS? | Status | Link / filename / notes |
| :--- | :--- | :--- | :--- |
| Photo of HA dashboard loaded on Raspberry Pi system | Yes | PENDING | |
| Screenshot/photo of ESPHome devices online | Yes | PENDING | |
| Screenshot of `light.bedroom_group` entity | Yes | PENDING | |
| Door-side boot photo/video | Yes | PENDING | |
| Video: long press opens Theme Selector | Yes | PENDING | |
| Video: long press does not toggle light | Yes | PENDING | |
| Video: browsing all 20 themes | Yes | PENDING | |
| Photo/video: selected theme persists after reboot | Yes | PENDING | |
| Video: door controls light group | Yes | PENDING | |
| Video: bedside LEFT/RIGHT gesture controls lights | Yes | PENDING | |
| Serial logs for any failure | Required if failure occurs | PENDING | |
| Final completed results template | Yes | PENDING | |

# Stage 9 — Final system sign-off

| Subsystem | Required result before system PASS | Result | Evidence link / notes |
| :--- | :--- | :--- | :--- |
| Raspberry Pi setup | PASS | PENDING | |
| Home Assistant setup | PASS | PENDING | |
| LocalTuya / `light.bedroom_group` setup | PASS | PENDING | |
| Door-side flashing | PASS | PENDING | |
| Door-side hardware validation | PASS or conditional PASS with documented limits | PENDING | |
| Bedside flashing | PASS | PENDING | |
| Bedside gesture validation | PASS or conditional PASS with documented limits | PENDING | |
| Full E2E command path | PASS | PENDING | |
| Evidence package complete | PASS | PENDING | |
| Final decision | `[ ] PASS` / `[ ] FAIL` / `[ ] CONDITIONAL PASS` | PENDING | |

## Conditional PASS rules

A conditional PASS is allowed only when the core system works and limitations are explicitly documented. Examples:

| Condition | Conditional PASS allowed? | Notes |
| :--- | :--- | :--- |
| One visual theme needs cosmetic refinement but controls work | Yes | Log visual issue and continue. |
| TSL2591 not present on board revision but core UI works | Yes, if documented | Mark TSL2591 as N/A for that hardware revision. |
| LED ring color order is wrong but controls work | Yes, if documented | Requires follow-up firmware fix before final polish. |
| Long press also toggles light | No | Safety gate failure. |
| `light.bedroom_group` cannot be controlled from HA | No | E2E gate failure. |
| Door-side firmware bootloops | No | Core hardware gate failure. |
| Bedside gestures cannot control lights | No for full system PASS | Can only pass door-side subsystem. |

## Final note

This checklist is intentionally strict. The current evidence supports a narrow door-side ON/OFF bridge PASS only. VelaDial is not fully physically validated until the filled results template contains evidence-backed PASS/FAIL results for the remaining door-side behavior, bedside behavior, and full-system scenarios.
