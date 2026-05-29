# Concept 18: Radar Sweep Animation — Implementation Notes

## 1. The Core Challenge: Continuous Animation on ESP32-S3

The defining feature of a radar sweep is continuous, smooth rotation. However, continuous animation is the most expensive rendering pattern for LVGL on an ESP32-S3, especially when combined with real-time input handling (rotary encoder) and Home Assistant communication.

**Risks of True Continuous Animation:**
- **Frame Drops:** The ESP32-S3 may struggle to maintain 30 FPS while redrawing a large rotating element and handling Wi-Fi/HA traffic.
- **Input Lag:** High CPU load from rendering can delay the processing of rotary encoder interrupts, making the knob feel sluggish or unresponsive.
- **Memory Bandwidth:** Pushing full-screen updates constantly stresses the SPI bus to the GC9A01 display.

## 2. LVGL-Safe Approximations for the Radar Sweep

To ensure a compile-passing, performant prototype, we must approximate the radar sweep using LVGL primitives that minimize redraw overhead.

### Approach A: The "Arc Trail" (Chosen for Prototype)
Instead of a rotating line, we use an `lv_arc` to represent the "trail" of the radar sweep.
- **Mechanism:** The arc's `value` (end angle) is tied to the brightness percentage.
- **Visual Effect:** At 100% brightness, the arc forms a nearly complete circle (a full 360° sweep trail). At 5% brightness, it forms a tiny sliver (a short sweep trail).
- **Why it works:** This avoids continuous animation entirely. The arc only updates when the user turns the knob. It provides the *aesthetic* of a radar scope (circular, technical, glowing) without the performance penalty of constant rotation.
- **Implementation:** A background `lv_arc` (faint, full circle) acts as the range ring. A foreground `lv_arc` (bright amber) acts as the sweep trail.

### Approach B: The "Stepped Sweep" (Alternative/Future)
If actual motion is strictly required, we can use a timer script to update the position of an element at a low frame rate (e.g., 5-10 FPS).
- **Mechanism:** An ESPHome `interval` or `script.execute` loop increments an angle variable. An `lv_arc` or `lv_line` updates its position based on this angle.
- **Why it's risky:** Even at low frame rates, constant updates can interfere with knob responsiveness. It also requires complex trigonometry in lambdas to calculate line endpoints if using `lv_line`.

### Approach C: The "Rotating Image" (Alternative/Future)
- **Mechanism:** Create a PNG image of a sweep gradient with a transparent background. Use LVGL's image rotation feature (`angle` property).
- **Why it's risky:** Image rotation in LVGL is computationally expensive on MCUs without dedicated 2D graphics hardware. It often results in severe frame drops.

## 3. Prototype Implementation Details (Approach A)

The prototype YAML (`door_side_concept_18_radar_sweep_animation.yaml`) implements the **Arc Trail** approximation to guarantee compilation and performance safety.

### Page 1: Brightness Hero (The Radar Scope)
- **Range Rings:** Three concentric `lv_obj` circles with thin (1px or 2px) dark gray borders (`0x333333`) provide the structural grid of the radar scope.
- **The Sweep Trail:** A large `lv_arc` (`id: sweep_arc`) sits on top of the range rings.
  - `arc_color`: `0xFFB000` (Warm Amber)
  - `arc_width`: 15px (thick enough to be visible, thin enough to look technical)
  - `value`: Dynamically bound to `id(brightness_pct)`.
- **Center Data:** A `Roboto Mono` label displays the brightness percentage in the center, reinforcing the instrument aesthetic.

### Page 0: Power (Scanner State)
- **Visual:** A simplified radar scope.
- **ON State:** A static "blip" (amber circle) appears in the center, and the text reads "SCAN ACTIVE".
- **OFF State:** The blip is dark gray, and the text reads "SCAN OFFLINE".

### Page 2: Presets (Target Acquisition)
- **Visual:** A 2x2 grid representing radar targets.
- **Elements:** Four buttons, each styled like a radar blip with a label (Warm, Amber, Neutral, Night).
- **Active State:** The selected preset button highlights in bright amber, simulating a "locked target."

## 4. Typography and Color Palette

- **Font:** `Roboto Mono` is used exclusively for numbers and status text to evoke a technical, computerized instrument panel.
- **Colors:**
  - Background: `0x111111` (Deep Charcoal)
  - Range Rings: `0x333333` (Dark Gray)
  - Sweep/Active Elements: `0xFFB000` (Warm Amber)
  - Inactive Text: `0x777777` (Medium Gray)

## 5. LED Ring Synchronization (V1 Expanded)

In a production environment with custom C++ components, the 5-LED WS2812 ring could be programmed to "chase" in a circle, simulating the physical rotation of the radar sweep. For this YAML prototype, we use the standard proportional brightness approach to ensure compile safety and adherence to v1 constraints.

## 6. Summary of Compromises

1. **No Continuous Rotation:** The prototype uses a static, adjustable arc trail rather than a continuously rotating sweep line to ensure UI responsiveness and compile safety.
2. **No Phosphor Decay Gradient:** Pure LVGL YAML cannot easily render a conical gradient (fading trail). The arc is a solid color.
3. **Simplified Geometry:** True radar scopes have complex graticules and crosshairs. We use simple concentric circles to maintain a clean, residential aesthetic and minimize rendering load.
