# Concept 05: Preset Ring UI — Research Findings

> 20-thread parallel internet research conducted 2026-05-27.

---

## 1. ESPHome LVGL multiple arc widgets configuration

### Key Findings

When configuring multiple LVGL arc widgets on the same page in ESPHome, especially for a circular UI like the Preset Ring, several key technical considerations emerge. First, LVGL allows stacking multiple widgets in the same container or page. To create a continuous ring of segments, you can define multiple arc widgets with the same center point and dimensions (e.g., width and height matching the display resolution, 240x240) but with different `start_angle` and `end_angle` properties. In LVGL, 0 degrees is at the 3 o'clock position (middle right), and angles increase clockwise. Therefore, to divide the ring into four quadrants, the angles would be approximately 315-45 (top), 45-135 (right), 135-225 (bottom), and 225-315 (left).

A critical finding regarding ESPHome's LVGL integration is that while updating the `value` of an arc dynamically via lambda (e.g., `lvgl.arc.update`) works reliably, attempting to update `start_angle` or `end_angle` dynamically currently fails to compile or apply correctly (as reported in ESPHome issues #12642 and community forums). Therefore, the design pattern must rely on statically defining the angles in the YAML configuration.

To manage the active state of the presets, instead of changing angles, you should manipulate the `value` or `color` of the statically defined arcs. For instance, an inactive preset could have a value of 0 (showing only the background arc, which can be styled to be transparent or dim), while the active preset has a value of 100 (filling the foreground indicator arc with its characteristic color). Additionally, to prevent unintended touch interactions if a touch screen were used, the arcs should be configured with `adjustable: false` and the knob styling removed, ensuring they act purely as visual indicators controlled by the physical rotary encoder.

### Implementation Recommendations

For the VelaDial project on a 240x240 GC9A01A display, implement the Preset Ring UI by stacking four separate arc widgets on the same page, all aligned to CENTER with width and height set to 240. Set `adjustable: false` for all arcs since navigation is handled by the rotary encoder, not touch. Configure the four arcs with distinct angle ranges: Top (315-45), Right (45-135), Bottom (135-225), and Left (225-315). To indicate the active preset, dynamically update the `value` of the arcs (e.g., 100 for active, 0 for inactive) or change their colors using `lvgl.arc.update` actions in ESPHome. Note that dynamically updating `start_angle` and `end_angle` via lambda is currently problematic in ESPHome, so it is better to define fixed angles in YAML and toggle visibility or values.

### Code/Config Examples

```
lvgl:
  pages:
    - id: presets_page
      widgets:
        - arc:
            id: preset_1
            align: CENTER
            width: 240
            height: 240
            start_angle: 315
            end_angle: 45
            value: 100
            adjustable: false
            color: 0xFF0000
        - arc:
            id: preset_2
            align: CENTER
            width: 240
            height: 240
            start_angle: 45
            end_angle: 135
            value: 100
            adjustable: false
            color: 0x00FF00
```

### Sources

https://esphome.io/components/lvgl/widgets/, https://lvgl.io/docs/open/8.3/widgets/core/arc, https://github.com/esphome/esphome/issues/12642, https://community.home-assistant.io/t/esphome-lvgl-widget-arc-update-problem/845590, https://forum.lvgl.io/t/how-can-i-stack-2-arcs-with-the-same-center-point/22199

---

## 2. Circular preset selector UI patterns and radial menu design

### Key Findings

Radial menus (also known as pie menus or circular menus) are highly effective for smartwatch and round IoT device interfaces because they align with the physical shape of the device and leverage Fitts' Law, making options equidistant from the center and easier to select. Key findings for circular UI design include:

1. **Spatial Efficiency and Aesthetics**: Circular designs naturally accommodate radial patterns. Placing options around the circumference maximizes the use of the screen edge while leaving the center open for critical information (like preset names and values).
2. **Muscle Memory and Speed**: Radial menus build muscle memory faster than linear lists. Users learn the directional location of options (e.g., top, bottom, left, right), allowing for quick, almost "eyes-free" selection, which is ideal for a rotary encoder interface.
3. **Option Limits**: Best practices suggest limiting radial menus to a maximum of 8 items to prevent clutter and ensure each segment is large enough to distinguish. The VelaDial's 4-preset design (quadrants) is optimal, providing 90-degree segments that are clearly defined.
4. **Visual Feedback**: Highlighting the active segment with a characteristic color while keeping inactive segments dimmed or transparent provides immediate, clear feedback.
5. **LVGL Implementation**: In LVGL, radial menus are typically constructed using multiple `arc` widgets or a modified pie chart. For ESPHome, using individual `arc` widgets for each segment is the most straightforward approach, allowing independent control over styling, color, and visibility.

### Implementation Recommendations

For the VelaDial project using a 240x240 GC9A01A display with ESPHome LVGL, implement the Preset Ring UI using multiple `arc` widgets overlaid on the same center point. Create four separate arcs, each spanning 90 degrees (e.g., 315-45, 45-135, 135-225, 225-315) with a thickness of 30-40 pixels to ensure they are easily visible but leave a large central area (approx 160x160 pixels) for the preset name and brightness value. 

Use `bg_opa: TRANSP` for the background of the arcs and only color the active preset's arc segment using the `color` property to indicate selection. Map the rotary encoder input to cycle through the four presets, updating the arc colors and central text dynamically. For the Brightness page, use a single continuous arc where the `end_angle` is dynamically updated based on the brightness value (0-255 mapped to 0-360 degrees). Ensure all arcs have `adv_hittest: true` if touch is added later, though navigation is primarily via the rotary encoder.

### Code/Config Examples

```
# ESPHome LVGL Arc configuration example
lvgl:
  pages:
    - id: preset_page
      widgets:
        - arc:
            id: preset_1_arc
            align: CENTER
            width: 240
            height: 240
            start_angle: 315
            end_angle: 45
            value: 100
            bg_opa: TRANSP
            color: 0xFF0000 # Active color
            width: 40 # Arc thickness
            on_click:
              - script.execute: select_preset_1
```

### Sources

https://esphome.io/cookbook/lvgl/, https://forum.lvgl.io/t/how-to-create-a-radial-menu/22429, https://www.sitepoint.com/smartwatch-ui-design-battle-circles-vs-squares/, https://bigmedium.com/ideas/radial-menus-for-touch-ui.html

---

## 3. LVGL arc segment styling in ESPHome

### Key Findings

1. **Widget Structure**: In LVGL, the `arc` widget consists of three main parts: `main` (background), `indicator` (foreground/active part), and `knob` (handle). Each part can be styled independently in ESPHome.
2. **Styling Properties**: The color of the arc is controlled by the `arc_color` property. To style the indicator, this property must be nested under the `indicator:` key. Similarly, the background is styled under the `main:` key (or at the root level).
3. **Angles**: The `start_angle` and `end_angle` properties define the span of the arc. In LVGL, 0 degrees is at the 3 o'clock position, and angles increase clockwise.
4. **Dynamic Updates**: ESPHome allows updating widget properties at runtime using the `lvgl.arc.update` action or `lvgl.widget.update`. This is crucial for changing the active preset color or brightness level dynamically.
5. **State-Based Styling**: Styles can be applied based on states (e.g., `focused`, `pressed`). This can be useful if the rotary encoder's focus is used to highlight the active preset.
6. **Limitations**: LVGL arcs are continuous. To create a segmented ring with different colors per segment, the best approach is to use multiple overlapping or adjacent arc widgets, each configured with specific start and end angles.

### Implementation Recommendations

For the VelaDial project on a 240x240 GC9A01A display using ESPHome LVGL:
1. **Layout Strategy**: Create 4 separate `arc` widgets, each covering a 90-degree quadrant (e.g., 225-315, 315-45, 45-135, 135-225). This allows independent styling of each preset segment.
2. **Styling**: Use the `indicator` part to set the active color via `arc_color` and `arc_width`. Use the `main` part for the inactive/background state. Hide the knob by setting `bg_opa: TRANSP` in the `knob` part.
3. **Interaction**: Since the rotary encoder navigates the presets, use ESPHome automations to update the `value` of each arc (100 for active, 0 for inactive) or dynamically change the `arc_color` of the indicator based on the selected preset.
4. **Center Content**: Place a `label` widget in the center of the display (using `align: CENTER`) to show the preset name and brightness. Ensure the arc widths leave enough room in the center (e.g., `arc_width: 15` leaves a 210px diameter center area).

### Code/Config Examples

```
- arc:
    id: preset_1_arc
    start_angle: 225
    end_angle: 315
    value: 100
    indicator:
      arc_color: 0xFF0000 # Red for preset 1
      arc_width: 15
    main:
      arc_color: 0x333333 # Dim background
      arc_width: 15
    knob:
      bg_opa: TRANSP # Hide knob
```

### Sources

https://esphome.io/components/lvgl/widgets/, https://esphome.io/cookbook/lvgl/, https://lvgl.io/docs/open/8.3/widgets/core/arc

---

## 4. Rotary encoder preset cycling UX and LVGL Arc integration

### Key Findings

**1. Rotary Encoder and LVGL Integration**
LVGL supports rotary encoders natively through its input device interface. In ESPHome, this is configured by defining an `encoders` list under the `lvgl` component, linking a `rotary_encoder` sensor and a binary sensor for the enter button. LVGL uses a "group" system to manage focus. When the encoder is rotated, focus moves between widgets in the group. When pressed, it clicks the focused widget or enters edit mode for complex widgets (like sliders).

**2. The Arc Widget Focus Issue**
A critical finding is that the LVGL `Arc` widget does not natively support being added to an input group for rotary encoder navigation. The `LV_OBJ_CLASS_GROUP_DEF_TRUE` flag is not set for the Arc object in LVGL's source code. This means you cannot simply add an Arc to a group and expect the rotary encoder to focus on it and adjust its value automatically like a slider.

**3. Workarounds for Arc Selection**
To achieve the "Preset Ring UI" where rotating the knob selects different arc segments, developers typically use one of two approaches:
*   **Manual State Management:** Do not use LVGL's native group focus for the arc. Instead, read the `rotary_encoder` value directly in ESPHome using `on_value` or `on_clockwise`/`on_anticlockwise` triggers. Use lambda functions or ESPHome actions to manually update the UI (e.g., changing the color of specific arc segments or updating a central label) based on the encoder's position.
*   **Hidden Focusable Widgets:** Place hidden or transparent focusable widgets (like buttons) over the arc segments and add those to the encoder group.

**4. Detent-to-Segment Alignment**
For a satisfying UX, the physical detents (clicks) of the rotary encoder must align perfectly with the visual changes on the screen. 
*   **Resolution Matching:** Ensure the rotary encoder's resolution (steps per revolution) maps cleanly to the UI elements. For 4 presets, an encoder with 16 or 20 detents per revolution works well if you divide the steps (e.g., 1 preset change per 4 or 5 detents, or use a 1:1 mapping if the encoder has very few detents).
*   **Debouncing and Filtering:** Hardware debouncing (capacitors) or software filtering in ESPHome is crucial to prevent skipped or double-registered detents, which ruins the alignment between physical feel and visual feedback.

**5. Multi-Page Handling**
ESPHome's LVGL implementation handles pages as separate screens. A known issue (Issue #6725) highlights that LVGL's input group might include widgets from inactive (hidden) pages, causing the encoder focus to seemingly disappear as it scrolls through off-screen elements. The workaround is to explicitly hide widgets on inactive pages or manage focus groups dynamically when switching pages. For the VelaDial, switching between Power, Brightness, and Presets pages should carefully manage which elements are active in the encoder group.

### Implementation Recommendations

For the VelaDial project using ESPHome and LVGL on a 240x240 GC9A01A display with a rotary encoder:

1. **Input Group Configuration**: Define an encoder group in the `lvgl` component. Map the rotary encoder sensor and the push button to this group. This allows LVGL to automatically route rotation and click events to the focused widget.

2. **Arc Widget Limitations**: In LVGL, the `Arc` widget does not natively support being added to an input group for rotary encoder selection (`LV_OBJ_CLASS_GROUP_DEF_TRUE` is not set for Arc). To work around this, you can use invisible buttons overlaid on the arc segments, or use a `roller` or `dropdown` widget styled to look like an arc, or handle the rotary encoder value changes manually via ESPHome `on_value` triggers to update the arc's value and visual state.

3. **Manual Mapping (Recommended)**: Instead of relying on LVGL's native focus navigation for the arc, use the ESPHome `rotary_encoder` component's `on_value` trigger. Map the encoder's integer value (e.g., 0 to 3) to the 4 presets. When the value changes, use `lvgl.arc.update` or `lvgl.widget.update` actions to change the active arc segment's color and update the central label text.

4. **Visual Feedback**: Use the `opa` (opacity) or `color` properties to highlight the active preset segment. For a 4-preset ring, divide the 360 degrees into 4 segments (e.g., 0-90, 90-180, etc.). You can use 4 separate arc widgets, one for each quadrant, and toggle their colors based on the selected preset.

5. **Page Navigation**: Use the encoder's push button (`on_click` or `on_press`) to trigger `lvgl.page.next` or `lvgl.page.show` to cycle through the Power, Brightness, and Presets pages. Ensure each page has its own focusable elements or manual encoder handling logic.

### Code/Config Examples

```
# ESPHome LVGL Rotary Encoder Arc Example
lvgl:
  encoders:
    - sensor: encoder
      enter_button: encoder_btn
      group: preset_group
  pages:
    - id: preset_page
      widgets:
        - arc:
            id: preset_arc
            group: preset_group
            range_from: 0
            range_to: 3
            value: 0
            bg_angle_start: 0
            bg_angle_end: 360
            rotation: 270
            # Note: Arc widgets might need custom focus handling or wrapper buttons in LVGL v8/v9

```

### Sources

https://esphome.io/components/lvgl/, https://github.com/eez-open/studio/issues/465, https://github.com/lvgl/lvgl/issues/6609, https://github.com/esphome/issues/issues/6725

---

## 5. Smart light preset UI design patterns

### Key Findings

Research into smart light apps (Philips Hue, LIFX, Nanoleaf) reveals several key design patterns for presenting color temperature and presets:

1. **Philips Hue**: Hue uses a scene gallery and a color wheel for custom settings. For color temperature specifically, they offer a "White Ambiance" range typically from 2200K (warm) to 6500K (cool). Presets like "Read" (around 2980K) and "Concentrate" are common. The UI often uses a circular color picker where the outer ring or a specific arc represents the white temperature spectrum.

2. **LIFX**: LIFX recently updated their app to reorder the white palette, starting with warm colors first, as this aligns with user behavior. They use a classic control wheel for color and saturation, and a specific wheel/slider for Kelvin values (typically 1500K to 9000K). The UI emphasizes visual feedback, showing the exact color on the wheel.

3. **Nanoleaf**: Nanoleaf's app features an intuitive color wheel for creating custom dynamic scenes. They also have a "Basic" tab for setting a single static color or tunable white. Their UI allows users to select palettes and colors, adjusting ratios for different panels.

**Common Patterns**:
- **Circular Interfaces**: All three apps heavily utilize circular color wheels or arcs for selecting colors and temperatures, making the "Preset Ring UI" concept highly aligned with industry standards.
- **Warm to Cool Spectrum**: Color temperature is almost always presented as a gradient from warm (orange/yellow, ~2000K) to cool (blue/white, ~6500K+).
- **Preset Naming**: Functional names (Read, Relax, Concentrate, Energize) are preferred over technical Kelvin values for average users.

**Application to Preset Ring UI**:
The concept of placing four presets around a round display perfectly mirrors the circular color pickers found in these apps. By dividing the ring into quadrants, users can quickly access their most used settings (e.g., Warm/Relax, Neutral/Read, Cool/Concentrate, and a Custom Color). The active segment filling with the characteristic color provides the immediate visual feedback that users expect from premium smart light apps.

### Implementation Recommendations

For the ESPHome LVGL implementation on the 240x240 GC9A01A round display, the "Preset Ring UI" should utilize the `arc` widget to create the four preset segments. Each arc should span 90 degrees (e.g., 45-135, 135-225, 225-315, 315-45) with a small gap between them for visual separation. The `radius` should be set to around 100-110 pixels to fit within the 240x240 screen while leaving room for the center text. The `width` of the arcs should be around 20-30 pixels to be easily selectable via the rotary encoder.

The rotary encoder should be configured to navigate between these four arc widgets. When an arc is focused (using the `focused` state in LVGL), its color should change or its width should increase to indicate selection. Pressing the encoder (the `ENTER` action) should trigger an `on_click` event on the focused arc, which sends a Home Assistant action to set the light to the corresponding color temperature or scene.

In the center of the ring, use a `label` widget to display the name of the active preset (e.g., "Warm", "Cool", "Reading") and the current brightness percentage. The brightness can be updated dynamically using a sensor that tracks the light's state.

For the Brightness page, a single continuous `arc` widget can be used as a slider, where the `value` corresponds to the brightness level (0-255 mapped to 0-100%). The rotary encoder's left/right actions will adjust this value.

### Code/Config Examples

```
lvgl:
  widgets:
    - arc:
        id: preset_arc_1
        start_angle: 225
        end_angle: 315
        width: 20
        radius: 100
        bg_color: 0xFF9900 # Warm White
        on_click:
          - homeassistant.action:
              action: light.turn_on
              data:
                entity_id: light.main
                color_temp: 400
    - label:
        id: center_text
        align: CENTER
        text: "Warm"
```

### Sources

https://esphome.io/cookbook/lvgl/, https://esphome.io/components/lvgl/widgets/, https://esphome.io/components/lvgl/, https://www.reddit.com/r/Hue/comments/7p6jjm/setting_hue_lights_to_specific_color_temperature/, https://hueblog.com/2024/02/14/displaying-the-exact-colour-temperature-for-philips-hue-in-kelvin/, https://www.reddit.com/r/lifx/comments/1lw3q95/lifx_app_update_4680_is_here/

---

## 6. ESPHome LVGL arc widget start_angle end_angle configuration

### Key Findings

1. **Coordinate System**: In LVGL, the 0-degree position for arcs is at the middle right (3 o'clock position). Degrees increase in a clockwise direction. The angle values should be in the range of 0 to 360 degrees.

2. **Arc Structure**: An LVGL arc consists of two main parts: a background arc and a foreground (indicator) arc. The `start_angle` and `end_angle` properties in ESPHome YAML configure the background arc's span. The foreground arc's span is determined by the `value` property relative to `min_value` and `max_value`.

3. **ESPHome Mapping**: In the ESPHome LVGL component source code (`esphome/components/lvgl/widgets/arc.py`), the YAML properties `start_angle` and `end_angle` are explicitly mapped to LVGL's `bg_start_angle` and `bg_end_angle` functions. This means configuring these properties in YAML sets the static background boundaries of the arc.

4. **Known Issues**: There are reported issues in ESPHome (e.g., Issue #12642 and #15853) where `start_angle` and `end_angle` might not behave as expected or fail to update dynamically at runtime. If static YAML configuration fails to render the quadrants correctly, a workaround is to use C++ lambdas to call the native LVGL functions directly: `lv_arc_set_bg_angles(arc_obj, start, end)`.

5. **Quadrant Design Pattern**: To create a 4-quadrant UI with gaps, you cannot use a single arc widget because an arc is continuous. Instead, you must instantiate four separate `arc` widgets, all centered at the same coordinates, with distinct start and end angles that leave gaps (e.g., 80-degree spans with 10-degree gaps).

6. **Interactivity**: For a rotary encoder-driven UI like VelaDial, the arcs should be set to `adjustable: false` to disable touch interactions and hide the knob part. The active state can be visually represented by changing the `value` of the respective arc to fill its indicator, or by changing its `arc_color` style.

### Implementation Recommendations

For the VelaDial project on a 240x240 GC9A01A display, implement the 4 quadrant arcs using four separate `arc` widgets overlaid on the same center point. To create gaps between the quadrants, configure the `start_angle` and `end_angle` for each arc with a 10-degree gap between them. 

For example:
- Top quadrant (Preset 1): `start_angle: 230`, `end_angle: 310`
- Right quadrant (Preset 2): `start_angle: 320`, `end_angle: 40`
- Bottom quadrant (Preset 3): `start_angle: 50`, `end_angle: 130`
- Left quadrant (Preset 4): `start_angle: 140`, `end_angle: 220`

Set `adjustable: false` for all arcs since navigation is handled by the rotary encoder, not touch. Use the `value` property (0-100) to fill the active preset's arc with its characteristic color, while keeping inactive presets at `value: 0` or a low value to show only the background arc.

Note that in ESPHome, `start_angle` and `end_angle` in the YAML configuration are automatically mapped to LVGL's `bg_start_angle` and `bg_end_angle` properties under the hood. If you encounter issues with angles not updating dynamically at runtime, this is a known bug in some ESPHome versions (e.g., #12642) where the angle properties might not apply correctly after initialization. In such cases, you may need to use a C++ lambda with `lv_arc_set_bg_angles(id(preset_1).get_obj(), 230, 310);` to enforce the angles.

### Code/Config Examples

```
lvgl:
  pages:
    - id: presets_page
      widgets:
        - arc:
            id: preset_1
            start_angle: 230
            end_angle: 310
            value: 100
            adjustable: false
        - arc:
            id: preset_2
            start_angle: 320
            end_angle: 40
            value: 100
            adjustable: false
        - arc:
            id: preset_3
            start_angle: 50
            end_angle: 130
            value: 100
            adjustable: false
        - arc:
            id: preset_4
            start_angle: 140
            end_angle: 220
            value: 100
            adjustable: false
```

### Sources

https://lvgl.io/docs/open/9.5/widgets/arc, https://esphome.io/components/lvgl/widgets/, https://github.com/esphome/esphome/issues/12642, https://github.com/esphome/esphome/issues/15853, ESPHome Source Code (esphome/components/lvgl/widgets/arc.py)

---

## 7. Radial menu animation patterns in LVGL

### Key Findings

Research into LVGL radial menu animation patterns reveals several key technical findings and best practices for creating a "Preset Ring UI":

1. **Arc Widget as Foundation**: The `lv_arc` widget is the standard method for creating circular/radial UI elements in LVGL. A radial menu can be constructed by placing multiple arc widgets on top of each other or side-by-side, each configured with specific background angles (`lv_arc_set_bg_angles`). For four quadrants, the angles would typically be 315°-45°, 45°-135°, 135°-225°, and 225°-315°.

2. **Animation System (`lv_anim_t`)**: LVGL provides a robust animation system that can animate almost any property of a widget. For a traveling highlight effect, the most effective approach is to animate the `value` or the `angles` of the arc. 
   - **Value Animation**: Animating the value from 0 to max (with `LV_ARC_MODE_NORMAL`) creates a filling effect.
   - **Angle Animation**: Animating the start and end angles (`lv_arc_set_angles`) allows a segment to physically move around the circle, which is perfect for a traveling highlight.

3. **Ease-in-out Paths**: To make transitions smooth, LVGL offers built-in animation paths. `lv_anim_path_ease_in_out` is highly recommended for UI transitions as it starts slow, accelerates, and then decelerates at the end, providing a natural, fluid motion compared to linear paths.

4. **Timelines for Complex Animations**: For coordinating multiple animations (e.g., one arc shrinking while another grows, and text fading in the center), LVGL's `lv_anim_timeline_t` allows precise synchronization of multiple `lv_anim_t` objects. However, in ESPHome's YAML configuration, complex timelines might require custom C++ lambdas if the native YAML actions are insufficient.

5. **Input Integration**: When using a rotary encoder, the encoder's left/right actions should change the focused object or a state variable, which in turn triggers the animation to move the highlight to the newly selected preset.

### Implementation Recommendations

For the VelaDial project on a 240x240 GC9A01A display using ESPHome LVGL, implement the Preset Ring UI using four `arc` widgets positioned at the quadrants (e.g., 315°-45°, 45°-135°, 135°-225°, 225°-315°). 

1. **Traveling Highlight Effect**: Since ESPHome's YAML wrapper for LVGL animations is somewhat limited compared to native C code, achieve the traveling highlight by animating the `value` property of the arcs. When the rotary encoder focuses on a new preset, trigger an animation that reduces the previous arc's value to 0 and increases the new arc's value to 100 simultaneously.
2. **Smooth Transitions**: Use the `ease_in_out` animation path (`lv_anim_path_ease_in_out` in C, or `ease_in_out` in ESPHome YAML) to ensure the arc filling and unfilling feels natural and not abrupt. Set the animation duration to around 300ms to 500ms for a responsive yet smooth feel.
3. **Arc Configuration**: Hide the knob part of the arc (`LV_PART_KNOB`) since navigation is done via the rotary encoder, not touch dragging. Set `mode: NORMAL` for standard filling.
4. **Performance**: The ESP32-S3 is powerful enough for these animations, but ensure the display buffer is adequately sized (e.g., 25% to 50% of screen size in PSRAM) to prevent tearing during the animation.

### Code/Config Examples

```
# ESPHome LVGL Radial Menu Animation Snippet
lvgl:
  pages:
    - id: preset_page
      widgets:
        - arc:
            id: preset_arc_1
            bg_angles: [315, 45]
            value: 100
            mode: NORMAL
            color: 0xFF0000
            # Animation triggered on focus/selection
            on_focus:
              - lvgl.anim.start:
                  id: anim_highlight_1
                  var: preset_arc_1
                  property: value
                  from: 0
                  to: 100
                  time: 300ms
                  path: ease_in_out

```

### Sources

https://lvgl.io/docs/open/8.3/overview/animation, https://lvgl.io/docs/open/8.3/widgets/core/arc, https://esphome.io/components/lvgl/, https://esphome.io/cookbook/lvgl/

---

## 8. Color temperature visualization on small displays

### Key Findings

Visualizing color temperature on small displays, especially round ones like the 240x240 GC9A01A, requires careful consideration of color palettes and UI elements. 

1. **Color Representation**: Color temperature is typically represented on a scale from warm (amber/orange) to cool (blue/white). For a smart light controller, distinguishing between specific presets like warm white (e.g., 2700K), soft amber (e.g., 2200K), neutral white (e.g., 4000K), and nightlight (often very warm and dim) is crucial. Using a gradient or distinct solid colors that match these physical light outputs helps users intuitively understand the setting.

2. **UI Patterns for Round Displays**: The "Preset Ring UI" concept is highly effective for round displays. Using arc segments (`lv_arc` in LVGL) around the circumference maximizes the use of screen real estate while leaving the center open for text (preset name and brightness value). 

3. **LVGL Specifics**: In LVGL (specifically v8, commonly used with ESPHome), native gradients on arcs might not be fully supported or performant on low-end MCUs like the ESP32. A common workaround is to use an image of a gradient arc as the background or to dynamically change the solid color of the arc indicator based on the current value. For discrete presets, dividing the ring into 4 distinct arc segments (quadrants) and filling the active one with its characteristic color is a clean and readable approach.

4. **Contrast and Legibility**: On a small 1.28" display, contrast is key. A dark or black background is recommended to make the colored arcs and white text stand out. The arc thickness should be substantial enough (e.g., 15-20 pixels) to be easily visible from a distance or at a glance.

5. **Interaction**: The rotary encoder is the primary input. The UI should provide immediate visual feedback when the knob is turned. For the Presets page, rotating the knob should snap the selection between the 4 quadrants, instantly updating the center text and the active arc's color.

### Implementation Recommendations

For ESPHome LVGL on a 240x240 round display (GC9A01A), use an `lv_arc` widget to represent color temperature. Map the color temperature range (e.g., 2700K to 6500K) to the arc's value. To distinguish warm white, soft amber, neutral white, and nightlight, you can use distinct colors for the arc indicator based on the current value, or use an image with a pre-rendered gradient as the arc's background if LVGL v8 doesn't support native arc gradients. Place the arc near the edge (radius ~100-110px) with a thickness of 15-20px for easy visibility and touch/knob interaction. Use a dark background (e.g., 0x000000 or 0x111111) to make the colors pop. Update the arc color dynamically in ESPHome using a lambda when the rotary encoder changes the value.

### Code/Config Examples

```
arc:
  id: temp_arc
  min_value: 2700
  max_value: 6500
  color: 0xFFA500 # Example warm color
  bg_color: 0x333333
  width: 20
  radius: 100
```

### Sources

https://dribbble.com/search/color-temperature, https://forum.lvgl.io/t/color-gradient-on-arc/9533, https://lvgl.io/docs/open/8.3/widgets/core/arc

---

## 9. LVGL lv_arc custom styling and performance

### Key Findings

The `lv_arc` widget in LVGL is a versatile component for creating circular UI elements, consisting of a background arc (`LV_PART_MAIN`) and an indicator arc (`LV_PART_INDICATOR`). The indicator arc is drawn on top of the background arc, allowing for progress bars or dial-like interfaces.

Custom styling of the arc involves several key properties. The width of the arc can be adjusted using the `arc_width` property, which dictates the thickness of the drawn line. To achieve rounded ends for the arc segments, the `arc_rounded` property must be set to `true`. This maps to the `lv_style_set_arc_rounded` function in the underlying C library. However, developers have noted a bug where setting rounded ends with an opacity other than fully opaque can cause rendering issues, so it is advisable to use solid colors for the arcs.

A significant finding regarding performance relates to how LVGL handles redrawing arcs. When the value of an arc changes, LVGL invalidates the entire rectangular bounding box containing the arc, rather than just the specific segment that changed. This behavior causes the entire area taken up by the arc to be redrawn. While there was a pull request (#7598) attempting to address this by optimizing the invalidation area, it reportedly caused issues for some users and may not be fully integrated or stable in all versions.

This redrawing behavior has profound implications for performance when using multiple overlapping arcs, such as in a preset ring UI design. If multiple arcs share the same center and cover a large portion of the screen, updating one arc will trigger a redraw of the entire overlapping area. On microcontrollers without internal GRAM (Graphics RAM) in the display controller, such as the ST7701 or the GC9A01A used in the VelaDial project, the entire framebuffer must be rendered and transmitted over the SPI bus. This process is bandwidth-intensive and can severely degrade the frame rate, especially if the arcs are updated continuously (e.g., during an animation or rapid knob rotation). To achieve acceptable performance, it is critical to minimize the frequency of arc updates and leverage compiler optimizations.

### Implementation Recommendations

For the VelaDial project using an ESP32-S3 and a 240x240 GC9A01A display running ESPHome with LVGL, implementing the Preset Ring UI requires careful consideration of hardware constraints and LVGL's rendering behavior.

The GC9A01A display is typically connected via SPI. Since it is a 240x240 display, the framebuffer size is 115.2 KB (240 * 240 * 2 bytes for RGB565). To ensure smooth rendering, allocate a sufficiently large buffer for LVGL, ideally a full framebuffer or at least 1/10th of the screen size, utilizing the ESP32-S3's PSRAM if available. Ensure the ESP32-S3 is running at its maximum clock speed of 240 MHz.

To implement the four preset segments around the circumference, use four separate `arc` widgets positioned at the same center. Configure each arc with a specific `start_angle` and `end_angle` corresponding to the four quadrants (e.g., 315-45, 45-135, 135-225, 225-315). Set `arc_rounded: true` to achieve the desired aesthetic for the preset ring, and adjust `arc_width` to a suitable value, such as 10 to 20 pixels, to make the presets visible without overwhelming the central information display.

Performance optimization is crucial due to the overlapping nature of the arcs. Since changing an arc's value causes LVGL to invalidate and redraw the entire rectangular bounding box, overlapping multiple arcs covering the full 240x240 area will lead to significant performance degradation if updated frequently. To mitigate this, avoid continuous updates of the arcs. Only update the active preset's arc color or value when the rotary encoder is turned to select a new preset. Minimize the number of overlapping transparent widgets and consider enabling compiler optimizations (e.g., `-O2` or `-O3`) in the ESPHome configuration to improve rendering speed.

### Code/Config Examples

```
```yaml
lvgl:
  pages:
    - id: preset_page
      widgets:
        - arc:
            id: preset_1
            align: CENTER
            width: 240
            height: 240
            arc_width: 15
            arc_rounded: true
            start_angle: 315
            end_angle: 45
            bg_opa: TRANSP
            color: 0xFF0000 # Active color
```
```

### Sources

https://esphome.io/components/lvgl/widgets/, https://esphome.io/cookbook/lvgl/, https://forum.lvgl.io/t/changing-arc-value-causes-entire-arc-to-be-redrawn/19798, https://github.com/lvgl/lvgl/issues/2868

---

## 10. Washing machine program selector UI metaphor - vintage radio tuner dial interface, rotary selection wheel digital implementation

### Key Findings

The research into UI/UX patterns for a washing machine program selector metaphor and vintage radio tuner dial interface, specifically applied to a digital rotary selection wheel using LVGL on a round display, yielded several key findings:

**1. The Washing Machine Selector Metaphor:**
The washing machine program selector is a classic example of a discrete rotary interface. Users expect distinct "clicks" or detents as they rotate the dial, with each position corresponding to a specific, mutually exclusive mode or preset. In a digital implementation like the VelaDial, this translates to dividing the circular UI into distinct segments. When the user rotates the physical knob, the UI should snap to the next segment, providing clear visual feedback of the active selection. This is different from a continuous volume knob.

**2. Vintage Radio Tuner Dial Interface:**
Vintage radio dials often feature a fixed scale with a moving indicator (needle) or a moving scale behind a fixed window. For a round display, the moving indicator on a fixed circular scale is the most applicable metaphor. This provides a strong sense of absolute position. The digital implementation can enhance this by illuminating the active segment or changing the color of the indicator based on the selected "station" (preset).

**3. LVGL Arc Widget Capabilities:**
The LVGL `arc` widget is the primary tool for building these interfaces. It consists of a background arc, an indicator arc, and a knob.
*   **Angles:** The arc's start and end angles can be precisely controlled (e.g., `lv_arc_set_bg_angles(arc, 0, 360)` for a full circle). Zero degrees is at the 3 o'clock position.
*   **Modes:** The arc supports different modes like `NORMAL`, `REVERSE`, and `SYMMETRICAL`, which dictate how the indicator is drawn relative to the value.
*   **Styling:** Every part of the arc (main/background, indicator, knob) can be styled independently. For a segmented preset ring, the knob can be hidden, and the indicator can be styled to fill a specific angular segment.

**4. ESPHome Integration:**
ESPHome provides excellent support for LVGL. The rotary encoder can be directly linked to LVGL widgets.
*   **Groups:** The encoder must be assigned to an LVGL `group`, and the arc widget must be added to that group to receive input.
*   **Value Mapping:** The encoder's discrete steps need to be mapped to the arc's value range. For four presets, the arc's range could be 0-3, with each value corresponding to a 90-degree segment.
*   **Event Handling:** The `on_value` trigger in ESPHome's LVGL configuration allows executing actions (like changing the central text or sending commands to Home Assistant) when the arc's value changes.

**5. Design Patterns for 240x240 Round Displays:**
*   **Maximize Edge Usage:** The perimeter of the round display is ideal for navigation and status indication (like the preset ring), leaving the center free for detailed information.
*   **Clear Typography:** Use large, legible fonts for the central information (preset name, brightness) as round displays can sometimes feel cramped.
*   **Color Coding:** Use distinct colors for each preset segment to provide immediate visual recognition, reinforcing the washing machine selector metaphor where different colors might indicate different wash types.

### Implementation Recommendations

For the VelaDial project using ESPHome LVGL on a 240x240 GC9A01A round display with an ESP32-S3 and rotary encoder:

1. **Arc Widget Configuration**: Use the `lv_arc` widget to create the preset ring. Set `width` and `height` to 240 to fill the display. Use `bg_angles: [0, 360]` to create a full circle.
2. **Preset Segments**: Divide the 360-degree arc into four 90-degree segments for the four presets. You can use multiple arc widgets overlaid or a single arc with custom drawing logic to show the active segment.
3. **Rotary Encoder Integration**: Map the rotary encoder input to the arc's value. Since the encoder provides discrete steps, map these steps to the preset selection (e.g., 0-25 for Preset 1, 26-50 for Preset 2, etc.).
4. **Visual Feedback**: Use the `indicator` part of the arc to highlight the active preset with its characteristic color. Hide the `knob` part (`bg_opa: TRANSP` or `pad_all: 0`) if you want a clean segment look rather than a continuous dial.
5. **Center Display**: Place a `label` widget in the center (`align: CENTER`) to show the preset name and brightness value. Update this label dynamically based on the selected preset.
6. **Page Navigation**: Implement the 3 pages (Power, Brightness, Presets) using LVGL's page or screen management, switching between them based on encoder button presses or specific gestures.

### Code/Config Examples

```
# ESPHome LVGL Arc Preset Selector
lvgl:
  pages:
    - id: preset_page
      widgets:
        - arc:
            id: preset_arc
            align: CENTER
            width: 240
            height: 240
            bg_angles: [0, 360]
            value: 25
            min_value: 0
            max_value: 100
            mode: NORMAL
            on_value:
              - logger.log: "Preset changed"
```

### Sources

https://lvgl.io/docs/open/8.3/widgets/core/arc, https://esphome.io/components/lvgl/widgets/, https://esphome.io/cookbook/lvgl/

---

## 11. ESPHome LVGL page swipe navigation with dot indicators

### Key Findings

Research into implementing a 3-page carousel with swipe navigation and dot indicators in ESPHome LVGL reveals several critical technical findings and best practices.

**1. Swipe Navigation Mechanism:**
The fundamental finding is that standard LVGL pages in ESPHome do not natively support swipe-to-navigate functionality out of the box. Users attempting to use `on_swipe` triggers on standard pages often encounter conflicts with interactive widgets like buttons or sliders. The official and most robust solution is to use the `tileview` widget. The `tileview` is a container object whose elements (tiles) can be arranged in a grid. It natively supports dragging or swiping to navigate between tiles. For a 3-page carousel, you arrange three tiles horizontally. 

**2. Scrollbar Management:**
By default, the `tileview` will show scrollbars when navigating. On a 240x240 round display, straight scrollbars clip awkwardly against the circular edges. It is a best practice to disable them by setting the `scrollbar_mode` property to `"OFF"` on the `tileview` widget.

**3. Custom Dot Indicators:**
LVGL does not provide a dedicated "carousel dot indicator" widget. While widgets like `slider`, `bar`, and `arc` have an `indicator` part, these are for continuous values, not discrete page dots. To implement page indicator dots, you must construct them manually using basic `obj` widgets. 
- Create a parent `obj` container positioned at the bottom center (`align: BOTTOM_MID`).
- Use LVGL's flex layout (`layout: FLEX`, `flex_flow: ROW`) to evenly space the dots.
- Create the dots using small `obj` widgets with equal width and height (e.g., 6x6 or 8x8 pixels) and set the `radius` to half the width to make them perfectly round.

**4. State Synchronization:**
To synchronize the dot indicators with the active page, you must utilize the `on_value` event of the `tileview` widget. This event is triggered whenever a new tile is loaded by scrolling. Within this trigger, a C++ lambda function is required to query the active tile index using `lv_tileview_get_tile_active()`. Based on the returned index, you dynamically update the styling (typically `bg_color` or `bg_opa`) of the dot objects to reflect the current page.

**5. Round Display Considerations:**
When designing for a 240x240 round display like the GC9A01A, placement is critical. The dot indicators must be placed high enough from the absolute bottom edge to avoid being cut off by the physical bezel or the curvature of the screen. A `y` offset of `-10` to `-15` pixels from `BOTTOM_MID` is typically required. Furthermore, when placing the arc segments for the Preset Ring UI, ensure their touch areas (if touch is enabled) or visual boundaries do not overlap with the swipe area needed for the `tileview` navigation.

### Implementation Recommendations

For the VelaDial project on a 240x240 round display (GC9A01A) running ESPHome with LVGL, implementing a 3-page carousel with dot indicators requires a specific approach. 

First, use the `tileview` widget instead of standard pages. Standard pages in ESPHome LVGL do not natively support swipe gestures for navigation. The `tileview` widget is designed exactly for this purpose, allowing you to arrange "tiles" (pages) horizontally and swipe between them. Set the `scrollbar_mode` to `"OFF"` to hide the default scrollbars, which look out of place on a round display.

Second, for the dot indicators, ESPHome LVGL does not have a built-in "carousel dots" widget. You must implement this manually by creating a container `obj` positioned at the bottom of the screen (`align: BOTTOM_MID`, with a negative `y` offset like `-10` to keep it visible on the curve of the round display). Inside this container, use a `FLEX` layout with `ROW` flow to arrange three small `obj` widgets. Set their `width` and `height` to `6` pixels and `radius` to `3` to make them perfect circles (dots).

Third, to make the dots functional, use the `on_value` trigger of the `tileview` widget. This trigger fires when the user swipes to a new tile. Inside the trigger, use a `lambda` to call `lv_tileview_get_tile_active()` to determine which tile is currently visible (0, 1, or 2). Based on this index, update the `bg_color` or `bg_opa` of the three dot objects to highlight the active page (e.g., solid white for active, dim gray for inactive).

Finally, for the Preset Ring UI concept, ensure that the swipe gestures do not conflict with the rotary encoder input or the arc widgets used for the presets. The `tileview` handles horizontal swipes, while the rotary encoder should be mapped to a specific group to control the active preset or brightness arc.

### Code/Config Examples

```
# ESPHome LVGL Tileview with Dot Indicators
lvgl:
  pages:
    - id: main_page
      widgets:
        - tileview:
            id: carousel
            width: 100%
            height: 100%
            scrollbar_mode: "OFF"
            on_value:
              - lambda: |-
                  int active_tile = lv_tileview_get_tile_active(id(carousel));
                  // Update dot indicators based on active_tile (0, 1, 2)
                  // e.g., lv_obj_set_style_bg_color(dot0, active_tile == 0 ? active_color : inactive_color, 0);
        - obj: # Dot container
            align: BOTTOM_MID
            y: -10
            layout: FLEX
            flex_flow: ROW
            flex_align_main: CENTER
            flex_align_cross: CENTER
            pad_column: 5
            widgets:
              - obj: { id: dot0, width: 6, height: 6, radius: 3, bg_color: 0xFFFFFF }
              - obj: { id: dot1, width: 6, height: 6, radius: 3, bg_color: 0x888888 }
              - obj: { id: dot2, width: 6, height: 6, radius: 3, bg_color: 0x888888 }
```

### Sources

https://esphome.io/components/lvgl/widgets/, https://lvgl.io/docs/open/9.1/widgets/tileview.html, https://community.home-assistant.io/t/problem-with-lvgl-swipes-and-click-button-togther/860276, https://forum.lvgl.io/t/how-to-display-a-dot-on-screen/339

---

## 12. WS2812 LED ring color temperature mapping

### Key Findings

The research into WS2812 LED color temperature mapping reveals several critical technical considerations for achieving accurate white and amber tones. WS2812 LEDs are RGB-based, meaning they lack a dedicated white phosphor channel. Consequently, producing white light requires mixing Red, Green, and Blue channels, which presents specific challenges.

**Color Temperature to RGB Mapping**
Based on established blackbody radiation conversions (such as Mitchell Charity's calculations and Andi Siess's dataset), specific RGB values can simulate various color temperatures on RGB LEDs. For the Preset Ring UI, the following mappings are recommended:
- **Amber (approx. 2200K):** RGB(255, 147, 44). This provides a deep, warm glow suitable for night-time or relaxed settings.
- **Warm White (approx. 2700K):** RGB(255, 169, 87). This mimics traditional incandescent lighting.
- **Gold / Soft Amber (approx. 3300K):** RGB(255, 190, 126). A balanced, inviting tone often used in hospitality lighting.
- **Cool White / Neutral (approx. 6000K):** RGB(255, 243, 239). A crisp, daylight-like white.

**The Dimming Challenge (Color Shift)**
A significant issue with WS2812 LEDs is color shifting during dimming. When scaling RGB values linearly to reduce brightness, the LEDs often shift towards yellow or red. This occurs because the human eye's perception of brightness is non-linear, and the LEDs themselves have different luminosity curves for each color channel (e.g., Green is typically much brighter than Red or Blue). Furthermore, at low brightness levels, the 8-bit resolution (0-255) per channel causes quantization errors, disrupting the delicate balance required for white light.

**Best Practices for Dimming**
To mitigate color shift when dimming warm white or amber tones on WS2812 LEDs, developers strongly recommend using the HSV (Hue, Saturation, Value) color space instead of RGB. By setting the desired color using Hue and Saturation, brightness can be adjusted purely by changing the Value parameter. This maintains the color ratio much better than linear RGB scaling. If using FastLED or similar libraries, utilizing built-in color correction and gamma correction functions is essential to maintain color fidelity at lower brightness levels.

**Hardware Alternatives**
While WS2812 LEDs can simulate these temperatures, they are not ideal for high-quality white light due to poor Color Rendering Index (CRI) and the aforementioned dimming issues. For projects heavily reliant on white/amber tones, the community frequently suggests switching to SK6812 WWA (Warm White, Cool White, Amber) strips. These strips use dedicated white and amber phosphors instead of RGB mixing, resulting in vastly superior color accuracy, perfect dimming without color shift, and higher overall brightness for white tones. However, since the VelaDial project specifies WS2812 LEDs, careful software calibration using the HSV method and precise RGB mapping is required.

### Implementation Recommendations

For the VelaDial project using ESPHome and LVGL on the 240x240 GC9A01A display with 5 WS2812 LEDs, the implementation should focus on precise color mapping and smooth UI integration. 

First, configure the WS2812 LEDs in ESPHome using the `neopixelbus` or `fastled_clockless` platform. Ensure the `color_correct` parameter is calibrated, as WS2812 LEDs often have a strong blue bias. A typical correction might be `[100%, 80%, 80%]` depending on the specific batch, but start with `[100%, 100%, 100%]` and adjust based on visual inspection.

For the LVGL UI, use the `lv_arc` widget to create the four preset segments. Set the arc's background color to a dim gray and the indicator color to the specific RGB value of the active preset. The active preset's arc should be filled with its characteristic color: Amber (255, 147, 44), Warm White (255, 169, 87), Gold (255, 190, 126), or Cool White (255, 243, 239).

When navigating with the rotary encoder, update the active arc segment and simultaneously send the corresponding RGB values to the WS2812 LEDs. To avoid the "yellowing" effect when dimming WS2812 LEDs, implement dimming in the HSV color space rather than scaling RGB values linearly. Convert the target RGB color to HSV, reduce the Value (V) for dimming, and convert back to RGB before sending to the LEDs.

For the Brightness page, use a single continuous `lv_arc` where the angle is proportional to the brightness value (0-100%). The color of this arc should match the currently selected preset color. On the Power page, a full ring indicates 'ON', while a dimmed ring indicates 'OFF'. Ensure smooth transitions between pages and states using LVGL animations (`lv_anim_t`) for a premium user experience.

### Code/Config Examples

```
// ESPHome YAML configuration for WS2812 preset colors
light:
  - platform: neopixelbus
    type: GRB
    variant: WS2812
    pin: GPIO4
    num_leds: 5
    name: "Preset Ring LEDs"
    color_correct: [100%, 100%, 100%]

// C++ LVGL Color Mapping for Presets
lv_color_t preset_amber = lv_color_make(255, 147, 44);  // 2200K
lv_color_t preset_warm = lv_color_make(255, 169, 87);   // 2700K
lv_color_t preset_gold = lv_color_make(255, 190, 126);  // 3300K
lv_color_t preset_cool = lv_color_make(255, 243, 239);  // 6000K
```

### Sources

https://andi-siess.de/rgb-to-color-temperature/, https://forum.arduino.cc/t/is-any-dimming-in-warm-white-feasible-with-ws2812b-and-fastled-or-should-i-just-switch-to-sk6812/1370275, https://www.reddit.com/r/led/comments/9meghu/rgb_to_create_warm_white_on_a_60_pixel_per_meter/

---

## 13. Round display UI quadrant layout using LVGL arcs

### Key Findings

The research into dividing a 360-degree round display into 4 segments using LVGL arcs for the Preset Ring UI concept yielded several critical technical findings and best practices.

**LVGL Arc Coordinate System**
A fundamental aspect of working with LVGL arcs is understanding its coordinate system. In LVGL, 0 degrees is located at the 3 o'clock position (middle right of the widget). Angles increase in a clockwise direction. The valid range for angles is 0 to 360 degrees. This differs from standard mathematical polar coordinates (where angles increase counter-clockwise) and must be accounted for when calculating segment positions.

**Segment Calculation and Gaps**
To divide a circle into 4 equal quadrants with visual gaps between them, the total 360 degrees must be distributed among the arcs and the gaps. A visually pleasing ratio is an 80-degree arc segment with a 10-degree gap between each segment. 
Based on the LVGL coordinate system (0° at 3 o'clock, clockwise), the calculated angles for the four quadrants are:
1. **Bottom Right**: 5° to 85° (Centered at 45°)
2. **Bottom Left**: 95° to 175° (Centered at 135°)
3. **Top Left**: 185° to 265° (Centered at 225°)
4. **Top Right**: 275° to 355° (Centered at 315°)
This layout ensures perfect symmetry and consistent 10-degree gaps at the 0°, 90°, 180°, and 270° marks.

**Arc Widget Properties in ESPHome**
In ESPHome's LVGL component, an arc is defined by both background angles (`bg_start_angle`, `bg_end_angle`) and foreground/indicator angles (`start_angle`, `end_angle`). 
- The background arc represents the track or the inactive state of the segment.
- The foreground arc represents the filled or active state.
To create a static segment that can be toggled on or off, the background angles define the permanent position of the segment. When a preset is active, the foreground angles are set to match the background angles, filling the segment with the active color.

**Styling and Parts**
An LVGL arc consists of three main parts: `LV_PART_MAIN` (background), `LV_PART_INDICATOR` (foreground), and `LV_PART_KNOB` (the draggable handle). For a display-only UI driven by a rotary encoder, the knob is unnecessary and should be hidden. This is achieved by removing the style for the knob or setting its opacity to transparent. Additionally, the `clickable` flag should be disabled (`lv_obj_clear_flag(arc, LV_OBJ_FLAG_CLICKABLE)`) to prevent unintended touch interactions if the display has a touch overlay.

**Dynamic Updates**
To animate or update the active preset based on rotary encoder input, ESPHome provides the `lvgl.arc.update` action. However, it is often more efficient to keep the angles static and update the `color` or `bg_color` properties, or toggle the visibility of the foreground indicator to show which preset is currently selected.

### Implementation Recommendations

For the VelaDial project using an ESP32-S3 and a 240x240 GC9A01A round display with ESPHome and LVGL, implementing the Preset Ring UI requires careful configuration of the `arc` widget.

1. **Widget Setup**: Create four separate `arc` widgets, all centered on the screen with a width and height of 240 (matching the display resolution). Set `arc_width` and `bg_arc_width` to a suitable thickness, such as 20 pixels, to create a ring effect along the outer edge.

2. **Angle Configuration**: LVGL defines 0 degrees at the 3 o'clock position (middle right), with angles increasing clockwise. To divide the 360-degree circle into 4 quadrants with gaps, use the following angle ranges:
   - Top Right (Preset 1): 275° to 355°
   - Bottom Right (Preset 2): 5° to 85°
   - Bottom Left (Preset 3): 95° to 175°
   - Top Left (Preset 4): 185° to 265°
   This provides an 80-degree arc for each preset, with a 10-degree gap between adjacent arcs.

3. **Active/Inactive States**: To show the active preset, set the `start_angle` and `end_angle` to match the `bg_start_angle` and `bg_end_angle` for the active arc, and set its `color` to the characteristic color. For inactive presets, either set the foreground arc angles to 0 or make the `color` transparent, allowing the `bg_color` (e.g., dark gray) to show.

4. **Knob Removal**: Since these arcs are used for display rather than direct touch interaction, remove the knob part by setting its style to hidden or removing it entirely, and disable the `clickable` flag to prevent touch interference.

5. **Rotary Encoder Integration**: Bind the rotary encoder to a variable that tracks the active preset (1-4). When the encoder is rotated, update the `color` or visibility of the foreground arcs to highlight the newly selected preset.

### Code/Config Examples

```
# ESPHome LVGL Arc Quadrant Configuration
lvgl:
  pages:
    - id: presets_page
      widgets:
        # Top Right Quadrant (Preset 1)
        - arc:
            id: preset_1_arc
            bg_start_angle: 275
            bg_end_angle: 355
            start_angle: 275
            end_angle: 355
            width: 240
            height: 240
            align: CENTER
            arc_width: 20
            bg_arc_width: 20
            color: 0xFF0000 # Active color
            bg_color: 0x333333 # Inactive color
            
        # Bottom Right Quadrant (Preset 2)
        - arc:
            id: preset_2_arc
            bg_start_angle: 5
            bg_end_angle: 85
            start_angle: 5
            end_angle: 85
            # ... other properties same as above
            
        # Bottom Left Quadrant (Preset 3)
        - arc:
            id: preset_3_arc
            bg_start_angle: 95
            bg_end_angle: 175
            start_angle: 95
            end_angle: 175
            # ... other properties same as above
            
        # Top Left Quadrant (Preset 4)
        - arc:
            id: preset_4_arc
            bg_start_angle: 185
            bg_end_angle: 265
            start_angle: 185
            end_angle: 265
            # ... other properties same as above
```

### Sources

https://lvgl.io/docs/open/9.5/widgets/arc, https://lvgl.io/docs/open/8.3/widgets/core/arc, https://esphome.io/components/lvgl/widgets/, https://esphome.io/cookbook/lvgl/

---

## 14. LVGL multiple arc widget update performance on ESP32-S3

### Key Findings

Research into LVGL widget update performance, specifically concerning multiple arc widgets on an ESP32-S3 driving a GC9A01A display, reveals several critical factors that impact rendering speed and overall frame rate.

**1. The "Math Problem" of Vector Graphics**
LVGL uses a vector engine (like ThorVG) to decode and render paths for widgets like arcs. This process is computationally intensive, relying heavily on calculating Bezier curves. The ESP32-S3's default CPU frequency of 160MHz is often insufficient for smooth animation of complex vector graphics, resulting in low frame rates (e.g., 7-9 FPS). Increasing the CPU frequency to 240MHz and enabling `-O3` compiler optimizations are essential first steps to provide the vector engine with the necessary processing power.

**2. SPI Bus Bottlenecks**
The GC9A01A display is typically driven over an SPI bus. A default SPI clock speed of 20MHz is a significant bottleneck. Increasing the SPI clock to 80MHz (the absolute limit for the ESP32-S3's SPI bus) is crucial for maximizing data transfer rates. Lower speeds can cause visible "tearing" as the bus struggles to keep up with frame updates.

**3. Buffer Strategies and Tiling Overhead**
The choice of buffer strategy profoundly affects performance. A naive implementation using a single full-frame buffer in internal SRAM requires a full refresh for every frame, which is slow. 
Implementing double buffering with partial buffers (e.g., 1/10th of the screen size) allows parallel processing: the CPU draws to one buffer while DMA transfers the other to the display. However, this introduces "tiling overhead." Because the buffer is small, the vector engine must recalculate the entire vector path multiple times (once for each strip) to generate the full image. This overhead can sometimes outweigh the benefits of parallel DMA transfer, leading to a regression in frame rate.

**4. The PSRAM Solution**
To eliminate tiling overhead, full-frame buffers are required. Since internal SRAM is limited, these buffers must be allocated in PSRAM. Utilizing Octal PSRAM (if available on the ESP32-S3 variant) allows for two full-frame buffers. This means the vector path is calculated only once per frame, significantly boosting performance (e.g., achieving ~26 FPS for complex animations).

**5. Arc Widget Redraw Behavior**
A specific issue with LVGL arc widgets is that changing the value of an arc can cause the entire rectangular bounding box of the arc to be redrawn, rather than just the updated segment. This behavior occurs even in direct render mode and can severely impact the refresh rate, especially if the arc covers a large portion of the screen. When updating four arcs simultaneously, this redraw area overlap can cause significant performance degradation.

**6. Stack Size Requirements**
Vector graphics rendering involves recursion. The default LVGL task stack size (often 8KB) is insufficient and will likely overflow, causing crashes when scaling or animating complex paths like multiple arcs. Increasing the task stack size to at least 64KB is a necessary precaution.

### Implementation Recommendations

For the VelaDial project using an ESP32-S3 with a 240x240 GC9A01A display running ESPHome and LVGL, implementing the Preset Ring UI with four simultaneous arc widgets requires careful optimization to maintain a smooth frame rate.

1. **Hardware Configuration**: Maximize the ESP32-S3's raw clock speeds. Set the CPU frequency to 240MHz and the SPI bus speed to 80MHz. This provides the vector engine with more cycles to calculate Bezier curves for the arcs and ensures the display highway speed is maximized. Enable `-O3` compiler optimizations to speed up the vector math engine.

2. **Buffer Strategy**: Implement double buffering using partial buffers. Allocate two separate buffers in the fast internal SRAM (e.g., 1/10th or 1/20th of the screen size). While the hardware DMA is sending Buffer A to the screen, the CPU can immediately start drawing Buffer B. Set the render mode to `PARTIAL` (`LV_DISPLAY_RENDER_MODE_PARTIAL`). This approach decouples the CPU from the display and eliminates screen tearing.

3. **Stack Size**: Increase the LVGL task stack size significantly. Vector graphics, such as arcs, use recursion. The default 8KB stack is prone to overflowing and crashing when scaling complex paths. Increase the stack size to at least 64KB (`65536` bytes) to ensure stability during vector math calculations.

4. **Widget Updates**: Avoid updating all four arcs simultaneously if possible. If only one preset is active, only update the active arc's value and color. If all arcs must be updated, ensure that the updates are batched and flushed efficiently. Be aware that changing an arc's value may cause the entire rectangular area taken up by the arc to be redrawn, even in direct render mode. To mitigate this, consider using smaller, separate arc widgets rather than one large widget that covers the entire screen.

5. **PSRAM Utilization**: If the ESP32-S3 has Octal PSRAM (e.g., 8MB), consider moving the buffers to PSRAM and using full-frame buffers. This eliminates the "tiling overhead" associated with partial buffers, where the vector engine must recalculate the path multiple times for each strip. Full-frame buffers in PSRAM can significantly boost the frame rate (e.g., from ~9 FPS to ~26 FPS for complex vector animations).

### Code/Config Examples

```
# ESPHome YAML optimization for LVGL arcs
lvgl:
  displays:
    - display_id: my_display
  # Use partial render mode for better performance
  render_mode: PARTIAL
  # Allocate two buffers for double buffering
  buffer_size: 20%
  double_buffer: true

# C++ optimization for LVGL porting
lvgl_config.malloc_caps = MALLOC_CAP_DMA | MALLOC_CAP_INTERNAL;
lvgl_config.double_buffered = true;
lvgl_config.render_mode = LV_DISPLAY_RENDER_MODE_PARTIAL;
lvgl_config.task_stack_size = 65536; // Increase stack for vector math
```

### Sources

https://forum.lvgl.io/t/render-time-optimization/23291, https://esphome.io/components/lvgl/widgets/, https://esphome.io/cookbook/lvgl/, https://community.home-assistant.io/t/esphome-lvgl-widget-arc-update-problem/845590, https://forum.lvgl.io/t/changing-arc-value-causes-entire-arc-to-be-redrawn/19798, https://wiki.seeedstudio.com/round_display_animation_workshop/, https://github.com/UsefulElectronics/esp32s3-gc9a01-lvgl

---

## 15. Smart home rotary controller preset selection UI/UX patterns

### Key Findings

Based on research into smart home rotary controllers like the Nest Thermostat, Brilliant Smart Home Control, and Aqara Dial, several key UI/UX patterns emerge for preset and mode selection.

**Nest Thermostat:**
The Nest Thermostat utilizes a physical rotating outer ring (or a touch bar on newer models) to navigate menus. For mode selection (Heat, Cool, Eco, Off), pressing the thermostat face opens a Quick View menu. Rotating the ring highlights different modes around the perimeter or in a list, and pressing the face again confirms the selection. The UI relies heavily on color coding (e.g., orange for heating, blue for cooling, green for Eco) to provide immediate visual feedback of the active mode.

**Brilliant Smart Home Control:**
Brilliant's interface relies more on touch sliders and screens rather than a physical rotary dial. However, their slider preferences allow users to configure "Activate Scenes" through double-tap features on the slider. This indicates a pattern where primary controls (like brightness) are handled by the main interaction (sliding), while secondary controls (like presets or scenes) are accessed via distinct gestures (double-tapping) or dedicated menu pages.

**Aqara Dial (H1/S1E):**
Aqara's rotary dials often support multiple interaction modes. The physical rotation typically controls continuous values like brightness or volume. For discrete selections like scenes or presets, they employ multi-click actions (single press, double press, long press) or distinct pages on touchscreen models (like the S1E).

**Design Patterns for VelaDial:**
The "Preset Ring UI" concept aligns well with these established patterns. Placing options around the circumference mimics the physical rotation of the knob, providing an intuitive mapping between physical action and digital response. Filling the active preset's arc segment with its characteristic color is a strong pattern seen in Nest devices, offering clear, glanceable status information. Using the center of the display for detailed text (preset name, brightness value) maximizes the use of the round screen's geometry.

### Implementation Recommendations

For the VelaDial project using ESPHome and LVGL on a 240x240 GC9A01A display, implement the Preset Ring UI using the LVGL `arc` widget. Set the `arc` width and height to approximately 200-220 pixels to fit comfortably within the 240x240 screen while leaving a small margin. Use the `arc_width` property (e.g., 16-20 pixels) to define the thickness of the ring.

To handle the rotary encoder input, assign the `arc` widget to a specific `group` (e.g., `group: control_group`) and ensure the rotary encoder sensor is configured to interact with this group. The encoder's `on_clockwise` and `on_anticlockwise` triggers can be used to navigate between the 4 presets.

For the 4 presets, configure the `arc` with `min_value: 0` and `max_value: 3`. You can use the `on_value` trigger of the arc to detect when the user rotates the knob to a new preset, and then update the central label (using a `label` widget aligned to `CENTER`) to display the corresponding preset name and brightness. To visually distinguish the active preset, you can dynamically update the `arc_color` or the `indicator` style properties based on the current value.

### Code/Config Examples

```
# ESPHome LVGL Arc Widget Example
- arc:
    width: 200
    height: 200
    align: CENTER
    group: control_group
    arc_width: 16
    id: preset_arc
    value: 0
    min_value: 0
    max_value: 3
    adjustable: true
    on_value:
      - logger.log: "Preset changed"
```

### Sources

Google Nest Help (Nest thermostat temperature modes), Brilliant Support (Slider Preferences), ESPHome LVGL Documentation, GitHub (GinAndBacon/ESPHome-LVGL-EncoderDial)

---

## 16. ESPHome LVGL arc widget on_click touchscreen detection

### Key Findings

When implementing a "Preset Ring UI" on a round display using ESPHome and LVGL, detecting which segment of a ring was tapped presents specific challenges and solutions.

**1. Advanced Hit Testing (`adv_hittest`)**
The most critical finding is the necessity of the `adv_hittest` flag in LVGL. By default, LVGL widgets process touch events based on their rectangular bounding box. For an arc widget, this means touching the empty center or the corners outside the arc would still trigger a click event. Enabling `adv_hittest: true` in the ESPHome YAML configuration forces LVGL to perform a more accurate hit test. For arcs, this ensures that clicks are only recognized on the actual drawn ring of the background arc, allowing the center to be clicked through to widgets underneath.

**2. Handling Partial Arcs**
There is a known behavior in LVGL (discussed in issue #4576) where clicking on the invisible part of the ring (outside the defined start and end angles but within the circular path) can still trigger events. To build a 4-segment preset ring, the best practice is to instantiate four distinct `arc` widgets, each configured with a 90-degree span (e.g., `start_angle: 315`, `end_angle: 45`). By applying `adv_hittest: true` to each, they will only respond to touches on their specific visible segments.

**3. Touch Coordinate Extraction**
While it is possible to extract raw touch coordinates using the `touchscreen` component's `on_touch` or `on_update` triggers (which provide `touch.x` and `touch.y`), calculating the angle mathematically via a lambda function to determine the quadrant is overly complex and unnecessary for this use case. Relying on LVGL's native widget event handling (`on_click` on individual arcs) is much more robust and aligns better with ESPHome's declarative YAML structure.

**4. Arc Configuration for Buttons**
When using arcs as buttons rather than progress bars or sliders, the knob part should be hidden or removed. In LVGL C code, this is done via `lv_obj_remove_style(arc, NULL, LV_PART_KNOB)`. In ESPHome, you can achieve this by setting the knob's opacity to transparent or ensuring the arc is not adjustable, so dragging does not change its value. The `on_click` trigger can then be used purely for selection.

### Implementation Recommendations

For the VelaDial project using a 240x240 GC9A01A round display with ESPHome LVGL, implementing the Preset Ring UI requires specific configurations. 

1. **Use Multiple Arcs:** Instead of trying to calculate the angle from a single full-circle arc's touch coordinates (which is complex in ESPHome's YAML), create four separate `arc` widgets, each representing a 90-degree quadrant (e.g., 315°-45°, 45°-135°, 135°-225°, 225°-315°). 

2. **Enable Advanced Hit Testing:** You MUST set `adv_hittest: true` on each arc widget. By default, LVGL widgets have a rectangular bounding box for touch detection. `adv_hittest` ensures that clicks are only registered when the user touches the actual drawn ring of the arc, not the empty space inside or the corners of its bounding box.

3. **Disable the Knob:** Since these arcs act as buttons rather than sliders, remove the knob styling and disable adjustability. Set `adjustable: false` (or clear the clickable flag for the knob part) so the user cannot drag the arc value.

4. **Touchscreen Calibration:** Ensure your touchscreen component is properly calibrated. Use the `touchscreen` component's `transform` options (`swap_xy`, `mirror_x`, `mirror_y`) if the touch coordinates don't align with the display coordinates.

5. **Visual Feedback:** Use the `on_click` trigger on each arc to change its `value` to maximum (filling the segment with color) while setting the other three arcs' values to minimum, providing immediate visual feedback of the active preset.

### Code/Config Examples

```
# ESPHome YAML example for 4 arc segments
lvgl:
  touchscreens:
    - touchscreen_id: my_touch
      transform:
        swap_xy: false
  pages:
    - id: presets_page
      widgets:
        - arc:
            id: arc_preset_1
            start_angle: 315
            end_angle: 45
            adv_hittest: true
            on_click:
              then:
                - logger.log: "Preset 1 tapped"
        - arc:
            id: arc_preset_2
            start_angle: 45
            end_angle: 135
            adv_hittest: true
            on_click:
              then:
                - logger.log: "Preset 2 tapped"
```

### Sources

https://esphome.io/components/lvgl/widgets/, https://lvgl.io/docs/open/8.3/widgets/core/arc, https://forum.lvgl.io/t/arc-clickable-area-outside-of-the-draw/13032, https://github.com/lvgl/lvgl/issues/4576, https://esphome.io/components/touchscreen/

---

## 17. Circular progress indicators with multiple segments - segmented ring charts, donut chart UI patterns for embedded displays

### Key Findings

Research into circular progress indicators and segmented ring charts for embedded displays using LVGL reveals several key patterns and best practices, particularly relevant for the VelaDial project's 240x240 round display.

**1. Widget Selection: Meter vs. Arc**
While the standard `arc` widget is suitable for simple progress bars (like the Brightness page), creating a multi-segment ring (like the Presets page) is best achieved using the `meter` widget. The `meter` widget in LVGL is designed to visualize data flexibly and supports adding multiple `arc` indicators to a single scale. This is far more efficient and easier to manage than attempting to overlay and align multiple individual `arc` widgets.

**2. Scale and Coordinate System**
In LVGL, the default 0-degree position is at 3 o'clock (middle right), and angles increase clockwise. When using a `meter`, you define a scale (e.g., 0 to 100) and an `angle_range` (e.g., 360 for a full circle). You can apply a `rotation` offset (e.g., 270 degrees) to shift the starting point to 12 o'clock (top), which is standard for UI dials.

**3. Creating Segments**
To create the four distinct preset segments, you add four `arc` indicators to the meter's scale. By mapping these to specific value ranges on a 0-100 scale, you can position them in the four quadrants. For example:
- Quadrant 1: values 0-25
- Quadrant 2: values 25-50
- Quadrant 3: values 50-75
- Quadrant 4: values 75-100
To create visual separation (the "segmented" look), you introduce gaps between the start and end values of each arc (e.g., 5-20, 30-45, 55-70, 80-95).

**4. Dynamic Styling and Interaction**
The active preset can be highlighted by changing the color or opacity of its corresponding arc indicator dynamically. In ESPHome, this is done using the `lvgl.indicator.update` action within automations triggered by the rotary encoder. Inactive segments can be drawn with a muted background color (e.g., dark gray), while the active segment is filled with its characteristic color.

**5. Center Content Placement**
For the center display (preset name and brightness), a `label` widget should be aligned to the `CENTER` of the screen or the parent container. Since the `meter` widget's arcs are drawn at the outer radius, the center remains empty, providing a perfect canvas for text.

**6. Performance Considerations**
Updating UI elements continuously (e.g., while rapidly turning the rotary encoder) can impact performance on the ESP32-S3. It is recommended to use efficient update actions and avoid redrawing the entire screen unnecessarily. For smooth operation, ensure that only the necessary indicators and labels are updated during encoder events.

### Implementation Recommendations

For the VelaDial project using an ESP32-S3 with a 240x240 GC9A01A round display and ESPHome LVGL, implement the "Preset Ring UI" using the `meter` widget rather than multiple `arc` widgets. The `meter` widget is highly optimized for circular displays and allows stacking multiple indicators (arcs) on a single scale.

1. **Base Configuration**: Create a `meter` widget sized 240x240 aligned to `CENTER`. Set the scale `range_from: 0` to `range_to: 100` with an `angle_range: 360`. Use `rotation: 270` to start the 0 value at the top (12 o'clock position). Disable ticks by setting `count: 0`.

2. **Preset Segments**: Add four `arc` indicators to the meter's scale. Map them to quadrants using the 0-100 scale:
   - Preset 1 (Top Right): `start_value: 5`, `end_value: 20`
   - Preset 2 (Bottom Right): `start_value: 30`, `end_value: 45`
   - Preset 3 (Bottom Left): `start_value: 55`, `end_value: 70`
   - Preset 4 (Top Left): `start_value: 80`, `end_value: 95`
   Leave gaps (e.g., 0-5, 20-30) between segments to create the distinct "segmented ring" look. Set the arc `width` to around 20-30 pixels to ensure touch/visibility without crowding the center.

3. **Dynamic Updates**: Use ESPHome lambda actions (`lvgl.indicator.update`) triggered by the rotary encoder to change the active preset. When a preset is selected, update its arc color to its characteristic color (e.g., `0xFF0000` for red) and dim the inactive arcs (e.g., `0x333333`).

4. **Center Content**: Place a `label` widget inside the meter (or as a sibling aligned to `CENTER`) to display the preset name and brightness. Update this label dynamically when the rotary encoder changes the selection.

5. **Pages**: For the Power and Brightness pages, use a standard `arc` widget or a `meter` with a single arc indicator. For Brightness, map the 0-255 brightness value to the 0-100 scale of the arc.

### Code/Config Examples

```
# ESPHome LVGL Segmented Ring Example
lvgl:
  pages:
    - id: preset_ring_page
      widgets:
        - meter:
            id: preset_meter
            align: CENTER
            width: 240
            height: 240
            scales:
              - range_from: 0
                range_to: 100
                angle_range: 360
                rotation: 270 # Start at top
                ticks:
                  count: 0
                indicators:
                  # Preset 1 (Top Right)
                  - arc:
                      id: preset_1_arc
                      color: 0xFF0000 # Red
                      width: 20
                      start_value: 5
                      end_value: 20
                  # Preset 2 (Bottom Right)
                  - arc:
                      id: preset_2_arc
                      color: 0x00FF00 # Green
                      width: 20
                      start_value: 30
                      end_value: 45
                  # Preset 3 (Bottom Left)
                  - arc:
                      id: preset_3_arc
                      color: 0x0000FF # Blue
                      width: 20
                      start_value: 55
                      end_value: 70
                  # Preset 4 (Top Left)
                  - arc:
                      id: preset_4_arc
                      color: 0xFFFF00 # Yellow
                      width: 20
                      start_value: 80
                      end_value: 95
        - label:
            id: center_text
            align: CENTER
            text: "Preset 1\n100%"
            text_align: CENTER
```

### Sources

https://lvgl.io/docs/open/8.3/widgets/core/arc, https://lvgl.io/docs/open/8.3/widgets/extra/meter, https://esphome.io/components/lvgl/widgets/, https://esphome.io/cookbook/lvgl/

---

## 18. Premium watch complication design and Garmin Venu circular widget patterns

### Key Findings

Research into premium watch complication design and Garmin Venu circular widget patterns reveals several key design principles applicable to the Preset Ring UI concept. 

**Radial Information Architecture:** Circular UIs, such as those on the Garmin Venu and luxury watch faces, utilize radial patterns to maximize the use of the round display. Information is placed along the perimeter, drawing the eye around the dial. This is highly effective for the Preset Ring UI, where the four presets act as complications placed at the cardinal points (top, bottom, left, right) or quadrants.

**Arc Segments as Indicators:** Garmin's UI frequently uses arc segments along the bezel to indicate progress, status, or categories. In the context of the VelaDial, using four distinct arc segments for the presets provides a clear visual mapping. When a preset is selected, filling its respective arc segment with a characteristic color provides immediate, glanceable feedback, mimicking the behavior of active complications on a digital watch face.

**Central Focus Area:** Both luxury watch faces and smartwatch UIs reserve the center of the display for the most critical information (e.g., the time, or the primary metric of a widget). For the VelaDial, the center should prominently display the active preset's name and its current brightness level. This creates a clear hierarchy: the perimeter provides context and navigation options, while the center provides the current state.

**Visual Separation and Margins:** Premium designs avoid clutter by maintaining clear margins. On a 240x240 display, placing elements too close to the edge can result in clipping or a cramped appearance. Leaving a margin (e.g., 10-15 pixels) around the outer edge and introducing small gaps between the arc segments enhances the premium feel and readability.

**Contextual Pages:** The Garmin Venu uses a widget loop to navigate through different data screens. The VelaDial's 3-page structure (Power, Brightness, Presets) mirrors this pattern. The transition between these pages should be distinct. For example, the Power page can use the entire ring as a binary indicator (full brightness for ON, dim for OFF), while the Brightness page uses a continuous arc proportional to the level, and the Presets page uses the segmented quadrant layout. This contextual use of the ring maximizes the utility of the limited screen real estate.

### Implementation Recommendations

For the ESPHome LVGL implementation on the 240x240 GC9A01A display, use the `lv_arc` widget to create the four preset segments. Since the display is 240x240, the arcs should have a radius of approximately 110 pixels to leave a 10-pixel margin from the edge, preventing clipping. Set the `arc_width` to 15-20 pixels to ensure they are easily visible but leave enough room in the center (approx 180x180 pixels) for the preset name and brightness value.

Configure four separate `lv_arc` widgets, each with a 90-degree span (e.g., 315-45, 45-135, 135-225, 225-315), minus a small gap (e.g., 5 degrees) between them for visual separation. Use the `LV_PART_INDICATOR` to fill the arc with the preset's characteristic color when active, and `LV_PART_MAIN` for the background of inactive presets.

For the rotary encoder navigation, map the encoder's value to the active preset index (0-3). When the index changes, update the `value` of the corresponding arc to 100 (filled) and the others to 0 (empty). For the Brightness page, use a single continuous arc from 135 to 45 degrees (270-degree span) and map the brightness value (0-255) to the arc's range (0-100). Enable the `adv_hittest` flag on the arcs if touch interaction is added later, to ensure accurate hit detection on the curved segments.

### Code/Config Examples

```
# ESPHome LVGL Arc Configuration Example
lvgl:
  displays:
    - display_id: my_display
  widgets:
    - arc:
        id: preset_arc_1
        x: 0
        y: 0
        width: 240
        height: 240
        bg_angles: [225, 315] # Top quadrant
        value: 100
        styles:
          main:
            bg_opa: 0
            arc_width: 20
            arc_color: 0x333333
          indicator:
            arc_width: 20
            arc_color: 0xFF5733 # Preset color
        flags:
          - adv_hittest

```

### Sources

https://lvgl.io/docs/open/8.3/widgets/core/arc, https://esphome.io/components/lvgl/widgets/, https://www.sitepoint.com/smartwatch-ui-design-battle-circles-vs-squares/, https://www8.garmin.com/manuals/webhelp/venu/EN-US/GUID-9DDEE202-9F27-4EA5-BD80-3ADEB0F0C806.html

---

## 19. LVGL color animation between states

### Key Findings

Research into LVGL color animations and pulse effects for the VelaDial project revealed several key technical findings and best practices:

1. **LVGL Animation System**: LVGL provides a robust animation system (`lv_anim_t`) that can automatically change the value of a variable between a start and end value over time. This system is highly flexible and can animate almost any property, including coordinates, dimensions, and custom values via callback functions.

2. **Style Transitions**: For color changes between states (e.g., dim to bright when a preset is selected), LVGL's style transitions are the recommended approach. Transitions allow properties like `LV_STYLE_ARC_COLOR` or `LV_STYLE_BG_COLOR` to animate smoothly when an object's state changes (e.g., from `LV_STATE_DEFAULT` to `LV_STATE_CHECKED`). This is more efficient and easier to manage than manually creating animations for color properties.

3. **Arc Widget Specifics**: The `lv_arc` widget consists of a background (`LV_PART_MAIN`), an indicator (`LV_PART_INDICATOR`), and a knob (`LV_PART_KNOB`). To animate the color of the arc segment representing a preset, the transition or animation must specifically target the `LV_PART_INDICATOR`.

4. **Pulse Animation Pattern**: A pulse confirmation animation can be achieved by animating the size (width/height) or scale of the arc widget. By setting a playback time (`lv_anim_set_playback_time`), the animation will automatically reverse, creating a pulse effect. Using easing paths like `lv_anim_path_overshoot` adds a dynamic, bouncy feel to the pulse.

5. **ESPHome Limitations and Workarounds**: ESPHome's native LVGL component provides excellent YAML-based configuration for static layouts and basic interactions. However, advanced animations and style transitions often require dropping into C++ lambdas. The ESPHome LVGL component allows referencing widget objects by ID, making it possible to apply custom LVGL C API calls (like `lv_style_transition_dsc_init` and `lv_anim_start`) directly to the widgets defined in YAML.

### Implementation Recommendations

For the VelaDial project using ESPHome LVGL on a 240x240 round display (GC9A01A) with an ESP32-S3, implementing color animations and pulse effects requires a combination of LVGL's animation system and style transitions.

1. **Color Animation (Dim to Bright)**: Use LVGL's style transitions to animate the arc color. Define a transition descriptor (`lv_style_transition_dsc_t`) that targets the `LV_STYLE_ARC_COLOR` property. Apply this transition to the arc's indicator part (`LV_PART_INDICATOR`). When the preset is selected (e.g., state changes to `LV_STATE_CHECKED`), the color will smoothly transition from the dim state to the bright state over the specified duration (e.g., 300ms).

2. **Pulse Confirmation Animation**: To create a pulse effect upon selection, use LVGL's animation system (`lv_anim_t`). Create an animation that targets the arc's width or radius, expanding it slightly and then returning to its original size. Use the `lv_anim_path_overshoot` or `lv_anim_path_ease_in_out` path for a natural feel. Set the animation to play back (`lv_anim_set_playback_time`) to complete the pulse.

3. **ESPHome Integration**: While ESPHome's YAML configuration supports basic LVGL widgets and styles, complex animations and transitions often require custom C++ code via lambdas. Use the `lvgl.widget.update` action or custom lambdas to trigger state changes and animations based on rotary encoder input.

4. **Performance Considerations**: The ESP32-S3 is capable, but the 240x240 display requires efficient rendering. Ensure `auto_clear_enabled: false` is set in the display config. Use PSRAM if available, and allocate a sufficient buffer size (e.g., 25% or more) for smooth animations. Avoid animating too many properties simultaneously to prevent frame drops.

### Code/Config Examples

```
# Example ESPHome LVGL config for arc color transition
lvgl:
  widgets:
    - arc:
        id: preset_arc
        align: CENTER
        width: 200
        height: 200
        indicator:
          arc_color: 0x0000FF # Dim color
        state:
          checked:
            indicator:
              arc_color: 0x00FFFF # Bright color
        # Transition properties can be set via C code or lambda

```

### Sources

https://lvgl.io/docs/open/8.3/overview/animation, https://esphome.io/components/lvgl/, https://esphome.io/components/lvgl/widgets/, https://forum.lvgl.io/t/changing-the-color-of-an-arc-indicator-seems-to-not-work-for-me/11778, https://lvgl.io/docs/open/8.3/overview/style.html

---

## 20. Dark room UI colored arc segments visibility and minimum brightness

### Key Findings

Research into dark room UI design, specifically focusing on colored arc segments on black or near-black backgrounds, reveals several critical considerations for visibility, contrast, and user experience.

**1. The "Pure Black" Problem**
While pure black (hex #000000) is often used in dark modes to save power on OLED screens, it is generally discouraged for UI backgrounds. Pure black creates extreme contrast with bright elements, leading to eye strain and a phenomenon known as "halation," where bright text or colored elements appear to glow or bleed into the dark background. This is especially problematic in dark rooms where the user's pupils are dilated. Furthermore, on OLED screens, transitioning pixels from completely off (pure black) to on can cause a "smearing" effect during motion. Best practice dictates using a very dark gray (such as Material Design's recommended #121212) as the base background color. This reduces eye strain, minimizes halation, and allows for the use of elevation (lighter grays) to indicate depth and hierarchy.

**2. Color Saturation and Contrast**
Highly saturated colors, which might look vibrant on a light background, often vibrate visually against dark backgrounds, making them difficult to look at and reducing readability. In dark UI design, colors should be desaturated. Lighter, pastel-like tones (often referred to as the 200-level colors in Material Design) provide better visibility and meet accessibility standards. The Web Content Accessibility Guidelines (WCAG) require a minimum contrast ratio of 4.5:1 for normal text and UI elements against their background. When designing colored arc segments, the chosen colors must meet this contrast requirement against the dark gray background to ensure they are easily identifiable.

**3. Minimum Brightness for Identification**
In a completely dark room, the human eye becomes highly sensitive to light. A display running at 100% brightness will be blinding and ruin night vision. However, if the brightness is too low, colors lose their perceived saturation (the Purkinje effect), making it difficult to distinguish between different colored arc segments. The minimum brightness required for color identification depends on the specific display hardware (in this case, the GC9A01A IPS LCD). Generally, a backlight PWM duty cycle of 10% to 15% is the minimum threshold where colors remain distinct without causing discomfort in a dark environment. Below this level, colors may appear muddy or indistinguishable from gray.

**4. LVGL Arc Widget Implementation**
In LVGL (Light and Versatile Graphics Library), the `arc` widget is ideal for creating the Preset Ring UI. An arc consists of a background (the track) and a foreground (the indicator). For the VelaDial concept, four separate arc widgets can be used, each configured with specific `start_angle` and `end_angle` parameters to occupy a quadrant of the screen. The `arc_width` property controls the thickness of the ring. To indicate the active preset, the `indicator_color` of the selected arc can be changed dynamically, while the inactive arcs remain a dim, neutral color. ESPHome's LVGL component supports these properties, allowing for precise control over the UI layout and styling directly via YAML configuration.

### Implementation Recommendations

For the VelaDial project using a 240x240 GC9A01A round display with ESPHome and LVGL, implement the Preset Ring UI with careful attention to contrast and visibility. 

1. **Background Color**: Avoid using pure black (0x000000) for the background. While the GC9A01A is an IPS LCD and not a true OLED, it has good contrast. Use a very dark gray (e.g., 0x121212 or 0x1A1A1A) to reduce eye strain in dark rooms and prevent "smearing" effects when transitioning colors.

2. **Arc Segments**: Configure four separate LVGL `arc` widgets positioned at the quadrants. For a 240x240 display, set the width and height to 240, and center them (x:0, y:0). Use `start_angle` and `end_angle` to define the quadrants (e.g., 315-45, 45-135, 135-225, 225-315). Leave a small gap (e.g., 5 degrees) between segments for visual separation.

3. **Arc Width**: Set `arc_width` and `bg_arc_width` to around 15-25 pixels. This provides enough surface area for the color to be visible without overwhelming the central text area.

4. **Brightness and Color**: For dark room visibility, highly saturated colors on dark backgrounds can cause halation (glowing effect) and reduce readability. Desaturate the preset colors slightly (e.g., use pastel or muted tones) and ensure a minimum contrast ratio of 4.5:1 against the background. The minimum brightness for the display backlight should be set to around 10-15% via PWM to ensure the colored arcs are identifiable without being blinding in a dark room.

5. **Active State**: When a preset is selected via the rotary encoder, fill the `indicator_color` of the active arc segment. Keep the inactive segments in a dim, neutral color (e.g., 0x333333) to maintain the ring shape context without distracting from the active selection.

### Code/Config Examples

```
# ESPHome LVGL Arc Configuration Example
lvgl:
  - id: preset_ring
    pages:
      - id: page_presets
        widgets:
          - arc:
              id: arc_preset_1
              x: 0
              y: 0
              width: 240
              height: 240
              start_angle: 225
              end_angle: 315
              bg_color: 0x1A1A1A # Dark gray background, not pure black
              indicator_color: 0xFF5733 # Preset color
              arc_width: 20
              bg_arc_width: 20
              adjustable: false

```

### Sources

https://m2.material.io/design/color/dark-theme.html, https://www.nngroup.com/articles/dark-mode-users-issues/, https://www.jamesrobinson.io/post/a-guide-to-dark-mode-design, https://esphome.io/components/lvgl/widgets/, https://esphome.io/cookbook/lvgl/

---

