# VelaDial — Final Completion Roadmap

**Status:** Active planning document
**Purpose:** Define exactly what remains before VelaDial v1 can be called *ready for hardware validation*, and what only Hardik can finish once physical hardware is available.
**Created:** 2026-05-26
**Owner:** Hardik
**Decision in force:** **No clean-repo migration yet.** Complete the current repo first; reconsider migration only after the project reaches *ready for hardware validation*.

---

## 1. Scope and intent

This document is the single answer to the question:

> *"What is left before VelaDial v1 is finished, and which of those things can be done without physical hardware?"*

It supplements `docs/MASTER_EXECUTION_ROADMAP.md` (which records the phase contract and validation gates) by translating that contract into a concrete, near-term punch list and a small PR sequence.

**This is not a re-plan.** The v1 scope locked in PRs #18–#27 is preserved verbatim. Nothing in this document changes:

- Door-side UI = exactly 3 pages (Power, Brightness, Presets).
- Presets = exactly 4 (Warm White, Soft Amber, Neutral White, Low Nightlight).
- Bedside = APDS-9960 standalone gestures only. No VL53L4CD firmware. No sensor fusion. No VL53L0X fallback.
- Wake-only-first behavior on the door side.
- Local-first control path: ESPHome → HA on RPi → LocalTuya → `light.bedroom_group`.

Any change to the above requires Hardik to explicitly re-open the relevant decision (see `docs/vl53l4cd_support_verification.md` for the re-open template).

---

## 2. Repo status after PRs #25–#27

| PR | Title | Status | Effect |
| :-- | :-- | :-- | :-- |
| #25 | Address immediate Claude review cleanup items | MERGED | Hardened `.gitignore`, changed preset values from strings to numerics *(later reverted in PR #30 — ESPHome's `homeassistant.action` schema requires string values, confirmed by CI)*, clarified APDS update_interval, recorded Step 15B process waiver. |
| #26 | Record VL53L4CD v1 decision — Option B (defer to v2) | MERGED | VL53L4CD removed from v1 software; PRD/Schema/Roadmap updated. |
| #27 | Prepare repo for clean migration | MERGED | Repo-level cleanup; main is in a good state for continued development. |

**Net effect:** The repo is cleaner and more internally consistent than at PR #24, but the project is **not** production complete. Firmware remains DRAFT, hardware validation remains entirely NOT TESTED, and the HA/RPi/LocalTuya path has been documented but never executed.

**In-flight (not yet merged):**
- **PR #28** (`firmware/v1-compile-hardening`) — opened by `nidhiuiux`. Inlines the door-side brightness arc `on_value` handler so direct touch/drag updates `brightness_pct`, preserving the wake-only-first guards. Removes the `IMPLEMENTATION PENDING` block. Single-file change to `esphome/door_side_rotary.yaml`. **No reviews yet. Do not merge before Hardik review.**

---

## 3. Completion matrix

### 3.1 Already complete

| Area | Artifact |
| :-- | :-- |
| Product spec | `docs/01_PRD.md` |
| Technical requirements | `docs/02_TRD.md` |
| App flow | `docs/03_App_Flow.md` |
| UI/UX brief | `docs/04_UI_UX_Design_Brief.md` |
| Entity model / backend schema | `docs/05_Backend_Schema.md` |
| Implementation plan | `docs/06_Implementation_Plan.md` |
| Research / alternatives | `docs/07_Research_Alternatives.md`, `docs/08_Sensor_Expansion_Research.md` |
| Firmware prep validation plan | `docs/13_Firmware_Prep_Validation_Plan.md` |
| Master roadmap | `docs/MASTER_EXECUTION_ROADMAP.md` |
| Reference resources | `docs/REFERENCE_RESOURCES.md` |
| RPi / HA setup guide | `docs/setup/raspberry_pi_home_assistant_setup.md` |
| Full E2E validation guide | `docs/setup/full_e2e_setup_and_validation_guide.md` |
| UI/UX integration plan | `docs/ui/ux_guide_integration_plan.md` |
| ELECROW pinout | `hardware/elecrow_pinout.md` |
| Sensor wiring plan | `hardware/sensor-wiring.md` |
| Source confirmation matrix | `docs/validation/elecrow_source_confirmation_matrix.md` |
| Step 15A pinout validation plan | `docs/validation/step_15a_elecrow_pinout_validation.md` |
| Validation results scaffold | `hardware/validation_results.md` |
| VL53L4CD decision (Option B) | `docs/vl53l4cd_support_verification.md` |
| Claude review package | `docs/review/claude_review_package.md` |
| Secrets handling baseline | `.gitignore`, `esphome/secrets.yaml.example` |

### 3.2 DRAFT — compile-passed in author env, not flashed

| Artifact | Notes |
| :-- | :-- |
| `esphome/door_side_rotary.yaml` | Full LVGL 3-page UI, wake-only-first dual-layer guard, knob-press-per-page, preset apply, adaptive backlight, HA state reconcile. Brightness-arc direct-touch fix in flight via PR #28. |
| `esphome/bedside_gesture.yaml` | APDS-9960 native gestures, 2 s cooldown, GPIO20 STEMMA QT power, no fusion, no VL53L4CD. |

### 3.3 NOT TESTED (only Hardik can resolve)

- All rows of `hardware/validation_results.md` (board revision photo, power-on, compile, flash, boot logs, I²C scan, display, backlight PWM, touch, encoder, WS2812, sensors, 30-minute soak).
- ESP32-C6 bedside compile, flash, GPIO20 power, APDS gestures, cooldown, soak.
- HA OS install, ESPHome add-on install, device adoption (door + bedside).
- LocalTuya install, Tuya bulb adoption, `light.bedroom_group` creation, on/off/brightness/color-temp control.
- Internet-off test, HA restart recovery, Wi-Fi reconnect, RPi reboot recovery.
- Full E2E scenarios (door knob press, swipes, presets, bedside gestures).
- UI/UX final visual sign-off on physical 240×240 round display.

### 3.4 Missing / not yet started

- A repeatable **compile CI workflow** (no `.github/workflows/` exists). "Compile PASSED" claims today are environment-specific and non-reproducible.
- **Home Assistant config examples** in-repo. There is no `homeassistant/` directory. The setup guide tells the user to create `light.bedroom_group` and LocalTuya entries manually with no template to copy.
- A **single-page hardware day-one runbook** that Hardik can follow when the box arrives. The E2E guide is comprehensive but long; a tight runbook keyed to `validation_results.md` would reduce setup-day friction.
- A **standalone UI/UX asset intake document**. The intake checklist currently lives inside the integration plan; promoting it to its own file makes it easier to hand to a designer.
- **OTA secret-key naming consistency** between door (`ota_key`) and bedside (`ota_password`). Both work today because `secrets.yaml.example` carries both names, but harmonizing reduces foot-gun risk.

---

## 4. Remaining tasks by area

### 4.1 Firmware

What Claude can complete without hardware:
- Add `.github/workflows/esphome-compile.yml` that generates a dummy `secrets.yaml` in CI (no real values) and runs `esphome compile` for both YAMLs. This catches future syntax regressions automatically.
- Update `esphome/secrets.yaml.example` for clarity, e.g. document the door-vs-bedside OTA key situation more explicitly or harmonize on a single key name.
- Source-only comment hardening in YAMLs (no behavior changes) to make compile assumptions explicit.
- Coordinate with **PR #28** for the brightness-arc fix (do not duplicate or conflict).
- Source-verify new YAML changes against ESPHome docs before opening any PR.

What only Hardik can do with hardware:
- Run `esphome compile` and `esphome run` on real boards and capture the logs.
- Flash both devices, observe boot, I²C scan, display, touch, encoder, WS2812, APDS gestures.
- Validate wake-only-first behavior on the physical knob and touchscreen.
- 30-minute stability soaks.
- Tune touch transform / encoder direction if real-hardware behavior diverges from source-confirmed expectations.

### 4.2 Raspberry Pi / Home Assistant / LocalTuya

What Claude can complete without hardware:
- Add a `homeassistant/` directory with safe placeholder config examples:
  - `homeassistant/README.md` — explains the directory, expected install path, redaction rules.
  - `homeassistant/packages/veladial_example.yaml` — example light group helper, optional template helpers (`binary_sensor.bedroom_occupied`, comfort index), all with placeholder entity IDs.
  - `homeassistant/localtuya_mapping_notes.md` — placeholder DP-mapping cheat sheet (no real `local_key`, no real device IDs).
- Cross-link those examples from `docs/setup/raspberry_pi_home_assistant_setup.md` Section 7/8.

What only Hardik can do with hardware:
- Flash HA OS to the RPi, complete onboarding, install the ESPHome add-on, install HACS + LocalTuya.
- Extract Tuya local keys and adopt the five bulbs.
- Create `light.bedroom_group` and confirm on/off/brightness/color-temp.
- Adopt the door + bedside devices in ESPHome.
- Verify local-only operation with internet disconnected.

### 4.3 UI / UX

What Claude can complete without hardware:
- If Hardik provides assets (style guide, fonts, icons, mockups, motion rules): apply allowed v1-scope-preserving polish to `esphome/door_side_rotary.yaml` in a separate PR.
- If Hardik does **not** provide assets: promote the existing intake checklist (currently inside `docs/ui/ux_guide_integration_plan.md` §5) into a standalone `docs/ui/asset_intake_checklist.md` keyed to LVGL widget IDs and the v1 lock.

What only Hardik can do with hardware:
- Visually sign off the rendered UI on the physical 240×240 round display.
- Approve or revise the backlight curve based on real bedroom-light measurements.

### 4.4 Hardware validation

What Claude can complete without hardware:
- Add a tight `docs/validation/HARDWARE_DAY_ONE_RUNBOOK.md` — a single-page sequence keyed to the rows of `hardware/validation_results.md`, listing exactly what to plug in, what to compile, what to flash, what logs to capture, and what photos to take, in execution order.

What only Hardik can do with hardware:
- Photograph the board front/back/silkscreen revision marking.
- Capture serial boot logs (redacted), I²C scan output.
- Fill in `hardware/validation_results.md` rows with PASS/PARTIAL/FAIL.
- Approve the Step 15B gate lift.

---

## 5. Recommended PR sequence

These are sized to stay small and reviewable. Hardik reviews and merges. Claude does **not** merge.

| # | Branch | PR title | Files | Notes |
| :-- | :-- | :-- | :-- | :-- |
| A (this) | `planning/final-completion-roadmap` | `Docs: add final completion roadmap before hardware validation` | `docs/FINAL_COMPLETION_ROADMAP.md` (new), `docs/MASTER_EXECUTION_ROADMAP.md` (one cross-reference line) | No firmware changes. |
| B | `firmware/compile-ci-and-secrets-hardening` | `Firmware: add ESPHome compile CI + secrets template hardening` | `.github/workflows/esphome-compile.yml` (new), `esphome/secrets.yaml.example` (clarification only). Door YAML deliberately untouched while PR #28 is open. | CI uses a generated dummy `secrets.yaml`; never reads real secrets. |
| C | `setup/home-assistant-config-examples` | `Setup: add Home Assistant config examples for VelaDial` | New `homeassistant/README.md`, `homeassistant/packages/veladial_example.yaml`, `homeassistant/localtuya_mapping_notes.md`. Update `docs/setup/raspberry_pi_home_assistant_setup.md` to link them. | All values placeholder. No real Tuya keys, Wi-Fi creds, or HA tokens. |
| D | `docs/ui-ux-asset-intake` *(default)* or `ui/door-side-v1-polish` *(only if Hardik provides assets)* | `Docs: add UI/UX asset intake checklist` *or* `UI: polish door-side v1 LVGL interface` | Standalone intake doc; or asset-driven polish PR if assets arrive. | Preserves 3 pages / 4 presets / no Environment page absolutely. |
| E | `validation/hardware-readiness-package` | `Validation: add hardware readiness package` | `docs/validation/HARDWARE_DAY_ONE_RUNBOOK.md` (new). Light edits to `hardware/validation_results.md` (clearer how-to-fill notes). | Links to existing E2E guide; does not duplicate it. |

**Parallel-agent coordination:** PR #28 from `nidhiuiux` is the only other PR currently in flight and is owned by them. My PR B excludes any edit to `esphome/door_side_rotary.yaml` while #28 is open; if Hardik merges #28 first, my PR B will rebase cleanly.

---

## 6. Definitions

### 6.1 Ready for hardware validation

VelaDial v1 is **ready for hardware validation** when **all** of the following are true:

- Both ESPHome YAMLs compile cleanly in a reproducible CI run from a fresh clone using a dummy secrets file.
- `esphome/secrets.yaml.example` is unambiguous and complete.
- A `homeassistant/` example directory exists with placeholder-only configs.
- The UI/UX asset intake is either filled (assets received) or explicitly opened (standalone intake checklist published).
- A single-page hardware day-one runbook exists.
- `hardware/validation_results.md` clearly tells Hardik how to fill it in.
- No physical PASS claim is made anywhere in the repo.

### 6.2 Production complete

VelaDial v1 is **production complete** when **all** of the following are true:

- Hardware day-one runbook has been executed end-to-end on real hardware.
- `hardware/validation_results.md` rows are PASS (or owner-approved PARTIAL) with evidence captured.
- Door-side compile + flash + boot + I²C scan + display + touch + encoder + WS2812 + adaptive backlight all PASS.
- Bedside compile + flash + GPIO20 + APDS left/right + cooldown + soak all PASS.
- HA OS, ESPHome add-on, LocalTuya, `light.bedroom_group` all PASS.
- All E2E scenarios in `docs/setup/full_e2e_setup_and_validation_guide.md` §10 PASS.
- Both 30-minute soaks PASS.
- Hardik explicitly signs off the Step 15B gate lift and the final acceptance criteria.

Until *every* item under §6.2 is checked, the repo says **"ready for hardware validation"** — not "production complete".

---

## 7. Explicit no-clean-repo-yet decision

Hardik's instruction as of 2026-05-26: **do not create a new clean repo yet.**

Reasons:

- The current repo is internally consistent and merge-history-rich. Migration now would discard valuable PR context.
- Firmware is still DRAFT; nothing has been validated on physical hardware.
- The cleanup that PR #27 performed is the last cleanup the migration would have needed; migrating now adds risk without benefit.

A clean-repo migration may be reconsidered **after** VelaDial reaches *production complete* (§6.2) and Hardik explicitly approves.

---

## 8. What this document does not do

- Does **not** authorize any v1 scope change.
- Does **not** claim any physical PASS result.
- Does **not** authorize a clean-repo migration.
- Does **not** override the Step 15B physical validation gate.
- Does **not** lift any approval gate from `docs/06_Implementation_Plan.md` or `docs/MASTER_EXECUTION_ROADMAP.md`.
