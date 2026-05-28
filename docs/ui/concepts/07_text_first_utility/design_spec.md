# Concept 07: Text-First Utility — Design Specification

## Classification

| Field | Value |
|-------|-------|
| Concept ID | 07 |
| Name | Text-First Utility |
| Category | Standard (Primary) |
| Difficulty | Easy |
| Gate Status | PASSES all gates (G1–G8) |
| v1 Candidate | Yes — v1 locked |
| Score Rank | High readability, low implementation risk |

---

## 1. Design Philosophy

Text-First Utility rejects all decorative elements. No arcs, no rings, no gradients, no glow effects. Every piece of information is communicated through **typography alone**. The display shows large, confident text labels — `ON`, `OFF`, `75%`, `WARM WHITE` — in a clean sans-serif font against a pure black background. The amber accent color (`0xFFA500`) is used only for the active state indicator and page dots.

The visual reference is not other smart-home devices but rather:
- **Massimo Vignelli's NYC subway signage** — clarity through type hierarchy
- **Aviation instrumentation** — information density via text
- **Luxury brand packaging** — confidence to use white space

> "When text is the only element, every detail matters: letter spacing, line height, weight, alignment."

The premium quality comes from the same place as a luxury brand's packaging — the confidence to use white space and let the content speak.

---

## 2. Typography System

| Role | Font | Size | Color | Usage |
|------|------|------|-------|-------|
| Primary Value | Roboto | 56pt | White `0xFFFFFF` (on) / Dim Gray `0x666666` (off) | ON, OFF, 75% |
| Secondary Label | Roboto | 16pt | Gray `0x888888` | "Power", "Brightness", "Presets" |
| Tertiary Info | Roboto | 12pt | Dark Gray `0x555555` | Status, step indicator |
| Preset Names | Roboto | 24pt | Amber `0xFFA500` (active) / Gray `0x555555` (inactive) | Warm White, Soft Amber, etc. |

### Font Loading Strategy

Load Roboto from Google Fonts at 4 sizes with minimal glyph subsets:
- **56pt:** `0123456789%` + `ONOFF` (primary values)
- **32pt:** Full uppercase + digits (preset names)
- **24pt:** Full uppercase + lowercase + digits (preset names)
- **16pt:** Full ASCII printable range (labels)
- **12pt:** Full ASCII printable range (status text)

---

## 3. Screen Architecture (3 Pages)

### Page 1: Power

```
┌─────────────────────────┐
│                         │
│       "Power"           │  ← 16pt gray, y: 50
│                         │
│         ON              │  ← 56pt white, CENTER
│                         │
│     ● ○ ○              │  ← page dots, y: 210
│                         │
└─────────────────────────┘
```

- Primary: `ON` or `OFF` at 56pt, centered
- `ON` = white, `OFF` = dim gray `0x666666`
- Secondary: "Power" label at 16pt gray above center
- Page dots at bottom (amber active, gray inactive)

### Page 2: Brightness

```
┌─────────────────────────┐
│                         │
│     "Brightness"        │  ← 16pt gray, y: 50
│                         │
│        75%              │  ← 56pt white, CENTER
│                         │
│       ±5%              │  ← 12pt dark gray, y: 170
│     ○ ● ○              │  ← page dots, y: 210
│                         │
└─────────────────────────┘
```

- Primary: percentage at 56pt white, centered
- Secondary: "Brightness" label at 16pt gray
- Tertiary: `±5%` step indicator at 12pt
- Knob CW/CCW adjusts in 5% steps

### Page 3: Presets

```
┌─────────────────────────┐
│                         │
│      "Presets"          │  ← 16pt gray, y: 35
│                         │
│    Warm White           │  ← 24pt AMBER (active)
│    Soft Amber           │  ← 24pt gray
│    Neutral White        │  ← 24pt gray
│    Low Nightlight       │  ← 24pt gray
│                         │
│     ○ ○ ●              │  ← page dots, y: 210
│                         │
└─────────────────────────┘
```

- Vertical text list of 4 presets
- Active preset highlighted in amber `0xFFA500`
- Inactive presets in dim gray `0x555555`
- Knob rotation moves highlight up/down
- Knob press applies selected preset

---

## 4. Color Palette

| Color | Hex | Role |
|-------|-----|------|
| Pure Black | `0x000000` | Background (all pages) |
| White | `0xFFFFFF` | Primary value (active state) |
| Amber | `0xFFA500` | Active indicator, page dots, active preset |
| Medium Gray | `0x888888` | Secondary labels |
| Dim Gray | `0x666666` | Primary value (off state) |
| Dark Gray | `0x555555` | Tertiary text, inactive presets |
| Dot Inactive | `0x444444` | Inactive page dots |

---

## 5. Interaction Model

| Input | Page 1 (Power) | Page 2 (Brightness) | Page 3 (Presets) |
|-------|----------------|---------------------|------------------|
| Knob CW | Navigate to Page 2 | Brightness +5% | Highlight next preset |
| Knob CCW | Navigate to Page 3 | Brightness -5% | Highlight previous preset |
| Knob Press | Toggle power | Navigate to Page 1 | Apply selected preset |
| Touch center | Toggle power | — | — |

### Wake-Only-First

All inputs on wake restore the display only — no action is executed. The first input after sleep is consumed by the wake event.

---

## 6. LED Ring Behavior

Minimal and binary:
- **Lights ON:** All 5 LEDs amber at 30% brightness
- **Lights OFF:** All LEDs off

No color variation, no brightness mapping. The LED ring is secondary to the text.

---

## 7. Sleep/Wake Behavior

- **Sleep:** After 60s inactivity, backlight fades to 10% (or off)
- **Wake:** Backlight restores to 80%, text appears immediately (v1 locked)
- **v1-expanded (future):** Typewriter effect on wake — text appears character by character at ~50ms/char

---

## 8. Safe Area Calculations

On a 240x240 round display with circular bezel:
- **Inscribed square:** ~170x170px centered
- **Usable text area:** x: 35–205, y: 35–205
- **Primary text (56pt):** Must stay within y: 80–160 for full visibility
- **Page dots:** y: 210 is safe (within bottom quadrant)
- **Presets list:** y: 60–200 with 35px spacing between items

---

## 9. What Makes It Premium vs. "Cheap LCD Clock"

| Premium | Cheap |
|---------|-------|
| Generous white space | Cramped layout |
| Precise alignment (pixel-perfect center) | Slightly off-center |
| Confident font sizing (56pt hero) | Timid sizing (24-32pt) |
| Minimal color palette (2 colors max per page) | Rainbow colors |
| Information hierarchy (clear primary/secondary/tertiary) | All text same size |
| Amber accent only for active state | Color used randomly |

---

## 10. Home Assistant Integration

- Target entity: `light.bedroom_group`
- State import: power state, brightness attribute
- Services called: `light.toggle`, `light.turn_on` (with brightness_pct and effect)
- Presets map to HA effects or scenes
