# UI Concept 11: Brightness-First UI — Implementation Notes

## Implementation Strategy

Concept 11 is architecturally identical to Concept 10 (Three-Screen Tab Carousel) with one structural change: the LVGL page order is swapped so Brightness is page index 0 (the default landing page). All widgets, styles, and interaction patterns are reused from Concept 10 with minimal modification.

## Key Implementation Differences from Concept 10

| Aspect | Concept 10 | Concept 11 |
|--------|-----------|-----------|
| Page 0 (default) | Power | **Brightness** |
| Page 1 | Brightness | **Power** |
| Page 2 | Presets | Presets (unchanged) |
| Knob press on page 0 | Toggle power | **Toggle power (shortcut)** |
| Wake target | Power page | **Brightness page** |
| Dot 1 meaning | Power | **Brightness** |

## LVGL Page Configuration

The page order in ESPHome LVGL is determined by the declaration order in the YAML. The first `page:` block becomes page index 0 and is shown on boot.

```yaml
lvgl:
  pages:
    - id: page_brightness    # Page 0 — default landing
    - id: page_power         # Page 1
    - id: page_presets       # Page 2
```

## Knob Press Dual-Function

The knob press handler must be context-aware:
- On Brightness page (page 0): Toggle power (shortcut — most common two-action flow)
- On Power page (page 1): Toggle power (standard)
- On Presets page (page 2): Apply selected preset

Implementation uses a `current_page` global variable updated by `on_load` callbacks:

```yaml
globals:
  - id: current_page
    type: int
    initial_value: '0'
```

## Swipe Guard Logic

Swipe direction is relative to page position:
- Page 0 (Brightness): Can only swipe LEFT → Power. Swipe RIGHT blocked (first page).
- Page 1 (Power): Can swipe LEFT → Presets. Can swipe RIGHT → Brightness.
- Page 2 (Presets): Can only swipe RIGHT → Power. Swipe LEFT blocked (last page).

## Wake-Only-First Integration

All input paths check `is_asleep` before executing actions:
1. If display is asleep → wake display, show Brightness page, consume input
2. If display is awake → execute the intended action

The wake target is always page 0 (Brightness), regardless of which page was active before sleep.

## Arc Widget Configuration

The brightness arc is the hero element on the default page:

| Property | Value |
|----------|-------|
| Width | 200px |
| Height | 200px |
| Arc thickness | 8px |
| Range | 0-100 |
| Start angle | 135° |
| End angle | 45° (270° sweep) |
| Indicator color | Amber `0xFFA500` |
| Background color | Dark gray `0x333333` |
| Mode | NORMAL (display-only) |

The arc value is bound to `brightness_pct` global and updated via:
- Knob rotation (±5% steps)
- Home Assistant state import
- Preset application

## Home Assistant Integration

```yaml
homeassistant.service:
  service: light.turn_on
  data:
    entity_id: light.bedroom_group
    brightness_pct: !lambda 'return id(brightness_pct);'
```

State import uses `homeassistant` text_sensor/sensor to pull current brightness from HA and update the arc on reconnection.

## Font Requirements

| Font ID | Size | Glyphs | Usage |
|---------|------|--------|-------|
| font_hero | 48pt | `0123456789%` | Brightness percentage |
| font_state | 48pt | `ONF` | Power state label |
| font_header | 14pt | `A-Z ` | Section headers |
| font_preset | 16pt | `A-Za-z ` | Preset names |

**Note:** Glyph strings must be deduplicated to avoid ESPHome compile errors.

## Performance Expectations

- Identical to Concept 10 (same widget count, same render complexity)
- Arc animation at 150ms is well within LVGL's 30fps budget on ESP32-S3
- Page transitions at 200ms use hardware-accelerated LVGL animations
- No performance risk from the page reorder

## Testing Priorities (Hardware Pending)

1. Verify Brightness page loads as default on boot
2. Verify knob rotation immediately adjusts brightness (no page navigation needed)
3. Verify knob press toggles power from Brightness page
4. Verify 3AM flow: wake → see brightness → rotate → press off (zero swipes)
5. Verify page dots correctly reflect Brightness/Power/Presets order
6. Verify swipe guards prevent invalid navigation at page boundaries

## Known Limitations

1. Page order change requires Hardik approval (non-standard hierarchy)
2. Knob press = power on Brightness page is non-obvious without onboarding
3. Users familiar with power-first controllers may need adaptation time
4. No user-configurable default page in v1 (v2 scope)

## Files

| File | Purpose |
|------|---------|
| `esphome/concepts/door_side_concept_11_brightness_first_ui.yaml` | Prototype YAML |
| `docs/ui/concepts/11_brightness_first_ui/research.md` | Research findings |
| `docs/ui/concepts/11_brightness_first_ui/design_spec.md` | Design specification |
| `docs/ui/concepts/11_brightness_first_ui/implementation_notes.md` | This file |
| `docs/ui/concepts/11_brightness_first_ui/validation_notes.md` | Validation plan |
