# End-to-End Test Results Template

> **Tester:** Hardik  
> **Date:** [YYYY-MM-DD]  
> **Repo commit tested:** [commit SHA]  
> **Door-side firmware:** `esphome/door_side_rotary.yaml`  
> **Bedside firmware:** `esphome/bedside_gesture.yaml`  
> **Target:** Full VelaDial system

Use this template to record physical validation results. Do not mark PASS from compile alone. Every PASS should be backed by observation, photo/video, HA screenshot, or serial log where applicable.

## 1. Test environment

| Item | Value / notes |
| :--- | :--- |
| Raspberry Pi model | |
| Home Assistant version | |
| ESPHome version | |
| Local bulb integration used | LocalTuya / other: |
| Door-side board | ELECROW CrowPanel 1.28 ESP32-S3 rotary display |
| Bedside board | Adafruit ESP32-C6 Feather |
| Bedroom bulbs tested | |
| Network type | Wi-Fi / Ethernet / mixed |
| Tester notes | |

## 2. Stage results summary

| Stage | Result (`PENDING` / `PASS` / `FAIL` / `N/A` / `RETEST REQUIRED`) | Evidence / notes |
| :--- | :--- | :--- |
| Stage 0 — Hardware/materials ready | PENDING | |
| Stage 1 — Raspberry Pi Home Assistant setup | PENDING | |
| Stage 2 — HA base config / ESPHome / LocalTuya / `light.bedroom_group` | PENDING | |
| Stage 3 — Door-side ELECROW flashing | PENDING | |
| Stage 4 — Door-side physical validation | PENDING | |
| Stage 5 — Bedside ESP32-C6 flashing | PENDING | |
| Stage 6 — Bedside gesture validation | PENDING | |
| Stage 7 — Full E2E command path | PENDING | |
| Stage 8 — Evidence collection | PENDING | |
| Stage 9 — Final sign-off | PENDING | |

## 3. Raspberry Pi / Home Assistant results

| Test item | Result | Evidence / notes |
| :--- | :--- | :--- |
| HA OS flashed to microSD | PENDING | |
| Raspberry Pi boots | PENDING | |
| HA URL/IP reachable | PENDING | |
| HA account created | PENDING | |
| HA dashboard loads | PENDING | |
| ESPHome add-on installed and opens | PENDING | |
| `secrets.yaml` configured locally only | PENDING | |
| No secrets committed to repo | PENDING | |

## 4. LocalTuya / bedroom light group results

| Test item | Result | Evidence / notes |
| :--- | :--- | :--- |
| LocalTuya / local bulb integration installed | PENDING | |
| Bedroom bulbs added | PENDING | |
| Individual bulb ON/OFF works | PENDING | |
| Individual bulb brightness works | PENDING | |
| Individual bulb color temperature works | PENDING | |
| `light.bedroom_group` created | PENDING | |
| `light.bedroom_group` ON/OFF works from HA | PENDING | |
| `light.bedroom_group` brightness works from HA | PENDING | |
| `light.bedroom_group` color temperature works from HA | PENDING | |
| Local-only behavior checked where applicable | PENDING | |

## 5. Door-side flashing and boot results

| Test item | Result | Evidence / notes |
| :--- | :--- | :--- |
| Door-side compile passes | PENDING | |
| Door-side USB flash succeeds | PENDING | |
| Serial boot logs clean | PENDING | |
| Display renders | PENDING | |
| Wi-Fi/API connection appears in HA | PENDING | |
| Active theme diagnostic text sensor appears | PENDING | |

## 6. Door-side control results

| Test item | Result | Evidence / notes |
| :--- | :--- | :--- |
| Touch works | PENDING | |
| Swipe/page navigation works | PENDING | |
| Knob rotation works | PENDING | |
| Short press works | PENDING | |
| Long press opens Theme Selector | PENDING | |
| Long press does not toggle lights | PENDING | |
| Wake-only-first touch works | PENDING | |
| Wake-only-first knob rotation works | PENDING | |
| Wake-only-first knob press works | PENDING | |
| Power page toggles `light.bedroom_group` | PENDING | |
| Brightness page changes brightness | PENDING | |
| Presets page applies all 4 presets | PENDING | |
| API unavailable state checked | PENDING | |
| Screen idle/sleep behavior checked | PENDING | |

## 7. All 20 theme results

| # | Theme | Selector reachable | Applies successfully | Visible UI change | LED ring change | Reboot persistence | Visual refinement notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Minimal Thermostat | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 2 | SmartKnob-Inspired Arc | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 3 | Large Center Power Button | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 4 | Single-Page Simple Mode | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 5 | Preset Ring UI | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 6 | Night Mode Ultra-Minimal | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 7 | Text-First Utility | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 8 | Apple Watch Complications | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 9 | LED-Ring Status-First | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 10 | Three-Screen Tab Carousel | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 11 | Brightness-First UI | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 12 | Door Switch Replacement | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 13 | Lunar Phase Visualization | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 14 | Sundial Shadow UI | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 15 | Tree Ring Growth Pattern | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 16 | Topographic Contour Map | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 17 | Iris Aperture Mechanism | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 18 | Radar Sweep Animation | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 19 | Vinyl DJ Crossfader | PENDING | PENDING | PENDING | PENDING | PENDING | |
| 20 | Eclipse Corona | PENDING | PENDING | PENDING | PENDING | PENDING | |

## 8. Bedside flashing and gesture results

| Test item | Result | Evidence / notes |
| :--- | :--- | :--- |
| Bedside compile passes | PENDING | |
| Bedside USB flash succeeds | PENDING | |
| Serial boot logs clean | PENDING | |
| Wi-Fi/API connection appears in HA | PENDING | |
| APDS-9960 detected | PENDING | |
| LEFT gesture turns `light.bedroom_group` OFF | PENDING | |
| RIGHT gesture turns `light.bedroom_group` ON | PENDING | |
| Gesture cooldown works | PENDING | |
| No repeated accidental triggers | PENDING | |
| No VL53L4CD v1 behavior unless explicitly implemented later | PENDING | |
| No sensor fusion behavior | PENDING | |

## 9. Full end-to-end results

| Test item | Result | Evidence / notes |
| :--- | :--- | :--- |
| Door-side → HA → lights power path works | PENDING | |
| Door-side → HA → lights brightness path works | PENDING | |
| Door-side → HA → presets path works | PENDING | |
| Door-side theme persistence survives reboot | PENDING | |
| Bedside LEFT → HA → lights OFF works | PENDING | |
| Bedside RIGHT → HA → lights ON works | PENDING | |
| HA dashboard reflects changes from both ESP devices | PENDING | |
| Raspberry Pi reboot recovery works | PENDING | |
| Door-side reboot recovery works | PENDING | |
| Bedside reboot recovery works | PENDING | |
| 30-minute soak test passes | PENDING | |

## 10. Evidence package

| Evidence item | Required for PASS? | Result | File/link/notes |
| :--- | :--- | :--- | :--- |
| HA dashboard photo/screenshot | Yes | PENDING | |
| ESPHome devices online screenshot | Yes | PENDING | |
| `light.bedroom_group` screenshot | Yes | PENDING | |
| Door-side boot video/photo | Yes | PENDING | |
| Theme Selector long-press video | Yes | PENDING | |
| Video proving long-press does not toggle lights | Yes | PENDING | |
| Video cycling all 20 themes | Yes | PENDING | |
| Video/photo proving reboot theme persistence | Yes | PENDING | |
| Door-side controls lights video | Yes | PENDING | |
| Bedside gesture controls lights video | Yes | PENDING | |
| Serial logs for failures | If failure occurs | PENDING | |

## 11. Failure log

| ID | Stage/test | Observed behavior | Expected behavior | Steps to reproduce | Evidence | Severity | Next action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| F-001 | Example: Long press isolation | Light toggled before Theme Selector opened | Long press opens selector only | Hold knob for 2 seconds on Power page | video/log | Critical | Firmware fix required |
| | | | | | | | |
| | | | | | | | |
| | | | | | | | |

## 12. Visual refinement log

Use this section for theme polish issues that do not block core functionality.

| Theme | Issue | Severity | Follow-up needed? |
| :--- | :--- | :--- | :--- |
| Example: Night Mode Ultra-Minimal | Red still too bright in dark room | Medium | Lower LED opacity / adjust palette |
| | | | |
| | | | |

## 13. Final sign-off

| Overall result | Meaning | Signature | Date |
| :--- | :--- | :--- | :--- |
| `[ ] PASS` | All required gates pass on physical hardware. | | |
| `[ ] CONDITIONAL PASS` | Core system works; documented non-critical limitations remain. | | |
| `[ ] FAIL` | One or more required gates failed. | | |

## 14. Final decision notes

- Final decision: PASS / CONDITIONAL PASS / FAIL
- Blocking failures:
- Conditional limitations:
- Follow-up PRs required:
- Ready for daily use? Yes / No / Conditional
