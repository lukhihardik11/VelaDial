# UI Concept 16: Topographic Contour Map — Design Specification

## 1. Concept Classification

| Attribute | Value |
|-----------|-------|
| **Concept Number** | 16 |
| **Name** | Topographic Contour Map |
| **Novelty Level** | 10/10 (High-Novelty Stretch Concept) |
| **Primary Metaphor** | Cartography / Elevation mapping to brightness |
| **Hardware Target** | ELECROW CrowPanel 1.28" ESP32-S3 Rotary Touch Display |
| **HA Target** | `light.bedroom_group` |

## 2. Design Philosophy & Core Metaphor

The Topographic Contour Map concept maps brightness to elevation. The round display acts as a "window" looking down at a terrain from above.
- **100% Brightness = Mountain Peak:** Dense, closely spaced contour lines filling the display, with a warm "summit glow" at the center.
- **50% Brightness = Hillside:** Moderate contour density, representing a gentle slope.
- **10% Brightness = Valley Floor:** Sparse, widely spaced contour lines, cool and dim.
- **OFF = Flat Plain:** No contours, just a dark, dormant background.

Why topography for lighting? Topographic maps are associated with exploration, precision, and the outdoors. They are inherently beautiful, information-dense, and deeply tied to circular forms (when viewed from above). This concept elevates a utilitarian light switch into a piece of interactive cartographic art, appealing to users who appreciate premium outdoor gear (e.g., Garmin, Suunto, Arc'teryx) and technical aesthetics.

## 3. Visual Identity & Color Palette

The aesthetic reference is a premium, dark-mode hiking trail map or a luxury outdoor watch UI. It must feel technical, precise, and architectural — not like a generic circular progress chart.

| State | Background | Contour Lines | Summit/Center Glow |
|-------|------------|---------------|--------------------|
| **100% (Peak)** | Dark Charcoal (`0x111111`) | Bright Amber (`0xFFB000`) | Intense Warm Amber |
| **50% (Hill)** | Very Dark Gray (`0x0A0A0A`) | Medium Amber (`0xAA7000`) | Soft Amber |
| **10% (Valley)** | Near Black (`0x050505`) | Dim Amber (`0x553800`) | None |
| **OFF (Plain)** | Pure Black (`0x000000`) | None | None |

## 4. Screen Architecture

The UI adheres to the locked v1 three-page structure, but with a highly distinctive visual execution.

### Page 1: Power (The Terrain State)
- **Visualization:** Peak (ON) vs. Flat Plain (OFF).
- **Center Element:** Small summit marker icon (triangle or cross).
- **Interaction:** Tap center to toggle power. Knob press toggles power.

### Page 2: Brightness (The Elevation Hero)
- **Visualization:** Contour density proportional to brightness. As brightness increases, new contour lines emerge from the center and push outward, "raising the mountain."
- **Center Element:** `%` value at the summit, using a technical, monospaced font if possible.
- **Interaction:** Rotate knob to raise/lower terrain (adjust brightness). Press knob to return to Power page.

### Page 3: Presets (Terrain Zones)
- **Visualization:** Four distinct terrain "regions" or zones.
- **Center Element:** Active region highlighted with a specific contour color.
- **Interaction:** Tap to select. Press knob to apply highlighted preset.

## 5. Interaction Model

| Action | Power Page | Brightness Page | Presets Page |
|--------|------------|-----------------|--------------|
| **Knob Rotate CW** | No action | Raise terrain (add contours/brightness) | Next preset |
| **Knob Rotate CCW** | No action | Lower terrain (remove contours/brightness) | Previous preset |
| **Knob Press** | Toggle Power | Return to Power | Apply Preset |
| **Screen Tap** | Toggle Power | No action | Select Preset |
| **Screen Swipe** | Next/Prev Page | Next/Prev Page | Next/Prev Page |

## 6. Sleep/Wake Behavior

- **Wake-only-first enforced:** The first interaction only wakes the screen.
- **Wake Animation (Emergence):** Contour lines emerge from the center outward to the current elevation level over 500ms, like a mountain rising from the sea.
- **Sleep Animation (Submergence):** Contour lines retract inward and fade to black over 500ms.

## 7. LED Ring Behavior

The 5-LED WS2812 ring acts as the "outer elevation boundary" or horizon line.

| Brightness | LED Color | LED Intensity |
|------------|-----------|---------------|
| **100%** | Warm Amber | 100% |
| **50%** | Medium Amber | 50% |
| **10%** | Dim Amber | 10% |
| **OFF** | OFF | 0% |

## 8. Dark-Room & Daylight Usability

- **Dark-Room:** Excellent. Thin contour lines on a dark background produce minimal light pollution. At low brightness, only a few widely-spaced, dim lines are visible.
- **Daylight:** Good. The high-contrast amber-on-charcoal palette ensures the contour density is readable in bright light.

## 9. Implementation Strategy & Visual Compromises (LVGL)

True, irregular, organic topographic contour lines are extremely difficult to render dynamically in LVGL 9.x on an ESP32-S3 without pre-rendered images.
- **The Abstraction:** We will approximate contour lines using a series of concentric, slightly offset `lv_arc` widgets or `lv_obj` circles with borders.
- **Avoiding the "Target" Look:** To prevent the UI from looking like a bullseye or Concept 15 (Tree Rings), we will use varying border widths, slightly offset centers (if possible), and a distinct cartographic color palette. The lines will be thinner and more numerous than the tree rings.
- **Visual Compromise:** The prototype will use perfect circles (or arcs) rather than wavy, irregular lines. The "topographic" feel will rely heavily on the density, thinness, and color of the lines, along with the summit marker metaphor.

## 10. v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | Abstracted concentric contour lines for Brightness, simple power state, standard presets. |
| **v1-expanded** | Pre-rendered images of true irregular topographic maps, smooth emergence animation. |
| **v2** | Weather contour page (barometric pressure), real terrain data integration. |
