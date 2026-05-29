# Concept 20: Eclipse Corona — Implementation Notes

## 1. The "Concentric Glow" Strategy

Rendering a smooth radial gradient (a corona) is natively difficult in LVGL without using large pre-rendered images (which consume massive flash memory). To keep this prototype compile-safe and lightweight, we use the **Concentric Glow Strategy**.

We stack multiple `lv_obj` circles on top of each other, all centered on the screen.
- The outermost circle is the largest but has the lowest opacity (e.g., 10%).
- The next circle is slightly smaller with higher opacity (e.g., 20%).
- This continues inward until the innermost glowing ring (100% opacity).
- Finally, a solid black circle (the "moon disk") is placed at the absolute center, covering the middle of the glow rings.

This creates a convincing radial falloff effect that looks like a glowing corona extending from behind a dark sphere.

## 2. Asymmetric Streamers

A perfect circle looks like a neon sign, not a solar corona. Real coronas have streamers (spikes of plasma).
To simulate this without images, we overlay several `lv_arc` widgets on top of the concentric circles.
- These arcs have wide `arc_width` values (e.g., 30-50px).
- They do not form full circles (e.g., start angle 45, end angle 135).
- They have low opacity (e.g., 30%).
- By placing 3-4 of these arcs at different angles and radii, we break the perfect symmetry and create the illusion of organic coronal streamers.

## 3. Brightness Mapping

The brightness percentage (0-100%) drives the dimensions and opacities of the corona elements.
- **Radius:** At 100%, the outermost glow ring reaches the edge of the display (radius 120). At 10%, it shrinks to just slightly larger than the moon disk.
- **Opacity:** The overall opacity of the streamer arcs scales with brightness.
- **Color:** We can shift the color from deep amber (`0xFF8C00`) at low brightness to brilliant white-gold (`0xFFD700`) at high brightness.

## 4. Page Structure in YAML

- **Page 0 (Power):** Contains the central moon disk and a single, very faint outer ring when OFF. When ON, it shows a brighter ring.
- **Page 1 (Brightness):** Contains the full stack of concentric glow rings, the streamer arcs, the central moon disk, and the percentage label.
- **Page 2 (Presets):** Contains four smaller instances of the moon disk + glow ring combo, arranged in a cross pattern. Invisible touch buttons overlay these zones to trigger the presets.

## 5. Performance Considerations

Stacking 6-8 semi-transparent objects requires the ESP32-S3 to blend multiple layers per pixel.
- **Mitigation:** We do not animate these rings continuously. They only update when the rotary encoder is turned.
- **Static Background:** The screen background is solid black (`0x000000`), which is fast to render.
- **No Shadows:** We avoid using LVGL's `shadow_width` property, as box-shadow calculations are computationally expensive. The concentric circles achieve a better glow effect with less CPU overhead.

## 6. LED Ring Integration

The 5 WS2812 LEDs on the Elecrow board are critical to this concept. They act as the outermost reaches of the corona, projecting light onto the wall behind the device.
- The LED brightness should scale non-linearly with the screen brightness to create a seamless transition from the digital screen to the physical wall.
