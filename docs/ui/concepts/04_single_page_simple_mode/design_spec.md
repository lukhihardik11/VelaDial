# Concept 04: Single-Page Simple Mode — Design Specification

## Gate Status

> **FAILS Gate G1 (Three-Page Lock).** This concept uses exactly ONE page instead of the mandated three. It is prototyped for exploration purposes only and cannot be selected for production without explicit owner approval to waive the 3-page requirement.

---

## 1. Concept Overview

Single-Page Simple Mode is the most radical simplification in the 20-concept matrix. It places ALL controls — power state, brightness, and active preset — on a single screen with zero page navigation. The design philosophy is "anti-dashboard": no tabs, no swipes, no page indicators, no hidden state. Everything the user needs is visible at all times.

The concept treats the 240x240 round display as three horizontal bands stacked vertically within the circular safe area. This is the simplest possible UI for a smart light controller — it does one thing (control bedroom lights) and does it with absolute clarity.

---

## 2. Visual Identity

| Property | Value |
|----------|-------|
| Background | Pure black (`0x000000`) |
| Primary text | White (`0xFFFFFF`) |
| Accent color | Amber (`0xFFA500`) |
| Secondary text | Dim gray (`0x888888`) |
| Divider lines | Dark gray (`0x333333`) |
| Font family | Roboto (3 sizes) |
| Design language | Sparse, typographic, information-dense |

### Color Palette

| Role | Hex | Usage |
|------|-----|-------|
| Background | `0x000000` | Full-screen fill |
| Hero value | `0xFFFFFF` | Center brightness percentage (48pt) |
| Power ON text | `0xFFA500` | "ON" label in top zone |
| Power OFF text | `0x888888` | "OFF" label in top zone |
| Preset name | `0xFFA500` | Active preset name in bottom zone |
| Divider | `0x333333` | Subtle zone separators |
| Arc indicator | `0xFFA500` | Thin brightness arc around center value |
| Arc background | `0x222222` | Unfilled arc portion |

---

## 3. Layout Architecture

### Circular Safe Area

The 240px diameter circle has a usable content area of approximately 200px width at the center. The safe area (avoiding clipped corners) is roughly:
- Full width (240px) available only at vertical center (y=120)
- At y=40 (top zone center): ~196px available width
- At y=200 (bottom zone center): ~196px available width

### Three-Band Layout

```
┌─────────────────────────┐
│     ╭─────────────╮     │  ← Round bezel
│   ╭─┤  TOP ZONE   ├─╮   │  y: 30-80
│  │  │ Power ● ON  │  │  │  24pt, tap to toggle
│  │  ├─────────────┤  │  │  ← Curved divider (y≈85)
│  │  │             │  │  │
│  │  │ CENTER ZONE │  │  │  y: 85-155
│  │  │   72%       │  │  │  48pt hero value
│  │  │  ╭───arc───╮│  │  │  Thin arc around value
│  │  ├─────────────┤  │  │  ← Curved divider (y≈160)
│  │  │ BOTTOM ZONE │  │  │  y: 160-210
│  │  │ Warm White  │  │  │  20pt, tap to cycle preset
│   ╰─┤             ├─╯   │
│     ╰─────────────╯     │  ← Round bezel
└─────────────────────────┘
```

---

## 4. Widget Inventory

| Widget | Type | ID | Position | Size |
|--------|------|----|----------|------|
| Power icon/label | `label` | `power_label` | align: TOP_MID, y: 45 | auto |
| Brightness value | `label` | `brightness_label` | align: CENTER, y: -5 | auto |
| Brightness arc | `arc` | `brightness_arc` | align: CENTER | 160x160 |
| Preset name | `label` | `preset_label` | align: BOTTOM_MID, y: -40 | auto |
| Top touch zone | `obj` | `top_touch` | x: 20, y: 10, w: 200, h: 75 | invisible |
| Bottom touch zone | `obj` | `bottom_touch` | x: 20, y: 155, w: 200, h: 75 | invisible |

---

## 5. Interaction Model

### Rotary Encoder

| Action | Behavior |
|--------|----------|
| CW rotation | Increase brightness by 5% (clamped 1-100%) |
| CCW rotation | Decrease brightness by 5% (clamped 1-100%) |
| Knob press | Toggle power ON/OFF |

### Touch Zones

| Zone | Tap Action | Long-Press |
|------|-----------|------------|
| Top third (y: 10-85) | Toggle power | None |
| Center (y: 85-155) | No action (knob only) | None |
| Bottom third (y: 155-230) | Cycle to next preset | None (v1-expanded: preset picker overlay) |

### Preset Cycling

Tap on the bottom zone advances through presets sequentially:
1. Warm White → 2. Soft Amber → 3. Neutral White → 4. Low Nightlight → (wrap to 1)

Each tap immediately applies the next preset via `homeassistant.action`.

---

## 6. Wake/Sleep Behavior

| Trigger | Action |
|---------|--------|
| Touch (any zone) while asleep | Wake display only; do NOT toggle/cycle |
| Knob rotate while asleep | Wake display only; do NOT adjust brightness |
| Knob press while asleep | Wake display only; do NOT toggle power |
| Second touch/knob while awake | Execute the mapped action |
| Inactivity timeout (30s) | Fade to sleep (300ms) |

All three zones fade in simultaneously on wake (300ms opacity transition). No staged reveal — the simplicity extends to transitions.

---

## 7. LED Ring Behavior

| State | LED Ring |
|-------|----------|
| Power ON | Amber glow (neopixelbus, RGB 255/165/0) |
| Power OFF | LEDs off |
| Wake transition | LEDs fade in with display |
| Sleep transition | LEDs fade out with display |

No per-zone LED mapping. The single-page concept is too simple to warrant LED complexity.

---

## 8. Animation Specification

| Animation | Duration | Easing | Trigger |
|-----------|----------|--------|---------|
| Wake fade-in | 300ms | ease-out | Any input while asleep |
| Sleep fade-out | 300ms | ease-in | Inactivity timeout |
| Brightness number change | Instant | N/A | Knob rotation |
| Power state change | Instant | N/A | Toggle action |
| Preset name change | Instant | N/A | Cycle action |

Almost zero motion. The only animations are wake/sleep fades. The concept's radical simplicity extends to its transitions.

---

## 9. Home Assistant Integration

| Entity | Purpose |
|--------|---------|
| `light.bedroom_group` | Primary control target |
| Attribute: `brightness` | Maps to 0-100% display value |
| Attribute: `color_temp` | Determines active preset name |
| Service: `light.toggle` | Power toggle |
| Service: `light.turn_on` | Apply preset (with brightness/color_temp data) |

### State Import

On HA state change, the display updates all three zones simultaneously:
- Power label: "ON" (amber) or "OFF" (gray)
- Brightness value: percentage from brightness attribute
- Preset name: derived from color_temp value matching

---

## 10. Differentiation from Other Concepts

| Aspect | Concept 04 (This) | Concept 01 (Minimal Thermostat) | Concept 03 (Large Power Button) |
|--------|-------------------|--------------------------------|-------------------------------|
| Pages | 1 (single) | 1 (single) | 3 |
| Hero element | Brightness % value | Temperature-style display | 140px power button |
| Navigation | None | None | Swipe |
| Preset access | Tap bottom zone | Not on main page | Dedicated page |
| Gate G1 | **FAILS** | FAILS | PASSES |
| Complexity | Extremely low | Low | Medium |

---

## 11. Gate G1 Failure Analysis

This concept explicitly violates Gate G1 (Three-Page Lock) which mandates exactly three pages: Power, Brightness, and Presets. The violation is intentional — the concept explores whether the three-page requirement is necessary or whether a single-page approach could be superior for the use case.

**Arguments for waiving G1:**
- Eliminates "which page am I on?" confusion
- Faster access to all functions (zero navigation)
- Simpler implementation (fewer bugs)
- Better dark-room usability (everything visible at glance)

**Arguments against waiving G1:**
- Cramped layout on 240x240
- Preset cycling is less discoverable than dedicated page
- May feel "too simple" for premium device
- Inconsistent with the locked v1 scope

**Recommendation:** Prototype for exploration; do not select for production unless G1 is explicitly waived by owner.
