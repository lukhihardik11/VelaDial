# Concept 08: Apple Watch Complications — Implementation Notes

## 1. LVGL Widget Inventory

| Widget | Count | Purpose |
|--------|-------|---------|
| `label` | 17 | Center values, complications, preset name, headers |
| `obj` | 9 | Page indicator dots (3 per page) |
| **Total** | **26** | Moderate complexity |

Compared to other concepts:
- Concept 07 (Text-First): 23 widgets
- Concept 08 (Complications): 26 widgets
- Concept 05 (Preset Ring): ~35 widgets

The widget count is manageable because complications are simple labels, not complex containers.

---

## 2. Font Configuration

```yaml
font:
  - file: "gfonts://Roboto"
    id: roboto48
    size: 48
    glyphs: "0123456789%ONF "
  - file: "gfonts://Roboto"
    id: roboto24
    size: 24
  - file: "gfonts://Roboto"
    id: roboto16
    size: 16
  - file: "gfonts://Roboto"
    id: roboto14
    size: 14
```

### Font Size Rationale

- **48pt:** Primary center value (reduced from 56pt to leave room for complications above)
- **24pt:** Active preset name
- **16pt:** Page headers
- **14pt:** Complications (minimum readable at 190 PPI per research)

---

## 3. Complication Label Pattern

Each complication is a single LVGL label positioned at the top corners:

```yaml
# Top-left complication (WiFi)
- label:
    id: wifi_complication
    align: TOP_LEFT
    x: 40
    y: 30
    text_font: roboto14
    text_color: 0x888888
    text: "W --dBm"

# Top-right complication (Lux)
- label:
    id: lux_complication
    align: TOP_RIGHT
    x: -40
    y: 30
    text_font: roboto14
    text_color: 0x888888
    text: "L --lx"
```

### Safe Area Positioning

On a 240px circle, the top corners become visible at approximately:
- Top-left safe: x:35, y:30 (inside inscribed area)
- Top-right safe: x:205, y:30 (using negative offset from right)

The y:30 position ensures the complication text is below the bezel curve.

---

## 4. Sensor Configuration for Compile

Since this is a prototype for compile testing, sensors are simulated:

```yaml
sensor:
  - platform: wifi_signal
    id: wifi_rssi
    name: "WiFi Signal"
    update_interval: 30s
    on_value:
      then:
        - lvgl.label.update:
            id: wifi_complication
            text: !lambda |-
              static char buf[12];
              snprintf(buf, sizeof(buf), "W %ddBm", (int)x);
              return buf;

  - platform: template
    id: ambient_lux
    name: "Ambient Light"
    unit_of_measurement: "lx"
    update_interval: 10s
    lambda: 'return 340.0;'
    on_value:
      then:
        - lvgl.label.update:
            id: lux_complication
            text: !lambda |-
              static char buf[12];
              snprintf(buf, sizeof(buf), "L %dlx", (int)x);
              return buf;
```

The `wifi_signal` sensor is real (built-in to ESP32). The `ambient_lux` is a template sensor returning a fixed value for compile — on hardware, this would be replaced with a TSL2591 I2C sensor.

---

## 5. Complication Update Strategy

- **WiFi RSSI:** Updated every 30s via `wifi_signal` platform sensor
- **Ambient Lux:** Updated every 10s via template sensor (simulated)
- **Label updates:** Triggered by `on_value` callbacks — no polling loop needed
- **Performance:** Updating 2 labels every 10-30s has negligible CPU impact

---

## 6. Page Structure

All 3 pages share the same complication labels at the top. In LVGL, each page has its own copy of the complication labels (since labels are page-scoped). The sensor `on_value` callbacks update ALL copies:

```yaml
on_value:
  then:
    - lvgl.label.update:
        id: wifi_comp_p1
        text: !lambda 'return buf;'
    - lvgl.label.update:
        id: wifi_comp_p2
        text: !lambda 'return buf;'
    - lvgl.label.update:
        id: wifi_comp_p3
        text: !lambda 'return buf;'
```

This ensures complications show current data regardless of which page is active.

---

## 7. Knob Navigation (Same as Concept 07)

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
  CW → cycle preset forward
  CCW → cycle preset backward
  Press → apply preset, return to Page 1
```

---

## 8. Wake-Only-First Implementation

Identical to all previous concepts:

```cpp
if (!id(display_awake)) {
  id(display_awake) = true;
  id(back_light).turn_on().set_brightness(0.8).perform();
  id(sleep_timer).execute();
  return;  // consume input
}
```

---

## 9. Performance Expectations

| Metric | Expected |
|--------|----------|
| LVGL render time per frame | < 8ms (labels only, slightly more than Concept 07) |
| Sensor update frequency | 10-30s intervals |
| Label redraws per update | 6 (2 complications × 3 pages) |
| Memory usage | ~70KB (fonts) + ~12KB (widgets) |
| Frame rate | 30+ FPS |

---

## 10. v1 vs v2 Feature Boundary

| Feature | v1-Expanded (This Prototype) | v2 (Future) |
|---------|------------------------------|-------------|
| Complication count | 2 per page | 4-6 per page |
| Complication interaction | Display-only | Tap to expand |
| Icon type | Text character prefix | Material Design Icon font |
| Color coding | Static gray | Dynamic (green/amber/red) |
| Data sources | WiFi + lux | + temp, humidity, uptime, HA entities |
| LED ring mapping | Binary (on/off) | Temperature color gradient |
| Sensor detail page | Not included | Tap complication → full screen |

---

## 11. Compile-Proven Patterns Used

From Concepts 01-07:
- Hardware config (SPI, display, touchscreen, encoder, button)
- WiFi/API/OTA configuration
- Global variables structure
- Sleep timer script
- Wake-only-first lambda pattern
- Home Assistant text_sensor import
- LED ring configuration
- `lvgl.page.show` for page navigation
- Font loading from Google Fonts
- Page indicator dots using `obj` widgets
