# Concept 12: Door Switch Replacement — Design Specification

## Overview

Door Switch Replacement treats VelaDial as a direct replacement for a traditional bedroom wall light switch. The UI is optimized for the "walk into room, hit the switch" use case. The Power page dominates with a massive full-screen touch target — the entire 240x240 display is the button. The device's primary job is to be the best light switch in the world.

## Classification

| Field | Value |
|-------|-------|
| Concept ID | 12 |
| Name | Door Switch Replacement |
| Category | Primary (door-side) |
| Direction Matrix Score | Ranked in top 10 |
| Gate Status | PASSES all gates (G1-G8) |
| Version | v1-expanded |

## Visual Identity

### Design Language
- **Metaphor:** Premium wall switch — the device IS the switch
- **Hero element:** Full-screen amber fill when ON (the display itself glows)
- **OFF state:** Dark display with thin white border circle (touch target indicator)
- **Philosophy:** The switch itself becomes part of the room's lighting

### Color Palette

| Element | Color | Hex | Usage |
|---------|-------|-----|-------|
| ON fill | Warm Amber | `0xFFA500` | Full-screen background when lights on |
| ON text | White | `0xFFFFFF` | "ON" label, 64pt |
| OFF border | White | `0xFFFFFF` | Thin circle border indicating touch area |
| OFF text | Dim Gray | `0x666666` | "OFF" label, 48pt |
| OFF background | Near Black | `0x0A0A0A` | Full-screen background when lights off |
| Page indicator active | White | `0xFFFFFF` | Active dot |
| Page indicator inactive | Dark Gray | `0x444444` | Inactive dots |

## Screen Architecture

### 3-Page Layout (Locked v1 Spec)

| Page | Role | Hero Element | Interaction |
|------|------|-------------|-------------|
| 0 — Power | PRIMARY | Full-screen touch target | Any touch = toggle, knob press = toggle |
| 1 — Brightness | Secondary | 200px arc + 48pt percentage | Knob CW/CCW ±5% |
| 2 — Presets | Secondary | 2x2 grid (4 presets) | Tap to apply |

### Page 0: Power (Hero Page)

The entire display is a single touch target. No button boundaries, no icons — just the state:

**ON State:**
- Background: Full amber fill (`0xFFA500`) covering entire 240x240
- Text: "ON" in 64pt white, centered
- Effect: The device GLOWS — visible from the doorway

**OFF State:**
- Background: Near-black (`0x0A0A0A`)
- Border: 2px white circle at r=110 (indicating "touch here")
- Text: "OFF" in 48pt dim gray, centered
- Effect: Subtle presence, not distracting to sleepers

### Page 1: Brightness (Standard)
- 200px display-only arc (270° sweep, amber)
- 48pt center percentage label
- Knob CW/CCW adjusts ±5%
- Knob press = power toggle shortcut

### Page 2: Presets (Standard)
- 2x2 FLEX ROW_WRAP grid
- 4 preset tiles (80x65px each)
- Active preset highlighted in amber
- Tap to apply

### 4 Locked Presets

| Preset | Brightness | Color Temp |
|--------|-----------|------------|
| Warm White | 80% | 3000K |
| Soft Amber | 60% | 2700K |
| Neutral White | 90% | 4000K |
| Low Nightlight | 15% | 2200K |

## Interaction Model

### Power Page Touch Behavior
- **Tap anywhere** on the 240x240 area = power toggle
- **Swipe left/right** = page navigation (distinguished from tap by movement threshold > 30px)
- **Knob press** = power toggle (redundant with touch)
- **Knob rotate** = no action on Power page (prevents accidental brightness changes)

### Tap vs Swipe Discrimination
- Touch down → track movement
- If total movement < 30px AND duration < 500ms → TAP (toggle power)
- If horizontal movement > 30px → SWIPE (navigate pages)
- This prevents accidental toggles during swipe navigation

### Sleep/Wake Behavior
- Wake-only-first enforced on ALL inputs
- Sleep timeout: 30 seconds on Power page (shorter — transient door-side use)
- Sleep timeout: 60 seconds on Brightness/Presets pages (standard)
- Always wakes to Power page (the primary use case)

## LED Ring Behavior

| State | LED Pattern | Brightness |
|-------|-------------|-----------|
| ON | All 5 amber | 50% (aggressive — door-side, not bedside) |
| OFF | All 5 off | 0% |
| Transitioning | Quick amber pulse | 30% |

Higher LED brightness than other concepts because the door-side location is not next to a sleeping person.

## Animation

### Power Toggle Animation (v1-expanded)
- **ON → OFF:** Amber background fades to black over 300ms (simple opacity transition)
- **OFF → ON:** Black background fades to amber over 300ms
- Note: Full radial wipe is v2 scope. v1-expanded uses a clean fade.

### LVGL Style Transition
```
transition_time: 300ms
transition_path: ease_in_out
properties: bg_color, bg_opa
```

## Typography

| Element | Font | Size | Weight |
|---------|------|------|--------|
| Power ON label | Roboto | 64pt | Bold |
| Power OFF label | Roboto | 48pt | Regular |
| Brightness percentage | Roboto | 48pt | Bold |
| Preset names | Roboto | 16pt | Regular |
| Page indicator dots | N/A | 8-10px | N/A |

## Guest-Friendly Design

This concept is the most guest-friendly in the entire matrix:
1. **Zero learning curve** — it looks like a switch, it acts like a switch
2. **No instructions needed** — touch the glowing thing to turn it off
3. **Visible state from doorway** — amber glow = lights on
4. **Forgiving touch target** — entire screen, impossible to miss
5. **No hidden gestures** — the primary action is the only obvious action

## Dark-Room Usability

- Power page: EXCELLENT — large touch target, visible amber state
- The thin white border circle in OFF state provides just enough guidance
- Backlight at 15% in dark room (enough to see the border, not enough to disturb)

## Daylight Readability

- EXCELLENT — full-screen amber fill is impossible to miss in any lighting
- High contrast: amber vs black, white text vs amber/black backgrounds

## Performance Expectations

- **Render cost:** Minimal — single rectangle fill + one text label on Power page
- **Animation cost:** Low — bg_color transition fires only on toggle
- **Memory:** Light — fewer widgets than any multi-element concept
- **Touch latency:** Must be < 100ms for switch-like responsiveness
