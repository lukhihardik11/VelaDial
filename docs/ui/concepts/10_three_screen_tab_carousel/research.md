# Concept 10: Three-Screen Tab Carousel — Research

**Generated:** 2026-05-28
**Method:** 20-thread parallel internet research

---

## 1. Tab carousel UI patterns for round smartwatch displays 240px

### Findings

1. **Circular Layout Optimization**: For a 240x240 round display, UI elements must be arranged in a radial pattern or centered to accommodate the circular shape. This ensures that all information is visible and accessible, avoiding the corners where content would be cut off on a square display.

2. **Tab Carousel Navigation**: The tab carousel pattern is effective for organizing content into distinct, swipeable pages. In LVGL, the `lv_tabview` widget can be used to create this layout, allowing users to switch between tabs (e.g., Power, Brightness, Presets) by swiping horizontally or using a physical rotary encoder.

3. **Visual Indicators**: A 3-dot page indicator is crucial for providing context within a tab carousel. It helps users understand their current position within the sequence of pages, enhancing navigation and overall user experience on small screens.

4. **Hardware Integration**: Combining touch interactions with a physical rotary encoder offers a versatile control scheme. The encoder can be used for precise adjustments, such as changing brightness levels, while touch gestures handle broader navigation between tabs.

5. **Performance with LVGL and ESP32-S3**: The ESP32-S3 microcontroller, paired with the LVGL graphics library, provides the necessary processing power and graphical capabilities to render smooth, interactive UIs on a 240x240 display. LVGL's support for animations and custom widgets makes it ideal for creating premium smartwatch-style interfaces.

### Sources

https://lvgl.io/docs/open/9.1/widgets/tabview, https://www.dofbot.com/post/1-28-inch-gc9a01-round-lcd-with-esp32-and-lvgl-ui-part4, https://developer.android.com/design/ui/wear/guides/m2-5/foundations/getting-started

### Application to VelaDial

The "Three-Screen Tab Carousel" concept is highly applicable to the VelaDial premium bedroom light controller. By utilizing a 240x240 round 1.28" display, the UI can be optimized for quick, glanceable interactions. The three horizontally swipeable pages (Power, Brightness, Presets) align perfectly with the tab carousel pattern, allowing users to navigate seamlessly between core functions. The 3-dot page indicator provides clear visual feedback on the current page, while the physical rotary encoder offers precise control over settings like brightness, complementing the touch interface.

Implementing this UI with the LVGL graphics library on an ESP32-S3 ensures smooth animations and responsive performance. The circular layout should be leveraged by placing primary interactive elements, such as the power toggle or brightness arc, in the center or along the curved edges to maximize the limited screen real estate. This approach not only enhances usability but also creates a premium, modern aesthetic suitable for a high-end bedroom light controller.

---

## 2. LVGL tileview swipe animation timing

### Findings

LVGL provides a dedicated `lv_tileview` widget which is perfectly suited for creating a smartwatch-like interface with horizontally swipeable pages. By default, the tileview allows users to navigate between tiles by swiping, and it automatically handles the snapping of tiles to the center of the screen.

The scroll animation speed in LVGL's tileview is governed by constants defined in the core scrolling implementation (`lv_obj_scroll.c`). Specifically, the constants `SCROLL_ANIM_TIME_MIN` (default 200ms) and `SCROLL_ANIM_TIME_MAX` (default 400ms) dictate the duration of the scroll animation based on the distance scrolled.

Developers have noted that the default animation times can feel slow or "annoying" in certain applications. To change this, developers must modify the `SCROLL_ANIM_TIME_MIN` and `SCROLL_ANIM_TIME_MAX` macros directly in the LVGL source code, as there is currently no public API method to change these specific scroll animation times dynamically in LVGL v9. Setting these constants to 0 results in instantaneous tile switching without smooth sliding.

LVGL supports various animation paths (easing functions) that dictate the acceleration and deceleration of animations. Built-in paths include `lv_anim_path_linear`, `lv_anim_path_ease_in`, `lv_anim_path_ease_out`, `lv_anim_path_ease_in_out`, `lv_anim_path_overshoot`, and `lv_anim_path_bounce`. For swipe transitions, an ease-out or ease-in-out path provides the most natural feel, mimicking physical momentum.

When implementing swipe gestures over interactive elements like buttons, there can be conflicts where a swipe accidentally triggers a button click. LVGL handles this via scroll thresholds, but developers must be careful to configure the UI so that horizontal swipes on the tileview are prioritized over vertical or tap actions on the child widgets.

### Sources

https://lvgl.io/docs/open/9.1/widgets/tileview.html, https://forum.lvgl.io/t/scroll-animation-speed-in-tile-view-widget/16361, https://github.com/lvgl/lvgl/issues/6239, https://lvgl.io/docs/open/9.0/overview/animations, https://lvgl.io/docs/open/9.0/overview/scroll

### Application to VelaDial

For the VelaDial premium bedroom light controller, the LVGL tileview widget is the ideal container for the 3 horizontally swipeable pages (Power, Brightness, Presets). Since it is a premium device with a physical rotary encoder and a 240x240 round display, the UI must feel responsive and fluid. The default scroll animation timing (200ms to 400ms) might feel too slow or "annoying" for a premium dial interface where users expect immediate feedback.

To achieve a premium feel, the firmware should override the default `SCROLL_ANIM_TIME_MIN` and `SCROLL_ANIM_TIME_MAX` constants in `lv_obj_scroll.c` to shorter durations (e.g., 100ms to 250ms) or even 0ms if instantaneous switching is preferred when using the rotary encoder. Furthermore, the `lv_tileview_set_anim_time()` function (available in older LVGL versions but handled via scroll properties in newer ones) or custom animation paths like `lv_anim_path_ease_out` should be used to ensure the swipe feels natural.

Additionally, to prevent accidental button clicks while swiping across the round display, the UI should leverage LVGL's scroll propagation and ensure that buttons have proper drag thresholds. The 3-dot page indicator can be synchronized with the `LV_EVENT_VALUE_CHANGED` event of the tileview to update immediately when a new page is snapped into focus.

---

## 3. 3-dot page indicator design best practices

### Findings

A page control is a navigational UI component that shows a user their current position within a sequence and the total number of pages. For a 3-page sequence, a dot indicator is the standard and most effective pattern. The dots should be large enough to perceive clearly without dominating the interface, with an 8-10px minimum diameter being a reasonable baseline.

The spacing between the indicators should be at least equal to the dot diameter to prevent them from looking cramped, which is a common issue on smaller screens. On round displays, such as smartwatches or round smart home controllers, the page indicator is often curved to follow the shape of the screen, maximizing the use of available space and aligning with circular UI design principles.

The active and inactive states must be clearly distinguishable at a glance. This can be achieved by making the active dot fully opaque while inactive dots are semi-transparent, or by making the active dot noticeably larger or elongated into a pill shape. When a user navigates to the next page, the active indicator should animate smoothly to the new position, with a 150-200ms transition recommended to reinforce the sense of lateral movement.

### Sources

https://baymard.com/learn/page-control-ui, https://developer.android.com/design/ui/wear/guides/m2-5/components/page-indicators, https://lvgl.io/docs/open/9.3/details/common-widget-features/basics, https://esphome.io/components/lvgl/widgets/

### Application to VelaDial

For the VelaDial premium bedroom light controller, the 3-dot page indicator should be placed at the bottom of the 240x240 round display, following the curve of the screen to maximize space and align with circular UI principles. The indicator will represent the three swipeable pages: Power, Brightness, and Presets.

The dots should have a minimum diameter of 8-10px to ensure visibility without dominating the small 1.28" screen. The spacing between the dots should be at least equal to their diameter (8-10px). To clearly distinguish the active page, the active dot should be visually distinct, either by being fully opaque while inactive dots are semi-transparent, or by being slightly larger (e.g., 12px) or elongated into a pill shape.

Since the device features a physical rotary encoder and LVGL graphics library, the page indicator can be implemented using LVGL's built-in styles and widgets. The indicator should update smoothly with a 150-200ms transition when the user swipes or uses the rotary encoder to switch between the Power, Brightness, and Presets pages, providing a premium and responsive feel.

---

## 4. Premium tab navigation UX for embedded displays small screens

### Findings

1. **Circular Layout Adaptation**: On round displays like a 240x240 screen, traditional rectangular tab bars consume too much valuable real estate. Best practices suggest hiding the standard tab buttons and using a curved page indicator (like a 3-dot pagination at the bottom arc) to maximize the central viewing area for content.
2. **Gesture and Hardware Integration**: For small embedded screens (1.28"), touch targets can be difficult to hit accurately. Combining horizontal swipe gestures with physical hardware controls, such as a rotary encoder, provides a more reliable and premium navigation experience, allowing users to switch tabs without obscuring the screen.
3. **LVGL Tabview Customization**: In LVGL, the `lv_tabview` widget can be heavily customized for round screens. Developers often remove the default tab buttons (`lv_obj_add_flag(tab_btns, LV_OBJ_FLAG_HIDDEN)`) and rely on the built-in swipe-to-change functionality, which is highly optimized for microcontrollers like the ESP32-S3.
4. **Animation and Performance**: Smooth transitions are critical for a premium feel. On the ESP32-S3, LVGL's hardware acceleration should be leveraged to ensure tab switching animations run at a consistent 30-60 FPS. However, animations should be kept short (e.g., 200-300ms) to prevent the interface from feeling sluggish.
5. **Content Centering and Margins**: Due to the circular nature of the display, content must be carefully padded. Critical interactive elements and text should be kept within the central "safe zone" (roughly the inner 70% of the diameter), while the edges are reserved for non-interactive indicators or decorative elements.

### Sources

https://ux.stackexchange.com/questions/141947/what-to-consider-when-designing-ux-for-smartwatch-and-wearables, https://weareaffective.com/learning-centre/how-do-i-design-for-such-a-small-smartwatch-screen, https://forum.lvgl.io/t/how-to-make-tabview-horizontal-scroll/2721, https://lvgl.io/docs/open/8.3/widgets/extra/tabview

### Application to VelaDial

For the VelaDial premium bedroom light controller, implementing a three-screen tab carousel on the 240x240 round 1.28" display requires careful adaptation of these findings. The physical rotary encoder should be mapped to tab switching, allowing users to navigate between Power, Brightness, and Presets without obscuring the small screen with their fingers. The 3-dot page indicator must be placed at the bottom curve of the display, following the circular contour to save vertical space while providing clear context of the current page. Since LVGL is used on the ESP32-S3, the `lv_tabview` widget should be customized to hide the default rectangular tab buttons, relying entirely on swipe gestures and the rotary encoder for navigation. The UI should employ a dark theme with high-contrast elements to ensure readability in a bedroom environment, and animations between tabs should be smooth but brief to maintain a premium, responsive feel.

---

## 5. ESPHome LVGL swipe navigation

### Findings

ESPHome's LVGL component supports swipe gestures on standard pages using `on_swipe_left`, `on_swipe_right`, `on_swipe_up`, and `on_swipe_down` triggers. However, a known issue (Issue #6777) occurs when a swipe gesture starts or ends on an interactive widget like a button: the `on_press` or `on_release` events for that widget are also triggered simultaneously, leading to unintended actions.

To avoid these event collisions, the recommended approach for swipe-based navigation is to use the `tabview` or `tileview` widgets instead of standard pages. These widgets natively handle dragging and swiping to switch views, moving the entire view with the finger and preventing accidental button clicks during the swipe gesture.

The `tabview` widget organizes content into tabs and allows selecting a new tab either by clicking a tab button or by sliding horizontally. The tab bar can be positioned on any side (top, bottom, left, right) or hidden entirely if only swipe navigation is desired.

The `tileview` widget is a container whose elements (tiles) are arranged in a grid form. Users can navigate between tiles by swiping. Swiping directions can be individually disabled for specific tiles to restrict movement.

For implementing a persistent page indicator (like a 3-dot carousel indicator) across multiple screens, ESPHome's LVGL component provides a `top_layer`. This is an "Always on Top" transparent page that sits above all other pages or views. Widgets placed in the `top_layer` remain visible regardless of which page or tab is currently active, making it ideal for navigation footers, status icons, or pagination dots.

### Sources

https://esphome.io/components/lvgl/widgets/, https://esphome.io/cookbook/lvgl/, https://github.com/esphome/issues/issues/6777, https://community.home-assistant.io/t/problem-with-lvgl-swipes-and-click-button-togther/860276, https://lvgl.io/docs/open/9.1/widgets/tabview, https://lvgl.io/docs/open/widgets/tileview

### Application to VelaDial

For the VelaDial premium bedroom light controller, implementing the "Three-Screen Tab Carousel" (Power, Brightness, Presets) on the 240x240 round 1.28" display should utilize the `tabview` or `tileview` widget rather than standard pages. This approach natively supports horizontal swiping between the three screens without the event collision issues seen when using `on_swipe_left` and `on_swipe_right` on standard pages with interactive buttons.

To implement the 3-dot page indicator, VelaDial should use a `top_layer` object that remains visible across all tabs. Within this layer, a custom indicator can be built using a `buttonmatrix` or a row of small `obj` widgets styled as circles (using `radius` equal to half their width/height). The active dot can be updated by changing its style or color based on the currently active tab index, which can be tracked using the `LV_EVENT_VALUE_CHANGED` event or by monitoring the active tab state.

The physical rotary encoder can be integrated by assigning the widgets on each tab to a specific `group`. As the user swipes between tabs, the active group should be updated to ensure the rotary encoder controls the widgets on the currently visible screen (e.g., adjusting brightness on the Brightness tab or selecting presets on the Presets tab).

---

## 6. Round display 2x2 grid layout

### Findings

1. **Safe Area Calculation**: For a 240x240 round display (radius 120px), the maximum inscribed square that avoids any corner clipping has a side length of approximately 169.7 pixels (calculated as R * √2).
2. **Grid Container Positioning**: To perfectly center this safe area, the top-left coordinate of the grid container must be placed at (35, 35) relative to the display's origin (0,0).
3. **Cell Dimensions**: Within this safe zone, a 2x2 grid layout will yield individual cells of approximately 84x84 pixels, which is sufficient for touch targets but requires careful icon sizing.
4. **LVGL Implementation**: LVGL's Grid layout (a subset of CSS Grid) or Flex layout can be used by creating a base `lv_obj` container sized to 169x169, aligned to `LV_ALIGN_CENTER`, with the grid applied to this container rather than the full screen.
5. **Edge Utilization**: The space outside the 169x169 safe zone (the curved edges) is typically used for circular elements like `lv_arc` (for brightness/volume) or small indicators, as rectangular widgets will be clipped by the physical circular mask of the GC9A01 driver.

### Sources

https://forum.lvgl.io/t/littlevgl-and-round-displays/419, https://lvgl.io/docs/open/9.2/layouts/grid, https://controllerstech.com/how-to-interface-gc9a01-round-display-with-stm32-using-spi-lvgl-integration/

### Application to VelaDial

For the VelaDial premium bedroom light controller, implementing a 2x2 grid on the 240x240 round display requires constraining the interactive elements within the 169x169 pixel inscribed square safe zone. This ensures that touch targets for lighting controls (like toggles or preset buttons) are fully visible and accessible without being clipped by the physical bezel.

By utilizing LVGL's flex or grid layouts with a base container sized to 169x169 and centered on the screen, the UI can maintain a clean, symmetrical appearance. The remaining curved edge space can be repurposed for non-interactive visual indicators, such as a circular brightness arc or the 3-dot page indicator, maximizing the utility of the round form factor while keeping the core 2x2 grid functional.

---

## 7. Smart home light controller page carousel interaction design

### Findings

1. **LVGL Tileview for Carousels**: In LVGL, the `lv_tileview` widget is the standard method for creating swipeable multi-page interfaces (carousels). It allows placing multiple screens side-by-side horizontally, enabling smooth touch swipe gestures between pages like Power, Brightness, and Presets.
2. **Rotary Encoder Integration**: LVGL supports rotary encoders through its input device interface (`lv_indev`). Encoders are mapped to an `lv_group`, where rotating the knob navigates between focusable objects or adjusts values (edit mode), and clicking the knob acts as an enter/confirm action or toggles between navigation and edit modes.
3. **Round Display Spatial Constraints**: On a 240x240 round display, the corners are physically cut off. UI elements must be centralized. For a 3-page carousel, a 3-dot page indicator is typically placed at the bottom center (around y=200 to 220) following the curve, leaving the central 160x160 pixel area for primary controls like circular sliders or large icons.
4. **Hybrid Interaction Model**: Combining a capacitive touch screen (like the CST816D often paired with GC9A01 displays) with a rotary encoder allows a dual-interaction paradigm: swipe left/right to change pages (handled by tileview), and rotate the knob to adjust the primary value on the current page (e.g., an `lv_arc` or `lv_slider` for brightness).
5. **Hardware & Performance**: The GC9A01 display driver is standard for 1.28" round displays. When driven by an ESP32-S3 via SPI, it provides smooth 30+ FPS animations for LVGL carousels. The ESP32-S3's PSRAM is crucial for buffering the full 240x240 16-bit color framebuffers to ensure tear-free swipe transitions between the three tabs.

### Sources

https://dev.to/jasonliu112/round-lcd-displays-for-embedded-ui-a-practical-guide-2lp0, https://www.reddit.com/r/homeassistant/comments/1mb446q/diy_rotary_touch_controller_for_home_assistant/, https://forum.lvgl.io/t/understanding-how-navigation-works-with-a-rotary-encoder/2657, https://esphome.io/components/lvgl/widgets/

### Application to VelaDial

For the VelaDial premium bedroom light controller, the Three-Screen Tab Carousel design can be implemented using LVGL's tileview widget to create a seamless horizontal swipe experience between the Power, Brightness, and Presets pages. The 1.28" 240x240 round display requires careful spatial planning, placing the 3-dot page indicator at the bottom curve to maximize the central active area for touch and visual feedback. The physical rotary encoder should be integrated using LVGL's group navigation system, allowing users to rotate the knob to adjust values (like brightness or preset selection) and click to confirm, while touch swipes handle page navigation. This hybrid interaction model ensures that users can quickly adjust settings in the dark using the tactile knob, or use intuitive swipes when looking at the screen.

---

## 8. Horizontal page carousel accessibility usability

### Findings

1. **Control and Navigation**: Accessible carousels must provide robust user control. This includes the ability to navigate between items using alternative methods beyond just swiping, such as keyboard navigation or, in the case of hardware, physical buttons or rotary encoders. The W3C ARIA guidelines emphasize that all functionality must be operable by these alternative inputs, ensuring users can easily move to the next or previous slide.

2. **Visibility of Current Position**: It is crucial to clearly indicate the user's current position within the carousel. While progress dots are common, they must be designed with sufficient contrast and size to be easily perceivable. To accommodate users with color vision deficiencies, the active state should not rely on color alone; for example, using a filled dot for the active page and unfilled dots for inactive pages is recommended.

3. **Avoid Auto-Rotation**: Auto-advancing carousels are generally discouraged, especially on small screens or when users need time to process information. If auto-rotation is used, there must be a clear, accessible way to pause or stop it. For a smart home controller, static pages that only change upon user interaction (swipe or rotary encoder turn) are preferable to prevent frustration and ensure the user remains in control.

4. **Content Clarity and Density**: On small displays, such as a 1.28" round screen, it is vital to keep the content crisp and uncluttered. Guidelines suggest limiting the number of frames (e.g., 5 or fewer) to prevent cognitive overload and ensure users can easily remember and navigate back to specific pages. Each page should focus on a single, clear function (like Power, Brightness, or Presets) with legible text and recognizable icons.

5. **Semantic Structure and Feedback**: For users relying on assistive technologies, the carousel must have a clear semantic structure. While a physical device might not use a screen reader in the traditional web sense, the principle applies: the system should provide clear feedback (visual or haptic) when a page changes. The UI should communicate the boundaries of the carousel and the current context, ensuring the user always knows where they are within the 3-page structure.

### Sources

https://www.w3.org/WAI/tutorials/carousels/, https://www.smashingmagazine.com/2022/04/designing-better-carousel-ux/, https://www.nngroup.com/articles/designing-effective-carousels/, https://www.w3.org/WAI/ARIA/apg/patterns/carousel/, https://www.smashingmagazine.com/2023/02/guide-building-accessible-carousels/

### Application to VelaDial

The research findings on horizontal page carousel accessibility and usability have direct implications for the VelaDial premium bedroom light controller. Given the 240x240 round 1.28" display, the UI must prioritize clarity and ease of navigation. The 3-dot page indicator should be highly visible, using high-contrast colors and distinct states (e.g., filled vs. unfilled) to clearly show the current page (Power, Brightness, Presets) without relying solely on color. The physical rotary encoder is a significant advantage, as it provides a tactile, accessible alternative to swiping, addressing the need for keyboard-like navigation and precise control, which is crucial for users with motor impairments or those who prefer physical controls.

To ensure a premium and accessible experience, the LVGL implementation on the ESP32-S3 should support smooth transitions between the 3 pages, avoiding auto-rotation entirely, as the user should have full control over the interface. The rotary encoder should map intuitively to the horizontal carousel, where turning the dial moves between pages, and pressing it selects or activates the current page's primary function. Additionally, the UI should ensure that touch targets (if touch is supported alongside the encoder) are large enough to prevent accidental activations, and the content on each page (Power, Brightness, Presets) should be concise and legible, adhering to the principle of not overcrowding small screens.

---

## 9. LVGL arc 270 degree brightness

### Findings

1. **Arc Angle Configuration**: In LVGL, the 270-degree arc is typically configured using `lv_arc_set_bg_angles(arc, 135, 45)`. Zero degrees is at the 3 o'clock position, and angles increase clockwise. A start angle of 135° and an end angle of 45° (which is 360 + 45 = 405° logically, but LVGL handles the wrap-around) creates a symmetrical 270-degree arc with the opening at the bottom.

2. **Value Mapping and Range**: The arc's value range is set using `lv_arc_set_range(arc, 0, 100)` to map directly to a 0-100% brightness scale. The current value is updated using `lv_arc_set_value(arc, target_value)`, which automatically adjusts the indicator arc's length and the knob's position.

3. **Styling Dimensions**: For a 240x240 display, a common and visually pleasing configuration is to set the arc's line width to 12 pixels (`lv_obj_set_style_arc_width(arc, 12, LV_PART_MAIN)`). The knob at the end of the indicator is often sized around 20x20 pixels to be easily visible but not overwhelming.

4. **Hardware Synchronization**: When using a physical rotary encoder, the encoder's delta is calculated in an ESPHome lambda. This delta adjusts a global brightness variable (clamped between 0 and 100). The new value is then passed to `lv_arc_set_value()`, and `lv_refr_now(NULL)` is called to force an immediate screen refresh, ensuring zero perceived latency between turning the knob and the screen updating.

5. **Textual Feedback**: It is standard practice to place a label in the center of the arc to display the exact brightness percentage. This is achieved by formatting the integer value into a string (e.g., using `snprintf(buf, sizeof(buf), "%d%%", brightness_value)`) and updating the label with `lv_label_set_text()`. Using a large, clear font like 36pt Montserrat ensures readability.

### Sources

https://lvgl.io/docs/open/9.5/widgets/arc, https://www.elecrow.com/wiki/1.28-ESPHOME-Lesson04-Adjust-Brightness-in-LVGL-Interface.html, https://esphome.io/cookbook/lvgl/

### Application to VelaDial

For the VelaDial premium bedroom light controller, the 270-degree arc widget provides an intuitive and visually appealing way to control brightness on the 240x240 round display. By mapping the physical rotary encoder's input to the LVGL arc's value, users can smoothly adjust brightness. The 270-degree span (from 135° to 45°) perfectly fits the circular screen, leaving the bottom 90 degrees open for other UI elements or just for aesthetic balance.

The integration involves using ESPHome's lambda functions to capture the rotary encoder's delta, update a global brightness variable, and then call `lv_arc_set_value()` to synchronize the visual arc with the new brightness level. This ensures that the physical knob and the on-screen UI are always in perfect sync.

Furthermore, the arc can be styled to match the premium feel of the VelaDial. For instance, setting the background arc to a dark gray with a width of 12 pixels and rounded ends, while the indicator arc uses a vibrant color (like sky-blue or a gradient if supported/faked via images) and a 20x20 knob, creates a modern, high-end look. The central area of the arc can house a dynamically updated text label showing the exact percentage, providing clear feedback to the user.

---

## 10. Amber accent color psychology premium product design warmth

### Findings

Amber (HEX: #FFBF00; RGB: 255, 191, 0) is a warm, inviting color that combines the golden glow of yellow with the richness of orange, exuding a sunny, cheerful vibe and creating a comforting atmosphere in UI design.

In premium product design, amber-based palettes (like "Amber Mirage") are utilized to convey warmth, radiance, and a sense of calm power, which helps brands appear more expensive, intelligent, and sophisticated.

Research indicates that amber light can effectively ease stress and anxiety. This soothing effect is theorized to occur because amber mimics natural, calming light sources experienced in nature, such as sunsets and firelight.

In UI/UX applications, an "Amber Glow" serves as an attention-grabbing warmth that maintains high readability. It adds a soft, energetic feel that is perfect for prompting user engagement without being visually overwhelming like pure yellow or aggressive like pure red.

Psychologically, amber symbolizes energy, vitality, and comfort. It captures the essence of autumn leaves and golden sunsets, translating to a cozy, inviting environment when applied to smart home interfaces and lighting controls.

### Sources

https://www.figma.com/colors/amber/, https://www.instagram.com/reel/DTDAwVbDFnD/?hl=en, https://www.ucdavis.edu/health/news/color-lab-amber-light-eases-stress-anxiety, https://www.instagram.com/reel/DTA_C71j2Z0/?hl=en, https://picsart.com/colors/color-meanings/amber/

### Application to VelaDial

The use of amber as an accent color in the VelaDial premium bedroom light controller perfectly aligns with the product's goal of creating a warm, relaxing environment. By utilizing amber (such as #FFBF00) for active states, the 3-dot page indicator, and the brightness slider, the UI will evoke the calming effects of a sunset or firelight, which is ideal for a bedroom setting where users want to wind down before sleep.

In a premium context, amber provides a sophisticated alternative to stark primary colors, offering a "calm power" that feels expensive and intelligent. Against a deep black background on the 240x240 round display, amber accents will provide high contrast and readability while maintaining a soft, non-intrusive glow that won't disrupt the user's circadian rhythm or cause eye strain in low-light conditions.

Furthermore, the psychological association of amber with comfort and vitality enhances the tactile experience of the physical rotary encoder. As the user adjusts the brightness or selects presets, the amber visual feedback will reinforce a sense of warmth and coziness, making the interaction feel intuitive, premium, and deeply satisfying.

---

## 11. Page indicator dot animation active state transition micro-interaction

### Findings

1. **Morphing Pill-to-Dot Animation**: A common and effective micro-interaction for page indicators is the "pill-to-dot" morph, where the active page is represented by an elongated pill shape (e.g., 12px width, 4px height) and inactive pages are represented by smaller circular dots (e.g., 4px diameter). This provides a clear visual distinction for the active state.
2. **Transition Duration and Easing**: For smooth and premium-feeling micro-interactions, transition durations typically range from 200ms to 300ms. Using easing functions, such as `ease-in-out` or LVGL's `lv_anim_path_ease_in_out`, ensures the animation starts and ends smoothly, avoiding abrupt changes that can feel jarring.
3. **Color and Opacity Transitions**: Alongside size changes, active dots often feature a higher opacity (e.g., 100% or alpha 255) and a prominent color (like the primary brand color or white on a dark background), while inactive dots use lower opacity (e.g., 30-50%) or a muted color. This dual-channel feedback (size + color/opacity) reinforces the active state.
4. **LVGL Implementation**: In LVGL, these transitions can be implemented using the style transition API (`lv_style_set_transition`). By defining different styles for the default and active states (e.g., changing `width` and `bg_color`), LVGL automatically interpolates the values during state changes, creating a smooth animation.
5. **Synchronization with Swipe/Scroll**: The dot animation should ideally be synchronized with the swipe or scroll action. As the user swipes between the Power, Brightness, and Presets pages, the active indicator should transition proportionally to the scroll progress, providing immediate and continuous feedback rather than waiting for the page snap to complete.

### Sources

https://lvgl.io/docs/open/9.1/overview/style, https://developer.android.com/jetpack/androidx/releases/wear-compose-m3, https://www.youtube.com/watch?v=F3tWmOsERVc, https://blog.prototypr.io/10-inspiring-and-creative-micro-interactions-36bb1accf465, https://forum.lvgl.io/t/transition-performance-issues/19560

### Application to VelaDial

For the VelaDial premium bedroom light controller, the page indicator dot animation active state transition micro-interaction can be implemented using LVGL's transition and animation features. The 3-dot indicator should use a "shrink-to-dot" or "pill-to-dot" morphing animation where the active dot expands horizontally into a pill shape (e.g., 12px wide) while inactive dots remain circular (e.g., 4px diameter). This provides clear visual feedback of the current page (Power, Brightness, Presets) on the 240x240 round display.

The transition should be smooth, utilizing LVGL's easing functions (like `lv_anim_path_ease_in_out`) with a duration of around 200-300ms to match the physical rotary encoder's tactile feedback. When swiping or rotating the encoder, the active pill shape should smoothly transition to the next dot, shrinking back to a circle while the new active dot expands. This micro-interaction enhances the premium feel of the device, ensuring users always know their position within the 3-page carousel without cluttering the small 1.28" screen.

---

## 12. LVGL swipe gesture detection threshold distance speed

### Findings

LVGL's gesture detection mechanism relies on two primary conditions being met simultaneously: the movement must be fast enough, and it must cover a sufficient distance. Specifically, the difference between the current and previous touch points must exceed the `gesture_min_velocity` threshold, and the total distance from the initial touch point must exceed the `gesture_min_distance` threshold.

In newer versions of LVGL (v9.5+), these parameters can be dynamically configured at runtime using the API functions `lv_indev_set_gesture_min_velocity()` and `lv_indev_set_gesture_min_distance()`. In older versions, these were hardcoded macros in `lv_conf.h`, specifically `LV_INDEV_DEF_GESTURE_MIN_VELOCITY` (which defaults to 3) and `LV_INDEV_DEF_GESTURE_LIMIT` (which defaults to 50 pixels).

A known architectural limitation in LVGL's gesture engine (documented in Issue #6640) is that it does not use actual timestamps to calculate true velocity. Instead, it simply compares the position difference between consecutive input device reads. This means that if the polling rate fluctuates, the perceived velocity changes, often resulting in gestures only being registered if the user swipes very forcefully.

To mitigate slow or unresponsive gesture detection, developers commonly employ several techniques. First, lowering the `LV_INDEV_DEF_GESTURE_LIMIT` makes the system require less physical distance to register a swipe, which is especially important on small displays. Second, ensuring highly accurate timekeeping by calling `lv_tick_inc(passed_time)` from a reliable hardware timer prevents lag in the input handling task.

Finally, for ESP32-S3 implementations using touch controllers like the CST816S, a highly effective workaround is to offload gesture detection to the hardware. By reading the gesture registers directly from the touch IC rather than passing raw X/Y coordinates to LVGL's software recognizer, developers achieve significantly more reliable and responsive swipe detection.

### Sources

https://lvgl.io/docs/open/9.5/main-modules/indev/gestures, https://github.com/lvgl/lvgl/issues/6640, https://forum.lvgl.io/t/gestures-are-slow-perceiving-only-detecting-one-of-5-10-tries/18515

### Application to VelaDial

For the VelaDial premium bedroom light controller, which utilizes a 240x240 round 1.28" display and an ESP32-S3, implementing smooth swipe gestures across the three pages (Power, Brightness, Presets) requires careful configuration of LVGL's gesture thresholds. Since the display is small, the default `LV_INDEV_DEF_GESTURE_LIMIT` of 50 pixels might be too large, requiring the user to swipe across nearly 20% of the screen. Lowering this limit to around 20-30 pixels will make the UI feel more responsive to shorter, more natural thumb swipes.

Additionally, because LVGL calculates velocity based on position differences between reads rather than true timestamps, the `LV_INDEV_DEF_GESTURE_MIN_VELOCITY` should be tuned (potentially lowered from the default of 3) to ensure that casual, slower swipes are reliably detected. To guarantee optimal performance on the ESP32-S3, it is crucial to maintain accurate timekeeping by calling `lv_tick_inc()` consistently, perhaps via a dedicated hardware timer interrupt, ensuring that the touch polling rate remains stable and gestures are not missed.

Alternatively, if the touch controller (such as the CST816S commonly found on these round displays) supports hardware-level gesture recognition, VelaDial could bypass LVGL's software gesture detection entirely. Reading the swipe direction directly from the touch IC's registers would provide flawless, zero-latency page transitions, enhancing the premium feel of the device.

---

## 13. ESPHome LVGL swipe page navigation

### Findings

ESPHome's LVGL component supports page navigation using the `lvgl.page.next` and `lvgl.page.previous` actions. These actions can be triggered by swipe gestures using the `on_swipe_left` and `on_swipe_right` event triggers defined at the page level. The `page_wrap` configuration option, which defaults to `true`, allows for cyclic navigation, wrapping from the last page to the first page and vice versa.

A significant issue with the current implementation of swipe gestures in ESPHome's LVGL integration is the lack of gesture prioritization over click events. When a user initiates or terminates a swipe gesture over an interactive widget, such as a button, the widget's `on_press` or `on_release` events are simultaneously triggered. This behavior is documented in GitHub issues #6777 and #3059, where users report that swiping across a button inadvertently activates it.

To address the swipe and click conflict, community experts recommend using the `tabview` or `tileview` widgets instead of standard pages for swipeable interfaces. These widgets natively support drag-to-switch functionality, where the entire view moves with the user's finger, effectively avoiding the simultaneous triggering of widget click events.

When using the `tabview` widget to simulate a swipeable page interface, the default tab buttons can be problematic for a clean UI design, especially on a small round display. Users have reported challenges in completely disabling or hiding the scrolling tabs (e.g., GitHub issue #7090). A common workaround is to style the tab buttons to be transparent or position them outside the visible area of the display.

For hardware interaction, ESPHome allows the integration of a physical rotary encoder with LVGL widgets. The encoder can be configured to navigate focus between widgets or adjust values when a widget is in edit mode. The encoder's push button acts as the `ENTER` key, which triggers the `on_press` event for simple widgets or toggles edit mode for complex widgets. This is particularly useful for adjusting sliders or arcs, such as a brightness control on a smart home display.

### Sources

https://esphome.io/components/lvgl/, https://esphome.io/components/lvgl/widgets/, https://github.com/esphome/issues/issues/6777, https://github.com/esphome/feature-requests/issues/3059, https://community.home-assistant.io/t/problem-with-lvgl-swipes-and-click-button-togther/860276

### Application to VelaDial

For the VelaDial premium bedroom light controller, implementing a 3-page swipeable interface on a 240x240 round display requires careful consideration of ESPHome's LVGL integration. While the `on_swipe_left` and `on_swipe_right` triggers combined with `lvgl.page.next` and `lvgl.page.previous` actions provide a straightforward way to navigate between the Power, Brightness, and Presets pages, this approach has a known limitation: swiping over interactive widgets (like buttons or sliders) can inadvertently trigger their `on_press` or `on_release` events. This could lead to accidental light toggles or brightness changes during navigation.

To mitigate this, the VelaDial UI should either restrict swipe detection to non-interactive areas of the screen (e.g., the top or bottom edges) or utilize the `tabview` or `tileview` widgets instead of standard pages. The `tabview` and `tileview` widgets natively support drag-to-switch functionality, which handles gesture prioritization better than the manual swipe-to-page-change implementation. If using `tabview`, the tabs themselves can be hidden, and the 3-dot page indicator can be updated based on the active tab.

Furthermore, the physical rotary encoder can be integrated by assigning it to a widget group. The encoder's left/right rotation can be mapped to focus navigation or value adjustment (e.g., changing brightness), while the button press can be used to enter edit mode or toggle the power state. The `page_wrap` configuration should be set to `true` to allow seamless cyclic navigation between the three pages.

---

## 14. Rotary encoder UI navigation vs adjustment

### Findings

1. **LVGL Group Navigation:** LVGL uses a group-based navigation system where objects are added to a group, and the encoder navigates between them. This is essential for moving between the Power, Brightness, and Presets pages.
2. **Navigate vs. Edit Modes:** LVGL natively supports two modes for encoders: 'navigate' and 'edit'. In navigate mode, turning the encoder moves focus between objects (e.g., pages). In edit mode, turning the encoder changes the value of the focused object (e.g., brightness level).
3. **Mode Switching Mechanism:** Switching between navigate and edit modes is typically achieved by clicking the rotary encoder button. This triggers the `LV_KEY_ENTER` event, which LVGL uses to toggle the `LV_STATE_EDITED` state of the focused widget.
4. **Visual Feedback:** It is crucial to provide clear visual feedback when switching modes. LVGL applies different styles based on the `LV_STATE_FOCUSED` and `LV_STATE_EDITED` states, allowing the UI to highlight the active mode (e.g., a glowing border for edit mode).
5. **Encoder Event Handling:** Custom event handlers can be used to override default LVGL behavior if needed. For example, `lv_group_set_editing()` can be called programmatically to force a mode change based on specific conditions or timeouts.

### Sources

https://lvgl.io/docs/open/8.4/overview/indev, https://forum.lvgl.io/t/understanding-how-navigation-works-with-a-rotary-encoder/2657, https://www.reddit.com/r/homeassistant/comments/1mb446q/diy_rotary_touch_controller_for_home_assistant/

### Application to VelaDial

For the VelaDial premium bedroom light controller, the findings suggest a two-mode interaction model using the physical rotary encoder. By default, rotating the encoder should navigate between the three main pages (Power, Brightness, Presets), with the 3-dot indicator showing the current page. To adjust brightness, the user must click the encoder to enter 'edit mode' on the Brightness page, allowing rotation to change the brightness value. A subsequent click or timeout should return the interface to 'navigate mode'. This prevents accidental brightness changes while swiping or rotating between pages. The LVGL library natively supports this through its `LV_STATE_EDITED` state and group focus mechanisms, ensuring a smooth and intuitive user experience.

---

## 15. Circular display safe area inset calculation 240x240 bezel clipping

### Findings

1. **Inscribed Square Dimensions**: For a 240x240 pixel circular display, the maximum safe area for rectangular content is an inscribed square. Using the formula `side = √2 × radius`, with a radius of 120 pixels, the side length of this safe square is approximately 169.7 pixels (typically rounded to 169x169 or 170x170 pixels).
2. **Bezel Clipping Padding**: To prevent UI elements from being clipped by the circular bezel, a padding must be applied from the edges of the 240x240 bounding box. The required padding on each side (top, bottom, left, right) is calculated as `(240 - 169.7) / 2`, which equals approximately 35.15 pixels.
3. **LVGL Coordinate System**: In the LVGL graphics library, round displays are still treated internally as a square memory block (240x240). The UI must be designed within this square, and developers must manually account for the circular physical shape by applying padding or using specific alignment techniques to keep content within the safe zone.
4. **LVGL Padding Properties**: To enforce the safe area in LVGL, developers can use style properties such as `pad_top`, `pad_bottom`, `pad_left`, and `pad_right` set to 35 pixels on the main screen or container object. This ensures that child widgets (like text, buttons, or indicators) are automatically constrained within the 170x170 inscribed square.
5. **Edge Utilization for Circular UI**: While the 170x170 square is the safe area for standard rectangular content, the remaining space (the 35-pixel margin) is not useless. It is ideal for circular UI elements such as arc widgets (`lv_arc`), which can be drawn along the outer edge (e.g., radius 110-120) to serve as progress bars or indicators controlled by a physical rotary encoder, thus maximizing the display's unique shape.

### Sources

https://www.omnicalculator.com/math/square-in-a-circle, https://lvgl.io/docs/open/9.3/details/common-widget-features/coordinates, https://forum.lvgl.io/t/littlevgl-and-round-displays/419, https://www.youtube.com/watch?v=GN7Kx46eS74

### Application to VelaDial

For the VelaDial premium bedroom light controller, these findings directly dictate the UI layout strategy on the 1.28" 240x240 circular display. To ensure the 3 horizontally swipeable pages (Power, Brightness, Presets) and the 3-dot page indicator are fully visible without being clipped by the physical bezel, the core interactive elements must be constrained within the 169x169 pixel safe area. 

The LVGL graphics library on the ESP32-S3 should be configured with a base screen padding of 35 pixels on all sides (`pad_top`, `pad_bottom`, `pad_left`, `pad_right`). This guarantees that critical UI components like the brightness slider or preset buttons remain within the inscribed square. The 3-dot page indicator can be placed at the very bottom of the safe area (around Y=205), while the physical rotary encoder can control elements that are visually represented along the curved edges outside the safe area, such as a circular progress bar for brightness, maximizing the use of the round display while keeping text and icons safely unclipped.

---

## 16. Smart home UI information hierarchy power vs brightness priority

### Findings

1. **Primary Interaction Frequency**: Research indicates that in smart lighting, brightness adjustment is often the most frequent interaction after the initial power toggle. Users frequently want to adjust the ambient light level rather than just turning it on or off, making brightness a high-priority UI element.
2. **UI Hierarchy and Separation**: A common complaint in smart home apps (like Home Assistant) is burying power, brightness, and color controls across multiple screens. For a small 240x240 round display, separating Power, Brightness, and Presets into distinct, swipeable pages is a practical solution to avoid clutter while maintaining accessibility.
3. **Rotary Encoder Integration**: The physical rotary encoder is ideal for continuous variables like brightness. Studies on embedded UIs show that rotary knobs paired with circular displays provide an ergonomic and intuitive way to adjust values, mimicking traditional dimmer switches.
4. **Visual Feedback on Round Displays**: Round displays (like the 1.28" ESP32-S3) benefit from radial UI elements. For brightness, a circular slider or progress bar around the edge of the screen maximizes the use of the 240x240 resolution and provides clear visual feedback of the current level.
5. **State Retention**: A critical aspect of lighting UI is state retention. When the power is toggled off and on, the system should remember the last set brightness level. The UI must clearly indicate this retained state, perhaps by showing the target brightness level even when the light is currently off.

### Sources

https://dev.to/jasonliu112/round-lcd-displays-for-embedded-ui-a-practical-guide-2lp0, https://www.reddit.com/r/homeassistant/comments/12tqkza/why_does_the_new_light_dialog_require_3_screens/, https://joshdotai.medium.com/the-hard-thing-about-designing-the-smart-home-lighting-control-3d09515986bb, https://www.reddit.com/r/homeautomation/comments/biqwrt/smart_bulbs_and_energy_use/

### Application to VelaDial

The findings directly influence the VelaDial premium bedroom light controller's UI design. Given the 240x240 round 1.28" display and physical rotary encoder, the primary interaction should focus on brightness control via the rotary knob, as users adjust brightness more frequently than power. The Power page should be the first in the 3-page carousel, offering a clear, unambiguous binary toggle, perhaps utilizing the entire screen area for easy tapping. The Brightness page, being the most used, should be the central (default) page, allowing the rotary encoder to smoothly adjust levels (e.g., 0-100%) with visual feedback like a circular progress bar. The Presets page should follow, offering quick access to predefined scenes (e.g., "Reading", "Sleep"). The 3-dot indicator will keep users oriented, ensuring they know which page they are on without cluttering the small screen.

---

## 17. Round Display UI Design Guidelines

### Findings

1. **Central Focus Layout**: For round displays, the main content should be placed in the center of the screen using a large font size to draw attention. This ensures scannability and quick comprehension, which is crucial for smart home controllers where interactions are brief.

2. **Typography and Spacing**: Typography must be highly readable. A recommended practice is to use a line spacing of 140%-180% for optimal readability. For hierarchical text, main values (like H1 headers) can be set around 48px, while body text should be smaller (e.g., 16px), ensuring clear visual hierarchy on a compact 240x240 screen.

3. **Dark Backgrounds**: Utilizing black or dark backgrounds is highly recommended for round displays. It reduces screen glare, blends naturally with the hardware bezel, and is more energy-efficient, which is particularly beneficial for always-on or frequently accessed smart home devices.

4. **Circular Alignment and Margins**: UI elements should align with the circular shape of the screen. This includes using circular progress bars and placing indicators (like navigation dots) along the curved boundaries. Additionally, when integrating a rotary encoder with a capacitive touch panel, a minimum clearance of 5mm should be maintained behind the module for airflow and alignment, which also translates to leaving adequate visual margins to avoid cramped layouts.

5. **High Contrast and Legibility**: To ensure the UI is readable at a glance, there must be a high contrast between the text and the background. Avoid using colors with similar hue and brightness. This is essential for a bedroom controller where lighting conditions can vary significantly.

### Sources

https://developer.samsung.com/one-ui-watch-tizen/principle.html, http://ticdesign.chumenwenwen.com/en/design/, https://dev.to/jasonliu112/round-lcd-displays-for-embedded-ui-a-practical-guide-2lp0, https://blog.prototypr.io/8-rules-for-perfect-typography-in-ui-21b37f6f23ce

### Application to VelaDial

The findings directly inform the design of the VelaDial premium bedroom light controller. By utilizing a dark background, the 240x240 display will blend seamlessly into the bedroom environment, reducing glare and saving power. The Three-Screen Tab Carousel (Power, Brightness, Presets) will benefit from a central focus layout, where the primary control (e.g., the brightness percentage or power toggle) is prominently displayed in the center, utilizing a large, readable font (e.g., 48px for main values).

To accommodate the physical rotary encoder, the UI will incorporate a 5mm clearance around the edges to prevent visual crowding and ensure smooth interaction. The 3-dot page indicator will be placed along the circular boundary, following the natural curve of the screen, providing clear navigation cues without cluttering the central content. Typography will be carefully selected, ensuring high contrast and adequate line spacing (140%-180%) for readability at a glance, even in low-light conditions.

---

## 18. LVGL button matrix preset selector

### Findings

The LVGL Button Matrix (`lv_buttonmatrix`) is a highly memory-efficient widget designed to display multiple buttons in rows and columns. It is lightweight because the buttons are virtually drawn on the fly rather than created as individual objects, requiring only about eight extra bytes of memory per button compared to the ~100-150 bytes needed for a standard Button widget plus a Label widget.

A key feature for creating a preset selector is the "One-checked" functionality. By calling `lv_buttonmatrix_set_one_checked(btn_matrix, true)` and enabling the `LV_BUTTONMATRIX_CTRL_CHECKABLE` flag on the buttons, the matrix behaves like a radio button group where only one option can be selected at a time. This is perfect for selecting a single active lighting preset.

The layout of the buttons is controlled by a descriptor string array (a map) passed to `lv_buttonmatrix_set_map()`. Line breaks are inserted using `"\n"`, and the width of each button can be proportionally adjusted relative to others in the same row using `lv_buttonmatrix_set_button_width()`, which accepts a width value in the range of 1 to 15 (default is 1).

For hardware integration, specifically with a physical rotary encoder, the Button Matrix supports encoder navigation out of the box when added to an input group. However, to avoid issues where long-pressing the encoder to leave edit mode accidentally triggers a button repeat, developers should apply `lv_buttonmatrix_set_button_ctrl_all(btn_matrix, LV_BUTTONMATRIX_CTRL_CLICK_TRIG | LV_BUTTONMATRIX_CTRL_NO_REPEAT)`.

Visually, the Button Matrix can be extensively customized using the `LV_EVENT_DRAW_TASK_ADDED` (or `LV_EVENT_DRAW_PART_BEGIN` in older versions) event. This allows developers to intercept the drawing process and modify the draw descriptors (e.g., `lv_draw_fill_dsc_t`) for individual buttons based on their ID. For a round display, buttons can be styled with `LV_RADIUS_CIRCLE` or custom background colors to fit the circular aesthetic and indicate the currently selected preset.

### Sources

https://lvgl.io/docs/open/9.5/widgets/buttonmatrix.html, https://lvgl.io/docs/open/8.3/widgets/core/btnmatrix, https://lvgl.io/docs/open/8.3/layouts/grid

### Application to VelaDial

For the VelaDial premium bedroom light controller, the LVGL button matrix is an ideal solution for the "Presets" page on the 240x240 round 1.28" display. By configuring the button matrix with `LV_BUTTONMATRIX_CTRL_CHECKABLE` and enabling the "One-checked" feature via `lv_buttonmatrix_set_one_checked(btn_matrix, true)`, the UI can function as an exclusive preset selector where only one lighting scene (e.g., "Reading", "Relax", "Sleep") is active at a time.

To optimize for the round display and rotary encoder input, the button matrix should be styled with custom draw descriptors to ensure buttons fit within the circular bounds, possibly using `LV_RADIUS_CIRCLE` for individual buttons. The encoder navigation can be seamlessly integrated by adding the button matrix to the default input group, allowing users to rotate the physical dial to highlight presets and press to select. To prevent accidental triggering when exiting edit mode with the encoder, the `LV_BUTTONMATRIX_CTRL_CLICK_TRIG` and `LV_BUTTONMATRIX_CTRL_NO_REPEAT` flags should be applied to all buttons.

---

## 19. Wake-to-known-state UX pattern always start from page 0

### Findings

1. **Predictability and Cognitive Load:** The "wake-to-known-state" pattern significantly reduces cognitive load by ensuring the device always returns to a predictable default screen (Page 0) after a timeout. This is crucial for smart home devices where users expect immediate, intuitive control without having to remember the previous state of the UI.

2. **Timeout Implementation:** Industry standards for smart displays typically employ a timeout period ranging from 15 to 60 seconds of inactivity before the screen dims or turns off. Upon waking, the system programmatically resets the UI view to the primary dashboard or control page.

3. **LVGL and ESP32-S3 Integration:** In LVGL (Light and Versatile Graphics Library) running on an ESP32-S3, this pattern is implemented using an inactivity timer (`lv_timer`). When the timer expires, the display goes to sleep. The wake interrupt (from touch or the rotary encoder) must include a callback that explicitly calls `lv_scr_load()` or a similar function to render Page 0, rather than simply turning the backlight back on.

4. **Rotary Encoder Context Reset:** When returning to Page 0, it is essential to reset the input focus of the physical rotary encoder. In LVGL, this means updating the `lv_group` associated with the encoder to focus on the primary interactive element of Page 0, ensuring that the first physical turn performs the expected action (e.g., toggling power) rather than an action from a previously active page.

5. **Visual Feedback (3-Dot Indicator):** The UI must immediately reflect the reset state. For a 3-page carousel, the 3-dot indicator must instantly highlight the first dot upon waking. This provides immediate visual confirmation to the user that they are on the primary page, reinforcing the known state before they even interact with the controls.

### Sources

https://www.nngroup.com/articles/empty-state-interface-design/, https://forum.lvgl.io/t/understanding-rotary-encoder-or-physical-input-to-ui-elements/18715, https://lvgl.io/docs/open/9.1/porting/indev, https://github.com/HASwitchPlate/openHASP/discussions/37, https://docs.unity3d.com/Packages/com.unity.inputsystem@1.8//changelog/CHANGELOG.html

### Application to VelaDial

For the VelaDial premium bedroom light controller, implementing the "wake-to-known-state" pattern means the 240x240 round display should always wake up to Page 0 (Power) after a period of inactivity, regardless of which page the user was on before the screen timed out. This ensures that when a user reaches for the dial in the middle of the night or early morning, they are presented with the most critical and expected control (Power) rather than being confused by a secondary page like Presets or Brightness.

The 3-dot page indicator should clearly highlight the first dot upon waking, reinforcing the user's location within the UI. The physical rotary encoder should also reset its contextual mapping to the Power page's functions. This predictable behavior reduces cognitive load and prevents accidental adjustments to brightness or presets when the user simply wants to turn the light on or off.

Using LVGL on the ESP32-S3, this can be implemented by setting an inactivity timer that triggers a screen sleep function. Upon waking (either via touch or rotary encoder movement), the LVGL screen manager should explicitly load the Page 0 object and reset the encoder's input group focus to the primary control on that page.

---

## 20. LVGL ESP32-S3 200ms ease-in-out animation performance

### Findings

Research into LVGL performance on the ESP32-S3 for a 240x240 round display reveals several critical factors for achieving smooth 200ms ease-in-out page transitions. First, the hardware configuration is paramount: the ESP32-S3 must be set to its maximum CPU frequency of 240MHz and the SPI display bus should be overclocked to 80MHz. This provides the necessary bandwidth and processing power to handle the vector math and pixel pushing required for high-frame-rate animations.

Second, memory architecture plays a massive role in transition smoothness. While the ESP32-S3 supports Octal PSRAM, relying on it for the active display buffer during animations can introduce latency. For a 240x240 display at 16-bit color (RGB565), a full frame is only 115.2KB. It is highly recommended to use double buffering allocated in the internal SRAM with DMA (Direct Memory Access) enabled. This allows the CPU to render the next frame into one buffer while the DMA hardware asynchronously transmits the other buffer to the display, eliminating screen tearing and CPU blocking.

Third, the specific animation timing and path are crucial for a premium feel. LVGL provides the `lv_anim_path_ease_in_out` function, which creates a smooth acceleration and deceleration curve. For a 200ms transition, this path ensures the movement doesn't feel abrupt or linear. However, because 200ms is a very short window (only 6 frames at 30 FPS), the system must maintain a consistent frame rate; any dropped frames will be highly noticeable as stuttering.

Fourth, rendering strategy impacts performance during transitions. Using LVGL's partial render mode (`LV_DISPLAY_RENDER_MODE_PARTIAL`) with small strip buffers (e.g., 1/10th of the screen) can sometimes cause a "tiling penalty" where the CPU recalculates vector paths multiple times. For full-screen page swipes, using larger buffers or a full-frame buffer (if it fits in SRAM) often yields better performance by reducing the overhead of multiple draw calls per frame.

Finally, compiler optimizations are necessary. Enabling `-O3` (Optimize for performance) in the ESP-IDF configuration significantly speeds up the LVGL rendering engine. Additionally, ensuring the LVGL task stack size is increased (e.g., to 64KB) prevents stack overflows during complex rendering operations, which can occur when animating multiple UI elements simultaneously during a page transition.

### Sources

https://forum.lvgl.io/t/transition-performance-issues/19560, https://wiki.seeedstudio.com/round_display_animation_workshop/, https://forum.lvgl.io/t/fps-drop-during-scrolling/23425, https://forum.lvgl.io/t/scroll-animation-speed-in-tile-view-widget/16361, https://forum.lvgl.io/t/screen-scrolling-speed/3294, https://forum.lvgl.io/t/how-to-render-objects-to-buffer-in-psram/23702

### Application to VelaDial

The findings directly apply to the VelaDial premium bedroom light controller's "Three-Screen Tab Carousel" UI. The 240x240 1.28" round display is well within the ESP32-S3's capabilities, especially when using a 16-bit color depth (RGB565) which requires only 115.2KB per full frame. To achieve the premium feel of a 200ms ease-in-out transition between the Power, Brightness, and Presets pages, the system should utilize double buffering in internal SRAM rather than PSRAM. Since the buffer size is small enough, keeping it in the fast internal memory avoids the latency penalty of PSRAM access during DMA transfers.

For the 200ms ease-in-out animation specifically, LVGL's `lv_anim_path_ease_in_out` provides the exact timing curve needed for a premium feel—starting slow, accelerating through the middle, and decelerating at the end. However, to maintain a high frame rate (30+ FPS) during this 200ms window, the ESP32-S3 must be configured to run at its maximum 240MHz CPU frequency and 80MHz SPI clock speed. Furthermore, using partial refresh rendering mode with the double buffers allows the CPU to render the next frame while the DMA controller transmits the current frame to the display, ensuring the swipe animation remains fluid and tear-free.

Finally, the physical rotary encoder integration should be decoupled from the display rendering loop. By handling the encoder inputs on the second core of the ESP32-S3 or via interrupts, the UI thread can dedicate its resources to maintaining the smooth 200ms transition animation without stuttering when the user interacts with the dial.

---
