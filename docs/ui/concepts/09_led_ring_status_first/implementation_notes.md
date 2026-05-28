# Concept 09: LED-Ring Status-First — Implementation Notes

**Date:** 2026-05-28  
**Status:** CONCEPT PROTOTYPE  
**Compile Target:** ESPHome 2026.5.0, ESP-IDF framework

---

## 1. LED Ring Hardware Configuration

```yaml
light:
  - platform: esp32_rmt_led_strip
    id: led_ring
    name: "LED Ring"
    pin: GPIO48
    num_leds: 5
    rgb_order: GRB          # HARDWARE TEST PENDING
    chipset: WS2812
    restore_mode: ALWAYS_OFF
    default_transition_length: 300ms
```

**Key decisions:**
- `GPIO48` — source confirmed from ELECROW schematic
- `GRB` color order — assumed from production YAML, HARDWARE TEST PENDING
- `ALWAYS_OFF` restore — prevents LED flash on boot
- `300ms` transition — smooth enough for ambient feedback, fast enough for responsiveness

---

## 2. LED Brightness Ceiling

The LED ring maximum brightness is capped at 50% (127/255) for bedroom safety. This is enforced in the update scripts, not in the light component configuration (which allows full range for error patterns).

```yaml
globals:
  - id: led_brightness_ceiling
    type: float
    initial_value: '0.50'
```

The actual LED brightness is calculated as:
```
led_brightness = light_brightness * led_brightness_ceiling
```

Where `light_brightness` is the current brightness of `light.bedroom_group` (0.0 to 1.0).

---

## 3. Preset Color Definitions

```yaml
globals:
  - id: preset_color_r
    type: int
    initial_value: '255'
  - id: preset_color_g
    type: int
    initial_value: '179'
  - id: preset_color_b
    type: int
    initial_value: '0'
```

Color lookup table (applied via script):

| Preset Index | R | G | B | Name |
|-------------|---|---|---|------|
| 0 | 255 | 179 | 0 | Warm White (#FFB300) |
| 1 | 204 | 136 | 0 | Soft Amber (#CC8800) |
| 2 | 224 | 224 | 224 | Neutral White (#E0E0E0) |
| 3 | 255 | 179 | 0 | Low Nightlight (same color, 10% brightness) |

---

## 4. LED Update Script Architecture

The LED ring state is updated by a single script (`update_led_ring`) called from:
- Power toggle handler
- Brightness change handler
- Preset change handler
- HA state import callback
- Wake-from-sleep handler

```yaml
script:
  - id: update_led_ring
    mode: restart
    then:
      - if:
          condition:
            lambda: 'return !id(light_is_on);'
          then:
            - light.turn_off:
                id: led_ring
                transition_length: 400ms
          else:
            - light.addressable_set:
                id: led_ring
                range_from: 0
                range_to: 4
                red: !lambda 'return id(preset_color_r) / 255.0;'
                green: !lambda 'return id(preset_color_g) / 255.0;'
                blue: !lambda 'return id(preset_color_b) / 255.0;'
            - light.turn_on:
                id: led_ring
                brightness: !lambda 'return id(brightness_pct) / 100.0 * id(led_brightness_ceiling);'
                transition_length: 200ms
```

---

## 5. Error State LED Patterns

Error states override normal LED behavior:

```yaml
script:
  - id: led_error_pattern
    mode: restart
    then:
      - while:
          condition:
            lambda: 'return id(ha_unavailable);'
          then:
            - light.addressable_set:
                id: led_ring
                range_from: 0
                range_to: 4
                red: 0.6
                green: 0.0
                blue: 0.0
            - light.turn_on:
                id: led_ring
                brightness: 0.15
            - delay: 1000ms
            - light.turn_off:
                id: led_ring
                transition_length: 100ms
            - delay: 1000ms
```

---

## 6. Screen UI (Secondary — Minimal)

The screen uses the absolute minimum LVGL widgets:

| Page | Widgets | Total |
|------|---------|-------|
| Power | 1 label (`ON`/`OFF`) | 1 |
| Brightness | 1 label (`75%`) | 1 |
| Presets | 1 label (`Warm White`) | 1 |
| All pages | 3 page dots | 3 |
| **Total** | | **6** |

This is the lightest screen UI in the entire 20-concept matrix — even lighter than Concept 07 (Text-First Utility, 23 widgets).

---

## 7. LED Ring Extended Timeout

After screen sleep, the LED ring remains active for 30 additional seconds:

```yaml
script:
  - id: screen_sleep
    then:
      - lambda: 'id(display_awake) = false;'
      - light.turn_off:
          id: backlight
          transition_length: 500ms
      - delay: 30s
      - light.turn_off:
          id: led_ring
          transition_length: 2000ms
```

If the user wakes the screen during the 30s window, the LED ring remains active (script restart cancels the fadeout).

---

## 8. Low-Brightness Single-LED Mode

When light brightness is ≤ 10%, only LED 0 (assumed top position, HARDWARE TEST PENDING) illuminates at minimum visible brightness:

```yaml
# In update_led_ring script
- if:
    condition:
      lambda: 'return id(brightness_pct) <= 10;'
    then:
      - light.addressable_set:
          id: led_ring
          range_from: 0
          range_to: 0
          red: !lambda 'return id(preset_color_r) / 255.0;'
          green: !lambda 'return id(preset_color_g) / 255.0;'
          blue: !lambda 'return id(preset_color_b) / 255.0;'
      - light.addressable_set:
          id: led_ring
          range_from: 1
          range_to: 4
          red: 0
          green: 0
          blue: 0
      - light.turn_on:
          id: led_ring
          brightness: 0.05
```

---

## 9. Wake-Only-First Integration

The wake-only-first pattern is identical to all other concepts:

1. Touch/knob input detected
2. If `display_awake == false`:
   - Set `display_awake = true`
   - Turn on backlight
   - Reset LED ring to current state
   - Set `touch_woke_this_cycle = true`
   - **Block the action**
3. If `display_awake == true` and `touch_woke_this_cycle == false`:
   - Execute the action normally
   - Update LED ring via `update_led_ring` script

---

## 10. Compile-Proven Patterns Used

| Pattern | Source |
|---------|--------|
| `esp32_rmt_led_strip` config | Production `door_side_rotary.yaml` |
| `light.addressable_set` action | ESPHome docs, verified in Concept 03 |
| LVGL minimal labels | Concept 07 (Text-First Utility) |
| Wake-only-first latch | All prior concepts (01-08) |
| Script `mode: restart` | Concept 06 (Night Mode) |
| `default_transition_length` | Production YAML |

---

## 11. Known Limitations

1. **LED 0 position unknown** — physical orientation of the ring is not documented
2. **GRB order assumed** — if wrong, red and green will be swapped
3. **5 LEDs cannot show gradients** — brightness mapping is quantized
4. **No individual LED patterns in v1** — all LEDs show same color (v2 feature)
5. **Cross-fade may show color banding** — WS2812 has 8-bit color depth per channel
6. **RMT channel conflicts possible** — GPIO48 RMT timing not validated with display SPI
