# Concept 10: Three-Screen Tab Carousel — Design Specification

**Status:** CONCEPT PROTOTYPE — NOT PRODUCTION
**Hardware:** ELECROW CrowPanel 1.28" ESP32-S3 (240x240 round, GC9A01A, CST816D touch, rotary encoder)
**Target:** `light.bedroom_group` via Home Assistant

---

## 1. Concept Overview

The Three-Screen Tab Carousel is the **direct, literal implementation** of the locked v1 specification. Three horizontally swipeable pages — Power, Brightness, Presets — with a 3-dot page indicator at the bottom. This is not a creative reinterpretation; it is the spec made visual with premium execution details.

The concept's value is **reliability and familiarity**. It is the safe baseline against which every other concept is compared. Its premium quality comes entirely from execution: consistent spacing, aligned elements, smooth 200ms transitions, and the amber accent creating visual warmth.

---

## 2. Visual Language

| Property | Value |
|----------|-------|
| Background | Pure black `0x000000` |
| Primary text | White `0xFFFFFF` |
| Active accent | Amber `0xFFA500` |
| Inactive/dim | Gray `0x666666` |
| Error state | Red `0xFF0000` |
| Font family | Roboto (Google Fonts) |
| Primary font size | 48pt (power state, brightness %) |
| Secondary font size | 16pt (labels, preset names) |
| Tertiary font size | 12pt (page dots, status) |

---

## 3. Page Architecture

### Page 1: Power

| Element | Position | Size | Description |
|---------|----------|------|-------------|
| Power state label | Center (x:120, y:100) | 48pt | "ON" (white) / "OFF" (gray) |
| Touch toggle area | Full center circle | 140px diameter | Tap anywhere in center to toggle |
| Page dots | Bottom center (y:215) | 8px each, 16px gap | Dot 1 amber, dots 2-3 gray |

### Page 2: Brightness

| Element | Position | Size | Description |
|---------|----------|------|-------------|
| Brightness arc | Center | 180px diameter, 8px width | 270° sweep (135° to 45°), amber fill |
| Percentage label | Arc center (x:120, y:110) | 48pt | "75%" white |
| "Brightness" label | Below arc (y:170) | 16pt | Gray, informational |
| Page dots | Bottom center (y:215) | 8px each | Dot 2 amber, dots 1-3 gray |

### Page 3: Presets

| Element | Position | Size | Description |
|---------|----------|------|-------------|
| 2x2 preset grid | Center area | 4 tiles, 90x50px each | Within circular safe area |
| Active preset | Highlighted | Amber border + text | Currently active preset |
| Inactive presets | Normal | Gray border + text | Available presets |
| Page dots | Bottom center (y:215) | 8px each | Dot 3 amber, dots 1-2 gray |

### Preset Tiles (2x2 Grid)

| Position | Preset | Color Accent |
|----------|--------|--------------|
| Top-left | Warm White | Amber `0xFFA500` |
| Top-right | Soft Amber | Deep Gold `0xCC8800` |
| Bottom-left | Neutral White | Cool White `0xCCDDFF` |
| Bottom-right | Low Nightlight | Dim Amber `0x664400` |

---

## 4. Circular Safe Area

The 240x240 round display clips corners. The safe area for content is a circle with radius 110px from center (220px diameter). The 2x2 grid on Page 3 is positioned within this safe area to avoid bezel clipping.

| Zone | Radius | Usage |
|------|--------|-------|
| Hero content | 0-80px | Primary elements (text, arc) |
| Secondary content | 80-100px | Grid tiles, labels |
| Edge zone | 100-110px | Page dots only |
| Clip zone | 110-120px | Avoid placing content here |

---

## 5. Interaction Model

| Input | Power Page | Brightness Page | Presets Page |
|-------|-----------|----------------|-------------|
| Knob CW | No action | Brightness +5% | No action |
| Knob CCW | No action | Brightness -5% | No action |
| Knob press | Toggle power | Return to Power page | Apply selected preset |
| Touch tap center | Toggle power | No action | Select preset tile |
| Swipe left | Go to Brightness | Go to Presets | No action |
| Swipe right | No action (first page) | Go to Power | Go to Brightness |

### Wake-Only-First Logic

All inputs when display is asleep ONLY wake the display. No action is executed. The display always wakes to Page 1 (Power) regardless of which page was active at sleep.

---

## 6. Page Transition Animation

| Property | Value |
|----------|-------|
| Type | Horizontal slide |
| Duration | 200ms |
| Easing | ease-in-out |
| Direction | Content slides left (swipe left) or right (swipe right) |
| Dot update | Instant on page change |

---

## 7. LED Ring Behavior

| State | LED Ring |
|-------|----------|
| Lights ON | All 5 amber, brightness proportional (capped 30%) |
| Lights OFF | All 5 off |
| No page-specific LED behavior | Standard across all pages |

---

## 8. Sleep/Wake Behavior

| Parameter | Value |
|-----------|-------|
| Sleep timeout | 30 seconds no interaction |
| Wake trigger | Any input (touch, knob rotate, knob press) |
| Wake page | Always Page 1 (Power) |
| Backlight off | Fade to 0% over 500ms |
| Backlight on | Fade to 100% over 200ms |

---

## 9. Dark-Room Usability

Each page uses large primary elements (48pt text, thick 8px arc) visible at arm's length even at minimum backlight. The amber accent provides sufficient contrast against pure black without causing eye strain in a dark bedroom.

---

## 10. What Makes It Premium

The premium quality comes from execution details, not novelty:
- Consistent 16px padding from safe area edge
- Vertically centered primary elements
- Smooth 200ms transitions (not instant, not sluggish)
- Amber accent intentionally chosen for warmth
- Page dots precisely aligned at bottom center
- Typography hierarchy (48pt/16pt/12pt) creates clear information layers

---

## 11. Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| Feels generic | Medium | Premium execution details differentiate |
| 2x2 grid corner clipping | Low | Grid positioned within safe area |
| Does not exploit round form | Medium | Accepted trade-off for familiarity |
| Swipe conflicts with tap | Low | Gesture threshold: 30px minimum distance |

---

## 12. v1 Classification

| Scope | Included |
|-------|----------|
| **v1 locked** | Exactly this implementation — 3 pages, 4 presets, wake-only-first |
| **v1-expanded** | Knob-cycle presets on Page 3, refined spacing |
| **v2** | Additional pages, custom themes, configurable page order |
