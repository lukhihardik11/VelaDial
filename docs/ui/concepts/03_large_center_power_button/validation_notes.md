# Concept 03: Large Center Power Button — Validation Notes

## Compile Validation

The prototype YAML must compile without errors using ESPHome 2026.5.0 with ESP-IDF framework. The compile validates that all LVGL widget definitions, style properties, lambda expressions, and component configurations are syntactically correct and compatible with the ESP32-S3 target.

## Physical Validation Requirements

All physical validation remains **NOT TESTED** and requires Hardik to perform on the actual ELECROW CrowPanel 1.28" hardware. The following checks are specific to this concept:

### Visual Validation

| Check | Expected Result | Status |
|-------|----------------|--------|
| Power button renders as perfect circle | 140px circle centered on 240x240 display, no clipping | NOT TESTED |
| Amber glow visible on GC9A01A panel | Shadow/glow effect renders visibly on the IPS panel | NOT TESTED |
| Ghost outline visible in darkness | Thin border visible at minimum backlight without being distracting | NOT TESTED |
| Text legible inside button | "ON"/"OFF" text centered and readable at 32px size | NOT TESTED |
| Concentric circle harmony | Button appears harmonious with physical bezel ring | NOT TESTED |
| 3-dot indicator visible at bottom | Page dots visible but not distracting | NOT TESTED |

### Interaction Validation

| Check | Expected Result | Status |
|-------|----------------|--------|
| Button tap registers reliably | Touch on 140px target registers on_click consistently | NOT TESTED |
| Knob press toggles power | Physical encoder press triggers toggle_power script | NOT TESTED |
| Wake-only-first on touch | First touch when asleep only wakes, does not toggle | NOT TESTED |
| Wake-only-first on knob press | First knob press when asleep only wakes, does not toggle | NOT TESTED |
| Wake-only-first on knob rotate | First rotation when asleep only wakes, does not adjust | NOT TESTED |
| LED ring syncs with button state | All 5 LEDs turn amber on toggle-on, off on toggle-off | NOT TESTED |

### Dark-Room Validation

| Check | Expected Result | Status |
|-------|----------------|--------|
| Button visible at 10% backlight | Amber glow distinguishable in complete darkness | NOT TESTED |
| Not too bright for sleeping | Glow does not disturb a sleeping partner | NOT TESTED |
| Touch target hittable in dark | 140px target easy to tap without looking directly | NOT TESTED |

### Performance Validation

| Check | Expected Result | Status |
|-------|----------------|--------|
| Toggle response time | Visual state change within 100ms of input | NOT TESTED |
| Shadow rendering smooth | No visible frame drops during glow appearance | NOT TESTED |
| Memory usage stable | No memory leaks over 30-minute soak test | NOT TESTED |

## Known Limitations

1. The shadow/glow effect rendering quality depends on the GC9A01A panel's color depth and the LVGL color_depth setting (16-bit in this prototype). The glow may appear banded rather than smooth.

2. The 140px button size is optimized for the 240px display but leaves only 50px of margin on each side. Page indicator dots and state labels must fit within this narrow peripheral area.

3. The diamond preset layout (Page 3) is not implemented in the prototype — only the Power page is prototyped. Diamond layout validation requires a future iteration.

## Prototype Scope vs Full Concept

The prototype YAML validates the core Power page concept only. The full 3-page implementation with brightness arc, diamond presets, and page navigation would be implemented in a subsequent iteration if this concept direction is selected for production.
