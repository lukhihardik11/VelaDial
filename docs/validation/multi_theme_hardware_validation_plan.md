# Multi-Theme Hardware Validation Plan

> **Status:** Validation package created. Hardware validation **NOT TESTED**.
> **Target:** `esphome/door_side_rotary.yaml` (Multi-Theme Firmware Engine)
> **Hardware:** ELECROW CrowPanel 1.28" ESP32-S3 Rotary Touch Display

This document provides the step-by-step plan for Hardik to physically validate the multi-theme firmware engine on the actual hardware.

> **See also:** For the complete end-to-end project validation (including Raspberry Pi setup, Home Assistant, LocalTuya, bedside controller, and full system tests), refer to [`end_to_end_project_validation_checklist.md`](end_to_end_project_validation_checklist.md).

## 1. Flashing Guide

Before testing, the firmware must be flashed to the physical board.

1. Connect the ELECROW board to your computer via USB-C.
2. Ensure you have ESPHome installed locally or via Home Assistant.
3. Create or verify `esphome/secrets.yaml` exists with valid credentials.
4. Run the compile and upload command:
   ```bash
   esphome run esphome/door_side_rotary.yaml
   ```
5. Select the appropriate serial port when prompted.
6. Wait for the flash to complete and the device to reboot.
7. Keep the serial monitor open to observe boot logs and any immediate crashes.

## 2. Core Hardware Validation

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **Flash succeeds** | ESPHome reports successful upload and device reboots. | PENDING |
| **Device boots** | Serial log shows successful boot without bootloops. | PENDING |
| **Display renders** | Screen turns on and displays the Power page. | PENDING |
| **Touch works** | Swiping left/right changes pages smoothly. | PENDING |
| **Knob rotation works** | Rotating the knob changes brightness (Page 2) or preset selection (Page 3). | PENDING |

## 3. Interaction & Wake Behavior Validation

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **Screen sleep/idle** | Screen turns off after 45 seconds of inactivity. | PENDING |
| **Wake-only-first (Touch)** | Touching a sleeping screen wakes it but does NOT trigger an action. | PENDING |
| **Wake-only-first (Rotate)** | Rotating knob while asleep wakes screen but does NOT change values. | PENDING |
| **Wake-only-first (Press)** | Pressing knob while asleep wakes screen but does NOT trigger action. | PENDING |
| **Short press behavior** | Pressing knob (<600ms) triggers page-specific action. | PENDING |
| **Long press behavior** | Holding knob (>1.5s) opens Theme Selector. | PENDING |
| **Long press isolation** | Long press does NOT also trigger the short-press action. | PENDING |

## 4. Theme Selector Validation

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **Theme Selector opens** | Long press transitions to Theme Selector page. | PENDING |
| **Theme Selector browse** | Rotating knob cycles through all 20 themes. | PENDING |
| **All 20 themes selectable** | Can reach Theme 01 through Theme 20. | PENDING |
| **Theme Selector apply** | Short press applies the selected theme and returns to Power page. | PENDING |
| **Theme persistence** | Active theme remains the same after a hard reboot. | PENDING |

## 5. Page Functionality Validation

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **Power page works** | Short press toggles `light.bedroom_group` power. | PENDING |
| **Brightness page works** | Rotation changes brightness; short press returns to Power page. | PENDING |
| **Presets page works** | Rotation highlights preset; short press applies it. | PENDING |
| **4 locked presets work** | Warm White, Soft Amber, Neutral White, Low Nightlight apply correctly. | PENDING |

## 6. Home Assistant & Network Validation

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **HA target correct** | All commands affect `light.bedroom_group`. | PENDING |
| **HA diagnostic sensor** | `text_sensor.veladial_doorside_active_theme` updates in HA. | PENDING |
| **API unavailable state** | Disconnecting WiFi/HA shows "Unavailable" badge on screen. | PENDING |

## 7. Sensor & LED Validation

| Test Item | Expected Behavior | Status |
| :--- | :--- | :--- |
| **LED ring color** | LED ring color matches the active theme's palette. | PENDING |
| **TSL2591 adaptive backlight** | (If connected) Display brightness adjusts to room ambient light. | PENDING |
| **SHT45 diagnostic** | (If connected) Temp/humidity report to HA but do not show on UI. | PENDING |

## 8. Theme-by-Theme Visual Validation

*Note: This checks if the theme engine successfully applies the distinct visual parameters for each theme. It does not demand visual perfection, only that the theme engine works.*

| Theme | Expected Visual Change | Status |
| :--- | :--- | :--- |
| 01: Minimal Thermostat | Blue palette, minimal arcs | PENDING |
| 02: SmartKnob-Inspired Arc | Purple palette, arc gauge layout | PENDING |
| 03: Large Center Power Button | Green palette, large center button | PENDING |
| 04: Single-Page Simple Mode | Warm amber palette, simplified layout | PENDING |
| 05: Preset Ring UI | Classic amber palette, ring layout | PENDING |
| 06: Night Mode Ultra-Minimal | Deep red palette, ultra-dim | PENDING |
| 07: Text-First Utility | Amber palette, text-heavy | PENDING |
| 08: Apple Watch Complications | Blue palette, complication layout | PENDING |
| 09: LED-Ring Status-First | Amber palette, relies on physical LEDs | PENDING |
| 10: Three-Screen Tab Carousel | Amber palette, tab carousel layout | PENDING |
| 11: Brightness-First UI | Amber palette, brightness dominates | PENDING |
| 12: Door Switch Replacement | Green palette, switch metaphor | PENDING |
| 13: Lunar Phase Visualization | Moon white palette, lunar metaphor | PENDING |
| 14: Sundial Shadow UI | Warm gold palette, shadow metaphor | PENDING |
| 15: Tree Ring Growth Pattern | Wood tone palette, concentric rings | PENDING |
| 16: Topographic Contour Map | Amber palette, contour lines | PENDING |
| 17: Iris Aperture Mechanism | Amber palette, aperture mechanism | PENDING |
| 18: Radar Sweep Animation | Amber palette, sweep animation | PENDING |
| 19: Vinyl DJ Crossfader | Amber palette, turntable metaphor | PENDING |
| 20: Eclipse Corona | Deep gold palette, corona glow | PENDING |
