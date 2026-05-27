# Concept 04: Single-Page Simple Mode — Research

## Research Overview

This document captures the findings from 20-thread parallel internet research conducted for UI Concept 04 (Single-Page Simple Mode). The concept places ALL controls (power, brightness, presets) on a single screen with no page navigation — the most radical simplification in the 20-concept matrix.

**IMPORTANT NOTE:** This concept FAILS Gate G1 (Three-Page Lock) per the selection criteria. It is prototyped for exploration purposes only and cannot be selected for production without explicit Hardik approval to waive the 3-page requirement.

---

## Thread 1: Single-page smart home controller UI design for round displays, focusing on minimal interfaces with all controls visible at once and no navigation.

### Key Findings

The concept of a "Single-Page Simple Mode" for smart home controllers emphasizes radical simplification by placing all essential controls on a single screen without any navigation menus or sub-pages. This approach is highly effective for quick, glanceable interactions, which are crucial for wearable and small-screen devices. Key design principles include prioritizing single-action interfaces, using bold and high-contrast colors for visibility in various lighting conditions, and ensuring that touch targets are large enough for finger taps. The "three-level rule" is often applied, limiting content to primary information (e.g., main action or status), secondary details (e.g., quick stats), and tertiary elements (e.g., minor settings), ensuring the interface remains uncluttered.

For round displays, the UI must account for the circular shape, which naturally draws the eye to the center. Essential controls should be centrally located, while secondary information can be placed along the curved edges. The use of arcs and circular sliders is particularly effective for adjusting values like brightness or volume, as they follow the screen's contour and maximize the use of available space. Typography must be highly legible, favoring sans-serif fonts with adequate weight and contrast, as small screens demand immediate readability.

Implementation on small screens also requires careful consideration of touch gestures. Since traditional tap-based interfaces can be challenging on tiny displays, incorporating swipe gestures or utilizing physical buttons (if available) can enhance usability. The goal is to create an interface that feels invisible, delivering exactly what the user needs without requiring them to think about how to navigate it.

### Reference URLs

- https://esphome.io/cookbook/lvgl/
- https://community.home-assistant.io/t/2424s012-round-display-lvgl/868243
- https://uxdesign.cc/your-guide-to-smartwatch-interface-design-designing-for-all-1a588a6a1181
- https://weareaffective.com/learning-centre/how-do-you-design-user-interfaces-for-tiny-wearable-screens

### LVGL Applicability

Applying the single-page concept to a 1.28" round 240x240 ESP32-S3 display using LVGL and ESPHome involves leveraging specific LVGL widgets optimized for circular interfaces. The `arc` widget is ideal for creating circular sliders to control light brightness or volume, fitting perfectly along the display's edge. Central placement of a `button` or `switch` widget can serve as the primary power toggle. By utilizing LVGL's styling capabilities, these widgets can be customized with high-contrast colors and bold fonts to ensure visibility.

In ESPHome, the integration of LVGL allows for direct binding of Home Assistant entities to UI elements. For instance, a light's brightness attribute can be mapped to an `arc` widget, updating in real-time as the state changes. It is crucial to handle the conversion between Home Assistant's value ranges (e.g., 0-255 for brightness) and LVGL's integer-based widget values. Additionally, using the `adv_hittest` option on sliders and arcs prevents accidental touches from causing sudden changes, which is vital on a small 240x240 touch area.

Performance optimization is also key when running LVGL on an ESP32-S3. While the S3 is capable, continuous updates from dragging a slider can impact performance. Using triggers like `on_release` instead of `on_value` for action calls ensures that commands are sent only after the interaction is complete, reducing the load on the microcontroller and ensuring a smooth user experience.

---

## Thread 2: Information density on small round displays — fitting power state, brightness value, and preset name within 240x240 circular viewport.

### Key Findings

Designing a "Single-Page Simple Mode" for a 1.28" 240x240 round display requires careful management of information density due to the circular viewport, which inherently clips the corners of a traditional rectangular layout. The primary challenge is maximizing legibility while fitting three distinct data points: power state, brightness value, and preset name.

A common and effective approach for circular UI is the "concentric layout." The outermost edge is utilized for a circular progress bar (arc) representing the brightness value, which naturally conforms to the display's shape and saves central space. The center of the screen, offering the widest horizontal area, is ideal for the most critical information, such as large, bold typography displaying the exact brightness percentage or the current preset name.

Power state can be integrated as a color-coded background or a prominent icon in the upper or lower central zones. For instance, a glowing accent color or a distinct toggle icon at the bottom can indicate the power status without cluttering the text. To avoid visual overload, minimalist typography and high-contrast color schemes are essential. Anti-aliasing is crucial on a 240x240 resolution to prevent jagged edges on curved elements like arcs and large fonts.

### Reference URLs

- https://www.panoxdisplay.com/solution/what-is-long-strip-stretched-display-panel
- https://community.home-assistant.io/t/1-28-inch-240-240-esp32c3-round-display-with-rotary-knob-uedx24240013-md50e-by-viewe-company/786687
- https://done.land/components/humaninterface/display/tft/gc9a01/1.28inch240x240round/

### LVGL Applicability

For an ESP32-S3 running ESPHome and LVGL, implementing this single-page UI is highly feasible and performant. LVGL provides native support for circular elements, specifically the `lv_arc` widget, which is perfect for the outer brightness indicator. The arc can be customized with gradient colors and rounded ends to look modern, and its value can be directly bound to the ESPHome light component's brightness state.

The central text can be rendered using `lv_label` with a custom, anti-aliased font (e.g., Montserrat or Roboto) converted via LVGL's font converter. Since the ESP32-S3 has ample PSRAM and processing power, it can easily handle custom fonts and smooth animations for the arc. The power state can be managed by dynamically changing the style of the central elements or the background color using `lv_obj_set_style_bg_color`.

To ensure the UI remains responsive, it is recommended to use LVGL's alignment features (`LV_ALIGN_CENTER`, `LV_ALIGN_BOTTOM_MID`) rather than absolute positioning. This ensures elements stay perfectly centered within the 240x240 circular mask. ESPHome's LVGL component allows seamless integration of these UI elements with Home Assistant entities, enabling real-time updates of the preset name and brightness without complex C++ coding.

---

## Thread 3: LVGL touch zone detection and dividing a round display into tap regions using lv_obj click areas for ESPHome on ESP32-S3.

### Key Findings

LVGL provides multiple mechanisms for handling touch zone detection and dividing a display into specific tap regions, which is particularly useful for a "Single-Page Simple Mode" on a round display. The most straightforward approach is to use invisible or transparent objects (`lv_obj`) positioned over the desired areas. By creating a base object and setting its background opacity to transparent (`LV_OPA_TRANSP` or `bg_opa: transp` in ESPHome) and removing borders, you can create a clickable region that does not visually obscure the UI elements beneath it. These transparent objects can be positioned at the top, center, and bottom of the screen to act as distinct touch zones.

Another critical feature in LVGL is the extended click area (`ext_click_area`). This allows an object's touchable region to be larger than its visual bounding box. Using `lv_obj_set_ext_click_area(obj, size)` in C, or the `ext_click_area` property in ESPHome, you can expand the sensitive area around a widget. This is highly beneficial for small buttons or specific UI elements where precision tapping on a small round screen might be difficult.

For more complex or non-rectangular touch zones, LVGL supports advanced hit testing via the `LV_EVENT_HIT_TEST` event. By intercepting this event, you can implement custom logic to determine if a touch point falls within a specific geometric shape, such as a circular sector or a custom polygon, rather than just the default rectangular bounding box. This is particularly relevant for round displays where rectangular touch zones might overlap awkwardly near the edges.

When implementing this in ESPHome, the `lvgl` component allows you to define these transparent touch zones using the `obj` or `button` widgets. You can set `bg_opa: transp` and `border_width: 0` to make them invisible. You can then use the `on_click` trigger to execute Home Assistant actions or update other UI elements. The `ext_click_area` property is also directly supported in ESPHome's YAML configuration, making it easy to enlarge the touch targets of existing widgets without writing custom C code.

### Reference URLs

- https://esphome.io/cookbook/lvgl/
- https://esphome.io/components/lvgl/widgets/
- https://forum.lvgl.io/t/transparent-button-or-obj-on-top-of-other-elements/12331
- https://forum.lvgl.io/t/increase-the-touch-area-of-an-object/2920
- https://lvgl.io/docs/open/8.3/widgets/obj

### LVGL Applicability

For the VelaDial project using a 1.28" round 240x240 ESP32-S3 display with ESPHome, implementing a "Single-Page Simple Mode" can be effectively achieved using transparent LVGL objects. You can divide the 240x240 screen into three horizontal bands (top, center, bottom) by creating three invisible `button` or `obj` widgets. For example, the top zone could be an object positioned at `y: 0` with a height of 80px, the center at `y: 80` with a height of 80px, and the bottom at `y: 160` with a height of 80px. All three would have `width: 240`, `bg_opa: transp`, and `border_width: 0`.

In ESPHome, these transparent objects will capture touch events while allowing the underlying UI (such as a large central value or background graphics) to remain visible. You can attach `on_click` or `on_press` triggers to these invisible zones to control power (e.g., top zone), brightness (e.g., center zone), and presets (e.g., bottom zone). This approach avoids the need for complex hit-testing logic and leverages the standard ESPHome LVGL configuration.

Additionally, if you have specific visual icons or small indicators within these zones, you can use the `ext_click_area` property on those specific widgets to make them easier to tap without needing full-width transparent overlays. However, for a radical "Single-Page Simple Mode" where the entire top/center/bottom regions act as massive buttons, the transparent overlay method is the most robust and easiest to configure within ESPHome's YAML structure.

---

## Thread 4: Horizontal band layout on circular display — three-zone vertical stacking with curved dividers that follow round bezel geometry

### Key Findings

The "Single-Page Simple Mode" concept for a circular display involves a horizontal band layout, typically divided into a three-zone vertical stack. This design pattern is highly effective for round screens, as it maximizes the usable horizontal space in the center while accommodating the curved geometry at the top and bottom. 

In this layout, the central zone is the widest and most prominent, making it ideal for primary controls or critical information, such as a large power toggle or the current temperature. The top and bottom zones are naturally constrained by the circular bezel, making them suitable for secondary controls, status indicators, or curved sliders (arcs) that follow the screen's edge. Curved dividers can be used to visually separate these zones, enhancing the aesthetic appeal and guiding the user's focus.

This approach eliminates the need for page navigation, aligning perfectly with the goal of radical simplification. By placing all necessary controls—power, brightness, and presets—on a single screen, users can interact with the device instantly without swiping or navigating menus. The use of arcs for adjustments like brightness or volume is particularly intuitive on a round display, as it mimics the physical rotation of a dial.

### Reference URLs

- https://esphome.io/cookbook/lvgl/
- https://esphome.io/components/lvgl/
- https://esphome.io/components/lvgl/widgets/

### LVGL Applicability

Applying this layout to a 1.28" 240x240 round display using LVGL on an ESP32-S3 with ESPHome requires careful use of LVGL's widget positioning and styling capabilities. Since LVGL operates on a rectangular coordinate system, the circular nature of the display must be managed through design rather than hardware constraints. 

To implement the three-zone layout, you can use LVGL `obj` (container) widgets to define the top, middle, and bottom areas. The central zone can house a large `button` or `switch` for the primary action (e.g., power). For the top and bottom zones, `arc` widgets are ideal for controls like brightness or presets, as they can be styled to follow the curvature of the screen. Curved dividers can be created using `line` widgets with custom styling or by using background images with transparent areas.

In ESPHome, this is configured within the `lvgl` component. You will define the widgets and their positions using absolute coordinates (`x` and `y`) or alignment options (`align: CENTER`, `align: TOP_MID`, etc.). It is crucial to account for the display's `240x240` resolution, ensuring that widgets in the top and bottom zones do not extend beyond the visible circular area. The `adv_hittest` property should be enabled for interactive widgets near the edges to ensure accurate touch detection despite the curved boundaries.

---

## Thread 5: Minimal smart home UI inspiration for single-screen dashboards on round displays, focusing on Xiaomi Aqara, Google Nest, and DIY ESP32-S3 LVGL implementations.

### Key Findings

The research into minimal smart home UI inspiration for single-screen dashboards on round displays reveals several key insights. A prominent example is the Google Nest Thermostat (Gen 1), which utilizes a highly simplified interface where the dial is the primary method of interaction, minimizing the need for complex on-screen navigation. The design focuses on essential information, such as current temperature and target temperature, displayed prominently in the center. However, critiques of this design suggest that while visually striking, the lack of tactile buttons can lead to usability issues, such as overshooting desired settings.

Another significant source of inspiration is the DIY community's approach to custom dashboards, particularly using the "Rounded" theme in Home Assistant. This approach emphasizes a clean, intuitive look that significantly improves the "Wife Acceptance Factor" (WAF). Key features include setting up zones by rooms with simple, recognizable icons and integrating specific controls like a "Now Playing" card for media. This demonstrates that a minimal UI does not mean sacrificing functionality; rather, it involves organizing controls logically and using visual cues effectively.

In the context of DIY hardware, projects utilizing the ESP32-S3 with a 1.28" GC9A01 circular display (240x240 resolution) have successfully implemented rotary and touch interfaces. These projects often use LVGL (Light and Versatile Graphics Library) to create smooth, dynamic UI widgets such as labels, buttons, and meters. The interface typically involves a multi-page rotary setup where the encoder click acts as confirmation, and the touch screen allows for direct interaction. This combination of physical and digital controls provides a robust solution for a single-screen dashboard.

### Reference URLs

- https://medium.com/@tinamar/a-beautiful-device-with-a-bad-experience-my-redesign-for-google-nest-thermostat-gen-1-64b0e76ba0bd
- https://forum.aqara.com/t/diy-dashboard-magic-how-i-transformed-my-home-with-rounded/1681
- https://www.reddit.com/r/homeassistant/comments/1mb446q/diy_rotary_touch_controller_for_home_assistant/

### LVGL Applicability

Applying these findings to a 1.28" round 240x240 ESP32-S3 display with LVGL involves several technical considerations. The "Single-Page Simple Mode" concept requires placing all essential controls—such as power, brightness, and presets—on a single screen without navigation. In LVGL, this can be achieved by utilizing an arc or meter widget around the perimeter of the display to represent adjustable values like brightness or temperature, similar to the Nest Thermostat's dial interface.

For ESPHome integration, the `lvgl:` component is essential for defining dynamic UI widgets. You can use `lv_label_set_text_fmt()` and `lv_obj_align()` to update labels in real-time as the user interacts with the touch screen or a rotary encoder. The central area of the 240x240 display should be reserved for the most critical information or the primary toggle button (e.g., power on/off), ensuring it is easily accessible and readable.

To maintain the radical simplification of the UI, avoid using tab views or complex menus. Instead, rely on touch events to trigger state changes directly. For instance, tapping the center could toggle power, while sliding along the edge could adjust brightness. This approach leverages the capabilities of the ESP32-S3 and LVGL to deliver a fluid, responsive, and highly intuitive user experience that aligns with the minimal smart home aesthetic.

---

## Thread 6: ESPHome LVGL single-page implementation with arc widget, multiple labels, and touch zones

### Key Findings

Based on the research into ESPHome's LVGL implementation, creating a single-page UI with multiple labels, an arc widget, and touch-clickable zones is fully supported and can be implemented entirely in YAML.

**Single-Page Architecture**
In ESPHome LVGL, pages are implemented as LVGL screens, which are special objects with no parent. For a single-page design, you simply define one page under the `pages` configuration block. If neither `pages` nor `widgets` is specified, a default "hello world" page is shown. By defining a single page, you eliminate the need for navigation logic or memory overhead associated with multiple screens.

**Arc Widget Implementation**
The `arc` widget in ESPHome LVGL is highly customizable and consists of three main parts: `main` (background), `indicator` (foreground), and `knob` (handle). It can be used to display values or act as an interactive control (e.g., for brightness or volume). The arc supports different modes such as `NORMAL`, `REVERSE`, and `SYMMETRICAL`. For touch interaction, the arc can be made adjustable by the user. A key feature for touch interfaces is the `adv_hittest` property, which allows for more accurate hit testing, particularly useful for curved shapes like arcs. Additionally, the `ext_click_area` property can be used to enlarge the touchable area around the widget, making it easier to interact with on small screens.

**Multiple Labels and Clickable Zones**
Labels can be easily added to the page to display text or dynamic values from Home Assistant sensors. To create clickable zones without visible buttons, you can use invisible `obj` (object) widgets or transparent buttons placed over specific areas of the screen. These objects can have the `clickable: true` property set and can trigger actions using the `on_click` event. The `hidden` property can also be used to manage visibility, but for clickable zones, setting the background opacity (`bg_opa`) to transparent is often more effective.

**Event Handling and Integration**
ESPHome LVGL provides robust event handling. Widgets can trigger actions based on events like `on_click`, `on_press`, `on_release`, and `on_value` (for arcs/sliders). These events can directly call Home Assistant services (e.g., `light.toggle`, `light.turn_on`) or update local ESPHome states. When using an arc to control a value like brightness, it's important to handle the value conversion between LVGL (which uses integers) and Home Assistant (which may use floats or specific integer ranges).

### Reference URLs

- https://esphome.io/components/lvgl/
- https://esphome.io/components/lvgl/widgets/
- https://esphome.io/cookbook/lvgl/
- https://lvgl.io/docs/open/8.3/widgets/core/arc

### LVGL Applicability

For a 1.28" round 240x240 ESP32-S3 display, the single-page simple mode is an ideal approach. The limited screen real estate makes navigation cumbersome, so placing all controls on one screen maximizes usability. The round form factor naturally lends itself to circular UI elements like the `arc` widget, which can be placed along the outer edge of the display to control primary functions like brightness or volume, leaving the center free for labels and other controls.

When implementing this on a 240x240 round display, positioning is critical. You should use the `align: CENTER` property for central elements and precise `x` and `y` coordinates or `align_to` for other widgets. Because the display is round, elements placed near the corners of the 240x240 bounding box will be cut off. The `arc` widget should be sized appropriately (e.g., `width: 240`, `height: 240`) to follow the curvature of the screen.

For touch interaction on such a small device, the `ext_click_area` property is essential to ensure that users can reliably tap controls without needing pinpoint accuracy. Invisible clickable zones (transparent `obj` widgets) can be placed over text labels (e.g., a power icon or text) to act as buttons without cluttering the UI with button borders. The ESP32-S3 has ample processing power and memory (especially if PSRAM is available) to handle the LVGL rendering smoothly, even with continuous updates from an interactive arc widget.

---

## Thread 7: Typography hierarchy and safe area constraints for a 240x240 round display using LVGL on ESP32-S3 with ESPHome, focusing on 48pt, 24pt, and 20pt font sizes.

### Key Findings

**Typography Hierarchy and Safe Area Constraints**
On a 240x240 round display, the circular geometry imposes strict constraints on horizontal space as you move away from the vertical center. The radius is 120px. At the exact center (Y=0), the full 240px width is available. However, as you move up or down, the available width decreases following the formula `width = 2 * sqrt(r^2 - y^2)`. 

For a "Single-Page Simple Mode" layout with a 24pt top label, 48pt center value, and 20pt bottom label, the total vertical text block height (including 10px padding between elements) is approximately 112px. If centered vertically:
- The top label (24pt) sits at a Y offset of roughly -44px, providing a safe width of ~223px.
- The center value (48pt) sits near the center (Y offset ~2px), providing the full ~240px safe width.
- The bottom label (20pt) sits at a Y offset of roughly +46px, providing a safe width of ~221px.

This layout fits comfortably within the circular safe area, leaving enough horizontal margin for typical smart home control values (e.g., "72°", "100%") and labels (e.g., "Living Room", "Brightness"). If the elements are pushed further apart (e.g., Y offsets of -60px and +60px), the safe width drops to ~207px, which is still adequate for short labels but requires careful text alignment and wrapping considerations.

**Visual Hierarchy in UI Design**
The proposed font sizes establish a clear visual hierarchy:
1. **Primary Focus (48pt):** The center value (e.g., current temperature, brightness level) is the most prominent element, instantly readable at a glance.
2. **Secondary Focus (24pt):** The top label provides context (e.g., device name or room), large enough to be legible but clearly subordinate to the main value.
3. **Tertiary Focus (20pt):** The bottom label offers supplementary information (e.g., status, mode, or secondary metric), using the smallest font size to avoid cluttering the interface.

**Font Handling in LVGL**
LVGL supports custom OpenType/TrueType fonts, which are converted into C arrays or loaded dynamically. For a 240x240 display, bitmap fonts are often preferred for performance and memory efficiency, especially on microcontrollers like the ESP32-S3. However, ESPHome's LVGL component allows rendering TrueType fonts directly, which simplifies the workflow but requires sufficient memory (PSRAM is highly recommended).

When using multiple font sizes in LVGL, each size must be generated as a separate font asset (or loaded separately if using TTF rendering). LVGL does not natively support scaling a single bitmap font to different sizes dynamically without significant quality loss. Therefore, the 48pt, 24pt, and 20pt fonts must be explicitly defined in the ESPHome configuration.

### Reference URLs

- https://esphome.io/components/lvgl/
- https://esphome.io/cookbook/lvgl/
- https://forum.lvgl.io/t/how-to-change-font-size/20531
- https://forum.lvgl.io/t/how-to-match-the-font-size-in-lvgl/6444
- https://github.com/esphome/esphome/issues/10124

### LVGL Applicability

**ESPHome LVGL Implementation on ESP32-S3**
The ESP32-S3 is well-suited for driving a 240x240 round display with LVGL, especially when equipped with PSRAM. ESPHome's LVGL component simplifies UI creation by allowing you to define widgets and layouts directly in YAML. For the "Single-Page Simple Mode," you can use `lvgl.label` widgets for the text elements.

**Font Configuration:**
In ESPHome, you must define each font size separately in the `font:` core component and then reference them in the LVGL configuration. For example:
```yaml
font:
  - file: "fonts/Roboto-Regular.ttf"
    id: font_large
    size: 48
  - file: "fonts/Roboto-Regular.ttf"
    id: font_medium
    size: 24
  - file: "fonts/Roboto-Regular.ttf"
    id: font_small
    size: 20
```
These fonts can then be applied to specific LVGL labels using the `text_font` property or via LVGL styles.

**Layout and Alignment:**
To achieve the desired layout, you can use an LVGL `obj` (container) with a `flex` layout (column direction, center alignment) or position the labels absolutely using `x` and `y` coordinates. Given the circular display, absolute positioning or careful use of padding in a flex layout is necessary to ensure the text stays within the calculated safe widths. The `align: center` property is crucial for keeping the text horizontally centered.

**Memory Considerations:**
Rendering large fonts (like 48pt) requires significant memory for glyph bitmaps. The ESP32-S3's PSRAM is essential here to prevent memory allocation failures (`LVGL memory allocation failed`). Ensure that only the necessary characters (glyphs) are compiled into the font if memory becomes an issue, though ESPHome's dynamic TTF rendering handles this relatively well if PSRAM is available.

---

## Thread 8: Preset cycling via touch tap for LVGL on ESP32-S3 with ESPHome

### Key Findings

The research into implementing a "Single-Page Simple Mode" for the VelaDial project using a 1.28" round 240x240 ESP32-S3 display with LVGL and ESPHome reveals several key insights regarding preset cycling via touch tap. 

First, LVGL handles touch events through its input device interface, triggering events such as `LV_EVENT_CLICKED` or `LV_EVENT_SHORT_CLICKED` when a user taps a widget like a button or an object. For a single-page interface, a large invisible or transparent button (or the screen background itself) can be used to capture these tap events. 

Second, to achieve sequential preset advancement (cycling through 4 options), a state machine approach is required. Since LVGL itself does not have a built-in "multi-state cycle" widget beyond simple checkable (toggle) buttons, the logic must be handled in the application layer. In ESPHome, this is typically done using global variables to store the current state (e.g., 0 to 3) and incrementing this variable within an `on_click` or `on_release` trigger using a lambda function. When the variable reaches 4, it wraps back to 0.

Third, visual feedback is crucial for user experience. When the state changes, the UI must update immediately to reflect the new preset. This can be achieved by updating a label's text, changing an image source, or modifying the color/style of an indicator widget (like an arc or LED) based on the new state value. The `lvgl.widget.update` action in ESPHome is used to apply these visual changes dynamically.

Implementation considerations for ESPHome include ensuring the touch controller (e.g., CST816S or similar common on round displays) is correctly configured and mapped to the LVGL component. The `adv_hittest` property might be useful if specific hit areas are needed, but for a full-screen tap, a simple full-sized object is sufficient. Additionally, debouncing the touch input might be necessary if the hardware is noisy, though LVGL's internal input handling usually manages basic debouncing.

### Reference URLs

- https://esphome.io/components/lvgl/widgets/
- https://esphome.io/cookbook/lvgl/
- https://forum.lvgl.io/t/multi-tap-keypad-support/8043
- https://github.com/esphome/issues/issues/6517

### LVGL Applicability

For a 240x240 round LVGL display powered by an ESP32-S3 running ESPHome, the "Single-Page Simple Mode" is highly applicable and efficient. The ESP32-S3 has ample processing power and memory (especially with PSRAM) to handle LVGL smoothly. The round form factor requires careful UI design; placing a large, transparent, clickable object over the central area (or the entire screen) ensures reliable tap detection without requiring precise targeting by the user.

In ESPHome, the implementation involves defining a `globals` variable to track the preset state (0-3). An LVGL `button` or `obj` widget covering the screen would have an `on_click` trigger. Within this trigger, a lambda increments the global variable modulo 4. Following the increment, the lambda or subsequent ESPHome actions update the visual elements—such as a central `label` displaying the preset name or an `arc` widget around the perimeter indicating the current selection—to provide immediate visual feedback. This approach leverages ESPHome's tight integration with LVGL, keeping the logic simple and the UI responsive.

---

## Thread 9: Anti-dashboard design philosophy and radical simplification in smart home UIs, focusing on single-page simple mode without navigation.

### Key Findings

The "anti-dashboard" design philosophy in smart home UIs represents a radical shift away from complex, multi-page control centers toward extreme simplification. This approach is driven by the need to eliminate decision fatigue and reduce cognitive load for users who interact with smart home devices in passing. Instead of presenting a comprehensive overview of every sensor and device, the anti-dashboard focuses on "one-second understanding," where the user can grasp the state and take action instantly without navigating through menus or tabs.

Key insights from this philosophy emphasize that "simplicity is not the absence of clutter; it's clarity of purpose." In practice, this means removing all unnecessary chrome, navigation bars, and secondary controls. For example, rather than having separate pages for lighting, climate, and security, a single-page simple mode consolidates the most critical, context-aware controls onto one screen. This aligns with trends in wearable and smartwatch UI design, which rely on glanceable information architectures and shallow interaction depths. The goal is to make technology frictionless, especially for users who are not tech-savvy, by providing a direct, immediate interface.

Examples of this approach can be seen in minimalist smart home setups that eschew complex hubs for simple, direct-control interfaces, and in the evolution of mobile UX toward "super apps" or widgets that offer consolidated, single-point interactions. The radical simplification process involves reducing, combining, or abstracting system nodes until only the essential functions remain. This design strategy is particularly effective for small screens, where every pixel must serve a clear, immediate purpose without requiring the user to scroll or swipe to find what they need.

### Reference URLs

- https://uxdesign.cc/radically-different-design-thinking-process-3aab29ba9a7d
- https://medium.com/design-bootcamp/how-minimalist-design-principles-can-make-or-break-your-product-2163fa1eb8a3
- https://www.transfunnel.com/blog/top-mobile-ux-design-factors-to-know
- https://esphome.io/cookbook/lvgl/
- https://esphome.io/components/lvgl/widgets/
- https://controllerstech.com/how-to-interface-gc9a01-round-display-with-stm32-using-spi-lvgl-integration/

### LVGL Applicability

Applying the anti-dashboard philosophy to a 1.28" round 240x240 ESP32-S3 display using LVGL requires a highly focused UI strategy. Given the small, circular form factor, traditional navigation elements like tabs or sidebars are impractical and waste valuable screen real estate. Instead, the UI should consist of a single, static page that houses all necessary controls—such as power, brightness, and presets—arranged concentrically or in a highly optimized grid that respects the circular boundary.

In ESPHome with LVGL, this can be implemented by utilizing the `meter` or `arc` widgets for continuous controls like brightness or volume, wrapping them around the edge of the display. Central elements can be dedicated to primary actions, such as a large toggle button for power or a prominent label displaying the current state. The `align: CENTER` property is crucial here to ensure elements are perfectly centered within the round display. By avoiding the `scrollable` property and keeping all widgets on a single `page`, the interface remains strictly "no navigation," adhering to the radical simplification concept.

Technical implementation on the ESP32-S3 involves configuring the GC9A01 display driver within ESPHome and setting up the LVGL component with a single page. Widgets should be styled to maximize contrast and readability at a glance. For instance, using `lvgl.widget.update` actions tied to Home Assistant sensors allows the display to instantly reflect the current state without user polling. The use of hardware-accelerated rendering and careful memory management (e.g., partial rendering buffers) ensures that the single-page UI remains highly responsive, providing the frictionless experience demanded by the anti-dashboard philosophy.

---

## Thread 10: Using transparent LVGL obj widgets as invisible touch targets for zone-based touch detection in ESPHome

### Key Findings

The research into using an LVGL `obj` widget as an invisible touch target for zone-based touch detection in ESPHome reveals several key insights and implementation details. The primary goal is to create a "Single-Page Simple Mode" UI where all controls are on one screen, and transparent touch zones overlay the visual elements to capture user input without altering the visual design.

**1. The Transparent Object Approach**
The most effective way to create an invisible touch target in LVGL is to use a base `obj` (or a `button`) and set its background opacity to transparent. In LVGL v7/v8, this is achieved by modifying the style properties of the object. Specifically, the `bg_opa` (background opacity) property must be set to `LV_OPA_TRANSP` (or `0%` in ESPHome's YAML configuration), and the border width should be set to `0` to ensure no outlines are drawn.

**2. ESPHome Configuration Specifics**
In ESPHome, LVGL widgets are configured via YAML. To create an invisible touch zone, you can define an `obj` or `button` widget and apply styles directly to it. The key properties to set are:
- `bg_opa: 0%` (or `transp`) to make the background fully transparent.
- `border_width: 0` to remove any borders.
- `shadow_width: 0` to remove any shadows.
- `clickable: true` to ensure the object can receive touch events.

Example ESPHome YAML snippet:
```yaml
lvgl:
  widgets:
    - obj:
        id: touch_zone_power
        x: 0
        y: 0
        width: 120
        height: 120
        clickable: true
        styles:
          main:
            bg_opa: 0%
            border_width: 0
        on_click:
          - logger.log: "Power zone clicked!"
```

**3. Z-Order and Layering**
A critical consideration is the Z-order (stacking order) of the widgets. In LVGL, widgets are drawn in the order they are created, meaning later widgets are drawn on top of earlier ones. To ensure the invisible touch target intercepts touch events, it must be created *after* the visual elements it overlays. In ESPHome's YAML, this means the transparent `obj` should appear lower down in the `widgets` list than the images or shapes it covers.

**4. Event Handling and Bubbling**
When a transparent object is clicked, it can trigger an `on_click` event in ESPHome. If you want the touch event to also pass through to underlying widgets (which is usually not the case for a dedicated touch zone, but possible), you would need to manage the `clickable` and `event_bubble` properties. For a dedicated invisible touch target, `clickable: true` is sufficient, and it will consume the touch event, preventing it from reaching widgets underneath.

**5. Potential Pitfalls**
A common issue reported in LVGL forums is that setting an object to transparent might cause crashes or unexpected behavior if not done correctly, particularly with older versions or specific style initializations. However, in ESPHome's managed LVGL component (which uses LVGL v8), setting `bg_opa: 0%` in the YAML is safe and handled correctly by the underlying wrapper. Another issue is the "focus" state; buttons might show an outline when focused (touched). To prevent this, the style for the `focused` or `pressed` state must also have `bg_opa: 0%` and `border_width: 0`.

### Reference URLs

- https://esphome.io/components/lvgl/
- https://esphome.io/components/lvgl/widgets/
- https://forum.lvgl.io/t/transparent-button-or-obj-on-top-of-other-elements/12331

### LVGL Applicability

For the VelaDial project using a 1.28" round 240x240 ESP32-S3 display, the transparent `obj` approach is highly applicable and efficient for a "Single-Page Simple Mode". Since the display is round, the touch zones can be mapped to specific quadrants or areas of the 240x240 grid. 

To implement this, you would design the single-page UI visually (e.g., drawing a power icon in the top half, brightness controls in the bottom half). Then, you overlay transparent `obj` widgets over these specific areas. For example, a transparent `obj` at `x: 0, y: 0, width: 240, height: 120` would cover the top half of the screen. When the user touches anywhere in that top half, the transparent `obj` intercepts the touch and triggers the `on_click` action, such as toggling the power.

This method is particularly advantageous for the ESP32-S3 because it offloads the complex hit-testing logic from custom C++ code to LVGL's highly optimized internal event system. It allows the visual design to remain completely decoupled from the touch interaction logic. You can have complex, non-rectangular visual elements (like curved sliders or circular buttons) drawn on the screen, while using simple rectangular transparent `obj` widgets to define the interactive hit boxes, simplifying the ESPHome YAML configuration significantly.

---

## Thread 11: Number-roll animation in LVGL — smooth percentage increment/decrement animation like odometer, triggered by rotary encoder steps

### Key Findings

The research into LVGL number-roll and odometer animations reveals several viable approaches for creating smooth, incrementing/decrementing number animations triggered by rotary encoder steps. The most straightforward and natively supported method is using the `lv_roller` widget. The roller widget allows users to select an item from a list by scrolling, and it supports an infinite mode (`LV_ROLLER_MODE_INFINITE`) which is ideal for continuous number rolling like an odometer. When setting a new value programmatically (e.g., from a rotary encoder step), the `lv_roller_set_selected(roller, id, LV_ANIM_ON)` function can be used to trigger a smooth transition to the new value. 

A critical technical detail discovered is that the animation time for the roller in LVGL v8+ is controlled via styles, specifically using `lv_obj_set_style_anim_time(roller, time_in_ms, LV_PART_MAIN)`. This replaces the older `lv_roller_set_anim_time` function from v7. Furthermore, for the animation to work correctly, the main loop must not contain blocking delays, as LVGL relies on its `lv_timer_handler` and custom tick configuration to calculate elapsed time. If a blocking delay is used, LVGL assumes the time has instantly elapsed and skips the animation, resulting in a hard transition.

Alternative approaches include using the `lv_meter` widget or custom animations on `lv_label` objects. The `lv_meter` is highly flexible and can be animated using the general `lv_anim_t` structure, where a custom execution callback updates the meter's value. For labels, one can use `LV_LABEL_LONG_SCROLL_CIRCULAR` for scrolling text, or create a custom timeline animation (`lv_anim_timeline_t`) that moves the Y-position of a label containing a column of numbers to simulate a mechanical odometer roll. However, the roller widget remains the most efficient and built-in solution for this specific requirement.

### Reference URLs

- https://lvgl.io/docs/open/8.3/widgets/core/roller
- https://lvgl.io/docs/open/8.3/overview/animation
- http://forum.squareline.io/t/roller-with-animation/1006
- https://forum.lvgl.io/t/how-to-set-roller-animation-time-speed/11019
- https://esphome.io/components/lvgl/

### LVGL Applicability

For a 1.28" round 240x240 ESP32-S3 display running ESPHome, implementing a smooth number-roll animation requires careful consideration of both LVGL capabilities and ESPHome's integration layer. The ESP32-S3 has ample processing power to handle smooth LVGL animations, but the round display means the UI must be carefully designed to fit within the circular bounds. The `lv_roller` widget is particularly well-suited for this, as its visible row count can be adjusted (`lv_roller_set_visible_row_count`) to ensure it fits neatly in the center of the screen without clipping at the edges.

In the context of ESPHome, the LVGL component provides native support for widgets, including rollers and animations. However, ESPHome's declarative YAML configuration might not expose every low-level LVGL C API function directly. To achieve the smooth animation triggered by a rotary encoder, you will likely need to use ESPHome's lambda functions to call the underlying LVGL C++ API. Specifically, you would map the rotary encoder's `on_value` or `on_clockwise`/`on_anticlockwise` triggers to a lambda that executes `lv_roller_set_selected(id, value, LV_ANIM_ON)`. 

Furthermore, you must ensure that the animation time is set correctly in the ESPHome LVGL configuration or via a setup lambda using `lv_obj_set_style_anim_time`. It is crucial to rely on ESPHome's asynchronous loop and avoid any `delay()` calls within lambdas, as this will block the ESPHome main loop and break the LVGL tick mechanism, causing the animations to snap instantly rather than roll smoothly. Using ESPHome's built-in rotary encoder component linked directly to the LVGL roller state via lambdas provides the most robust solution for the VelaDial "Single-Page Simple Mode" concept.

---

## Thread 12: Single-function premium product design and its application to a 240x240 round LVGL display with ESP32-S3 and ESPHome.

### Key Findings

### The Philosophy of Single-Function Premium Design

In an era defined by feature bloat and multi-purpose devices, luxury brands and premium product designers are increasingly embracing "strategic restraint." This design philosophy posits that true luxury is expressed not through an abundance of features, but through the mastery of a single, focused function. As noted by industry observers, "The world's most powerful luxury brands are now whispering – expressing confidence not through volume, but through control."

This approach, often termed "quiet design," replaces excess with intention. It is not merely aesthetic minimalism, but a structural logic where the decision of what to remove is as important as what to include. In a culture of visual saturation and constant connectivity, absence and deliberate restraint become alluring. The modern luxury identity behaves like a perfectly edited sentence: the meaning lives in what is left unsaid.

### The Rise of Single-Purpose Devices

The consumer market is seeing a counter-cultural reaction to the "ultimate bundle" of the smartphone. There is a growing demand for single-purpose devices that offer a focused, distraction-free experience. Examples include the resurgence of dedicated MP3 players (like the iPod Classic), premium point-and-shoot cameras (like the Contax T2), and minimalist phones (like the Light Phone).

These devices are prized because they provide a "cognitive shift." Using a device with a single, clear purpose puts the user in a focused state of mind, free from the constant interruptions of multi-functional technology. This intentional limitation is increasingly viewed as a luxury—the luxury of focus and control.

### Implementation Considerations for UI Design

When applying this philosophy to UI design, particularly for a premium product, several key principles emerge:

1.  **Clarity Before Visibility:** The interface must have a clear strategic foundation. Every element—color, typography, layout—must be intentional.
2.  **Strategic Restraint:** The UI should resist the urge to over-explain or clutter the screen with secondary functions. The focus must remain entirely on the primary task.
3.  **Precision over Absence:** Restraint is not simply about removing elements; it is about executing the remaining elements with absolute precision and high quality.
4.  **Tactile and Visual Feedback:** In the absence of complex navigation, the interaction with the single screen must be flawless, providing immediate and satisfying feedback.

### Reference URLs

- https://www.iconeye.com/sponsored-content/quiet-design-how-luxury-brands-are-redefining-power-through-restraint
- https://medium.com/@info_30784/strategic-restraint-why-luxury-brands-win-through-silence-and-clarity-31bc2a733f1b
- https://molodtsov.me/2025/11/retro-tech-luxury/
- https://betterhumans.pub/in-praise-of-single-purpose-devices-5f35267902d4
- https://esphome.io/components/lvgl/
- https://esphome.io/cookbook/lvgl/
- https://community.home-assistant.io/t/2424s012-round-display-lvgl/868243

### LVGL Applicability

### Applying Single-Function Design to VelaDial

The concept of "Single-Page Simple Mode" aligns perfectly with the philosophy of single-function premium design. For the VelaDial project, utilizing a 1.28" round 240x240 ESP32-S3 display, this means creating an interface that is entirely focused on the immediate control task (e.g., adjusting volume, temperature, or lighting) without any distracting navigation menus or secondary screens.

### Technical Implementation with ESPHome and LVGL

Implementing this in ESPHome with LVGL requires specific technical considerations to maximize the premium feel on a constrained device:

1.  **Optimized Layout for Round Displays:** The 240x240 round form factor dictates a central focus. The primary control (e.g., an `arc` widget for a dial) should dominate the screen, utilizing the natural curvature of the display. Secondary information (like current value or status) should be placed centrally using `label` widgets with high-quality, anti-aliased fonts (e.g., Montserrat or Roboto).
2.  **Fluid Interactions:** To convey luxury, the UI must be highly responsive. The ESP32-S3 provides sufficient processing power, but the LVGL configuration must be optimized. The `buffer_size` should be set appropriately (e.g., `25%` or higher if PSRAM is available) to ensure smooth animations and transitions. The `update_interval` for the display should generally be set to `never`, relying on LVGL's internal rendering engine to update only when necessary.
3.  **Input Handling:** If VelaDial uses a rotary encoder or touch interface, the input handling must be precise. In ESPHome, the `touchscreen` or `encoder` components must be carefully tuned. For an encoder, the `long_press_time` and `long_press_repeat_time` should be configured to provide intuitive secondary actions (like toggling power) without cluttering the screen with buttons.
4.  **Visual Polish:** The use of a solid, deep background color (`disp_bg_color`) or a subtle, high-quality background image (`disp_bg_image`) can elevate the perceived value. The `color_depth` should be maintained at `16` (RGB565) for optimal performance while still providing rich colors. The `draw_rounding` parameter can be used to ensure smooth rendering of curved elements.

---

## Thread 13: Round display safe area calculation for 240x240 pixel circular viewports

### Key Findings

The usable pixel area within a circular viewport, commonly referred to as the "safe area," is mathematically defined by the largest square that can be inscribed within the circle. For a 240x240 pixel round display, the diameter (d) is 240 pixels, making the radius (r) 120 pixels. The side length of the inscribed square is calculated using the formula: side = r * sqrt(2). 

Applying this formula, the side length of the safe area square is approximately 169.7 pixels (effectively 170x170 pixels). To center this square within the 240x240 bounding box, the top-left corner coordinates are calculated as (r - side/2, r - side/2), which results in (35.15, 35.15). Therefore, the safe area starts at pixel coordinates (35, 35) and extends to (205, 205). Any UI elements placed outside this 170x170 square risk having their corners clipped by the physical circular bezel of the display.

When designing a "Single-Page Simple Mode" UI where all controls (power, brightness, presets) are on one screen, it is crucial to constrain all interactive elements and critical text within this 170x170 pixel bounding box. The remaining space (the segments of the circle outside the inscribed square) can be used for non-critical visual elements, such as curved progress bars (e.g., for brightness or volume) or decorative background elements, which naturally follow the curvature of the display without risking functional clipping.

### Reference URLs

- https://forum.lvgl.io/t/littlevgl-and-round-displays/419
- https://github.com/lvgl/lvgl/issues/7750
- https://community.home-assistant.io/t/2424s012-round-display-lvgl/868243
- https://controllerstech.com/how-to-interface-gc9a01-round-display-with-stm32-using-spi-lvgl-integration/

### LVGL Applicability

In LVGL, handling a round display requires specific configuration to ensure the UI renders correctly and efficiently. First, the display driver must be configured with the physical resolution of 240x240. However, to prevent LVGL from drawing interactive elements in the clipped corners, you should define a base container (e.g., `lv_obj_t * safe_area = lv_obj_create(lv_scr_act());`) with a size of 170x170 pixels and center it using `lv_obj_align(safe_area, LV_ALIGN_CENTER, 0, 0)`. All buttons, labels, and sliders for the Single-Page Simple Mode should be added as children to this `safe_area` container.

For ESP32-S3 with ESPHome, the LVGL component allows you to define the display geometry. While ESPHome handles the low-level SPI/I2C communication with the display driver (like GC9A01 commonly used for 1.28" round displays), the UI layout must be managed within the LVGL configuration block. You can use ESPHome's LVGL integration to create the centered 170x170 container. Additionally, LVGL provides a specific feature for round displays: `LV_USE_ROUNDER` or setting the display radius, which can help optimize rendering by invalidating only the visible circular area during screen updates, saving CPU cycles on the ESP32-S3.

To maximize the use of the 240x240 space while respecting the safe area, consider using LVGL's `lv_arc` widget for controls like brightness. An arc can be drawn along the outer edge of the 240x240 display (e.g., radius 110, centered), utilizing the space outside the 170x170 safe area for touch interaction and visual feedback, while keeping text and toggle buttons safely inside the central square. This approach perfectly suits the "Single-Page Simple Mode" concept, providing a highly intuitive and visually appealing interface without navigation.

---

## Thread 14: ESPHome LVGL arc widget with label overlay for a single-page simple mode UI

### Key Findings

Based on the research into ESPHome LVGL implementations, creating a single-page simple mode with an arc widget and a centered label overlay is highly feasible and well-supported. The LVGL library provides an `arc` widget that consists of a background arc, a foreground (indicator) arc, and a knob. To overlay text, a `label` widget can be used.

Key insights for implementing this:
1. **Widget Hierarchy**: In ESPHome's LVGL component, widgets can have children. To center a label inside an arc, you typically create a parent object (like an `obj` or the `arc` itself if supported as a container) and place the `label` inside it with `align: CENTER`.
2. **Arc Configuration**: The arc can be configured with `min_value` and `max_value` to represent the control range (e.g., 0-255 for brightness or 0-100 for percentage). The `start_angle` and `end_angle` define the visual span of the arc (e.g., 135 to 45 degrees for a typical dial).
3. **Label Overlay**: The label widget is used to display the current value. It can be dynamically updated using a lambda function tied to the arc's `on_value` trigger or a sensor's `on_value` trigger.
4. **Advanced Hit Testing**: For a touch interface, enabling `adv_hittest: true` on the arc ensures that touches are accurately registered on the arc's ring, allowing the center area to be clicked through or used for other interactions if needed.
5. **Styling**: Both the arc and the label can be extensively styled. The arc has parts like `main` (background), `indicator` (foreground), and `knob`. The label can have custom fonts and colors to ensure readability against the background.

Example implementation concept in ESPHome YAML:
```yaml
lvgl:
  pages:
    - id: main_page
      widgets:
        - arc:
            id: brightness_arc
            align: CENTER
            width: 200
            height: 200
            min_value: 0
            max_value: 100
            adv_hittest: true
            on_value:
              - lvgl.label.update:
                  id: brightness_label
                  text:
                    format: "%d%%"
                    args: [ 'x' ]
            widgets:
              - label:
                  id: brightness_label
                  align: CENTER
                  text: "50%"
                  text_font: montserrat_48
```

### Reference URLs

- https://esphome.io/cookbook/lvgl/
- https://esphome.io/components/lvgl/widgets/
- https://lvgl.io/docs/open/9.0/widgets/arc

### LVGL Applicability

For a 1.28" round 240x240 ESP32-S3 display running ESPHome and LVGL, this "Single-Page Simple Mode" is an ideal UI paradigm. The round display naturally complements the circular shape of the arc widget. By setting the arc's width and height to nearly the full dimensions of the screen (e.g., 220x220 or 240x240 with padding), you maximize the touch area for the user.

The ESP32-S3 has ample processing power to handle LVGL's rendering and touch events smoothly. When implementing the arc, you should configure the `bg_angles` to utilize the circular edge of the display, perhaps leaving a small gap at the bottom if you want a traditional dial look (e.g., start at 135 degrees, end at 45 degrees). The centered label will sit perfectly in the middle of the round screen, providing clear, immediate feedback on the current value (like brightness or temperature).

Specific to ESPHome, you will bind the arc's value to your smart home entities (like a light's brightness). Since LVGL handles integers, you may need to scale values (e.g., mapping a 0.0-1.0 float to 0-100 for the arc). The `adv_hittest` is crucial here so that the user can drag the arc's knob along the edge of the round screen without accidental triggers in the center, ensuring a robust and intuitive "VelaDial" experience.

---

## Thread 15: Dark room optimized single-screen UI — sparse layout with large center value, minimal elements for bedroom ambient light control

### Key Findings

**Dark Room Optimized UI Design Principles**
A dark room optimized UI for a single-screen control, such as a bedroom ambient light controller, must prioritize minimal light emission and high legibility. The core concept revolves around a sparse layout where the background is pure black (RGB 0,0,0) to prevent light bleed, especially on IPS or OLED displays. The central element should be a large, easily readable value (e.g., brightness percentage or color temperature) using a dark red or dim amber hue, as these colors preserve night vision and minimize sleep disruption. Minimalist design dictates the removal of all non-essential elements—no borders, no complex icons, and no navigation bars. 

**Single-Screen Control Mechanics**
For a "Single-Page Simple Mode," all interactions must occur on one screen without navigating to sub-pages. This can be achieved through gesture-based controls or multi-function touch areas. For example, swiping up/down could adjust brightness, while swiping left/right could change color temperature or cycle through presets. A tap in the center could toggle the power state. This approach eliminates the need for on-screen buttons, keeping the interface clean and reducing cognitive load for the user in a dark environment.

**Technical Implementation Considerations**
When implementing this on an ESP32-S3 with a 240x240 round display, performance and memory management are key. The UI should use anti-aliased fonts for the large central text to ensure smooth edges on the relatively low-resolution screen. To achieve the pure black background, the display's backlight should be dynamically controlled via PWM, allowing the screen to dim significantly or turn off completely when not in use, waking up upon touch or proximity detection. This not only enhances the dark room experience but also conserves power and extends the display's lifespan.

### Reference URLs

- https://community.home-assistant.io/t/2424s012-round-display-lvgl/868243
- https://esphome.io/cookbook/lvgl/
- https://lvgl.io/docs/open/9.1/overview/style
- https://community.home-assistant.io/t/1-28-inch-240-240-esp32c3-round-display-with-rotary-knob-uedx24240013-md50e-by-viewe-company/786687

### LVGL Applicability

**ESPHome LVGL Integration**
Applying this design to LVGL within ESPHome on an ESP32-S3 involves specific configuration steps. First, the LVGL theme should be set to a custom dark theme where the default background color is explicitly set to `lv_color_black()`. This ensures that any unstyled areas do not emit light. The central value can be implemented using an `lv_label` widget, styled with a large, custom font (e.g., Montserrat 48 or larger) and a dark red text color. 

**Handling Interactions and Display Shape**
Given the 1.28" round display, the UI must account for the circular clipping area. The central label should be perfectly centered using `lv_obj_align(label, LV_ALIGN_CENTER, 0, 0)`. For interactions, an invisible `lv_obj` covering the entire screen can capture touch events. ESPHome's LVGL component allows mapping these touch events (like `LV_EVENT_GESTURE` or `LV_EVENT_CLICKED`) to Home Assistant actions. For instance, a swipe gesture detected by LVGL can trigger an ESPHome script to adjust the light's brightness via the Home Assistant API, all while updating the central label's text dynamically.

**Backlight and Power Management**
To truly optimize for a dark room, the ESP32-S3 must control the display's backlight. In ESPHome, this is done by configuring a `ledc` output for the backlight pin and linking it to a `monochromatic` light component. This allows Home Assistant or local automation to dim the screen based on ambient light or time of day. Additionally, LVGL's inactivity timer can be used to automatically turn off the backlight after a period of no interaction, waking it up instantly upon the next touch event, ensuring the room remains completely dark when the control is not actively being used.

---

## Thread 16: Implementing wake-only-first touch behavior on a single-page LVGL UI to prevent accidental control activation when waking the display.

### Key Findings

The research into implementing a "wake-only-first" touch behavior on a single-page LVGL UI reveals several key insights and technical challenges. When a display goes to sleep (e.g., backlight turns off after a period of inactivity), the touchscreen controller often remains active. If a user touches the screen to wake it up, the touch coordinates are typically passed directly to the LVGL input device (indev) driver. Because there is no page navigation or screen change to naturally "absorb" or debounce this initial touch, the touch event can inadvertently trigger a button or slider located at the touched coordinates. This is a common issue reported across various platforms using LVGL, including ESPHome and Linux-based systems.

To solve this, developers have employed a few different strategies. One common approach is to intercept the touch event at the input driver level. When the screen is in a "sleep" state, the first touch is detected, the backlight is turned back on, and the touch state is explicitly set to `LV_INDEV_STATE_RELEASED` to prevent LVGL from processing it as a click. However, because the user's finger might remain on the screen for multiple polling cycles, a simple state override is often insufficient. A "hack" or workaround frequently used is to implement a debounce timer (e.g., ignoring all touch inputs for 100-500ms after the initial wake touch) or to require the touch state to transition to released before accepting new pressed states.

Another strategy involves using a transparent or semi-transparent overlay (a full-screen object) that is brought to the foreground when the screen goes to sleep. The first touch hits this overlay, waking the screen and hiding the overlay, thereby protecting the underlying UI elements from accidental activation. In the context of ESPHome, users have modified the touchscreen component's `on_release` or `on_touch` triggers to handle backlight control and LVGL pause/resume states, but handling the exact "ignore first touch" logic often requires custom lambda code or specific state tracking within the ESPHome configuration to ensure the first touch only wakes the display and doesn't trigger a Home Assistant action.

### Reference URLs

- https://forum.lvgl.io/t/screen-saver-advice-on-linux-with-lvgl/12128
- https://community.home-assistant.io/t/disable-touchscreen-on-cheap-yellow-display-using-lvgl/942366
- https://github.com/RyanEwen/esphome-lvgl/issues/22
- https://github.com/littlevgl/lvgl/issues/109

### LVGL Applicability

For a 1.28" round 240x240 ESP32-S3 display running LVGL via ESPHome, implementing a wake-only-first touch on a single-page UI requires careful handling of the touchscreen component and LVGL state. Since the UI is a single page with all controls present, any accidental touch during wake-up is highly likely to trigger an unintended action (like toggling power or changing brightness).

The most robust implementation in ESPHome involves using a global variable to track the screen's sleep state and intercepting the touch events. You can configure the touchscreen component (e.g., CST816S or similar I2C touch controller common on these round displays) to trigger an `on_touch` event. If the screen is asleep (backlight off), the event should turn on the backlight, reset the inactivity timer, and crucially, prevent the touch coordinates from being passed to the LVGL widgets. This might require using a custom lambda in the ESPHome configuration to filter the touch data sent to the LVGL indev, or utilizing the overlay method where a hidden, full-screen transparent LVGL object is shown during sleep to absorb the wake touch.

Additionally, you can leverage ESPHome's `lvgl.pause` and `lvgl.resume` actions. When the inactivity timeout is reached, turn off the backlight and call `lvgl.pause`. Upon the next touch, turn on the backlight and call `lvgl.resume`. However, you must ensure that the touch event that triggers the resume is consumed and not processed by the resumed LVGL instance, which often necessitates the aforementioned debounce timer or overlay technique to ensure the "wake guard" is respected across all zones of the single-page UI.

---

## Thread 17: Implementation of curved horizontal divider lines on round LVGL displays

### Key Findings

Research into implementing curved horizontal divider lines on a round LVGL display reveals several viable approaches, each with distinct technical trade-offs. The core challenge is that LVGL's native `lv_line` widget is designed for straight lines between points, and does not have a built-in "curved line" primitive that automatically renders a smooth arc or Bezier curve out of the box without manual point calculation.

**Approach 1: The Arc Widget (`lv_arc`) as a Divider**
The most native and performant way to create a curved line in LVGL is using the `lv_arc` widget. While typically used for loaders or sliders, an arc can be repurposed as a static curved divider. By setting the background angles (`lv_arc_set_bg_angles`) to a specific range (e.g., 160 to 200 degrees for a subtle bottom curve, or 340 to 20 degrees for a top curve) and removing the knob and indicator parts, you get a perfectly smooth, anti-aliased curved line. The radius of the arc can be set to match or slightly offset the display's radius (e.g., 120px for a 240x240 display). This approach is highly efficient as it uses LVGL's optimized arc drawing routines.

**Approach 2: Calculated Points with `lv_line` and Bezier Functions**
If a specific non-circular curve is needed, the `lv_line` widget can be used by feeding it an array of points calculated to form a curve. LVGL includes a built-in cubic Bezier function (`lv_bezier3()`) that can generate these points. A developer must allocate an array of `lv_point_t`, iterate through a loop (e.g., 50-100 segments) calculating the X and Y coordinates using `lv_bezier3()`, and then pass this array to `lv_line_set_points()`. To ensure smoothness, the line style should have `lv_style_set_line_rounded` enabled. While flexible, this consumes more memory (to store the point array) and CPU cycles during initialization compared to the arc method.

**Approach 3: Canvas Widget (`lv_canvas`)**
For maximum control, the `lv_canvas` widget allows direct drawing of arcs, lines, and pixels into a dedicated memory buffer. This allows drawing complex curves or even anti-aliased Bezier curves if custom drawing logic is implemented. However, a canvas requires a dedicated RAM buffer (e.g., `width * height * bytes_per_pixel`), which is highly prohibitive on memory-constrained microcontrollers like the ESP32-S3 unless the canvas is kept very small (e.g., just bounding the divider line itself).

**Approach 4: Image Widget (`lv_img`)**
The simplest approach from a code perspective is to design the curved divider in a graphics program (like Photoshop or Figma), export it as a transparent PNG, convert it to a C array using the LVGL image converter, and display it using an `lv_img` widget. This guarantees perfect anti-aliasing and allows for complex gradients or drop shadows, at the cost of flash memory storage for the image asset.

### Reference URLs

- https://lvgl.io/docs/open/9.2/widgets/arc
- https://esphome.io/components/lvgl/widgets/
- https://forum.lvgl.io/t/may-i-ask-how-to-draw-a-smooth-curve-using-lvgl/11769
- https://lvgl.io/docs/open/widgets/line

### LVGL Applicability

For the VelaDial project using an ESP32-S3 and a 1.28" 240x240 round display with ESPHome, the **Arc Widget (`lv_arc`) approach is strongly recommended**. ESPHome's LVGL component fully supports the arc widget natively via YAML configuration. You can define an arc, set it to be non-clickable, hide the knob and indicator, and position it absolutely. Because the display is round, an arc perfectly complements the physical bezel. For example, to create a subtle divider separating the top status area from the main controls, you can create an arc with a radius of 110, centered, with start and end angles spanning just the top section (e.g., 225 to 315 degrees).

Using the calculated Bezier curve with `lv_line` is technically possible in ESPHome but requires writing custom C++ lambdas to allocate the point array, calculate the curve, and apply it to the line widget. This adds unnecessary complexity and memory overhead for a simple UI divider. The Canvas approach is highly discouraged due to the ESP32-S3's limited SRAM; allocating a canvas buffer large enough to span the 240px width would consume significant memory that is better saved for the display buffer and other widgets.

To implement the Arc divider in ESPHome YAML, you would define an `arc` widget within your page, set `bg_opa: cover`, `bg_color` to your divider color, and define the `width` (thickness of the line). Crucially, set `clickable: false` so it doesn't intercept touch events meant for the single-page controls. You can use `align: CENTER` and adjust the `radius` and `bg_angles` to position the curve exactly where the visual separation is needed.

---

## Thread 18: Smart watch single-screen complications — how watchOS and Wear OS fit multiple data points on small round displays

### Key Findings

Research into watchOS and Wear OS single-screen complications reveals several key strategies for fitting multiple data points onto small, round displays. Both platforms rely heavily on a structured hierarchy of information, utilizing specific complication "families" or "types" to standardize data presentation.

Apple's watchOS utilizes a variety of complication families (e.g., Graphic Circular, Graphic Corner, Graphic Rectangular) to dictate how data is displayed. A critical insight is the use of "Graphic Corner" complications, which curve text and small icons along the outer edge of the display. This maximizes the use of the circular screen's perimeter, leaving the center free for primary information or additional complications. Apple's Human Interface Guidelines emphasize that complications should provide "quick, information-rich" interactions, often using simple icons, short text strings, or small gauges (like a circular progress bar) to convey data at a glance.

Wear OS employs a similar approach with its Complications API, defining types such as `SHORT_TEXT`, `SMALL_IMAGE`, and `RANGED_VALUE`. A notable strategy in Wear OS is the use of dynamic expressions and platform data sources to update complications without constantly waking the main application, saving battery life. For instance, a `RANGED_VALUE` complication can show progress toward a goal using a circular gauge, which is highly space-efficient on a round display. Wear OS also supports `ComplicationDataTimeline`, allowing the watch face to pre-load a schedule of data (like calendar events) to display at specific times, reducing the need for real-time data fetching.

Both platforms demonstrate that successful single-screen UI design on round displays involves:
1.  **Radial Layouts:** Utilizing the curved edges for secondary data (like battery or temperature) using curved text or small icons.
2.  **Iconography over Text:** Replacing words with universally understood symbols to save space.
3.  **Gauges and Progress Bars:** Using circular or semi-circular bars to represent data ranges (e.g., volume, brightness) instead of raw numbers.
4.  **Strict Data Typing:** Limiting the type of data a specific slot can hold to ensure visual consistency and prevent overcrowding.

### Reference URLs

- https://developer.apple.com/design/human-interface-guidelines/complications
- https://developer.android.com/training/wearables/complications
- https://android-developers.googleblog.com/2025/08/building-complication-data-sources-wear-os.html
- https://spin.atomicobject.com/watchos-complications-families-templates/
- https://www.glimsoft.com/02/18/watchos-complications/
- https://www.sitepoint.com/smartwatch-ui-design-battle-circles-vs-squares/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC12221094/
- https://www.panoxdisplay.com/solution/round-display-applications

### LVGL Applicability

Applying these findings to a 1.28" 240x240 round LVGL display on an ESP32-S3 using ESPHome requires adapting these high-level concepts to the specific constraints of the hardware and software stack. The "Single-Page Simple Mode" concept for VelaDial can benefit significantly from the radial layout strategies observed in watchOS and Wear OS.

In LVGL, you can implement "Graphic Corner" style complications by using the `lv_arc` widget combined with `lv_label` widgets positioned along the arc's path. For example, a brightness control could be a semi-circular arc on the left edge, while volume is on the right. The center of the 240x240 screen should be reserved for the primary control or status indicator (e.g., the current power state or active preset). To mimic the `SHORT_TEXT` and `SMALL_IMAGE` complication types, you can use small `lv_img` widgets paired with concise `lv_label` widgets, strategically placed in the four "corners" (top-left, top-right, bottom-left, bottom-right) of the circular display area, ensuring they don't overlap with the central content or the edge arcs.

For ESPHome implementation, the key is managing the data updates efficiently, similar to Wear OS's `UPDATE_PERIOD_SECONDS`. Since the ESP32-S3 has limited resources compared to a full smartwatch, you should bind ESPHome sensor values directly to LVGL widget properties (like the value of an `lv_arc` or the text of an `lv_label`) using ESPHome's lambda functions. To avoid UI blocking, ensure that data updates are event-driven rather than polled continuously in the main loop. Use custom fonts with only the necessary glyphs to save memory, and rely heavily on LVGL's built-in symbols (e.g., `LV_SYMBOL_POWER`, `LV_SYMBOL_SETTINGS`) instead of loading external images wherever possible to maintain high performance and a clean, complication-style aesthetic.

---

## Thread 19: ESPHome text_sensor preset name display — showing active preset name from Home Assistant attribute in LVGL label widget

### Key Findings

To display a preset name from a Home Assistant attribute on an ESPHome LVGL display, two main components are required: importing the attribute into ESPHome as a text sensor, and updating the LVGL label widget when the sensor value changes.

First, ESPHome's `homeassistant` text sensor platform allows importing specific attributes from Home Assistant entities. By defining a `text_sensor` with `platform: homeassistant`, you can specify the `entity_id` and the specific `attribute` (e.g., `preset_mode` or `effect`) to track. This creates a local text sensor in ESPHome that automatically updates whenever the attribute changes in Home Assistant.

Second, to display this text on the LVGL UI, ESPHome provides the `lvgl.label.update` action. This action can be triggered using the `on_value` automation of the text sensor. When the text sensor receives a new value from Home Assistant, the `on_value` trigger fires, and the `lvgl.label.update` action is called to update the `text` property of the target LVGL label widget. The new value is passed using a lambda expression, typically `!lambda return x;`, where `x` represents the new string value of the text sensor.

Example implementation:
```yaml
text_sensor:
  - platform: homeassistant
    id: active_preset
    entity_id: climate.living_room
    attribute: preset_mode
    on_value:
      - lvgl.label.update:
          id: preset_label
          text:
            format: "%s"
            args: [ 'x.c_str()' ]
```
Note that when passing string values to LVGL labels via formatting, it is often necessary to convert the C++ `std::string` to a C-style string using `.c_str()`.

### Reference URLs

- https://esphome.io/components/text_sensor/homeassistant/
- https://esphome.io/components/lvgl/widgets/
- https://esphome.io/cookbook/lvgl/

### LVGL Applicability

For a 1.28" round 240x240 ESP32-S3 display running ESPHome and LVGL, this approach is highly applicable for creating a "Single-Page Simple Mode" UI. Since the goal is to put all controls and information on one screen without navigation, displaying the active preset name directly on the main screen is essential.

The 240x240 resolution provides limited space, so the label widget displaying the preset name should be carefully positioned and sized. Using `align: CENTER` or specific `x` and `y` coordinates relative to other controls (like a central temperature arc or power button) will ensure the UI remains uncluttered. The text font size should be chosen to be readable but not overpowering.

Performance-wise, updating a single label widget on an ESP32-S3 is trivial and will not cause any lag. The ESP32-S3 has ample processing power and memory to handle the LVGL rendering and the Home Assistant API communication simultaneously. The `on_value` trigger ensures that the display only updates when the preset actually changes, minimizing unnecessary redraws and keeping the UI responsive.

---

## Thread 20: Documenting a concept that violates a hard gate (Three-Page Lock) while prototyping it for exploration, specifically focusing on a Single-Page Simple Mode for a 1.28" round 240x240 ESP32-S3 display using LVGL and ESPHome.

### Key Findings

In product development, a "hard gate" (or phase gate) is a critical decision point where a project must meet specific criteria to proceed to the next phase. Violating a hard gate—such as the "Three-Page Lock" requirement—typically means the concept should be rejected or halted. However, in the context of prototyping for exploration, documenting a failed concept that violates a hard gate is crucial for learning and future reference.

When documenting a concept that intentionally violates a hard gate, the focus shifts from proving viability to capturing insights. The documentation should clearly state the constraint being violated (e.g., the Three-Page Lock) and the rationale for exploring the concept anyway (e.g., radical simplification). It is essential to document the "Works-like" and "Looks-like" aspects of the prototype, detailing how the single-page UI functions and how users interact with it.

Key elements to include in the documentation:
1. **Constraint Identification**: Explicitly name the hard gate being violated and explain why it exists.
2. **Exploration Rationale**: Justify the need to explore the concept despite the constraint. What potential value does the Single-Page Simple Mode offer?
3. **Prototype Details**: Describe the technical implementation, including the hardware (ESP32-S3, 1.28" round display) and software (LVGL, ESPHome).
4. **Failure Modes and Learnings**: Document where the concept fails to meet the hard gate criteria and what was learned from the failure. Does the single-page UI become too cluttered? Is the user experience compromised?
5. **Future Recommendations**: Provide guidance on whether the concept should be revisited if constraints change or if elements of the concept can be integrated into a compliant design.

By treating the failed prototype as a learning opportunity, teams can avoid repeating mistakes and uncover novel solutions that might eventually challenge existing constraints.

### Reference URLs

- https://printform.com/when-prototypes-fail-in-production-and-how-to-prevent-it/
- https://www.cognidox.com/blog/effective-product-development-with-gate-phase-control-and-a-dms
- https://learn.forclimatetech.org/an-introduction-to-prototype-stages-phase-gates/
- https://esphome.io/components/lvgl/
- https://esphome.io/cookbook/lvgl/

### LVGL Applicability

Applying the Single-Page Simple Mode concept to a 1.28" round 240x240 ESP32-S3 display using LVGL and ESPHome presents unique challenges and opportunities. The primary constraint is the limited screen real estate and the circular form factor, which makes fitting all controls (power, brightness, presets) onto a single page difficult without causing clutter.

In LVGL, this can be implemented by utilizing a central `meter` or `arc` widget for the primary control (e.g., brightness), surrounded by smaller `button` or `switch` widgets for secondary controls (e.g., power, presets). The circular nature of the display naturally lends itself to radial layouts. ESPHome's LVGL component allows for precise positioning of these widgets using absolute coordinates or alignment properties (e.g., `align: CENTER`).

However, violating the "Three-Page Lock" gate by cramming everything onto one screen may lead to a poor user experience due to touch target sizes being too small. When documenting this prototype, it is crucial to note the usability issues encountered, such as accidental touches or difficulty reading labels. The documentation should highlight whether the radical simplification achieved by the Single-Page Simple Mode outweighs the usability drawbacks on this specific hardware configuration.

---
