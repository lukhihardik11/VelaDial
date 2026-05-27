# Concept 04: Single-Page Simple Mode — Implementation Notes

## Gate Status

> **FAILS Gate G1 (Three-Page Lock).** Prototyped for exploration only.

---

## 1. LVGL Widget Strategy

This is the simplest LVGL implementation in the entire 20-concept matrix. The entire UI consists of:

| Widget | LVGL Type | Purpose |
|--------|-----------|---------|
| `brightness_arc` | `arc` | Thin visual indicator around center value |
| `brightness_label` | `label` | Hero 48pt percentage display |
| `power_label` | `label` | Power state "ON"/"OFF" in top zone |
| `preset_label` | `label` | Active preset name in bottom zone |
| `top_touch` | `obj` | Invisible clickable zone for power toggle |
| `bottom_touch` | `obj` | Invisible clickable zone for preset cycling |

Total: 6 widgets on 1 page. No page transitions, no tileview, no tabview.

---

## 2. Touch Zone Implementation

The key technical challenge is dividing the round display into three tap regions. ESPHome LVGL uses `obj` widgets with `on_click` handlers as invisible touch targets:

```yaml
- obj:
    id: top_touch
    x: 20
    y: 10
    width: 200
    height: 75
    bg_opa: TRANSP
    border_width: 0
    clickable: true
    on_click:
      - lambda: |-
          // Wake guard + power toggle
```

The center zone has NO touch target — brightness is knob-only. This prevents accidental touches while adjusting the knob.

---

## 3. Arc Widget Configuration

The brightness arc is purely visual (no touch interaction). It uses a thin line width to avoid dominating the display:

| Property | Value | Rationale |
|----------|-------|-----------|
| Width/Height | 160px | Leaves room for top/bottom labels |
| Arc width (indicator) | 4px | Thin, subtle indicator |
| Arc width (main/bg) | 4px | Matches indicator |
| Start angle | 135° | Standard bottom-open arc |
| End angle | 45° | 270° sweep |
| Mode | NORMAL | Visual only, no touch |
| Indicator color | Amber (`0xFFA500`) | Matches accent |
| Background color | Dark (`0x222222`) | Subtle unfilled portion |

---

## 4. Preset Cycling Logic

The preset cycling uses a global variable to track the current index:

```cpp
globals:
  - id: current_preset_index
    type: int
    initial_value: '0'

// On bottom zone tap:
id(current_preset_index) = (id(current_preset_index) + 1) % 4;
// Then call the corresponding HA action
```

Preset mapping:
| Index | Name | color_temp | brightness_pct |
|-------|------|-----------|----------------|
| 0 | Warm White | 370 | 100 |
| 1 | Soft Amber | 454 | 75 |
| 2 | Neutral White | 285 | 100 |
| 3 | Low Nightlight | 454 | 10 |

---

## 5. Wake-Only-First Implementation

Single-page wake guard is simpler than multi-page because there is no page-specific behavior to track:

```cpp
globals:
  - id: display_awake
    type: bool
    initial_value: 'false'
  - id: touch_woke_this_cycle
    type: bool
    initial_value: 'false'
```

Every input handler (touch zones, knob rotate, knob press) checks `display_awake` first. If false, it wakes the display and returns without executing the mapped action.

---

## 6. State Import from Home Assistant

A single `homeassistant.text_sensor` imports the full light state. The `on_value` lambda parses brightness and updates all three display elements:

```yaml
text_sensor:
  - platform: homeassistant
    id: bedroom_light_state
    entity_id: light.bedroom_group
    attribute: brightness
    on_value:
      - lambda: |-
          // Update brightness label and arc
          // Update power label based on state
          // Update preset label based on color_temp
```

---

## 7. Performance Characteristics

| Metric | Expected Value | Notes |
|--------|---------------|-------|
| RAM usage | ~15 KB LVGL | Minimal widgets |
| Render time | <5ms per frame | Single page, no transitions |
| Touch response | <50ms | Direct click handler |
| Knob response | <20ms | ISR-driven encoder |
| Wake time | ~300ms | Opacity animation only |

This concept is dramatically over-provisioned for the ESP32-S3's capabilities. The hardware could handle 10x the complexity.

---

## 8. Known Limitations

1. **No page indicator** — there is only one page, so no indicator needed
2. **Preset cycling is not discoverable** — new users may not realize bottom zone is tappable
3. **Center zone is touch-dead** — intentional, but may confuse users expecting tap-anywhere
4. **240x240 cramping** — three zones in a circle means each gets limited vertical space
5. **No visual feedback on preset cycle** — the name changes but there is no animation

---

## 9. Files

| File | Purpose |
|------|---------|
| `esphome/concepts/door_side_concept_04_single_page_simple.yaml` | Prototype YAML |
| `docs/ui/concepts/04_single_page_simple_mode/research.md` | Research findings |
| `docs/ui/concepts/04_single_page_simple_mode/design_spec.md` | Visual specification |
| `docs/ui/concepts/04_single_page_simple_mode/implementation_notes.md` | This file |
| `docs/ui/concepts/04_single_page_simple_mode/validation_notes.md` | Physical test plan |
