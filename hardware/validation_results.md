# VelaDial — Hardware Validation Results

**Status:** Active validation log — fill in incrementally as physical validation
phases execute.  
**Project:** VelaDial  
**Scope:** Single consolidated results document, as defined by
`docs/13_Firmware_Prep_Validation_Plan.md` §7.

---

## How to use this document

- One section per validation phase (per
  `docs/13_Firmware_Prep_Validation_Plan.md` §3).
- Each section is owned by a named tester for a specific board / device in
  hand.
- Do **not** invent results. Any field whose value has not been physically
  observed must be recorded as
  `NOT TESTED — awaiting Hardik physical validation input`.
- Do **not** edit `hardware/elecrow_pinout.md` based on guesses. Only update
  the pinout doc when physical deltas have been confirmed by Hardik on the
  actual board in hand.

---

## Step 15B — ELECROW Door-Side Board / Pinout Validation Results

Implements the validation defined in
`docs/validation/step_15a_elecrow_pinout_validation.md` (the Step 15A plan).

### Identification

| Field | Value |
| --- | --- |
| Board (model) | ELECROW CrowPanel 1.28 in ESP32-S3 round rotary touch display |
| Board revision / silkscreen / sticker | NOT TESTED — awaiting Hardik physical validation input |
| Validation date | NOT TESTED — awaiting Hardik physical validation input |
| Tester name | NOT TESTED — awaiting Hardik physical validation input |
| Bring-up YAML used | `esphome/door_side_rotary.yaml` (existing bring-up, unchanged) |
| Pinout source-of-truth used | `hardware/elecrow_pinout.md` |

### Evidence checklist

| Evidence item | Status | Location / reference |
| --- | --- | --- |
| Board front photo captured | NOT TESTED — awaiting Hardik physical validation input | — |
| Board back photo captured | NOT TESTED — awaiting Hardik physical validation input | — |
| Revision marking (silkscreen / sticker) photo captured | NOT TESTED — awaiting Hardik physical validation input | — |
| Serial logs from bring-up flash captured | NOT TESTED — awaiting Hardik physical validation input | — |
| I²C scan output captured | NOT TESTED — awaiting Hardik physical validation input | — |

> Photos and logs will be attached to the Step 15B PR (or stored in an
> agreed-upon location) once Hardik provides them. Do not commit raw binary
> assets to the repo unless the team explicitly decides to.

### Subsystem results

Statuses: **PASS** / **PARTIAL** / **FAIL** / **BLOCKED** / **NOT TESTED**.
(**PARTIAL** is only acceptable with a documented workaround approved by
Hardik — see the Step 15B gate below.)

| # | Check | Expected result | Actual result | Status | Evidence / notes |
| -: | --- | --- | --- | --- | --- |
| 1 | Power-on baseline | Board powers up on USB 5 V / 2 A; power LED on GPIO40 behaves as documented; no brownout, no overheat, no reboot loop. | — | NOT TESTED — awaiting Hardik physical validation input | — |
| 2 | Compile `esphome/door_side_rotary.yaml` | ESPHome compile completes without errors for the existing bring-up YAML, no pin / component conflicts. | — | NOT TESTED — awaiting Hardik physical validation input | Compile only — no behavior changes to YAML. |
| 3 | Flash bring-up YAML | Flash succeeds; device boots; serial log shows clean boot, no panic, no strap-pin warning, no reboot loop. | — | NOT TESTED — awaiting Hardik physical validation input | — |
| 4 | Display — GC9A01A | Display initializes over SPI on the documented pins (SCLK 10, MOSI 11, DC 3, CS 9, RST 14); LVGL bring-up page renders ("VelaDial" / "Press dial"). | — | NOT TESTED — awaiting Hardik physical validation input | Confirm `invert_colors: true` is still correct for this panel; record if not. |
| 5 | Backlight PWM | Backlight on GPIO46 sweeps smoothly 0 % → 100 % via the `back_light` light entity; visible change; no flicker / whine. | — | NOT TESTED — awaiting Hardik physical validation input | — |
| 6 | CST816 I²C address `0x15` | I²C scan on SDA GPIO6 / SCL GPIO7 reports a device at `0x15`. | — | NOT TESTED — awaiting Hardik physical validation input | — |
| 7 | CST816 coordinate orientation | Touches generate coordinates; existing transform (`mirror_y: true`, `swap_xy: true`) maps touches to visible panel correctly. | — | NOT TESTED — awaiting Hardik physical validation input | Record any required transform change — do not "fix" in production YAML in Step 15B. |
| 8 | Rotary encoder CW / CCW | Rotating CW and CCW changes encoder count in opposite directions on pins A=GPIO45, B=GPIO42. | — | NOT TESTED — awaiting Hardik physical validation input | Record observed direction. Do not "fix" direction in production YAML in Step 15B. |
| 9 | Rotary encoder press | Pressing the encoder shaft registers a press on GPIO41 (`rotary_button` in bring-up YAML). | — | NOT TESTED — awaiting Hardik physical validation input | — |
| 10 | WS2812 LED ring (5 LEDs) | All 5 LEDs on GPIO48 light up in order from a temporary test entity; no dropped pixels. | — | NOT TESTED — awaiting Hardik physical validation input | Use a temporary test entity only — do not commit it to production YAML. |
| 11 | 30-minute stability soak | Board runs ≥ 30 min with display + backlight + touch active; no reboots, no I²C bus errors, no thermal issues. | — | NOT TESTED — awaiting Hardik physical validation input | Attach final serial-log tail to evidence. |

### Pinout delta vs `hardware/elecrow_pinout.md`

#### Confirmed matches

| Subsystem | Function | Documented (`hardware/elecrow_pinout.md`) | Physically confirmed? |
| --- | --- | --- | --- |
| Display | SCLK | GPIO10 | NOT TESTED — awaiting Hardik physical validation input |
| Display | MOSI | GPIO11 | NOT TESTED — awaiting Hardik physical validation input |
| Display | DC | GPIO3 | NOT TESTED — awaiting Hardik physical validation input |
| Display | CS | GPIO9 | NOT TESTED — awaiting Hardik physical validation input |
| Display | RST | GPIO14 | NOT TESTED — awaiting Hardik physical validation input |
| Display | Backlight (PWM) | GPIO46 | NOT TESTED — awaiting Hardik physical validation input |
| Touch | SDA | GPIO6 | NOT TESTED — awaiting Hardik physical validation input |
| Touch | SCL | GPIO7 | NOT TESTED — awaiting Hardik physical validation input |
| Touch | INT | GPIO5 | NOT TESTED — awaiting Hardik physical validation input |
| Touch | RST | GPIO13 | NOT TESTED — awaiting Hardik physical validation input |
| Touch | I²C address `0x15` | `0x15` | NOT TESTED — awaiting Hardik physical validation input |
| Encoder | A | GPIO45 | NOT TESTED — awaiting Hardik physical validation input |
| Encoder | B | GPIO42 | NOT TESTED — awaiting Hardik physical validation input |
| Encoder | Switch | GPIO41 | NOT TESTED — awaiting Hardik physical validation input |
| WS2812 | Data | GPIO48 | NOT TESTED — awaiting Hardik physical validation input |
| WS2812 | LED count | 5 | NOT TESTED — awaiting Hardik physical validation input |
| Power indicator | Power LED | GPIO40 | NOT TESTED — awaiting Hardik physical validation input |

#### Confirmed mismatches

| Subsystem | Function | Documented | Observed on board | Notes / proposed action |
| --- | --- | --- | --- | --- |
| _none yet_ | — | — | — | NOT TESTED — awaiting Hardik physical validation input |

> ⚠️ Do **not** edit `hardware/elecrow_pinout.md` based on entries here until
> Hardik has physically confirmed the delta and explicitly approved the
> pinout-doc update in a separate, narrowly-scoped PR.

#### Unresolved TBDs

| Item | Why it is TBD | Resolution path |
| --- | --- | --- |
| Board revision / silkscreen | Not yet observed in hand | Photograph silkscreen, record value here. |
| Onboard OLED presence (`SDA GPIO38`, `SCL GPIO39`) | Optional on some board revisions | Visual inspection + I²C scan on bus, record presence/absence. |
| Test IO usage (`GPIO4`, `GPIO12`) | Listed as test IO in pinout doc; intended door-side function not finalized | Out of scope for Step 15B unless they conflict with bring-up. |

---

## Step 15B Gate

> 🛑 **Do not proceed to production door-side firmware, production door-side
> YAML, the 3-page LVGL UI, door-side sensor wiring (TSL2591 / SHT45), Home
> Assistant action wiring, VL53L4CD work, or any sensor fusion work until
> Step 15B is completed end-to-end and the results in this document have been
> reviewed and approved by Hardik.**

Specifically, until this gate is lifted, do **not**:

- Modify `esphome/door_side_rotary.yaml` for production behavior.
- Create any new production YAML under `esphome/`.
- Begin door-side sensor (TSL2591, SHT45) validation
  (`docs/13_Firmware_Prep_Validation_Plan.md` §3.C).
- Begin VL53L4CD / VL53L0X work
  (`docs/13_Firmware_Prep_Validation_Plan.md` §3.A / §3.E).
- Begin LVGL Power / Brightness / Presets page implementation.
- Begin sensor fusion work (explicitly out of v1).

Gate lift requires: all subsystem rows above marked **PASS** (or **PARTIAL**
with documented workaround approved by Hardik), all evidence items captured,
and any pinout deltas resolved.

### Step 15B gate — process waiver note (owner-acknowledged)

> 📝 **Process waiver acknowledged by Hardik (owner).**
>
> PR #19 created a production-oriented door-side YAML
> (`esphome/door_side_rotary.yaml`, full LVGL 3-page UI + wake-only-first
> guards + preset/brightness/HA-state logic) **before** any physical Step 15B
> validation was completed. This is accepted **only** as a pre-hardware draft
> for compile-time and design-level review.
>
> This acceptance does **NOT** lift the Step 15B physical validation gate.
> No physical PASS is claimed. All hardware rows in this document remain
> **NOT TESTED**. The YAML's own header continues to mark it
> `DRAFT — COMPILE PASSED / HARDWARE VALIDATION PENDING`.
>
> The gate continues to apply to:
> - All further production-firmware-behavior changes to
>   `esphome/door_side_rotary.yaml` (beyond compile/lint cleanups that do not
>   change behavior).
> - Any new production YAML under `esphome/`.
> - Door-side sensor wiring (TSL2591 / SHT45) validation.
> - LVGL UI sign-off as production-ready.
> - VL53L4CD / VL53L0X / sensor fusion work (independently gated).
>
> The gate is **lifted only** when Hardik records physical Step 15B PASS
> results (or owner-approved PARTIAL with workaround) in the subsystem table
> above, with evidence captured, and explicitly signs off in this section.

---

## Phase 1 — Door-side ESP32-S3 Firmware Draft
	
	**Date:** 2026-05-24  
	**Status:** Draft created (PR #19 merged)  
	**Compile status:** PASSED in Manus environment using ESPHome 2026.5.0 (binary size 1,194,131 bytes, 0 errors, 4 non-blocking warnings). Flash and physical validation remain NOT TESTED.  
	
	A draft production-oriented YAML (`esphome/door_side_rotary.yaml`) has been created. The YAML compiled without errors in the sandbox environment (ESPHome 2026.5.0, ESP-IDF framework). It has **not** been flashed to or tested on physical hardware. The Step 15B gate above remains **unlifted** — physical board validation is still required before this draft can be considered production-ready.
	
	## Phase 2 — Bedside ESP32-C6 APDS Firmware Draft
	
	**Date:** 2026-05-24  
	**Status:** Draft created  
	**Compile status:** PASSED (ESPHome 2026.5.0, ESP-IDF, 0 errors, 0 warnings, 181.5s)  
	**Physical Validation:** NOT TESTED  
	
	A draft production-oriented YAML (`esphome/bedside_gesture.yaml`) has been created on branch `firmware/bedside-apds-v1-draft`. It uses native ESPHome APDS-9960 gesture support. No sensor fusion is implemented. Physical validation remains pending.

### Pin Cross-Validation (Source Research)

The YAML pin assignments were cross-validated against **three independent sources** on 2026-05-24:

| Source | URL | Agreement |
| --- | --- | --- |
| Elecrow Official GitHub (factory source code) | https://github.com/Elecrow-RD/CrowPanel-1.28inch-HMI-ESP32-Rotary-Display-240-240-IPS-Round-Touch-Knob-Screen | Full match |
| Incipiens Community ESPHome YAML (XDA/HA) | https://github.com/Incipiens/Elecrow-Rotary-Displays | Full match |
| Makerguides.com Tutorial (pin table) | https://www.makerguides.com/getting-started-crowpanel-1-28inch-hmi-esp32-rotary-display/ | Full match |

All three sources confirm: Display SPI (SCLK=10, MOSI=11, CS=9, DC=3, RST=14), Backlight=GPIO46, Touch (SDA=6, SCL=7, INT=5, RST=13, addr 0x15), Encoder (A=45, B=42, SW=41), WS2812 (GPIO48, 5 LEDs), Power LED (GPIO40), invert_colors=true, touch transform (mirror_y=true, swap_xy=true).

This does NOT replace physical validation. It confirms the YAML is aligned with all known documentation sources.

---

## Phase 3 — Raspberry Pi / Home Assistant Setup Guide

**Date:** 2026-05-25  
**Status:** Setup guide created (`docs/setup/raspberry_pi_home_assistant_setup.md`)  
**HA / ESPHome / LocalTuya Setup:** NOT TESTED  
**Physical PASS Results Added:** No  

A comprehensive setup guide has been created documenting the intended deployment path for Home Assistant OS on Raspberry Pi, ESPHome add-on configuration, LocalTuya integration, and `light.bedroom_group` creation. All steps remain NOT TESTED until Hardik performs the physical setup.

---

## Phase 4 — Full E2E Setup and Validation Guide

**Date:** 2026-05-25  
**Status:** E2E guide created (`docs/setup/full_e2e_setup_and_validation_guide.md`)  
**E2E Validation:** NOT TESTED  
**Physical PASS Results Added:** No  

A full E2E setup and validation guide has been created. It defines the complete validation sequence for door-side, bedside, HA/LocalTuya command path, and full system scenarios. All results remain NOT TESTED until Hardik performs them on physical hardware.

---

## Phase 5 — UI/UX Guide Integration Plan

**Date:** 2026-05-25  
**Status:** UI/UX integration plan created (`docs/ui/ux_guide_integration_plan.md`)  
**UI Implementation:** NOT PERFORMED  
**Hardik UI/UX Guide:** PENDING HARDIK INPUT  
**Physical PASS Results Added:** No  

A UI/UX guide integration plan has been created documenting the agreed visual themes, locked v1 rules, pending inputs from Hardik, integration decision matrix, and acceptance criteria for future UI implementation. No UI code changes were made. No ESPHome YAML was edited. Physical validation remains NOT TESTED.

---

## Phase 6 — Claude Review Package

**Date:** 2026-05-25  
**Status:** Claude review package created (`docs/review/claude_review_package.md`)  
**Validation Performed:** No  
**Physical PASS Results Added:** No  

A Claude review package has been created for independent review of the VelaDial repository after Phases 0–5. This is a documentation-only deliverable. No validation was performed, no firmware was changed, and all hardware/E2E validation remains NOT TESTED.

---

## UI Concept Direction Package Note

**Date:** 2026-05-26  
**Status:** UI concept direction package created (6 documents in `docs/ui/`)  
**Validation Performed:** No  
**Physical PASS Results Added:** No  

A premium UI concept direction package has been created with 20 deeply expanded concepts, a scored shortlist, selection criteria with pass/fail gates, a creative design notebook, a visual batch plan, and implementation candidates. This is a documentation-only deliverable. No firmware was changed, no YAML was modified, and all hardware/E2E validation remains NOT TESTED. Hardik must select a concept direction before any UI implementation begins.

---

## UI Concept 01: Minimal Thermostat Note

**Date:** 2026-05-27  
**Status:** Concept prototype YAML created and compile PASSED (ESPHome 2026.5.0, 177s, 0 errors)  
**Validation Performed:** No  
**Physical PASS Results Added:** No  

Concept 01 (Minimal Thermostat) has been prototyped as a single-page UI concept for the door-side ELECROW controller. The concept uses a central label for power/brightness display and a surrounding arc for brightness indication. Knob CW/CCW adjusts brightness, knob press toggles power. Wake-only-first logic is preserved. This is a concept exploration only — not production, not tested on physical hardware. Four documentation files (research.md, design_spec.md, implementation_notes.md, validation_notes.md) accompany the YAML.

---

## UI Concept 02: SmartKnob-Inspired Arc Note

**Date:** 2026-05-27  
**Status:** Concept prototype YAML created and compile PASSED (ESPHome 2026.5.0, 361s, 0 errors)  
**Validation Performed:** No  
**Physical PASS Results Added:** No  

Concept 02 (SmartKnob-Inspired Arc) has been prototyped as a single-page UI concept for the door-side ELECROW controller. The concept features a prominent SmartKnob-style arc widget spanning 270 degrees (135° to 45°) with an amber indicator and white knob handle for brightness visualization. The arc is visual-only (no touch drag) — all interaction flows through the physical rotary encoder (CW/CCW for brightness, press for power toggle). A large central percentage label doubles as a touch-to-toggle power button. Wake-only-first logic is enforced on all input paths (touch, knob CW/CCW, knob press). Home Assistant state import keeps the UI synchronized with `light.bedroom_group`. This is a concept exploration only — not production, not tested on physical hardware. Four documentation files (research.md, design_spec.md, implementation_notes.md, validation_notes.md) accompany the YAML.

---

## UI Concept 03: Large Center Power Button Note

| Field | Value |
| --- | --- |
| Concept YAML | `esphome/concepts/door_side_concept_03_large_center_power.yaml` |
| Compile status | **PASSED** (ESPHome 2026.5.0, ESP-IDF, 0 errors, 386s) |
| Physical validation | NOT TESTED — awaiting Hardik physical validation input |

Concept 03 (Large Center Power Button) has been prototyped as a 3-page UI concept for the door-side ELECROW controller. The concept features an oversized 140px glowing circular power button as the hero element on the Power page, with a brightness arc on Page 2 and a diamond-layout preset selector on Page 3. The power button uses LVGL shadow properties to create an amber glow halo when on, and a ghost outline when off. Wake-only-first logic is enforced on all input paths (touch, knob CW/CCW, knob press). LED ring mirrors the power state (amber when on, off when off). Home Assistant state import keeps the UI synchronized with `light.bedroom_group`. This is a concept exploration only — not production, not tested on physical hardware. Four documentation files (research.md, design_spec.md, implementation_notes.md, validation_notes.md) accompany the YAML.

---

## UI Concept 04: Single-Page Simple Mode Note

| Field | Value |
| --- | --- |
| Concept YAML | `esphome/concepts/door_side_concept_04_single_page_simple.yaml` |
| Compile status | **PASSED** (ESPHome 2026.5.0, ESP-IDF, 0 errors, 374s) |
| Gate G1 status | **FAILS** — uses 1 page instead of mandated 3 |
| Physical validation | NOT TESTED — awaiting Hardik physical validation input |

Concept 04 (Single-Page Simple Mode) has been prototyped as a single-page UI concept for the door-side ELECROW controller. The concept places ALL controls on one screen: power state in the top band (tap to toggle), brightness percentage with thin arc in the center band (knob-only control), and active preset name in the bottom band (tap to cycle through 4 presets). This is the most radical simplification in the 20-concept matrix — no page navigation, no swipe gestures, no page indicator dots. Wake-only-first logic is enforced on all input paths. **This concept FAILS Gate G1 (Three-Page Lock) and cannot be selected for production without explicit owner waiver.** This is a concept exploration only — not production, not tested on physical hardware.

---

## UI Concept 05: Preset Ring UI Note

| Field | Value |
| --- | --- |
| Concept YAML | `esphome/concepts/door_side_concept_05_preset_ring.yaml` |
| Compile status | **PASSED** (ESPHome 2026.5.0, ESP-IDF, 0 errors, 378s) |
| Gate status | **PASSES all gates (G1-G8)** |
| Physical validation | NOT TESTED |

Concept 05 (Preset Ring UI) uses 4 colored arc segments around the ring as a preset selector on the Presets page. The ring metaphor unifies all 3 pages: full ring (power), proportional arc (brightness), segmented ring (presets). Knob rotation cycles through presets; press applies. LED ring mirrors preset color. Wake-only-first enforced.

---

## UI Concept 06: Night Mode Ultra-Minimal Note

| Field | Value |
| --- | --- |
| Concept YAML | `esphome/concepts/door_side_concept_06_night_mode_ultra_minimal.yaml` |
| Compile status | **PASSED** (ESPHome 2026.5.0, ESP-IDF, 0 errors, 86s incremental) |
| Gate status | **PASSES all gates (G1-G8)** — Layer concept |
| Physical validation | NOT TESTED |

Concept 06 (Night Mode Ultra-Minimal) is a layer concept — the sleep/dark-room state that overlays any active primary concept. When the TSL2591 ambient light sensor reads below 5 lux for 10 seconds, the display enters Night Mode: backlight drops to 10% PWM, all UI elements disappear except a single 8px amber dot (lights off) or dim percentage value (lights on). ALL inputs in Night Mode are wake-only — no action is executed, only the full UI is restored. Hysteresis band (5 lux entry, 15 lux exit) prevents oscillation.

---

## UI Concept 07: Text-First Utility Note

| Field | Value |
| --- | --- |
| Concept YAML | `esphome/concepts/door_side_concept_07_text_first_utility.yaml` |
| Compile status | **PASSED** (ESPHome 2026.5.0, ESP-IDF, 0 errors, 464s) |
| Gate status | **PASSES all gates (G1-G8)** — v1 candidate |
| Physical validation | NOT TESTED |

Concept 07 (Text-First Utility) rejects all decorative elements. No arcs, rings, gradients, or glow effects. Every piece of information is communicated through typography alone: 56pt primary values (ON/OFF/percentage), 24pt preset names in a vertical list, 16pt section headers. The amber accent color is used only for active state and page dots. Design reference: Massimo Vignelli subway signage, aviation instrumentation. Best daylight readability of any concept in the matrix.

---

## UI Concept 08: Apple Watch Complications Note

| Field | Value |
| --- | --- |
| Concept YAML | `esphome/concepts/door_side_concept_08_apple_watch_complications.yaml` |
| Compile status | **PASSED** (ESPHome 2026.5.0, ESP-IDF, 0 errors, 326s) |
| Gate status | **PASSES all gates (G1-G8)** — v1-expanded adaptation only |
| v1 Recommendation | **NOT RECOMMENDED** per direction matrix — too complex |
| Physical validation | NOT TESTED |

Concept 08 (Apple Watch Complications) borrows the watchOS Infograph "complications" metaphor: small data widgets at the top corners of each page showing ambient sensor data (WiFi RSSI top-left, ambient lux top-right). The v1-expanded adaptation uses only 2 complications per page in static gray, display-only (not interactive). The center content remains the hero element on each page. At 190 PPI (vs Apple Watch 326 PPI), information density is deliberately reduced. Full complication face with 4-6 interactive complications is v2 scope only.

---

## UI Concept 09: LED-Ring Status-First Note

| Field | Value |
| --- | --- |
| Concept YAML | `esphome/concepts/door_side_concept_09_led_ring_status_first.yaml` |
| Compile status | **PASSED** (ESPHome 2026.5.0, ESP-IDF, 0 errors, 81s) |
| Gate status | **PASSES all gates (G1-G8)** — Layer concept |
| Physical validation | NOT TESTED |
| LED hardware tests | ALL PENDING (color order, LED 0 position, brightness ceiling, RMT conflicts) |

Concept 09 (LED-Ring Status-First) inverts the typical smart-display hierarchy. The 5-LED WS2812 ring on GPIO48 is the PRIMARY output — visible from across the room — while the 240x240 screen is secondary with only 6 LVGL widgets (3 labels + 3 page dots). LED color maps to active preset (amber/gold/white/dim amber), LED brightness is proportional to light brightness (capped at 50% for bedroom safety), and error states use red pulse patterns. The LED ring has an extended 30-second timeout after screen sleep for ambient room awareness. All LED behavior is hardware-test-pending: color order (GRB assumed), LED 0 physical orientation, brightness ceiling comfort, and RMT channel conflicts are unverified.

---

## UI Concept 10: Three-Screen Tab Carousel Note

| Field | Value |
| --- | --- |
| Concept YAML | `esphome/concepts/door_side_concept_10_three_screen_tab_carousel.yaml` |
| Compile status | **PASSED** (ESPHome 2026.5.0, ESP-IDF, 0 errors, 417s) |
| Gate status | **PASSES all gates (G1-G8)** — Direct v1 spec implementation |
| Physical validation | NOT TESTED |

Concept 10 (Three-Screen Tab Carousel) is the direct, literal implementation of the locked v1 specification. Three horizontally swipeable pages (Power, Brightness, Presets) with a 3-dot page indicator in the LVGL top_layer (always visible). The concept uses the production YAML's exact architecture: FLEX-layout dot container, guarded swipe scripts, 2x2 preset grid with FLEX ROW_WRAP, and a display-only brightness arc controlled exclusively by the rotary encoder. Premium execution details: 200ms MOVE_LEFT/MOVE_RIGHT transitions, active dot grows from 8px to 10px, amber accent throughout. This is the safe baseline against which all other concepts are compared. Zero compile errors on first attempt.

---

## UI Concept 11: Brightness-First UI Note

| Field | Value |
| --- | --- |
| Concept YAML | `esphome/concepts/door_side_concept_11_brightness_first_ui.yaml` |
| Compile status | **PASSED** (ESPHome 2026.5.0, ESP-IDF, 0 errors, 429s) |
| Gate status | **PASSES all gates (G1-G8)** — IA innovation |
| Physical validation | NOT TESTED |

Concept 11 (Brightness-First UI) is architecturally identical to Concept 10 with one critical information architecture change: the default landing page is Brightness (page 0) instead of Power. The hypothesis is that "adjust brightness" is the most common bedroom action — especially at night. Users wake the device and immediately see the current brightness level and can rotate the knob to adjust without any page navigation. Knob press on the Brightness page is a power toggle shortcut, making the two most common actions (adjust brightness, toggle power) accessible without any swipe. Zero compile errors on first attempt. Page order: Brightness → Power → Presets.

---

## Document control

**Version:** 0.18 — Added Concept 11 Brightness-First UI note; compile PASSED, IA innovation (brightness as default landing), zero errors on first attempt.
**Owner approval required:** Yes, before lifting the Step 15B gate.  
**Next phase after sign-off:** Door-side sensor validation
(`docs/13_Firmware_Prep_Validation_Plan.md` §3.C).
