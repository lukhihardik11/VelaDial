# UI Concept 17: Iris Aperture Mechanism — Design Specification

## 1. Concept Classification

| Attribute | Value |
|-----------|-------|
| **Concept Number** | 17 |
| **Name** | Iris Aperture Mechanism |
| **Novelty Level** | 10/10 (High-Novelty Stretch Concept) |
| **Primary Metaphor** | Optical/Mechanical Aperture mapping to brightness |
| **Hardware Target** | ELECROW CrowPanel 1.28" ESP32-S3 Rotary Touch Display |
| **HA Target** | `light.bedroom_group` |

## 2. Design Philosophy & Core Metaphor

The Iris Aperture Mechanism concept maps brightness to the physical opening of a camera lens aperture. The round display acts as the lens barrel, and the UI renders the overlapping mechanical blades of the iris.
- **100% Brightness = Wide Open (f/1.4):** The blades are fully retracted, revealing a large, bright amber circle (the "light" entering the lens).
- **50% Brightness = Mid-Aperture (f/4):** The blades form a distinct polygon (e.g., hexagon or octagon), partially obscuring the light.
- **10% Brightness = Stopped Down (f/16):** The blades are nearly closed, leaving only a tiny pinhole of light at the center.
- **OFF = Fully Closed:** The blades converge completely at the center, blocking all light.

Why an iris aperture for lighting? An aperture literally controls the amount of light passing through a lens. It is the most mechanically coherent metaphor possible for a rotary dial controlling brightness. Turning the physical knob feels exactly like turning the aperture ring on a manual camera lens (e.g., Leica, Zeiss). It elevates the device from a "smart switch" to a "precision optical instrument."

## 3. Visual Identity & Color Palette

The aesthetic reference is high-end photography equipment (Leica, Hasselblad) and luxury mechanical watches. It must feel precise, metallic, and engineered.

| Element | Color / Texture | Purpose |
|---------|-----------------|---------|
| **Iris Blades** | Dark Metallic Gray (`0x222222` to `0x444444`) | Represents the physical mechanism. Must have subtle contrast to show overlap. |
| **Aperture Opening** | Warm Amber (`0xFFB000`) | Represents the light source behind the blades. |
| **Typography** | Light Gray (`0xAAAAAA`) | Technical, precise, high-legibility (Roboto Mono). |
| **Background (Closed)** | Deep Charcoal (`0x111111`) | The dormant state of the mechanism. |

## 4. Screen Architecture

The UI adheres to the locked v1 three-page structure, but with a highly distinctive visual execution.

### Page 1: Power (The Shutter State)
- **Visualization:** Fully closed iris (OFF) vs. partially open iris (ON).
- **Center Element:** Power icon visible in the center (when closed) or through the aperture (when open).
- **Interaction:** Tap center to toggle power. Knob press toggles power.

### Page 2: Brightness (The Optical Hero)
- **Visualization:** Iris opening diameter is proportional to brightness. As brightness increases, the blades retract outward.
- **Center Element:** `%` value visible through the aperture opening.
- **Interaction:** Rotate knob to open/close iris (adjust brightness). Press knob to return to Power page.

### Page 3: Presets (Aperture Modes)
- **Visualization:** Four distinct "f-stop" or aperture modes.
- **Center Element:** Active mode highlighted.
- **Interaction:** Tap to select. Press knob to apply highlighted preset.

## 5. Interaction Model

| Action | Power Page | Brightness Page | Presets Page |
|--------|------------|-----------------|--------------|
| **Knob Rotate CW** | No action | Open Iris (increase brightness) | Next preset |
| **Knob Rotate CCW** | No action | Close Iris (decrease brightness) | Previous preset |
| **Knob Press** | Toggle Power | Return to Power | Apply Preset |
| **Screen Tap** | Toggle Power | No action | Select Preset |
| **Screen Swipe** | Next/Prev Page | Next/Prev Page | Next/Prev Page |

## 6. Sleep/Wake Behavior

- **Wake-only-first enforced:** The first interaction only wakes the screen.
- **Wake Animation (Opening):** The iris blades smoothly retract from fully closed to the current brightness level over 400ms.
- **Sleep Animation (Closing):** The iris blades smoothly converge to the center, completely closing the aperture over 400ms, followed by the backlight fading out. This mimics a mechanical shutter closing.

## 7. LED Ring Behavior

The 5-LED WS2812 ring acts as the "lens rim" or "light leak" around the aperture housing.

| Brightness | LED Color | LED Intensity |
|------------|-----------|---------------|
| **100%** | Warm Amber | 100% |
| **50%** | Medium Amber | 50% |
| **10%** | Dim Amber | 10% |
| **OFF** | OFF | 0% |

## 8. Dark-Room & Daylight Usability

- **Dark-Room:** Excellent. When stopped down (low brightness), the dark metallic blades block most of the screen, leaving only a small, dim amber polygon in the center. This produces minimal light pollution.
- **Daylight:** Good. The high contrast between the dark metallic blades and the bright amber aperture opening ensures readability.

## 9. Implementation Strategy & Visual Compromises (LVGL)

True rotating, overlapping polygon blades with dynamic shadows are extremely difficult to render in real-time using pure LVGL 9.x YAML on an ESP32-S3.
- **The Abstraction:** We will approximate the iris mechanism using a series of overlapping `lv_arc` widgets or `lv_line` segments arranged radially to form a polygon (e.g., an octagon).
- **The "Compile-Safe" Approach:** For the v1 prototype, we will use a central expanding/contracting circle (the aperture) surrounded by a dark mask (the blades). To give it the "iris" feel, we will overlay static lines radiating outward from the aperture edge to simulate the blade edges.
- **Visual Compromise:** The prototype will not have true rotating blade animation. Instead, the central aperture will scale in size, and the "blade edges" will adjust to match. This captures the essence of the aperture opening/closing without the severe performance penalty of real-time polygon rotation.

## 10. v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | Abstracted scaling aperture with static blade edge overlays, simple power state, standard presets. |
| **v1-expanded** | Pre-rendered image sequence of a true 3D mechanical iris opening/closing (8-12 frames). |
| **v2** | Real-time polygon rendering of rotating blades with dynamic shadows, exposure triangle visualization. |
