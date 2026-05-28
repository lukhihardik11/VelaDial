# Concept 08: Apple Watch Complications — Design Specification

## Classification

| Field | Value |
|-------|-------|
| Concept ID | 08 |
| Name | Apple Watch Complications |
| Category | Standard (Primary) |
| Difficulty | Hard |
| Gate Status | PASSES gates G1-G8 (v1-expanded adaptation) |
| v1 Candidate | **NOT RECOMMENDED** for v1 per direction matrix |
| v1-Expanded | 2 corner complications (WiFi + lux) as decorative additions |
| v2 | Full complication face, interactive complications, sensor detail pages |
| Risk Level | Medium-High (information density at 190 PPI) |

---

## 1. Design Philosophy

> **WARNING:** The direction matrix explicitly states this concept is "NOT RECOMMENDED for v1 — too complex, violates scope." This prototype implements a v1-compatible adaptation that uses complications as **decorative, non-interactive data labels** on the existing 3-page structure. The full watchOS Infograph experience requires v2 hardware/resolution improvements.

Apple Watch Complications borrows the "complications" metaphor from watchOS: small data widgets arranged around a central element on a circular face. Each complication shows a single piece of information in a compact format. The center of the display shows the primary control value, and 2 complications orbit in the upper corners.

The v1 adaptation is deliberately restrained:
- Only **2 complications per page** (top-left, top-right corners)
- Complications are **display-only** (no tap interaction)
- Complications use **14pt minimum** font (readability at 190 PPI)
- Primary center content remains the hero element
- Complications are secondary/ambient information

---

## 2. Visual Reference

The aesthetic reference is the **watchOS Infograph face** — information-rich, colorful, and complex. However, at 240x240 (190 PPI vs Apple Watch's 326 PPI), we must reduce density significantly.

**Adaptation from watchOS to VelaDial:**

| watchOS Infograph | VelaDial v1-Expanded |
|-------------------|---------------------|
| 8 complications + center | 2 complications + center |
| Interactive (tap to expand) | Display-only |
| Full color icons | Monochrome icons (text glyphs) |
| 44mm / 326 PPI | 32.5mm / 190 PPI |
| Multiple data sources | WiFi RSSI + ambient lux only |

---

## 3. Screen Architecture (3 Pages)

### Page 1: Power

```
┌─────────────────────────┐
│  ⚡-72dBm    ☀ 340lx   │  ← complications (14pt, gray)
│                         │
│                         │
│         ON              │  ← 48pt white, CENTER
│                         │
│                         │
│     ● ○ ○              │  ← page dots, y: 210
└─────────────────────────┘
```

- **Center:** Power state `ON`/`OFF` at 48pt (reduced from 56pt to accommodate complications)
- **Top-left complication:** WiFi signal strength (RSSI in dBm)
- **Top-right complication:** Ambient light (lux from TSL2591, simulated)
- **Bottom:** Page indicator dots

### Page 2: Brightness

```
┌─────────────────────────┐
│  ⚡-72dBm    ☀ 340lx   │  ← complications (14pt, gray)
│                         │
│                         │
│        75%              │  ← 48pt white, CENTER
│                         │
│                         │
│     ○ ● ○              │  ← page dots, y: 210
└─────────────────────────┘
```

- **Center:** Brightness percentage at 48pt
- **Complications:** Same as Page 1 (persistent ambient data)
- Knob CW/CCW adjusts brightness ±5%

### Page 3: Presets

```
┌─────────────────────────┐
│  ⚡-72dBm    ☀ 340lx   │  ← complications (14pt, gray)
│                         │
│                         │
│    Warm White           │  ← 24pt amber (active preset)
│                         │
│                         │
│     ○ ○ ●              │  ← page dots, y: 210
└─────────────────────────┘
```

- **Center:** Active preset name at 24pt amber
- **Complications:** Same as Page 1 (persistent ambient data)
- Knob rotation cycles presets, press applies

---

## 4. Complication Design

### Layout Specifications

| Complication | Position | Content | Font | Color |
|-------------|----------|---------|------|-------|
| WiFi Signal | Top-left (x:35, y:30) | Icon + RSSI value | 14pt | Gray `0x888888` |
| Ambient Light | Top-right (x:155, y:30) | Icon + lux value | 14pt | Gray `0x888888` |

### Complication Format

Each complication is a single LVGL label with format: `{icon} {value}{unit}`

- WiFi: `W -72dBm` (W = WiFi glyph substitute)
- Lux: `L 340lx` (L = Light glyph substitute)

In v1, text characters substitute for icons (no custom icon font needed). In v2, Material Design Icon glyphs would replace the text prefixes.

### Color Coding (v2 future)

| Complication | Color Logic |
|-------------|-------------|
| WiFi | Green (> -50dBm), Amber (-50 to -70), Red (< -70) |
| Lux | Amber (> 100 lux), Gray (10-100), Blue (< 10) |

In v1-expanded, both complications use static gray `0x888888`.

---

## 5. Color Palette

| Color | Hex | Role |
|-------|-----|------|
| Pure Black | `0x000000` | Background |
| White | `0xFFFFFF` | Primary value (active) |
| Amber | `0xFFA500` | Active preset, page dots |
| Medium Gray | `0x888888` | Complications, secondary text |
| Dim Gray | `0x666666` | Primary value (off state) |
| Dark Gray | `0x555555` | Inactive presets |
| Dot Inactive | `0x444444` | Inactive page dots |

---

## 6. Interaction Model

| Input | Page 1 (Power) | Page 2 (Brightness) | Page 3 (Presets) |
|-------|----------------|---------------------|------------------|
| Knob CW | Navigate to Page 2 | Brightness +5% | Cycle preset forward |
| Knob CCW | Navigate to Page 3 | Brightness -5% | Cycle preset backward |
| Knob Press | Toggle power | Return to Page 1 | Apply preset, return to Page 1 |
| Touch center | Toggle power | — | — |

**Complications are NOT interactive in v1.** They are display-only ambient data.

### Wake-Only-First

All inputs on wake restore the display only — no action is executed.

---

## 7. Sensor Data Sources

| Complication | Sensor | ESPHome Platform | Notes |
|-------------|--------|-----------------|-------|
| WiFi Signal | ESP32 WiFi | `wifi_signal` | Built-in, no extra hardware |
| Ambient Light | TSL2591 | `template` (simulated) | Real sensor on hardware, simulated for compile |

### Update Frequency

- WiFi RSSI: Every 30 seconds
- Ambient Lux: Every 10 seconds (via template sensor for compile)

---

## 8. LED Ring Behavior

Binary state (same as Text-First Utility):
- **Lights ON:** All 5 LEDs amber at 30%
- **Lights OFF:** All LEDs off

v2 future: LED ring could map ambient temperature (blue=cold, amber=warm, red=hot).

---

## 9. Sleep/Wake Behavior

- **Sleep:** After 60s inactivity, backlight fades to 10%
- **Wake:** Backlight restores to 80%, center content appears first
- **Staged reveal (v1-expanded future):** Center fades in at 0ms, complications fade in at 200ms delay

---

## 10. What Could Go Wrong

| Risk | Mitigation |
|------|-----------|
| 14pt text too small at 190 PPI | Test on hardware; fall back to 16pt if unreadable |
| Complications distract from primary content | Use dim gray color; keep complications subtle |
| Information overload in bedroom | Complications are ambient-only; no action required |
| Sensor data stale/unavailable | Show `--` placeholder when no data |
| Too similar to Concept 07 (Text-First) | Complications add visual structure; distinct layout |

---

## 11. Home Assistant Integration

- Target entity: `light.bedroom_group`
- State import: power state, brightness attribute
- Services called: `light.toggle`, `light.turn_on` (with brightness_pct)
- Sensor data: WiFi RSSI (built-in), lux (simulated template)
