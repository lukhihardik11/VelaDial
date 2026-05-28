# Concept 12: Door Switch Replacement — Implementation Notes

## LVGL Architecture

### Widget Count
- **Power page:** 3 widgets (full-screen obj, ON label, OFF label with visibility toggle)
- **Brightness page:** 5 widgets (arc, percentage label, page dots reference)
- **Presets page:** 6 widgets (4 preset labels + container + page dots reference)
- **Top layer:** 3 widgets (dot indicator container + 3 dots)
- **Total:** ~17 widgets (lightweight)

### Full-Screen Touch Target Implementation

The Power page hero is a single `obj` widget covering the entire 240x240 display area:

```yaml
lvgl:
  pages:
    - id: page_power
      widgets:
        - obj:
            id: power_touch_area
            width: 240
            height: 240
            x: 0
            y: 0
            bg_color: 0xFFA500  # amber when ON
            bg_opa: COVER
            border_width: 0
            radius: 120  # circular clip
            on_click:
              - script.execute: guarded_power_toggle
```

### ON/OFF State Visualization

Two approaches for the full-screen state change:

**Approach A (Selected): Background color swap**
- ON: `bg_color: 0xFFA500`, label shows "ON" in white 64pt
- OFF: `bg_color: 0x0A0A0A`, label shows "OFF" in gray 48pt, border_color: white, border_width: 2

**Approach B (Alternative): Opacity-based**
- Single amber obj with opa transition between COVER and TRANSP
- Simpler animation but less control over OFF state border

### Style Transition for Animation

LVGL style transitions provide the 300ms fade between ON and OFF states:

```yaml
# Applied via lvgl.widget.update with transition properties
- lvgl.widget.update:
    id: power_touch_area
    bg_color: 0xFFA500
    transition:
      time: 300ms
      path: ease_in_out
```

Note: ESPHome LVGL may not support inline transition properties. Fallback is immediate color change (still premium — the LED ring provides the animation feedback).

### Tap vs Swipe Discrimination

ESPHome LVGL handles this automatically:
- `on_click` fires only on tap (touch down + touch up in same area)
- LVGL's built-in gesture detection handles swipes separately
- The `on_click` event has built-in movement threshold (~10px)
- Additional guard: if `is_asleep` is true, first touch only wakes

### Shorter Sleep Timeout

```yaml
script:
  - id: reset_sleep_timer
    then:
      - if:
          condition:
            lambda: 'return id(current_page) == 0;'
          then:
            - delay: 30s  # Power page: shorter timeout
          else:
            - delay: 60s  # Other pages: standard timeout
      - script.execute: enter_sleep
```

## Hardware Configuration

### Display
- GC9A01A (240x240 round IPS)
- SPI bus: GPIO6 (MOSI), GPIO4 (CLK), GPIO3 (DC), GPIO1 (CS)
- Backlight: GPIO5 (PWM, ledc channel 0)

### Touch
- CST816S on I2C (GPIO7 SDA, GPIO8 SCL)
- Interrupt: GPIO14
- Reset: GPIO13

### Rotary Encoder
- EC11 on GPIO9 (A), GPIO10 (B), GPIO11 (switch)
- Mechanical detent: 20 pulses/revolution

### LED Ring
- WS2812 x5 on GPIO48
- GRB color order (hardware test pending)
- 50% max brightness for door-side use

## Font Requirements

| Font ID | Family | Size | Glyphs | Purpose |
|---------|--------|------|---------|---------|
| font_power_on | Roboto Bold | 64pt | `ON` + space | Power ON label |
| font_primary | Roboto | 48pt | `0123456789%OFf ` | Power OFF + brightness |
| font_small | Roboto | 16pt | Full alpha + digits | Preset names |

Note: Separate 64pt font for "ON" only (2 glyphs) keeps memory minimal.

## Scripts Architecture

| Script | Trigger | Action |
|--------|---------|--------|
| `guarded_power_toggle` | Touch on power_touch_area, knob press | Wake-check → HA toggle → update UI |
| `guarded_swipe_left` | Swipe left gesture | Wake-check → show next page |
| `guarded_swipe_right` | Swipe right gesture | Wake-check → show prev page |
| `guarded_brightness_adjust` | Knob CW/CCW on brightness page | Wake-check → HA brightness call |
| `guarded_preset_apply` | Tap on preset tile | Wake-check → HA preset call |
| `update_power_display` | HA state change | Update bg_color, label text, label color |
| `reset_sleep_timer` | Any interaction | Context-aware timeout (30s/60s) |
| `enter_sleep` | Timer expiry | Dim backlight, set sleep flag |

## State Management

```cpp
globals:
  - id: is_asleep
    type: bool
    initial_value: 'true'
  - id: current_page
    type: int
    initial_value: '0'
  - id: current_brightness
    type: int
    initial_value: '0'
  - id: light_is_on
    type: bool
    initial_value: 'false'
  - id: active_preset
    type: int
    initial_value: '0'
```

## Home Assistant Integration

```yaml
text_sensor:
  - platform: homeassistant
    entity_id: light.bedroom_group
    attribute: state
    id: ha_light_state

sensor:
  - platform: homeassistant
    entity_id: light.bedroom_group
    attribute: brightness
    id: ha_light_brightness
```

## Performance Optimization

1. **Power page render:** Single obj + single label = ~1ms render time
2. **Color transition:** bg_color change triggers single full-screen redraw
3. **No continuous animation:** All animations are event-triggered (toggle only)
4. **Font memory:** 64pt font with only 2 glyphs ("O", "N") = ~8KB
5. **Touch latency target:** < 100ms from touch to HA service call

## Known Limitations

1. LVGL style transitions may not be available in ESPHome's LVGL integration — fallback to immediate color change
2. Radial wipe animation is v2 scope — v1-expanded uses simple fade/immediate
3. Tap vs swipe on full-screen obj relies on LVGL's built-in gesture handling
4. 64pt font requires careful glyph subsetting to avoid memory bloat
5. Shorter sleep timeout on Power page requires page-aware timer logic
