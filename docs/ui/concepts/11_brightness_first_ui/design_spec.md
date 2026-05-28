# UI Concept 11: Brightness-First UI — Design Specification

## Concept Identity

| Field | Value |
| --- | --- |
| Concept Number | 11 |
| Name | Brightness-First UI |
| Category | Primary (v1 candidate) |
| Direction Matrix Score | Ranked in top tier |
| Gate Status | PASSES all gates (G1-G8) |
| Key Innovation | Information architecture — brightness as default landing page |

## Design Philosophy

Brightness-First UI is architecturally identical to the Three-Screen Tab Carousel (Concept 10) with one critical difference: **the default landing page is Brightness, not Power**. The hypothesis is that "adjust brightness" is a more common bedroom action than "toggle power" — especially at night, when the user wants to dim the lights rather than turn them off entirely.

This concept optimizes for the user's most common need rather than the designer's conceptual hierarchy. Premium products anticipate behavior; this concept anticipates that bedroom users adjust brightness more often than they toggle power.

## Screen Architecture

| Page Order | Page | Content | Rationale |
|:----------:|------|---------|-----------|
| 0 (default) | Brightness | 200px arc + 48pt `%` value | Most frequent bedroom action |
| 1 | Power | 120px toggle button + state label | Second most frequent |
| 2 | Presets | 2x2 FLEX grid (4 presets) | Least frequent (set once, rarely changed) |

### Page Indicator Dots

3-dot indicator in LVGL `top_layer` (always visible):
- Dot 1 (leftmost) = Brightness (default active)
- Dot 2 (center) = Power
- Dot 3 (rightmost) = Presets

Active dot: 10px, amber (`0xFFA500`)
Inactive dot: 8px, gray (`0x444444`)

## Page Layouts

### Page 0 — Brightness (Default Landing)

The hero page. Users wake the device and immediately see the current brightness level.

| Element | Size | Position | Color |
|---------|------|----------|-------|
| Background arc | 200px diameter, 8px width | Centered | Dark gray `0x333333` |
| Indicator arc | 200px diameter, 8px width | 270° sweep (135° to 45°) | Amber `0xFFA500` |
| Percentage label | 48pt Roboto Bold | Center of arc | White `0xFFFFFF` |
| "BRIGHTNESS" header | 14pt | Top center (y: 20) | Gray `0x888888` |

The arc is **display-only** — not touch-draggable. All brightness adjustment flows through the physical rotary encoder (CW/CCW ±5% steps). The arc value is proportional to `brightness_pct` (0-100%).

### Page 1 — Power

| Element | Size | Position | Color |
|---------|------|----------|-------|
| Power button | 120px circle | Centered | Amber fill (ON) / dark gray border (OFF) |
| State label | 48pt | Center of button | White "ON" / dim gray "OFF" |
| "POWER" header | 14pt | Top center (y: 20) | Gray `0x888888` |

Tap the button to toggle power. Knob press also toggles power from this page.

### Page 2 — Presets

| Element | Size | Position | Color |
|---------|------|----------|-------|
| 2x2 grid container | 200x180px | Centered | Transparent |
| Preset tiles (4x) | 90x75px each | FLEX ROW_WRAP | Dark gray bg, rounded 8px |
| Active preset text | 16pt | Center of tile | Amber `0xFFA500` |
| Inactive preset text | 16pt | Center of tile | Gray `0x888888` |

### 4 Locked Presets

| Index | Name | Brightness | Color Temp |
|-------|------|-----------|------------|
| 0 | Warm White | 80% | 2700K |
| 1 | Soft Amber | 60% | 2200K |
| 2 | Neutral White | 90% | 4000K |
| 3 | Low Nightlight | 15% | 2200K |

## Interaction Model

| Input | Brightness Page (default) | Power Page | Presets Page |
|-------|--------------------------|-----------|-------------|
| Knob rotate CW | Brightness +5% | No action | No action |
| Knob rotate CCW | Brightness -5% | No action | No action |
| Knob press | Toggle power (shortcut) | Toggle power | Apply selected preset |
| Swipe left | Go to Power | Go to Presets | No action (last page) |
| Swipe right | No action (first page) | Go to Brightness | Go to Power |
| Touch center | No action | Toggle power | Apply preset |

**Critical UX decision:** Knob press on the Brightness page toggles power. This makes the two most common actions (adjust brightness, toggle power) accessible without any page navigation.

## Wake/Sleep Behavior

- **Wake target:** Always Brightness page (page 0)
- **Wake-only-first:** Enforced on all input paths
- **Sleep timeout:** 30 seconds of inactivity
- **3AM use case:** User wakes device → immediately sees current brightness → rotates knob to adjust → presses knob to turn off → done (zero swipes required)

## LED Ring Behavior

Same as Concept 10:
- ON: All 5 LEDs amber, brightness proportional (capped 30%)
- OFF: All LEDs off
- No page-specific LED behavior

## Color Palette

| Token | Hex | Usage |
|-------|-----|-------|
| Primary | `0xFFA500` | Active arc, active dot, active preset, ON state |
| Background | `0x000000` | All page backgrounds |
| Surface | `0x1A1A1A` | Button/tile backgrounds |
| Text Primary | `0xFFFFFF` | Percentage, ON label |
| Text Secondary | `0x888888` | Headers, inactive presets, OFF label |
| Text Dim | `0x444444` | Inactive dots |
| Arc Background | `0x333333` | Unfilled arc portion |

## Typography

| Role | Font | Size | Weight |
|------|------|------|--------|
| Hero value | Roboto | 48pt | Bold |
| Section header | Roboto | 14pt | Regular |
| Preset name | Roboto | 16pt | Medium |
| Page dots | n/a | 8-10px | n/a (circles) |

## Animation

| Trigger | Animation | Duration |
|---------|-----------|----------|
| Swipe left | `MOVE_LEFT` | 200ms |
| Swipe right | `MOVE_RIGHT` | 200ms |
| Wake | Fade in from black | 300ms (backlight ramp) |
| Sleep | Fade to black | 500ms (backlight ramp) |
| Arc value change | Smooth LVGL animation | 150ms |

## What Makes It Premium

The premium quality comes from **anticipating the user's intent**. Instead of forcing the user to navigate to brightness (the most common action), the device presents it immediately. This is the same philosophy behind:
- Nest thermostat showing current temperature on wake (not settings)
- Apple Watch showing the time face (not app grid)
- Tesla showing drive controls on entry (not media)

## What Could Go Wrong

1. Users who expect "page 1 = power" may be confused initially
2. Knob press = power toggle on Brightness page is non-obvious (needs onboarding)
3. If Hardik prefers Power-first hierarchy, this concept is simply rejected
4. The page reorder requires Hardik approval to change the default page

## Hardware Dependencies

- ELECROW CrowPanel 1.28" ESP32-S3 (240x240 round GC9A01A)
- Rotary encoder on GPIO45/GPIO42 (A/B), GPIO41 (switch)
- CST816S touch on I2C (SDA=6, SCL=7, INT=5, RST=13)
- WS2812 5-LED ring on GPIO48
- Backlight on GPIO46

All hardware validation: **NOT TESTED**
