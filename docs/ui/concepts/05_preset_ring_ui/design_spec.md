# Concept 05: Preset Ring UI — Design Specification

## Gate Status

> **PASSES all gates (G1–G8).** This concept uses exactly 3 pages and meets all selection criteria.

---

## 1. Concept Overview

Preset Ring UI places the four light presets around the circumference of the round display as arc segments, creating a ring of options navigated by rotating the knob. The physical metaphor is a vintage radio tuner or washing machine program selector: rotate to the setting you want, press to confirm. The round form factor is celebrated — this ring layout could not exist on a rectangular display.

The hero page is the Presets page (Page 3), where four colored arc segments divide the ring into quadrants. The active preset's segment is filled with its characteristic color while inactive segments show dim outlines. The center shows the preset name and a brightness value.

---

## 2. Visual Identity

| Property | Value |
|----------|-------|
| Background | Pure black (`0x000000`) |
| Primary text | White (`0xFFFFFF`) |
| Accent color | Amber (`0xFFA500`) — used for Warm White preset |
| Secondary text | Dim gray (`0x888888`) |
| Font family | Roboto (3 sizes: 48pt, 24pt, 16pt) |
| Design language | Radial, ring-centric, color-coded segments |

### Preset Color Palette

| Preset | Active Color | RGB Values | Arc Position |
|--------|-------------|------------|--------------|
| Warm White | Amber | `0xFFA500` (255, 165, 0) | Top-right quadrant (350°–70°) |
| Soft Amber | Deep Gold | `0xCC8800` (204, 136, 0) | Bottom-right quadrant (80°–160°) |
| Neutral White | Cool White | `0xCCDDFF` (204, 221, 255) | Bottom-left quadrant (170°–250°) |
| Low Nightlight | Dim Amber | `0x664400` (102, 68, 0) | Top-left quadrant (260°–340°) |

### Inactive Segment Styling

| Property | Value |
|----------|-------|
| Arc color (inactive) | `0x333333` |
| Arc width (inactive) | 12px |
| Arc width (active) | 14px |
| Gap between segments | 10° (no arc drawn in gap) |

---

## 3. Screen Architecture (3 Pages)

### Page 1: Power

| Element | Description |
|---------|-------------|
| Ring | Full amber ring when ON; dim gray ring when OFF |
| Center | Power icon label ("ON"/"OFF") + state text |
| Ring width | 12px |
| Ring color ON | `0xFFA500` (full 360° amber) |
| Ring color OFF | `0x333333` (full 360° dim) |

### Page 2: Brightness

| Element | Description |
|---------|-------------|
| Ring | Amber arc proportional to brightness (0-100%) |
| Center | Large percentage value (48pt) |
| Arc start | 135° |
| Arc end | 45° (270° sweep, bottom-open) |
| Ring width | 12px |

### Page 3: Presets (Hero Page)

| Element | Description |
|---------|-------------|
| Ring | 4 arc segments at quadrants, active filled with preset color |
| Center | Preset name (24pt) + "Selected" label (16pt) |
| Segment width | 12px (inactive), 14px (active) |
| Gap | 10° between each segment |
| Selection indicator | Active segment filled with preset color; others dim gray |

---

## 4. Layout Architecture

```
┌─────────────────────────┐
│     ╭─────────────╮     │  ← Round bezel
│   ╭─┤             ├─╮   │
│  │  │  ╭───────╮  │  │  │  ← Outer ring (4 arc segments)
│  │  │  │       │  │  │  │
│  │  │  │ CENTER│  │  │  │  ← Preset name + label
│  │  │  │       │  │  │  │
│  │  │  ╰───────╯  │  │  │
│   ╰─┤             ├─╯   │
│     ╰─────────────╯     │  ← Round bezel
│       ● ○ ○             │  ← Page indicator dots
└─────────────────────────┘
```

### Arc Segment Angles (Presets Page)

| Segment | Start Angle | End Angle | Span | Gap After |
|---------|-------------|-----------|------|-----------|
| Warm White (TR) | 350° | 70° | 80° | 10° |
| Soft Amber (BR) | 80° | 160° | 80° | 10° |
| Neutral White (BL) | 170° | 250° | 80° | 10° |
| Low Nightlight (TL) | 260° | 340° | 80° | 10° |

---

## 5. Widget Inventory

### Page 1: Power

| Widget | Type | ID | Position |
|--------|------|----|----------|
| Power ring | `arc` | `power_ring` | align: CENTER, 220x220 |
| Power label | `label` | `power_label` | align: CENTER, y: -10 |
| State text | `label` | `power_state_text` | align: CENTER, y: 25 |
| Page dots container | `obj` | — | align: BOTTOM_MID |

### Page 2: Brightness

| Widget | Type | ID | Position |
|--------|------|----|----------|
| Brightness arc | `arc` | `brightness_arc` | align: CENTER, 220x220 |
| Brightness label | `label` | `brightness_label` | align: CENTER |
| Page dots container | `obj` | — | align: BOTTOM_MID |

### Page 3: Presets

| Widget | Type | ID | Position |
|--------|------|----|----------|
| Preset arc 1 (Warm White) | `arc` | `preset_arc_1` | align: CENTER, 220x220 |
| Preset arc 2 (Soft Amber) | `arc` | `preset_arc_2` | align: CENTER, 220x220 |
| Preset arc 3 (Neutral White) | `arc` | `preset_arc_3` | align: CENTER, 220x220 |
| Preset arc 4 (Low Nightlight) | `arc` | `preset_arc_4` | align: CENTER, 220x220 |
| Preset name label | `label` | `preset_name_label` | align: CENTER, y: -10 |
| Selected text | `label` | `preset_selected_text` | align: CENTER, y: 20 |
| Page dots container | `obj` | — | align: BOTTOM_MID |

---

## 6. Interaction Model

### Rotary Encoder

| Page | CW Rotation | CCW Rotation | Press |
|------|-------------|--------------|-------|
| Power | Navigate to Brightness page | Navigate to Presets page | Toggle power |
| Brightness | Increase brightness +5% | Decrease brightness -5% | Navigate to Power page |
| Presets | Cycle to next preset (clockwise) | Cycle to previous preset (CCW) | Apply highlighted preset |

### Touch

| Page | Tap Center | Swipe Left | Swipe Right |
|------|-----------|------------|-------------|
| Power | Toggle power | Go to Presets | Go to Brightness |
| Brightness | No action | Go to Power | Go to Presets |
| Presets | Apply current preset | Go to Brightness | Go to Power |

### Preset Cycling (Page 3)

Knob rotation on the Presets page moves the selection around the ring:
- CW: Warm White → Soft Amber → Neutral White → Low Nightlight → (wrap)
- CCW: Reverse direction
- Press: Apply the currently highlighted preset

---

## 7. Wake/Sleep Behavior

| Trigger | Action |
|---------|--------|
| Any input while asleep | Wake display only; do NOT execute action |
| Second input while awake | Execute the mapped action |
| Inactivity timeout (60s) | Fade to sleep |

### Wake Animation Sequence

1. Ring fades in first (200ms ease-out)
2. Center content appears (200ms ease-out, 100ms delay after ring)
3. Page dots appear last (instant, after center)

This creates a "frame first, content second" reveal — like a spotlight warming up.

---

## 8. LED Ring Behavior

The 5 WS2812 LEDs mirror the active preset's color:

| Preset | LED Color | RGB |
|--------|-----------|-----|
| Warm White | Amber | (255, 165, 0) |
| Soft Amber | Deep Gold | (204, 136, 0) |
| Neutral White | Cool White | (180, 200, 255) |
| Low Nightlight | Very Dim Amber | (40, 25, 0) |
| Power OFF | Off | (0, 0, 0) |

---

## 9. Animation Specification

| Animation | Duration | Easing | Trigger |
|-----------|----------|--------|---------|
| Wake ring fade-in | 200ms | ease-out | Any input while asleep |
| Wake content fade-in | 200ms | ease-out | 100ms after ring |
| Sleep fade-out | 300ms | ease-in | Inactivity timeout |
| Preset selection change | Instant | N/A | Knob rotation on Presets page |
| Brightness arc update | Instant | N/A | Knob rotation on Brightness page |
| Page transition | Instant (LVGL page show) | N/A | Navigation action |

Note: The "traveling highlight" animation (smooth slide between segments) is classified as v1-expanded. The v1 implementation uses instant segment switching.

---

## 10. Home Assistant Integration

| Entity | Purpose |
|--------|---------|
| `light.bedroom_group` | Primary control target |
| Attribute: `brightness` | Maps to 0-100% display value and brightness arc |
| Attribute: `color_temp` | Determines which preset segment is active |
| Service: `light.toggle` | Power toggle |
| Service: `light.turn_on` | Apply preset (with brightness + color_temp) |

### State Import

On HA state change:
- Power ring: full amber (on) or dim gray (off)
- Brightness arc: proportional to brightness attribute
- Preset segments: active segment determined by matching color_temp to preset table

---

## 11. Differentiation from Other Concepts

| Aspect | Concept 05 (This) | Concept 02 (SmartKnob Arc) | Concept 03 (Large Power Button) |
|--------|-------------------|---------------------------|-------------------------------|
| Hero element | 4-segment preset ring | Single brightness arc | 140px power button |
| Ring usage | Multi-segment, color-coded | Single continuous arc | None |
| Preset access | Dedicated page with ring | Not on main page | Diamond layout on Page 3 |
| Physical metaphor | Radio tuner / program selector | SmartKnob haptic dial | Light switch |
| Unique to round display? | **YES** — impossible on rectangular | Partially | No |
