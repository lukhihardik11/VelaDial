# Concept 13: Lunar Phase Visualization — Design Specification

## Classification

| Attribute | Value |
|-----------|-------|
| Concept ID | 13 |
| Name | Lunar Phase Visualization |
| Category | High-Novelty Visual Metaphor |
| Direction Matrix Score | Novelty 9/10, Feasibility 6/10 |
| Gate G1 (Three-Page Lock) | PASS |
| Gate G2 (Knob-First) | PASS |
| Gate G3 (Wake-Only-First) | PASS |
| Gate G4 (HA Integration) | PASS |
| Version | v1 locked (LVGL arc-based moon approximation) |
| Target | `light.bedroom_group` |

## Design Philosophy

Lunar Phase Visualization maps bedroom light brightness to the phase of the moon. The round display IS a moon — the physical form factor and the visual metaphor are one and the same. At 0% brightness (lights off), the display shows a new moon (dark circle with faint limb). At 100% brightness, the display shows a full moon (luminous amber disc). Intermediate levels show corresponding lunar phases: crescent at 25%, half moon at 50%, gibbous at 75%.

The metaphor is poetic: controlling bedroom light is like controlling the moon. This is the most bedroom-appropriate high-novelty concept — a dim crescent moon on a dark display is beautiful, calming, and provides just enough visual information.

## Visual Language

### Color Palette

| Element | Color | Hex | Usage |
|---------|-------|-----|-------|
| Lit lunar surface | Warm Amber | `0xFFA500` | Moon's illuminated face |
| Dim lunar surface | Deep Charcoal | `0x1A1A1A` | Moon's shadowed face |
| Lunar limb | Soft Gray | `0x333333` | Faint edge of new moon |
| Background | Pure Black | `0x000000` | Space / void |
| Percentage overlay | White 40% opacity | `0xFFFFFF` | Semi-transparent readability |
| Active preset | Bright Amber | `0xFFCC00` | Selected preset highlight |
| Inactive preset | Dim Gray | `0x555555` | Unselected preset |

### Moon Phase Approximation (LVGL Arc-Based)

Since pre-rendered PNG images would consume significant flash storage (~100KB each × 8-12 phases), the v1 locked implementation uses an **LVGL arc-based approximation**:

- A full 240px amber circle represents the full moon (lit surface)
- A 240px black overlay circle (offset horizontally) creates the shadow/terminator
- The offset position maps to brightness: offset = 0 (full moon/100%), offset = 120 (half moon/50%), offset = 240 (new moon/0%)
- The terminator boundary is approximated by overlapping two circular objects

This approach uses zero image assets and renders in real-time.

### Typography

| Element | Font | Size | Weight | Color |
|---------|------|------|--------|-------|
| Percentage value | Roboto | 32pt | Regular | White @ 40% opacity |
| Power state label | Roboto | 20pt | Regular | White @ 60% opacity |
| Preset names | Roboto | 16pt | Regular | Amber or Gray |
| Page indicator dots | N/A | 6px circles | N/A | White/Gray |

## Screen Architecture

### Page 1: Power (Moon State)

The entire display shows the moon in its current state:
- **ON:** Full moon (amber disc, 200px diameter, centered)
- **OFF:** New moon (dark circle with 2px gray limb border)
- Small semi-transparent power icon (20pt "ON"/"OFF") overlaid at bottom
- Tap anywhere on the moon = toggle power

### Page 2: Brightness (Moon Phase — Hero Page)

The hero interaction page. The moon phase changes in real-time as the user rotates the knob:
- 200px amber circle = lit surface
- 200px black overlay circle = shadow, offset based on brightness
- Offset formula: `shadow_x = 200 - (brightness_pct * 200 / 100)`
- At 100%: shadow fully off-screen (full moon)
- At 50%: shadow centered (half moon)
- At 0%: shadow fully covers (new moon)
- Semi-transparent 32pt percentage value at center
- Knob CW/CCW = brightness ±5% = moon waxes/wanes

### Page 3: Presets (Moon Grid)

Four preset labels arranged in a 2×2 grid, each showing the preset name and its default brightness as a text percentage. Active preset highlighted in amber.

| Position | Preset | Default Brightness |
|----------|--------|-------------------|
| Top-left | Warm White | 80% |
| Top-right | Soft Amber | 60% |
| Bottom-left | Neutral White | 90% |
| Bottom-right | Low Nightlight | 15% |

## Interaction Model

| Input | Page 1 (Power) | Page 2 (Brightness) | Page 3 (Presets) |
|-------|----------------|--------------------|--------------------|
| Knob CW | Navigate right | Brightness +5% | Next preset |
| Knob CCW | Navigate left | Brightness -5% | Previous preset |
| Knob Press | Power toggle | Power toggle | Apply preset |
| Touch tap | Power toggle | — | Apply tapped preset |
| Swipe left | → Page 2 | → Page 3 | — (blocked) |
| Swipe right | — (blocked) | → Page 1 | → Page 2 |

## Wake/Sleep Behavior

- **Wake-only-first:** ALL inputs on first interaction after sleep = wake display only
- **Wake animation:** Display fades from black → current moon phase over 300ms (moonrise)
- **Sleep animation:** Moon fades to black over 300ms (moonset)
- **Sleep timeout:** 45 seconds of inactivity
- **Backlight:** 80% during active use, 0% during sleep

## LED Ring Behavior

| State | LED Ring |
|-------|----------|
| Full moon (100%) | All 5 LEDs warm amber, 40% brightness |
| Half moon (50%) | 3 LEDs warm amber, 20% brightness |
| Crescent (25%) | 1 LED warm amber, 10% brightness |
| New moon (OFF) | All LEDs off |
| HA unavailable | 1 LED red, 15% |

## Page Indicators

Three 6px dots at bottom (y: 225):
- Active page: white filled
- Inactive pages: gray outline
- Positioned at x: 108, 120, 132

## Hardware Test Pending

- Moon phase visual quality on actual GC9A01A display
- Arc-based terminator smoothness
- Backlight at 80% sufficient for amber visibility
- LED ring warm amber color accuracy
- Touch responsiveness on full-screen moon target
