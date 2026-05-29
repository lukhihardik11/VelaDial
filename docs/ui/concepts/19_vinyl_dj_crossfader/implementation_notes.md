# Concept 19: Vinyl DJ Crossfader — Implementation Notes

## 1. The Core Challenge: Rendering Vinyl Grooves on ESP32-S3

The defining visual feature of a vinyl record is the dense pattern of concentric grooves. Rendering this on a 240x240 pixel display using an ESP32-S3 presents several challenges:

**Risks of True Groove Rendering:**
- **Aliasing (Jagged Edges):** Drawing many thin, closely spaced concentric circles in LVGL often results in severe aliasing (stair-stepping) on low-resolution displays, destroying the premium aesthetic.
- **Performance Overhead:** Drawing 20+ individual `lv_arc` or `lv_obj` circles with borders consumes significant rendering time, potentially causing frame drops during UI updates.
- **Moire Patterns:** At 240x240 resolution, closely spaced concentric lines can create distracting optical illusions (Moire patterns) when the display is viewed at certain angles or when elements move.

## 2. LVGL-Safe Approximations for the Vinyl Aesthetic

To ensure a compile-passing, performant prototype that looks premium, we must approximate the vinyl grooves using LVGL primitives that minimize redraw overhead and aliasing.

### Approach A: The "Abstract Track Bands" (Chosen for Prototype)
Instead of rendering dozens of individual grooves, we represent the record using a few distinct, thicker bands (representing "tracks" on an LP).
- **Mechanism:** We use 4-5 concentric `lv_arc` widgets with moderate thickness (e.g., 10-15px) and small gaps between them.
- **Visual Effect:** This creates the unmistakable geometry of a record (concentric rings around a center label) without the aliasing issues of thin lines.
- **Why it works:** It is computationally cheap. The arcs only update when their color changes (e.g., when highlighting a preset). It avoids Moire patterns entirely.
- **Implementation:** Background arcs are dark gray (`0x222222`). The active "track" or the "needle position" is highlighted in warm amber (`0xFFB000`).

### Approach B: Pre-rendered Background Image (Alternative/Future)
- **Mechanism:** Create a high-quality PNG image of a vinyl record with perfect anti-aliasing and subtle lighting gradients. Use this as the background for the Brightness and Preset pages.
- **Why it's risky for the prototype:** It requires external asset generation and increases the firmware binary size. It also makes dynamic color changes (like highlighting a specific track) much more difficult, requiring multiple image assets or complex masking.

## 3. Prototype Implementation Details (Approach A)

The prototype YAML (`door_side_concept_19_vinyl_dj_crossfader.yaml`) implements the **Abstract Track Bands** approximation to guarantee compilation and performance safety.

### Page 1: Brightness Hero (The Tonearm/Needle)
- **The Grooves:** Four concentric `lv_arc` widgets represent the record surface. They are static and dark gray.
- **The Needle Indicator:** A single, bright amber `lv_arc` segment (e.g., a 30-degree slice) acts as the "needle."
- **Mapping:** As brightness increases, the amber needle segment moves from the innermost track to the outermost track. This is achieved by dynamically changing which of the four arcs has the amber segment visible, or by using a single adjustable arc whose radius changes (though LVGL doesn't easily support dynamic radius changes on a single arc, so toggling visibility of concentric arcs is safer).
- **Center Label:** A solid circular `lv_obj` in the center acts as the record label, displaying the brightness percentage in a clean `Roboto` font.

### Page 0: Power (The Platter State)
- **Visual:** A simplified view of the center label.
- **ON State:** The label displays a "SPINNING" or "ACTIVE" indicator.
- **OFF State:** The label displays "STOPPED".

### Page 2: Presets (Track Selection)
- **Visual:** The four concentric track bands are visible.
- **Mapping:** Each band corresponds to a preset (Outer = Preset 0, Inner = Preset 3).
- **Active State:** The currently selected preset's track band is fully illuminated in amber, while the others remain dark gray. This perfectly mimics the visual of selecting a specific track on an LP.

## 4. Typography and Color Palette

- **Font:** `Roboto` is used for a clean, modern, high-end audio equipment feel (like Bang & Olufsen). We avoid stylized "DJ" fonts to maintain a residential aesthetic.
- **Colors:**
  - Background/Platter: `0x111111` (Deep Charcoal)
  - Inactive Grooves/Tracks: `0x222222` to `0x333333` (Dark Grays)
  - Active Elements/Needle: `0xFFB000` (Warm Amber, representing analog warmth)
  - Center Label: `0x1A1A1A` (Slightly lighter than the platter to stand out)

## 5. LED Ring Synchronization (V1 Expanded)

In a production environment, the 5-LED WS2812 ring could be programmed to simulate the strobe dots on the edge of a turntable platter. A subtle chasing animation could indicate that the "record is spinning" when the light is on. For this YAML prototype, we use the standard proportional brightness approach to ensure compile safety and adherence to v1 constraints.

## 6. Summary of Compromises

1. **No Thin Grooves:** The prototype uses thick "track bands" instead of thin, dense grooves to avoid aliasing and Moire patterns on the 240x240 display.
2. **No Spinning Animation:** Pure LVGL YAML cannot easily rotate a complex group of arcs smoothly at 30 FPS. The record appears static, with state changes indicated by color highlighting rather than physical rotation.
3. **Abstract Tonearm:** Instead of drawing a literal tonearm (which would require a rotating image asset), we use an amber arc segment to represent the *position* of the needle on the grooves. This is cleaner and more performant.
