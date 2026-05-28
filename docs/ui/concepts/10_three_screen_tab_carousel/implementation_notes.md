# Concept 10: Three-Screen Tab Carousel — Implementation Notes

**Status:** CONCEPT PROTOTYPE — NOT PRODUCTION
**YAML:** `esphome/concepts/door_side_concept_10_three_screen_tab_carousel.yaml`

---

## 1. Architecture Overview

This concept is the closest to the existing production YAML (`door_side_rotary.yaml`) in architecture. It implements the locked v1 spec directly with premium execution details. The implementation uses standard LVGL page navigation with swipe gestures and a 3-dot indicator.

---

## 2. LVGL Page Structure

```
lvgl:
  pages:
    - id: page_power      # Page 0 — Power toggle
    - id: page_brightness  # Page 1 — Brightness arc
    - id: page_presets     # Page 2 — 2x2 preset grid
```

Each page contains its own set of widgets. The page dots are replicated on each page (not a persistent overlay) because LVGL pages are independent screens.

---

## 3. Key Widget IDs

| Widget | Type | Page | Purpose |
|--------|------|------|---------|
| `power_label` | label | Power | "ON"/"OFF" 48pt |
| `power_btn` | button | Power | Touch toggle area |
| `brightness_arc` | arc | Brightness | 270° sweep, amber |
| `brightness_label` | label | Brightness | "75%" center |
| `preset_btn_0` | button | Presets | Warm White tile |
| `preset_btn_1` | button | Presets | Soft Amber tile |
| `preset_btn_2` | button | Presets | Neutral White tile |
| `preset_btn_3` | button | Presets | Low Nightlight tile |
| `dot_1_p*` | obj | All | Page indicator dot 1 |
| `dot_2_p*` | obj | All | Page indicator dot 2 |
| `dot_3_p*` | obj | All | Page indicator dot 3 |

---

## 4. Page Navigation Implementation

### Swipe Gestures

ESPHome LVGL supports `on_swipe` events on pages:

```yaml
pages:
  - id: page_power
    on_swipe_left:
      - lvgl.page.show:
          id: page_brightness
          animation: MOVE_LEFT
          time: 200ms
```

### Knob Press Navigation

Knob press on Brightness page returns to Power page:

```yaml
- script.execute: navigate_to_power
```

---

## 5. Brightness Arc Configuration

| Property | Value |
|----------|-------|
| Type | `arc` |
| Range | 0-100 |
| Rotation | 135° |
| Span | 270° (from 135° to 45°) |
| Width | 8px |
| Color (active) | Amber `0xFFA500` |
| Color (inactive) | Gray `0x333333` |
| Knob visible | No (arc is display-only, knob controls value) |

The arc is NOT interactive via touch — brightness is controlled exclusively by the rotary encoder. This prevents accidental brightness changes from swipe gestures.

---

## 6. 2x2 Preset Grid Layout

The grid is positioned within the circular safe area (radius 100px from center):

```
Grid center: (120, 105)
Tile size: 85x45px each
Gap: 8px horizontal, 8px vertical
Total grid: 178x98px

Top-left:     x:35,  y:58  → Warm White
Top-right:    x:128, y:58  → Soft Amber
Bottom-left:  x:35,  y:111 → Neutral White
Bottom-right: x:128, y:111 → Low Nightlight
```

Active preset: amber border (2px), amber text
Inactive preset: gray border (1px), gray text

---

## 7. Page Dot Indicator

Each page has 3 dots at the bottom:

```
Dot size: 8px diameter (obj with border_radius: 4)
Dot spacing: 16px center-to-center
Position: y=215, centered at x=104, 120, 136
Active dot: amber background (0xFFA500)
Inactive dot: gray background (0x444444)
```

Dots are updated via `lvgl.widget.update` on page load.

---

## 8. Wake-to-Page-0 Logic

When the display wakes from sleep, it always shows Page 0 (Power):

```yaml
script:
  - id: wake_display
    then:
      - lvgl.page.show:
          id: page_power
          animation: NONE
          time: 0ms
      - light.turn_on:
          id: backlight
          brightness: 100%
          transition_length: 200ms
```

---

## 9. Compile-Proven Patterns Used

All patterns are drawn from Concepts 01-09 (all compile-PASSED):

| Pattern | Source |
|---------|--------|
| `lvgl.label.update` for text changes | Concept 08, 09 |
| `lvgl.page.show` with animation | Concept 06 |
| `on_swipe_left` / `on_swipe_right` | New for Concept 10 (standard LVGL) |
| Wake-only-first with `display_awake` global | All concepts |
| `ili9xxx` with `model: GC9A01A` | Concept 09 |
| Rotary encoder with `on_clockwise`/`on_anticlockwise` | All concepts |
| LED ring `addressable_set` | Concept 09 |
| Font glyph deduplication | Concept 07, 09 |

---

## 10. Performance Expectations

| Metric | Expected |
|--------|----------|
| Compile time | ~80-400s (depends on cache) |
| Widget count | ~30 (moderate) |
| RAM usage | Low-medium |
| Page transition | 200ms (smooth at 30fps) |
| Arc render | Instant (LVGL native) |

---

## 11. Known Limitations

1. Page dots are replicated per-page (no persistent overlay in LVGL pages)
2. 2x2 grid wastes some round-display area (accepted for familiarity)
3. No knob-cycle presets in v1 (v1-expanded feature)
4. Swipe gesture threshold must be tuned on hardware (30px assumed)
5. Arc is display-only — no touch-drag brightness (knob-only)
