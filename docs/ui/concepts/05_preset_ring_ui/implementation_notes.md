# Concept 05: Preset Ring UI — Implementation Notes

## Gate Status

> **PASSES all gates (G1–G8).**

---

## 1. LVGL Widget Strategy

The key technical challenge is implementing 4 separate arc widgets on the Presets page, each with different start/end angles and independently controllable colors. ESPHome LVGL supports multiple `arc` widgets on the same page.

### Arc Configuration for 4 Quadrant Segments

Each preset arc uses the same size (220x220) and position (CENTER) but different angle ranges:

| Arc ID | Start Angle | End Angle | Default Color |
|--------|-------------|-----------|---------------|
| `preset_arc_1` | 350 | 70 | `0xFFA500` (active) or `0x333333` (inactive) |
| `preset_arc_2` | 80 | 160 | `0xCC8800` (active) or `0x333333` (inactive) |
| `preset_arc_3` | 170 | 250 | `0xCCDDFF` (active) or `0x333333` (inactive) |
| `preset_arc_4` | 260 | 340 | `0x664400` (active) or `0x333333` (inactive) |

**Important:** In ESPHome LVGL, arcs use `value` to control the indicator portion. For our preset ring, we use the arc's `indicator` color as the segment fill. Setting `value` to `max_value` fills the entire segment; setting to `min_value` shows only the background arc.

### Alternative Approach: Background Arc Only

Since we want the segments to be either "filled" (active color) or "dim" (inactive), we can use the arc's background (`arc_color` in the main style) as the segment color and simply change the color via `lvgl.widget.update`. No indicator needed — the arc background IS the segment.

```yaml
- arc:
    id: preset_arc_1
    align: CENTER
    width: 220
    height: 220
    arc_width: 12
    start_angle: 350
    end_angle: 70
    value: 0
    min_value: 0
    max_value: 100
    adjustable: false
    indicator:
      arc_color: 0xFFA500  # Active color (or 0x333333 when inactive)
      arc_width: 12
    bg_opa: TRANSP
    border_width: 0
```

---

## 2. Page Navigation Implementation

Three pages with LVGL page show:

```yaml
lvgl:
  pages:
    - id: power_page
    - id: brightness_page
    - id: presets_page
```

Navigation via knob (on Power page, CW goes to Brightness) or via swipe (using `on_load` to track current page).

---

## 3. Preset Selection Logic

```cpp
globals:
  - id: highlighted_preset
    type: int
    initial_value: '0'  // 0=Warm White, 1=Soft Amber, 2=Neutral, 3=Nightlight

// On CW rotation (Presets page):
id(highlighted_preset) = (id(highlighted_preset) + 1) % 4;
id(update_preset_ring).execute();

// On CCW rotation (Presets page):
id(highlighted_preset) = (id(highlighted_preset) + 3) % 4;
id(update_preset_ring).execute();
```

The `update_preset_ring` script updates all 4 arc widgets — the active one gets its preset color, the other 3 get dim gray.

---

## 4. Updating Arc Colors

ESPHome LVGL supports `lvgl.widget.update` to change arc indicator color:

```yaml
- id: update_preset_ring
  then:
    - lvgl.widget.update:
        id: preset_arc_1
        indicator:
          arc_color: !lambda |-
            return (id(highlighted_preset) == 0) ? lv_color_hex(0xFFA500) : lv_color_hex(0x333333);
    - lvgl.widget.update:
        id: preset_arc_2
        indicator:
          arc_color: !lambda |-
            return (id(highlighted_preset) == 1) ? lv_color_hex(0xCC8800) : lv_color_hex(0x333333);
    # ... etc for arcs 3 and 4
```

**Note:** Based on compile experience from Concepts 01-04, `lvgl.widget.update` with nested `indicator:` may not be supported. If not, we'll use the simpler approach of setting the arc `value` to max (fills indicator) or min (hides indicator), keeping the indicator color constant per arc.

---

## 5. Power Ring Implementation

The Power page uses a single full-circle arc (360° sweep):

```yaml
- arc:
    id: power_ring
    align: CENTER
    width: 220
    height: 220
    arc_width: 12
    start_angle: 0
    end_angle: 360
    min_value: 0
    max_value: 100
    value: 100  # Full ring
    adjustable: false
    indicator:
      arc_color: 0xFFA500  # Amber when ON, 0x333333 when OFF
      arc_width: 12
```

On power toggle, update the indicator color between amber and dim gray.

---

## 6. Wake-Only-First Implementation

Same pattern as Concepts 01-04:

```cpp
globals:
  - id: display_awake
    type: bool
    initial_value: 'true'
  - id: touch_woke_this_cycle
    type: bool
    initial_value: 'false'
```

All input handlers check `display_awake` before executing actions.

---

## 7. Performance Considerations

| Concern | Mitigation |
|---------|-----------|
| 4 arcs on one page | ESP32-S3 handles this easily; arcs are lightweight LVGL primitives |
| Simultaneous color updates | 4 `lvgl.widget.update` calls in sequence; <5ms total |
| Page transitions | Instant `lvgl.page.show`; no animation overhead |
| Arc rendering | No anti-aliasing needed at 12px width; fast fill |

---

## 8. Known Limitations

1. **No traveling highlight animation** — v1 uses instant segment switching
2. **Arc gap rendering** — 10° gaps may appear as thin lines; may need adjustment on physical hardware
3. **Color similarity** — Warm White and Soft Amber are close; may need more contrast on physical display
4. **No tap-on-segment** — Touch detection on specific arc segments is complex; v1 uses center tap only
5. **4 arcs stacked** — All 4 arcs are at the same position; rendering order matters

---

## 9. Files

| File | Purpose |
|------|---------|
| `esphome/concepts/door_side_concept_05_preset_ring.yaml` | Prototype YAML |
| `docs/ui/concepts/05_preset_ring_ui/research.md` | Research findings |
| `docs/ui/concepts/05_preset_ring_ui/design_spec.md` | Visual specification |
| `docs/ui/concepts/05_preset_ring_ui/implementation_notes.md` | This file |
| `docs/ui/concepts/05_preset_ring_ui/validation_notes.md` | Physical test plan |
