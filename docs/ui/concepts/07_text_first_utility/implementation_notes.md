# Concept 07: Text-First Utility — Implementation Notes

## 1. LVGL Widget Inventory

This concept uses the absolute minimum widget set — only `lv_label` and `lv_obj` (for page dots):

| Widget | Count | Purpose |
|--------|-------|---------|
| `label` | 14 | All text content across 3 pages |
| `obj` | 9 | Page indicator dots (3 per page) |
| **Total** | **23** | Lightest widget count in the 20-concept matrix |

No arcs, no buttons, no images, no custom drawing. This is the simplest possible LVGL implementation.

---

## 2. Font Configuration

```yaml
font:
  - file: "gfonts://Roboto"
    id: roboto56
    size: 56
    glyphs: "0123456789%ONOFF "
  - file: "gfonts://Roboto"
    id: roboto24
    size: 24
  - file: "gfonts://Roboto"
    id: roboto16
    size: 16
  - file: "gfonts://Roboto"
    id: roboto12
    size: 12
```

### Memory Considerations

- 56pt with limited glyphs: ~15KB
- 24pt full ASCII: ~25KB
- 16pt full ASCII: ~12KB
- 12pt full ASCII: ~8KB
- **Total font memory:** ~60KB (well within ESP32-S3 PSRAM capacity)

---

## 3. Page Structure (LVGL YAML)

### Power Page
```yaml
- id: power_page
  bg_color: 0x000000
  bg_opa: COVER
  widgets:
    - label:  # "Power" header
        align: TOP_MID
        y: 50
        text_font: roboto16
        text_color: 0x888888
        text: "Power"
    - label:  # Primary value
        id: power_label
        align: CENTER
        text_font: roboto56
        text_color: 0x666666
        text: "OFF"
    - obj: # dot container with 3 dot children
```

### Brightness Page
```yaml
- id: brightness_page
  bg_color: 0x000000
  bg_opa: COVER
  widgets:
    - label:  # "Brightness" header
    - label:  # Primary percentage value
        id: brightness_label
    - label:  # Step indicator "±5%"
    - obj:    # dot container
```

### Presets Page
```yaml
- id: presets_page
  bg_color: 0x000000
  bg_opa: COVER
  widgets:
    - label:  # "Presets" header
    - label:  # Preset 1 (Warm White)
        id: preset_1_label
    - label:  # Preset 2 (Soft Amber)
        id: preset_2_label
    - label:  # Preset 3 (Neutral White)
        id: preset_3_label
    - label:  # Preset 4 (Low Nightlight)
        id: preset_4_label
    - obj:    # dot container
```

---

## 4. Preset Highlight Logic

The preset page uses a `selected_preset` global (0-3) to track which item is highlighted:

```cpp
// On knob CW (Presets page):
id(selected_preset) = (id(selected_preset) + 1) % 4;
id(update_preset_highlight).execute();

// update_preset_highlight script:
// Set all labels to gray, then set selected to amber
```

Each preset label update:
```yaml
- lvgl.label.update:
    id: preset_1_label
    text_color: !lambda |-
      return (id(selected_preset) == 0) ? lv_color_hex(0xFFA500) : lv_color_hex(0x555555);
```

---

## 5. Knob Navigation Pattern

```
Page 1 (Power):
  CW → show Page 2
  CCW → show Page 3
  Press → toggle power

Page 2 (Brightness):
  CW → brightness +5%
  CCW → brightness -5%
  Press → return to Page 1

Page 3 (Presets):
  CW → highlight next preset
  CCW → highlight previous preset
  Press → apply selected preset, return to Page 1
```

---

## 6. Wake-Only-First Implementation

Same pattern as all previous concepts:

```cpp
if (!id(display_awake)) {
  id(display_awake) = true;
  id(back_light).turn_on().set_brightness(0.8).perform();
  id(sleep_timer).execute();
  return;  // consume input, no action
}
```

---

## 7. Text Update Patterns

### Power Toggle
```yaml
- lvgl.label.update:
    id: power_label
    text: !lambda 'return id(light_is_on) ? "ON" : "OFF";'
    text_color: !lambda 'return id(light_is_on) ? lv_color_hex(0xFFFFFF) : lv_color_hex(0x666666);'
```

### Brightness Update
```yaml
- lvgl.label.update:
    id: brightness_label
    text: !lambda |-
      static char buf[8];
      snprintf(buf, sizeof(buf), "%d%%", id(brightness_pct));
      return buf;
```

---

## 8. Performance Expectations

| Metric | Expected |
|--------|----------|
| LVGL render time per frame | < 5ms (text only) |
| Memory usage (widgets) | < 10KB |
| Font memory | ~60KB |
| Total LVGL memory | < 100KB |
| Frame rate | 30+ FPS (trivial for text) |

Text-First Utility will be the fastest-rendering concept in the entire matrix.

---

## 9. Compile-Proven Patterns Used

From Concepts 01-06, the following patterns are reused exactly:
- Hardware config (SPI, display, touchscreen, encoder, button)
- WiFi/API/OTA configuration
- Global variables structure
- Sleep timer script
- Wake-only-first lambda pattern
- Home Assistant text_sensor import
- LED ring configuration
- `lvgl.page.show` for page navigation (NOT `lv_disp_load_scr`)

---

## 10. Key Differences from Other Concepts

| Aspect | Text-First Utility | Other Concepts |
|--------|-------------------|----------------|
| Visual elements | Labels only | Arcs, buttons, rings |
| Widget count | 23 | 30-50+ |
| Decorative elements | Zero | Shadows, glows, gradients |
| Information density | High (text is dense) | Medium (graphical) |
| Implementation complexity | Very Easy | Easy to Medium |
| Render performance | Fastest | Fast |
| Daylight readability | Best | Good to Excellent |
| Dark-room readability | Excellent | Good to Excellent |
