# VelaDial — Premium UI Concept Direction Matrix

**Status:** Premium concept exploration. No firmware changes. No implementation claims. No physical PASS.
**Supersedes:** Claude PR #33 concept direction matrix (too conservative, too generic).
**Scope:** All 20 concepts expanded with real product design thinking.
**Author context:** This document treats VelaDial as a premium bedroom product, not a generic IoT widget. Every concept is explored as if it were the hero direction for a product launch.

---

## How to Read This Document

Each of the 20 concepts below is expanded into a full design direction. The structure for each concept includes: visual identity, screen architecture, page mapping, interaction model, motion concept, LED ring behavior, dark-room and daylight behavior, LVGL feasibility, implementation risk, what makes it premium, what makes it unique, what could go wrong, required assets, and a v1/v1-expanded/v2 classification.

The concepts are numbered 1–20 matching the original concept table. Concepts 1–12 are "Standard" novelty (familiar smart-home patterns adapted to the round display). Concepts 13–20 are "9/10" novelty (original visual metaphors designed to make VelaDial iconic).

**Important:** Concept exploration is deliberately expansive. Pages beyond the locked v1 three (Power, Brightness, Presets) are explored freely but clearly marked as requiring Hardik approval before any firmware work. The locked v1 constraints are preserved as the implementation baseline; everything beyond them is labeled.

---

## Master Concept Table

| # | Concept | Novelty | Feasibility | Visual Batch? |
|---:|---------|---------|-------------|:-------------:|
| 1 | Minimal Thermostat | Standard | Easy | Yes |
| 2 | SmartKnob-Inspired Arc | Standard | Medium | Yes |
| 3 | Large Center Power Button | Standard | Easy | Yes |
| 4 | Single-Page Simple Mode | Standard | Easy | Yes |
| 5 | Preset Ring UI | Standard | Medium | Yes |
| 6 | Night Mode Ultra-Minimal | Standard | Easy | Yes |
| 7 | Text-First Utility | Standard | Easy | Yes |
| 8 | Apple Watch Complications | Standard | Hard | No |
| 9 | LED-Ring Status-First | Standard | Medium | No |
| 10 | Three-Screen Tab Carousel | Standard | Easy | Yes |
| 11 | Brightness-First UI | Standard | Easy | Yes |
| 12 | Door Switch Replacement | Standard | Easy | Yes |
| 13 | Lunar Phase Visualization | 9/10 | Medium | Yes |
| 14 | Sundial Shadow UI | 9/10 | Medium | Yes |
| 15 | Tree Ring Growth Pattern | 9/10 | Medium | Yes |
| 16 | Topographic Contour Map | 9/10 | Medium | Yes |
| 17 | Iris Aperture Mechanism | 9/10 | Medium | Yes |
| 18 | Radar Sweep Animation | 9/10 | Medium | Yes |
| 19 | Vinyl DJ Crossfader | 9/10 | Medium | Yes |
| 20 | Eclipse Corona | 9/10 | Medium | Yes |

---

## Concept 1 — Minimal Thermostat

### Visual Identity

The Minimal Thermostat draws from the visual language of the Nest Learning Thermostat and the Ecobee SmartThermostat: a single dominant value in the center of a circular field, surrounded by a thin status ring that communicates system state at a glance. The round GC9A01A display is perfectly suited to this metaphor because the physical bezel becomes the "housing" of a virtual thermostat dial. The key difference from an actual thermostat is that VelaDial controls light, not temperature — so the central value is brightness percentage rather than degrees, and the status ring communicates light-on/off rather than heating/cooling.

The visual register is deliberately understated. Pure black background bleeds into the black bezel, making the display appear to float. The amber accent color (`#FFB300`) is used exclusively for the status ring and the active-state indicator, never as a fill. White text at high contrast provides the primary readability layer. Typography is large and confident — a single two- or three-digit number dominates the center, with a small unit label (`%` or `ON`/`OFF`) below it.

### Screen Architecture

| Page | Center Content | Ring Element | Knob Behavior | Touch Behavior |
|------|---------------|-------------|---------------|----------------|
| Power | Large `ON` / `OFF` label, 48pt | Thin amber ring (on) or no ring (off) | Press = toggle | Tap center = toggle |
| Brightness | Large `75%` value, 56pt | Amber arc showing current level | Rotate = adjust, Press = return to Power | Swipe to navigate |
| Presets | 2x2 grid, preset names | Amber highlight on active preset | Press = apply highlighted | Tap preset tile |

The Minimal Thermostat naturally maps to the locked v1 three-page structure. No additional pages are required, though an optional Status page showing ambient lux, temperature, and humidity could be added as a v1-expanded fourth page if Hardik approves.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Status/Diagnostics | Show TSL2591 lux, SHT45 temp/humidity, WiFi signal, uptime | v1-expanded |
| Timer | Sleep timer countdown with arc visualization | v2 |
| Scene | Multi-room scene triggers beyond single bedroom group | v2 |

### Interaction Model

The knob is the primary input. Rotating the knob on the Brightness page directly drives the arc and the center value with zero perceptible latency (local update, debounced HA command). The physical rotation maps 1:1 to the visual arc movement, creating the "connected instrument" feel described in the UI Brief. On the Power page, the knob has no rotation function — only press-to-toggle. On the Presets page, rotation could optionally cycle through presets (v1-expanded), but the locked v1 behavior is press-to-apply with touch-to-select.

Touch is reserved for deliberate actions: wake from sleep, toggle power, select a preset, navigate between pages via horizontal swipe. The 3-dot page indicator sits at the bottom of the round screen, with the active dot in amber.

### Sleep/Wake Behavior

Wake-only-first is strictly enforced. Any input while asleep (touch, knob rotation, knob press) wakes the display only. The display fades in from black over approximately 300ms. No light state changes occur until a second deliberate action while awake. Sleep timeout is 60 seconds of inactivity, with the display fading to black over approximately 500ms.

### LED Ring Behavior

The 5-LED WS2812 ring mirrors the power state: all LEDs glow warm amber at low brightness (approximately 15/255) when lights are on, all LEDs off when lights are off. In Night mode (TSL2591 lux < 5), LED ring brightness is further reduced to approximately 5/255 to avoid being a light source itself. No animation, no flashing, no color changes.

### Backlight Behavior

TSL2591-driven adaptive backlight with three tiers: Night (lux < 5) at 10% PWM, Dim (lux 5-50) at 40% PWM, Bright (lux > 50) at 100% PWM. Smooth transitions between tiers over approximately 1 second. The backlight never exceeds the ambient brightness enough to be a disturbance.

### Motion/Animation Concept

Minimal motion by design. The only animations are: fade-in on wake (300ms), fade-out on sleep (500ms), page transition slide (200ms horizontal translate), brightness arc smooth tracking during knob rotation. No continuous animation. No particle effects. No breathing. This is the quietest concept in the matrix.

### Dark-Room Usability

Excellent. The pure black background with amber accent is specifically designed for dark bedrooms. The large center value is readable at arm's length even at minimum backlight. No bright fills, no white backgrounds, no elements that could cause eye strain at 3 AM.

### Daylight Readability

Good. High contrast white-on-black with amber accent remains readable in bright conditions. The adaptive backlight at 100% PWM provides sufficient brightness for daylight viewing. The simplicity of the layout means there are no fine details that could wash out.

### LVGL Implementation Feasibility

**Easy.** All elements are native LVGL primitives: `lv_label` for center text, `lv_arc` for the brightness ring, `lv_btn` for preset tiles, `lv_obj` with border-radius for the status ring. No custom drawing, no image assets, no complex animations. This concept could be implemented entirely with LVGL style properties and standard widgets. Estimated LVGL widget count per page: 5-8.

### Expected Performance Risk

**Very Low.** No continuous animation, no image decoding, no complex redraw. The ESP32-S3 handles this with significant headroom. Page transitions are simple horizontal slides. The brightness arc updates are lightweight arc redraws.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| Roboto font, 3 sizes (24pt, 48pt, 56pt) | TTF/OTF converted to LVGL font | Google Fonts |
| Power icon (optional) | Simple circle/line glyph or text "ON"/"OFF" | Text-based, no asset needed |

### What Makes It Premium

The premium quality comes from restraint. Like a Braun clock or a Dieter Rams radio, the Minimal Thermostat says "I am confident enough to show you only what matters." The black-on-black bezel integration, the single amber accent, and the large typography create a product that looks expensive because it looks intentional. Premium is not about complexity — it is about the confidence to be simple.

### What Makes It Unique

On a market full of rainbow-LED smart switches and busy dashboard displays, a device that shows a single number on a black field is distinctive by absence. The round form factor reinforces the thermostat metaphor in a way that no rectangular display can. The physical knob completing the visual arc creates a tangible-digital connection that most smart home devices lack entirely.

### What Could Go Wrong

The concept is so minimal that it risks feeling "unfinished" or "cheap" if the typography or spacing is not pixel-perfect. A poorly kerned font or misaligned center value would destroy the premium feel instantly. The concept also offers limited visual differentiation between pages — all three pages are essentially "black circle with something in the middle" — which could make navigation feel undifferentiated.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | 3 pages (Power, Brightness, Presets), wake-only-first, 4 presets, adaptive backlight, LED ring on/off |
| **v1-expanded** | Optional Status page, knob-cycle presets, ambient-responsive accent intensity |
| **v2** | Timer page, scene triggers, SHT45 comfort display, multi-room control |

---

## Concept 2 — SmartKnob-Inspired Arc

### Visual Identity

This concept takes direct inspiration from Scott Bezek's SmartKnob project — a haptic input device where the physical rotation of a knob is visually mirrored by an arc on a round display. The key insight is that the knob and the arc are the same gesture: turning the physical dial IS drawing the visual arc. This creates a deeply satisfying input-output loop that makes the device feel like a precision instrument rather than a remote control.

The visual language is more dynamic than the Minimal Thermostat. The arc is the hero element — a thick, luminous amber sweep that grows and shrinks with knob rotation. The arc starts from the 7 o'clock position and sweeps clockwise to the 5 o'clock position, leaving a gap at the bottom for the page indicator. The center of the circle contains the numeric value and a small contextual label. The background is pure black, and the arc itself provides the ambient glow that makes the display feel alive.

The arc thickness is critical to the premium feel. Too thin and it looks like a loading spinner. Too thick and it overwhelms the center content. The recommended thickness is approximately 12-16 pixels on the 240x240 display, with rounded endcaps that give the arc a polished, organic feel rather than a harsh geometric cut.

### Screen Architecture

| Page | Arc Element | Center Content | Knob Behavior | Touch Behavior |
|------|------------|---------------|---------------|----------------|
| Power | Full amber arc (on) / empty arc track (off) | Power icon + `ON`/`OFF` | Press = toggle | Tap center = toggle |
| Brightness | Amber arc proportional to brightness % | Large `%` value + "Brightness" label | Rotate = sweep arc, Press = return to Power | Drag arc (PENDING), swipe to navigate |
| Presets | Four small arcs at cardinal positions, active one filled amber | Preset name in center | Press = apply, Rotate = cycle (v1-expanded) | Tap quadrant to select |

The Presets page reimagines the 2x2 grid as four arc segments arranged at the cardinal points of the circle (top, right, bottom-right, bottom-left), each representing one preset. The active preset's arc segment is filled amber; inactive ones show a dim track. This is more visually cohesive with the arc language than a rectangular grid, but requires more LVGL work.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Ambient | TSL2591 lux displayed as a thin outer arc, SHT45 as center readout | v1-expanded |
| LED Preview | Shows current LED ring color/brightness as a simulated ring on screen | v1-expanded |
| Favorites | Quick-access to 2-3 most-used presets as large arc buttons | v2 |

### Interaction Model

The knob-to-arc relationship is the defining interaction. When the user rotates the knob clockwise on the Brightness page, the arc sweeps forward in real-time. The visual feedback is immediate — the arc endpoint tracks the knob position with no perceptible delay. The HA brightness command is debounced (200ms after last rotation), but the visual arc updates on every encoder tick. This creates the illusion of direct manipulation even though the actual light change is slightly delayed.

On the Power page, the arc serves as a status indicator rather than an input. When lights are on, the arc is fully swept in amber. When lights are off, only the dim track is visible. The transition between states is a smooth sweep animation (approximately 400ms) that visually "fills" or "drains" the arc.

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, the arc fades in from zero opacity to full over 300ms, followed by the center content appearing over 200ms. This staged reveal creates a sense of the display "powering up" rather than simply appearing. Sleep reversal: center content fades first (200ms), then arc drains to zero (300ms).

### LED Ring Behavior

The LED ring echoes the arc. When the brightness arc is at 50%, approximately 2-3 of the 5 LEDs glow amber. At 100%, all 5 glow. At 0% (lights off), all LEDs are off. The LED ring becomes a physical extension of the on-screen arc — visible from across the room when the display itself is too small to read.

### Backlight Behavior

Same TSL2591-driven adaptive system as Concept 1. The arc's amber color is chosen to remain visible even at minimum backlight in Night mode.

### Motion/Animation Concept

The primary motion is the arc sweep — smooth, continuous tracking of knob rotation. Secondary motions include: power toggle arc fill/drain (400ms), page transition with arc morphing (the arc on one page shrinks while the arc on the next page grows, creating a visual handoff), preset selection pulse (selected arc segment briefly brightens then settles). All animations are ease-in-out curves, never linear, never bouncy.

### Dark-Room Usability

Excellent. The amber arc on black is the ideal dark-room combination — warm, non-stimulating, and visible without being bright. The arc provides enough ambient glow to locate the device without the backlight needing to be high.

### Daylight Readability

Good. The thick arc and large center value maintain readability in bright conditions. The arc's amber color has sufficient contrast against black even in direct light.

### LVGL Implementation Feasibility

**Medium.** The brightness arc is a native `lv_arc` widget — straightforward. The power page arc fill/drain animation requires a timed arc value change — achievable with `lv_anim`. The Presets page with four arc segments is more complex: it requires either four separate `lv_arc` widgets with different start/end angles, or a custom canvas draw. The arc morphing page transition is the hardest element — it may need to be simplified to a standard slide transition for v1.

### Expected Performance Risk

**Low-Medium.** Single arc updates are lightweight. The risk comes from the Presets page (four simultaneous arcs) and the page transition morphing. If morphing is too expensive, falling back to a standard slide transition preserves the concept without the performance cost.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| Roboto font, 3 sizes | TTF/OTF to LVGL font | Google Fonts |
| No image assets required | N/A | Arc is pure LVGL drawing |

### What Makes It Premium

The 1:1 knob-to-arc mapping creates a tactile-visual connection that most smart home devices completely lack. When you turn the knob and watch the arc sweep, the device feels like a precision instrument — like adjusting a high-end audio amplifier or a camera lens. This is the kind of interaction that makes people want to show the device to guests.

### What Makes It Unique

The SmartKnob-inspired arc is the only concept that fully exploits the physical rotary encoder as a design element rather than just an input method. The knob IS the UI. No other smart light controller on the market offers this level of physical-digital integration on a round display.

### What Could Go Wrong

The arc sweep must be perfectly smooth. Any stutter, lag, or quantization in the arc movement would destroy the premium feel. The encoder resolution (typically 20-24 detents per revolution) means the arc moves in discrete steps — if those steps are visible as jumps rather than smooth motion, the concept loses its magic. The Presets page with four arcs may look cluttered on 240x240 if not carefully sized.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | Brightness arc, power arc status, 2x2 preset grid (simplified from arc segments), wake-only-first |
| **v1-expanded** | Four-arc preset page, arc morphing transitions, LED ring arc echo |
| **v2** | Ambient arc page, favorites, arc-based timer countdown |

---

## Concept 3 — Large Center Power Button

### Visual Identity

This concept puts a single, oversized power button at the absolute center of the display. The button is a perfect circle — approximately 140px diameter on the 240x240 screen — that dominates the visual field. When lights are on, the button glows amber with a soft radial gradient that fades into the black background, creating a "lamp" effect where the button itself appears to be a light source. When lights are off, the button is a dark circle with a thin white border, barely visible — a ghost of the control waiting to be activated.

The metaphor is the most primal one in lighting: a light switch. But instead of a cheap plastic rocker, this is a luminous disc floating in darkness. The round display and the round button create concentric circles that feel harmonious and intentional. The bezel is the outer ring, the button is the inner ring, and the glow bridges them.

### Screen Architecture

| Page | Center Content | Peripheral Content | Knob Behavior | Touch Behavior |
|------|---------------|-------------------|---------------|----------------|
| Power | 140px glowing circle, power icon inside | Small `ON`/`OFF` label below button, 3-dot indicator at bottom | Press = toggle | Tap button = toggle |
| Brightness | 100px circle showing `%` value, surrounded by arc | Thin arc ring at edge of display | Rotate = adjust, Press = return to Power | Swipe to navigate |
| Presets | 4 smaller circles (60px each) arranged in diamond pattern | Active preset circle glows amber | Press = apply | Tap circle to select |

The design language is circles within circles. Every interactive element is circular, reinforcing the round display form factor. The Presets page uses a diamond arrangement (top, left, right, bottom) rather than a 2x2 grid to better fit the circular viewport — the corners of a 2x2 grid get clipped by the round bezel, but a diamond arrangement keeps all four elements within the safe circular area.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Quick Toggle | Single large button for most-used action (e.g., "Nightlight") | v1-expanded |
| Ambient | Concentric rings showing lux (outer) and temp (inner) | v2 |

### Interaction Model

The Power page is touch-first. The large button is an irresistible touch target — users will instinctively tap it. The knob press serves as a redundant toggle for users who prefer physical input. On the Brightness page, the knob takes over as primary input, with the center circle shrinking to make room for the surrounding arc. On the Presets page, touch is primary (tap a circle), with knob press applying the currently highlighted one.

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, the center button fades in first (200ms), then peripheral elements appear (200ms delay). This creates a "spotlight" effect where the most important element appears first. On sleep, the reverse: peripherals fade, then the center button dims to black.

### LED Ring Behavior

The LED ring mirrors the center button: all 5 LEDs glow amber when the button is "on," all off when "off." During the power toggle animation, the LEDs transition in sync with the button glow — creating a unified "the whole device lights up" moment.

### Motion/Animation Concept

The hero animation is the power toggle. When toggling on: the center button expands slightly (scale 1.0 to 1.05 over 150ms), fills with amber glow (200ms radial gradient animation), then settles back to 1.0 (100ms). When toggling off: the glow contracts inward and disappears (300ms), leaving the ghost outline. This is a single-event animation, not continuous — it fires once per toggle and costs almost nothing in performance.

### Dark-Room Usability

Excellent. The glowing center button is specifically designed to be visible in complete darkness without being bright enough to disturb. The amber glow is warm and non-stimulating. The large touch target (140px) is easy to hit even when half-asleep.

### Daylight Readability

Good. The large button and high contrast ensure readability. The glow effect is less visible in daylight, but the button outline and icon remain clear.

### LVGL Implementation Feasibility

**Easy.** The center button is an `lv_btn` with `lv_style_set_radius(LV_RADIUS_CIRCLE)` and background gradient. The glow effect can be approximated with a shadow or a second larger circle behind the button with lower opacity. The brightness arc is standard `lv_arc`. The diamond preset layout is four `lv_btn` widgets positioned manually. No custom drawing required.

### Expected Performance Risk

**Very Low.** The toggle animation is a one-shot style transition — LVGL handles these natively with `lv_anim`. No continuous rendering cost.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| Power icon glyph | Simple line art or Unicode character | Text-based or minimal SVG |
| Roboto font, 2 sizes | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

The oversized glowing button creates an emotional response — it feels like pressing a luxury car's ignition button or tapping a high-end audio system's power control. The glow animation adds a moment of delight to every interaction. The concentric-circle design language feels cohesive and intentional.

### What Makes It Unique

Most smart light controllers hide the power toggle behind a menu or make it one of many small buttons. Making it the hero element — 140px of glowing amber — is a bold design choice that says "this device does one thing beautifully."

### What Could Go Wrong

The concept is power-page-centric. The Brightness and Presets pages may feel like afterthoughts if they don't carry the same visual weight. The diamond preset layout needs careful sizing to avoid feeling cramped. The glow effect must be subtle — too bright and it becomes garish, too dim and it disappears.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | Large power button, brightness arc, 2x2 presets (or diamond), wake-only-first |
| **v1-expanded** | Glow animation on toggle, diamond preset layout, Quick Toggle page |
| **v2** | Ambient rings, multi-room power buttons |

---

## Concept 4 — Single-Page Simple Mode

### Visual Identity

This is the most radical simplification in the matrix. Instead of three pages with swipe navigation, Single-Page Simple Mode puts everything on one screen: power state at the top, brightness in the middle, and the active preset name at the bottom. The entire UI is visible at all times — no swiping, no page indicator, no hidden state. The visual language is pure information density within a minimal frame: black background, white text, amber accents, and nothing else.

The layout exploits the round display's geometry by placing content in three horizontal bands. The top band (above center) shows the power state as a small icon and label. The middle band (center) shows the brightness as a large percentage value with a thin surrounding arc. The bottom band (below center) shows the active preset name. The three bands are separated by subtle horizontal dividers that curve to follow the round bezel.

### Screen Architecture

This concept has exactly **one page** for the locked v1 scope:

| Zone | Content | Interaction |
|------|---------|-------------|
| Top (above center) | Power icon + `ON`/`OFF` label, 24pt | Tap to toggle |
| Center | Brightness `%` value, 48pt, with thin arc | Knob rotate = adjust brightness |
| Bottom (below center) | Active preset name, 20pt, amber text | Tap to cycle presets |
| Knob press | Toggle power (same as top zone tap) | N/A |

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Preset Picker | Full-screen 2x2 grid overlay triggered by long-press on bottom zone | v1-expanded |
| Settings | WiFi status, sensor readings, firmware version | v2 |

### Interaction Model

The knob is dedicated to brightness — always. There is no page-specific knob behavior because there is only one page. Knob press toggles power. Touch zones are divided into three regions: top third = power toggle, center = no touch action (knob only), bottom third = cycle to next preset. This is the simplest possible interaction model.

The absence of page navigation means no swipe gestures, no page indicator dots, and no "which page am I on?" confusion. The trade-off is information density — all three functions share 240x240 pixels, so each gets less visual space than it would on a dedicated page.

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, all three zones fade in simultaneously (300ms). On sleep, all fade out together. No staged reveal — the simplicity of the concept extends to the transitions.

### LED Ring Behavior

Same as Concept 1: amber glow when on, off when off. No per-zone LED mapping (the single-page concept is too simple to warrant LED complexity).

### Motion/Animation Concept

Almost zero motion. The only animations are wake/sleep fade and brightness value number-roll (the percentage smoothly increments/decrements as the knob turns, like an odometer). No page transitions because there are no pages.

### Dark-Room Usability

Excellent. The sparse layout with large center value is ideal for dark-room use. The user can read the brightness and power state at a glance without any interaction.

### Daylight Readability

Good. High contrast, large text, simple layout.

### LVGL Implementation Feasibility

**Very Easy.** Three `lv_label` widgets, one `lv_arc`, and touch zone detection via `lv_obj` click areas. This is the simplest possible LVGL implementation in the entire matrix. Could be implemented in under 100 lines of LVGL configuration.

### Expected Performance Risk

**Extremely Low.** One page, no transitions, no animation. The ESP32-S3 is dramatically over-provisioned for this concept.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| Roboto font, 3 sizes | TTF/OTF to LVGL font | Google Fonts |
| Power icon (optional) | Text or simple glyph | Text-based |

### What Makes It Premium

Simplicity taken to its logical extreme becomes a design statement. Like a single-function kitchen tool from a premium brand — it does one thing, it does it perfectly, and it wastes nothing. The absence of navigation is itself a luxury: the user never has to think about where they are.

### What Makes It Unique

No other smart light controller puts everything on one screen. The industry default is tabs, pages, or menus. Single-Page Simple Mode rejects all of that. It is the anti-dashboard.

### What Could Go Wrong

The three zones may feel cramped on 240x240, especially with the round bezel cutting into the corners. The preset cycling (tap bottom to advance) is less discoverable than a dedicated preset page — new users may not realize they can change presets. The concept may feel "too simple" for users who expect a premium device to have more UI depth.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | Single page with all three functions, wake-only-first, 4 presets via cycling |
| **v1-expanded** | Preset picker overlay on long-press, ambient lux indicator as thin outer ring |
| **v2** | Settings page, timer overlay, multi-room |

---

## Concept 5 — Preset Ring UI

### Visual Identity

The Preset Ring UI places the four presets around the circumference of the round display as arc segments, creating a ring of options that the user navigates by rotating the knob. The active preset's arc segment is filled with its characteristic color (amber for Warm White, deep gold for Soft Amber, cool white for Neutral White, dim amber for Low Nightlight), while inactive segments show dim outlines. The center of the display shows the currently selected preset's name and a brightness value.

This concept treats the round display as a literal dial — the presets are "positions" on the dial that the knob rotates between. The physical metaphor is a vintage radio tuner or a washing machine program selector: rotate to the setting you want, press to confirm. The round form factor is not just accommodated but celebrated — the ring of presets could not exist on a rectangular display.

### Screen Architecture

| Page | Ring Content | Center Content | Knob Behavior | Touch Behavior |
|------|-------------|---------------|---------------|----------------|
| Power | Full amber ring (on) / dim ring (off) | Power icon + state label | Press = toggle | Tap center = toggle |
| Brightness | Amber arc proportional to brightness | Large `%` value | Rotate = adjust, Press = return to Power | Swipe to navigate |
| Presets | 4 arc segments at quadrants, active filled | Preset name + color temp label | Rotate = cycle presets, Press = apply | Tap segment to select |

The Presets page is the hero page of this concept. The four arc segments divide the ring into quadrants: top-right = Warm White, bottom-right = Soft Amber, bottom-left = Neutral White, top-left = Low Nightlight. Each segment spans approximately 80 degrees of arc with 10-degree gaps between them. The active segment is filled with its color; the selection indicator (a small dot or tick mark) sits at the segment's midpoint.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Scene Ring | Additional ring with scene presets (Movie, Reading, Romance, Sleep) | v1-expanded |
| LED Preview | Center shows simulated LED ring color matching selected preset | v1-expanded |
| Favorites | 2-slot quick-access ring with user's most-used presets | v2 |

### Interaction Model

On the Presets page, knob rotation moves the selection indicator around the ring. Each detent of the encoder advances to the next preset segment. The physical rotation directly maps to the visual rotation of the selection — turn right to move clockwise through presets, turn left to move counter-clockwise. Knob press applies the currently highlighted preset. Touch can also select a preset by tapping its arc segment directly.

The ring metaphor extends to the other pages: on the Power page, the ring is a status indicator (full = on, empty = off). On the Brightness page, the ring becomes the brightness arc. The ring is the unifying visual element across all pages, but its function changes per page.

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, the ring fades in first (200ms), then center content appears (200ms). The ring appearing first creates a "frame" that the content fills — like a spotlight warming up before the performer appears.

### LED Ring Behavior

The LED ring mirrors the preset ring. When a preset is selected, the 5 WS2812 LEDs show the preset's characteristic color temperature: warm amber for Warm White, deeper gold for Soft Amber, cool white for Neutral White, very dim amber for Low Nightlight. This creates a physical echo of the on-screen ring that is visible from across the room.

### Motion/Animation Concept

The primary motion is the preset selection rotation. When the knob moves the selection from one segment to the next, the highlight smoothly slides along the ring (approximately 200ms ease-in-out). The previous segment dims while the next segment brightens, creating a "traveling light" effect around the ring. On preset apply (knob press), the selected segment briefly pulses brighter (100ms) as confirmation.

### Dark-Room Usability

Excellent. The ring segments glow against the black background, and the color differentiation between presets is visible even at minimum backlight. The large arc segments are easy to identify by position (top-right is always Warm White) even without reading text.

### Daylight Readability

Good. The filled arc segments have sufficient contrast. The center text provides redundant information for cases where the ring colors are hard to distinguish in bright light.

### LVGL Implementation Feasibility

**Medium.** The four arc segments require four `lv_arc` widgets with custom start/end angles, or a single `lv_arc` with a custom indicator. The selection animation (traveling highlight) requires coordinated `lv_anim` on multiple arcs. The color-per-preset styling needs per-segment style overrides. This is achievable but requires more LVGL configuration than Concepts 1, 3, or 4.

### Expected Performance Risk

**Low-Medium.** Four simultaneous arc widgets with style transitions are heavier than a single arc, but well within ESP32-S3 capability. The traveling highlight animation is the most expensive element — if it stutters, it can be simplified to an instant switch.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| Roboto font, 3 sizes | TTF/OTF to LVGL font | Google Fonts |
| No image assets | N/A | All arcs are LVGL drawing |

### What Makes It Premium

The ring of presets feels like a luxury watch complication — multiple functions elegantly arranged around a circular face. The color-coded segments add visual richness without clutter. The knob-to-ring rotation creates the same satisfying physical-digital connection as Concept 2, but applied to preset selection rather than brightness.

### What Makes It Unique

No smart light controller presents presets as a ring. The industry standard is a list, a grid, or a dropdown. The ring layout is only possible on a round display with a rotary encoder — it is a design that could not exist on any other form factor. This makes VelaDial's Presets page instantly recognizable and impossible to confuse with a competitor.

### What Could Go Wrong

Four arc segments on 240x240 may feel tight. The 10-degree gaps between segments must be large enough to be visible but small enough to not waste space. The color differentiation between Warm White (amber) and Soft Amber (deep gold) may be too subtle on a small display — they need to be visually distinct. If the knob rotation mapping is not perfectly aligned with the visual segments, the interaction will feel broken.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | 4-segment preset ring, brightness arc, power ring, wake-only-first |
| **v1-expanded** | Traveling highlight animation, LED ring color echo, scene ring page |
| **v2** | Favorites ring, user-customizable preset colors, 8-segment ring |


---

## Concept 6 — Night Mode Ultra-Minimal

### Visual Identity

Night Mode Ultra-Minimal is not a standalone UI theme — it is the *sleep state* of every other concept. When the TSL2591 ambient light sensor reads below approximately 5 lux, the display enters Night Mode: backlight drops to minimum, all UI elements except a single small dot or a single dim number disappear, and the WS2812 LED ring either turns off entirely or glows at the absolute minimum visible level (approximately 3/255). The screen appears almost off — a faint heartbeat of information in an otherwise dark room.

The visual register is the opposite of every other concept in this matrix. Where other concepts fill the 240x240 canvas with arcs, buttons, and labels, Night Mode strips everything away. A single amber dot (8px diameter) at the center of the screen communicates "the device is alive and the lights are off." If the lights are on, the dot is replaced by a dim `%` value showing current brightness. Nothing else. No page indicator, no ring, no labels.

This is the concept that remembers VelaDial lives in a bedroom. At 3 AM, the last thing anyone wants is a glowing rectangle on the wall. Night Mode Ultra-Minimal makes the device nearly invisible while remaining instantly accessible — any touch, knob rotation, or knob press wakes the full UI.

### Screen Architecture

Night Mode has exactly one visual state, not pages:

| Light State | Display Content | LED Ring | Backlight |
|-------------|----------------|----------|-----------|
| Lights OFF, room dark | Single 8px amber dot at center | All off | 10% PWM |
| Lights ON, room dark | Dim `%` value at center, 24pt, amber | 1-2 LEDs at minimum (3/255) | 10% PWM |
| Lights OFF, room dim | Normal UI with reduced backlight | All off | 40% PWM |
| Lights ON, room dim | Normal UI with reduced backlight | Low amber (10/255) | 40% PWM |

Night Mode is triggered by the TSL2591 ambient light classifier, not by the user. The transition from normal UI to Night Mode happens automatically when ambient lux drops below the threshold. The transition is a slow fade (approximately 2 seconds) — elements gradually disappear rather than snapping off, creating a "the room is getting dark and so is the display" feel.

### Interaction Model

In Night Mode, all inputs are wake-only-first. Any touch, knob rotation, or knob press wakes the display to the full UI of whichever concept is active (Minimal Thermostat, SmartKnob Arc, etc.). The wake animation is a reverse of the sleep fade: the full UI fades in over approximately 500ms. After the wake timeout (60 seconds of inactivity), the display returns to Night Mode.

There is no interaction within Night Mode itself — no page navigation, no brightness adjustment, no preset selection. The user must wake the display first. This is a deliberate design choice: in a dark bedroom, any accidental touch should produce the minimum possible visual disruption.

### Sleep/Wake Behavior

Night Mode IS the sleep state. Wake-only-first is inherent. The distinction between "display sleep" and "Night Mode" is that Night Mode shows the minimal dot/value, while display sleep shows nothing (backlight fully off). Night Mode is the intermediate state between full UI and full sleep.

### LED Ring Behavior

In Night Mode with lights off, the LED ring is completely dark. In Night Mode with lights on, 1-2 LEDs glow at the absolute minimum visible level (3/255) in warm amber — enough to confirm "yes, the lights are still on" without being a light source itself. This is critical for bedroom use: the LED ring must never be bright enough to disturb sleep.

### Motion/Animation Concept

The only motion is the slow fade transition between full UI and Night Mode (approximately 2 seconds). The dot/value in Night Mode is static — no pulsing, no breathing, no animation. Stillness is the design intent.

### Dark-Room Usability

**This is the dark-room concept.** It is specifically designed for the use case of "it's 3 AM and I need to check if the lights are on without waking my partner." The single dot or dim value is visible at arm's length but invisible from across the room.

### Daylight Readability

N/A — Night Mode only activates in dark/dim conditions. In daylight, the full UI of the active concept is displayed.

### LVGL Implementation Feasibility

**Easy.** Night Mode is implemented as a style overlay: when the TSL2591 triggers the dark threshold, all LVGL widgets except the center dot/label are set to `opa: 0` (fully transparent), and the remaining widget's opacity is reduced to approximately 30%. The backlight PWM is reduced via the `output` component. No new widgets, no new pages, no custom drawing.

### Expected Performance Risk

**Extremely Low.** Night Mode reduces rendering load — fewer visible widgets means less redraw work.

### Required Assets

None beyond what the active concept already uses.

### What Makes It Premium

Night Mode Ultra-Minimal is the feature that separates a bedroom product from a generic smart-home gadget. It shows that the designers thought about the full 24-hour lifecycle of the device, not just the "demo at a trade show" moment. Premium products anticipate needs; Night Mode anticipates the need for darkness.

### What Makes It Unique

Most smart-home displays have a single brightness slider for night use. Night Mode Ultra-Minimal goes further: it redesigns the entire visual output for darkness, stripping away everything except the minimum viable information. The single dot is a design signature — recognizable and memorable.

### What Could Go Wrong

The transition threshold (5 lux) may need tuning per installation. If the TSL2591 is positioned where it reads ambient light inaccurately (e.g., facing a window), Night Mode may activate/deactivate at wrong times. The single dot may be too minimal for some users who want to see more information without waking the full UI.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | TSL2591-driven backlight reduction, minimal center content in dark mode |
| **v1-expanded** | Slow fade transition, dot vs. value distinction, LED ring minimum mode |
| **v2** | User-configurable Night Mode thresholds, scheduled Night Mode override |

---

## Concept 7 — Text-First Utility

### Visual Identity

Text-First Utility rejects all decorative elements. No arcs, no rings, no gradients, no glow effects. Every piece of information is communicated through typography alone. The display shows large, confident text labels — `ON`, `OFF`, `75%`, `WARM WHITE`, `SOFT AMBER` — in a clean sans-serif font against a pure black background. The amber accent color is used only for the active state indicator and the page dots.

The visual reference is not other smart-home devices but rather high-end industrial controls, aviation instrumentation, and the typography of Massimo Vignelli's New York subway signage. Information is organized by size hierarchy: the primary value is the largest element (48-56pt), secondary labels are medium (20-24pt), and tertiary information (page indicator, status) is small (14-16pt). The round bezel crops the text at the edges, which is intentional — it creates a "window into a larger information space" effect.

### Screen Architecture

| Page | Primary Text (center) | Secondary Text | Tertiary Text | Knob Behavior |
|------|----------------------|----------------|---------------|---------------|
| Power | `ON` or `OFF`, 56pt, white (on) or dim gray (off) | "Power" label, 16pt, above center | Connection status, 12pt, bottom | Press = toggle |
| Brightness | `75%`, 56pt, white | "Brightness" label, 16pt, above center | Step indicator `±5%`, 12pt, bottom | Rotate = adjust, Press = return |
| Presets | Active preset name, 32pt, amber | "Presets" label, 16pt, above center | 4 preset names listed vertically, 16pt, active highlighted | Press = apply |

The Presets page is the most text-dense: it lists all four preset names vertically (Warm White, Soft Amber, Neutral White, Low Nightlight) with the active one highlighted in amber and the others in dim gray. The knob cycles the highlight up and down the list. This is a classic menu pattern — familiar, discoverable, and impossible to misunderstand.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Info | Device name, IP address, firmware version, uptime, WiFi signal | v1-expanded |
| Sensor | TSL2591 lux value, SHT45 temp/humidity, raw text readout | v2 |

### Interaction Model

The knob is the primary input on every page. On Power, press toggles. On Brightness, rotate adjusts the number. On Presets, rotate moves the highlight, press applies. Touch is secondary: tap center on Power to toggle, swipe to navigate between pages. The interaction model is identical to the locked v1 spec — Text-First Utility changes only the visual presentation, not the behavior.

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, text appears character by character in a rapid typewriter effect (approximately 50ms per character, total 200-300ms for the primary value). On sleep, text fades uniformly to black. The typewriter wake is a subtle premium touch that reinforces the text-first identity.

### LED Ring Behavior

Minimal. The LED ring shows only on/off state: all amber when on, all off when off. No color variation, no brightness mapping. The LED ring is secondary to the text — this concept trusts the screen to communicate everything.

### Motion/Animation Concept

The typewriter wake effect is the only animation. All other transitions are instant or simple fades. Page transitions are horizontal slides with text sliding as a block. No continuous animation. The concept's motion philosophy is: "text does not need to move to be interesting."

### Dark-Room Usability

Excellent. Large white text on black is the highest-contrast combination possible. The 56pt primary value is readable at arm's length even at minimum backlight. No decorative elements compete for attention.

### Daylight Readability

Excellent. Text-First Utility has the best daylight readability of any concept in the matrix. Large text, maximum contrast, no fine details that could wash out.

### LVGL Implementation Feasibility

**Very Easy.** The entire UI is `lv_label` widgets with style properties (font size, color, alignment). The Presets page list is four `lv_label` widgets with conditional color. The typewriter effect can be implemented with a timed string append. No arcs, no custom drawing, no image assets. This is the second-simplest implementation after Concept 4.

### Expected Performance Risk

**Extremely Low.** Text rendering is the lightest possible LVGL workload.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| Roboto or Inter font, 4 sizes (12pt, 16pt, 32pt, 56pt) | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

Typography IS the design. When text is the only element, every detail matters: letter spacing, line height, weight, alignment. A well-typeset screen feels more intentional than a screen full of widgets. The premium quality comes from the same place as a luxury brand's packaging — the confidence to use white space and let the content speak.

### What Makes It Unique

Smart-home devices almost universally use icons, arcs, and colored fills. A device that uses only text is visually distinctive and immediately communicates "this is a tool, not a toy." The round bezel cropping the text edges creates a unique visual signature.

### What Could Go Wrong

If the font choice or sizing is wrong, the concept collapses into "cheap LCD clock." The difference between premium text-first and budget text-only is entirely in the typographic details. The Presets page with four vertical text items may feel cramped on 240x240 with the round bezel cutting into the top and bottom entries.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | 3 text pages, wake-only-first, 4 presets as text list |
| **v1-expanded** | Typewriter wake effect, Info page, font weight variations |
| **v2** | Sensor readout page, multi-language support, custom font upload |

---

## Concept 8 — Apple Watch Complications

### Visual Identity

This concept borrows the "complications" metaphor from watchOS: multiple small data widgets arranged around a central element on a circular face. Each complication shows a single piece of information — power state, brightness level, active preset, ambient temperature, humidity, lux level — in a compact visual format (small icon + value). The center of the display shows the most important value (brightness or power state), and the complications orbit around it.

The visual density is high. A typical screen might show 4-6 complications plus a center value, all within 240x240 pixels. The aesthetic reference is the watchOS Infograph face: information-rich, colorful, and complex. Each complication has its own accent color and icon.

### Screen Architecture

This concept fundamentally conflicts with the locked v1 three-page structure because it attempts to show multiple data types simultaneously. A v1-compatible adaptation would use complications as decorative elements on the existing three pages rather than as a replacement for them:

| Page | Center Content | Complications (corners) | Knob Behavior |
|------|---------------|------------------------|---------------|
| Power | Power state icon | WiFi signal (top-left), uptime (top-right) | Press = toggle |
| Brightness | `%` value | Lux (top-left), color temp (top-right) | Rotate = adjust |
| Presets | Active preset | Temp (top-left), humidity (top-right) | Press = apply |

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Dashboard | Full complication face with 6 data slots | v2 |
| Sensor Detail | Tap a complication to expand it full-screen | v2 |

### Interaction Model

Complications are display-only in v1 — they show data but are not interactive. The knob and touch behavior remains identical to the locked v1 spec. In v2, tapping a complication could expand it to a full-screen detail view.

### Sleep/Wake Behavior

Wake-only-first enforced. Complications fade in after the center content (staged reveal).

### LED Ring Behavior

Could map one complication to the LED ring (e.g., ambient temperature as color: blue = cold, amber = comfortable, red = hot). This is a v2 feature.

### Motion/Animation Concept

Complications could have subtle update animations (value counter roll, icon pulse on change). These are v2 features. In v1, complications are static text.

### Dark-Room Usability

Poor to marginal. The high information density means many small elements compete for attention in low light. Small text (12-14pt) is hard to read at minimum backlight. This concept is fundamentally at odds with the bedroom-first design philosophy.

### Daylight Readability

Marginal. Small complications may be hard to read even in good light on a 240x240 display. The Apple Watch achieves this with a much higher pixel density (326 PPI vs. VelaDial's approximately 190 PPI).

### LVGL Implementation Feasibility

**Hard.** Each complication needs its own container with icon, value, and optional mini-chart. Positioning 4-6 complications around a center element on a round display requires precise manual layout. The icons need custom font glyphs or small image assets. The data sources (SHT45, TSL2591, WiFi) need to be wired to individual LVGL labels. This is the most complex LVGL implementation in the matrix.

### Expected Performance Risk

**Medium-High.** Multiple simultaneous data updates, multiple label redraws, and potential icon animations create a heavier rendering load than any other concept.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| Icon font with 6-8 glyphs | Custom LVGL font | Material Symbols or custom |
| Roboto font, 4 sizes | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

Information density, when executed well, communicates sophistication. The Apple Watch Infograph face is one of the most recognizable premium UI patterns in consumer electronics.

### What Makes It Unique

No smart light controller shows this level of ambient data. It would make VelaDial feel like a room intelligence hub rather than a light switch.

### What Could Go Wrong

The 240x240 resolution is too low for this concept to work well. Small complications will be blurry, cramped, and hard to read. The concept also promotes SHT45 temperature/humidity to first-class UI elements, which violates the v1 scope (SHT45 is secondary/diagnostics only). The bedroom-first philosophy is compromised by the visual complexity.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | NOT RECOMMENDED for v1 — too complex, violates scope |
| **v1-expanded** | 2 corner complications (WiFi + lux) as decorative additions to existing pages |
| **v2** | Full complication face, interactive complications, sensor detail pages |

---

## Concept 9 — LED-Ring Status-First

### Visual Identity

This concept inverts the typical smart-display hierarchy: the 5-LED WS2812 ring is the primary output, and the 240x240 screen is secondary. The room reads the device's state from the LED ring across the room; the screen is only consulted up close for precise adjustments. The screen UI is deliberately minimal — almost a Night Mode Ultra-Minimal at all times — because the LED ring carries the communication burden.

The visual identity is defined by the LED ring's behavior rather than the screen's layout. The ring uses color temperature to indicate the active preset (warm amber, deep gold, cool white, dim amber), brightness to indicate light level (brighter ring = brighter lights), and on/off state (ring glowing = lights on, ring dark = lights off). The screen shows only a small confirmation label and the page indicator.

### Screen Architecture

| Page | Screen Content | LED Ring Behavior | Knob Behavior |
|------|---------------|-------------------|---------------|
| Power | Small `ON`/`OFF` label, 24pt, center | Full ring: amber (on) or off | Press = toggle |
| Brightness | Small `%` value, 24pt, center | Ring brightness proportional to light brightness | Rotate = adjust |
| Presets | Small preset name, 20pt, center | Ring color matches preset color temperature | Press = apply |

The screen is intentionally understated. No arcs, no large buttons, no decorative elements. The screen exists to confirm what the LED ring is already showing and to provide the precision that 5 LEDs cannot (exact percentage, exact preset name).

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| LED Config | Adjust LED ring brightness ceiling, color preferences | v2 |
| Ambient | LED ring reacts to TSL2591 lux changes in real-time | v2 |

### Interaction Model

Same as locked v1 spec. The knob and touch behavior are unchanged. The only difference is that the visual feedback comes primarily from the LED ring rather than the screen. When the user rotates the knob to adjust brightness, they watch the LED ring get brighter/dimmer rather than watching an arc on the screen.

### Sleep/Wake Behavior

Wake-only-first enforced for the screen. The LED ring has its own sleep behavior: when the display sleeps, the LED ring remains in its current state (showing power/brightness/preset) for an additional 30 seconds, then fades to off. This allows the LED ring to serve as a "nightlight confirmation" after the screen has gone dark.

### LED Ring Behavior

This is the hero element. The 5 LEDs are individually addressable, enabling patterns:

| State | LED Pattern |
|-------|------------|
| Lights on, 100% brightness | All 5 LEDs at full preset color |
| Lights on, 50% brightness | All 5 LEDs at 50% of preset color |
| Lights on, 20% brightness | 1 LED at preset color, others off |
| Lights off | All LEDs off |
| Preset: Warm White | Amber (#FFB300) |
| Preset: Soft Amber | Deep gold (#CC8800) |
| Preset: Neutral White | Cool white (#E0E0E0) |
| Preset: Low Nightlight | Very dim amber (#FFB300 at 10%) |

### Motion/Animation Concept

The LED ring has subtle transition animations: when changing brightness, the LEDs fade smoothly (200ms). When changing presets, the LEDs cross-fade between colors (300ms). When toggling power, the LEDs fade in/out (400ms). These animations are visible from across the room and create a sense of the device "breathing" through state changes.

### Dark-Room Usability

**Excellent for the LED ring, excellent for the screen.** The LED ring at minimum brightness is visible without being disturbing. The minimal screen content is easy to read. The combination of ambient LED feedback and minimal screen creates the ideal dark-room experience.

### Daylight Readability

**Marginal for the LED ring, marginal for the screen.** The LED ring is hard to see in bright daylight. The minimal screen content (small text, no decorative elements) may feel insufficient in a well-lit room where the user expects more visual feedback.

### LVGL Implementation Feasibility

**Easy for the screen, Medium for the LED ring.** The screen UI is trivially simple — three `lv_label` widgets across three pages. The LED ring behavior requires careful WS2812 programming with color temperature mapping, brightness scaling, and smooth transitions. The WS2812 control is done via ESPHome's `light` component, not LVGL.

### Expected Performance Risk

**Low.** The screen rendering is minimal. The LED ring transitions are handled by the WS2812 driver, not the LVGL renderer.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| Roboto font, 2 sizes | TTF/OTF to LVGL font | Google Fonts |
| Color temperature map | Configuration values | Manual tuning |

### What Makes It Premium

The LED ring as primary output creates an ambient intelligence feel — the device communicates its state to the entire room, not just to the person standing in front of it. This is the "smart home that knows you're there" experience. Walking into a bedroom and seeing the warm amber glow of the LED ring confirming "lights are on, Warm White preset" is a premium moment.

### What Makes It Unique

No smart light controller prioritizes the LED ring over the screen. This inversion of the typical hierarchy is distinctive and practical — it acknowledges that most of the time, the user is not standing at the device but somewhere else in the room.

### What Could Go Wrong

Five LEDs is a very limited canvas. The brightness-to-LED mapping (e.g., "1 LED at 20%") may feel crude. The color temperature differences between presets may be hard to distinguish on 5 small LEDs. Users may feel the screen is "wasted" if it only shows small text. The concept requires careful LED brightness calibration to avoid the ring being too bright in a dark bedroom.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | LED ring on/off echo, minimal screen labels, wake-only-first |
| **v1-expanded** | LED ring brightness scaling, preset color mapping, extended LED timeout |
| **v2** | Individual LED patterns, LED config page, ambient-reactive LED |

---

## Concept 10 — Three-Screen Tab Carousel

### Visual Identity

The Three-Screen Tab Carousel is the direct, literal implementation of the locked v1 specification. Three horizontally swipeable pages — Power, Brightness, Presets — with a 3-dot page indicator at the bottom. Active dot is amber, inactive dots are dim gray. Each page has a focused, single-purpose layout. This is not a creative reinterpretation; it is the spec made visual.

The visual language is clean and functional. Black background, white primary text, amber accents for active states and the page indicator. Each page uses the full 240x240 canvas for its single purpose: Power gets a large toggle area, Brightness gets a large arc with center value, Presets gets a 2x2 grid. The pages are visually distinct enough that the user always knows which page they are on without checking the dots.

### Screen Architecture

| Page | Layout | Primary Element | Secondary Element | Indicator |
|------|--------|----------------|-------------------|-----------|
| Power | Center-focused | Large power icon + `ON`/`OFF`, 48pt | Connection status label, 14pt | 3 dots, dot 1 amber |
| Brightness | Arc + center | Amber arc (270° sweep), center `%` value, 48pt | "Brightness" label, 16pt | 3 dots, dot 2 amber |
| Presets | 2x2 grid | 4 preset tiles, active highlighted amber | Preset names, 16pt | 3 dots, dot 3 amber |

This is the architecture already implemented in `esphome/door_side_rotary.yaml`. The concept exploration here focuses on how to make this standard layout feel premium rather than generic.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| None | The 3-page lock is the defining constraint of this concept | N/A |

### Interaction Model

Exactly the locked v1 spec:

| Input | Power Page | Brightness Page | Presets Page |
|-------|-----------|----------------|-------------|
| Knob rotate | No action | Adjust brightness ±5% | No action (v1-expanded: cycle presets) |
| Knob press | Toggle power | Return to Power page | Apply highlighted preset |
| Touch tap | Toggle power (center button) | No action | Select preset tile |
| Swipe left | Go to Brightness | Go to Presets | No action (wrap disabled) |
| Swipe right | No action (first page) | Go to Power | Go to Brightness |

### Sleep/Wake Behavior

Wake-only-first enforced. Display wakes to the Power page (page 0) regardless of which page was active when it went to sleep. This is a deliberate choice: the user always starts from a known state.

### LED Ring Behavior

Standard: amber glow when lights on, off when lights off. No page-specific LED behavior.

### Motion/Animation Concept

Page transitions are horizontal slides (200ms, ease-in-out). The 3-dot indicator updates instantly on page change. Brightness arc tracks knob rotation smoothly. Power toggle has a simple opacity transition (on: white to amber, off: amber to dim gray, 200ms). No continuous animations.

### Dark-Room Usability

Excellent. Each page is simple enough to read at minimum backlight. The large primary elements (48pt text, thick arc) are visible at arm's length.

### Daylight Readability

Good. Standard high-contrast layout works in all lighting conditions.

### LVGL Implementation Feasibility

**Easy.** This is already implemented. The concept is the current YAML made visual. All widgets are standard LVGL primitives. The page carousel uses `lvgl.page.next` / `.previous` actions.

### Expected Performance Risk

**Very Low.** Already proven to compile and (presumably) run on the target hardware.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| Roboto font, 3 sizes | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

The premium quality of a standard carousel comes entirely from execution details: consistent spacing, aligned elements, smooth transitions, and the amber accent creating visual warmth. A well-executed standard layout is more premium than a poorly executed creative one. The 3-dot indicator in amber is a small but important detail — it says "we chose this color intentionally."

### What Makes It Unique

Nothing — and that is the point. The Three-Screen Tab Carousel is the safe baseline. It is the concept that every other concept is compared against. Its value is reliability and familiarity, not novelty.

### What Could Go Wrong

The concept risks feeling generic. Without a distinctive visual signature, VelaDial looks like "another round display with pages." The 2x2 preset grid on a round display wastes the corner pixels that the bezel clips. The concept does not exploit the round form factor — it would look the same on a square display.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | Exactly the current implementation — 3 pages, 4 presets, wake-only-first |
| **v1-expanded** | Knob-cycle presets, refined spacing/typography, transition polish |
| **v2** | Additional pages, custom themes, user-configurable page order |


---

## Concept 11 — Brightness-First UI

### Visual Identity

Brightness-First UI is architecturally identical to the Three-Screen Tab Carousel (Concept 10) with one critical difference: the default landing page is Brightness, not Power. The user wakes the device and immediately sees the brightness arc and percentage — the most frequently adjusted value in a bedroom lighting system. Power is demoted to page 2, and Presets remain on page 3.

The visual language is the same as Concept 10 (or whichever concept is chosen as the base). The innovation is in information architecture, not visual design. The hypothesis is that "adjust brightness" is a more common bedroom action than "toggle power" — especially at night, when the user wants to dim the lights rather than turn them off entirely.

### Screen Architecture

| Page Order | Page | Content | Rationale |
|:----------:|------|---------|-----------|
| 0 (default) | Brightness | Arc + `%` value | Most frequent bedroom action |
| 1 | Power | Toggle + state label | Second most frequent |
| 2 | Presets | 2x2 grid or ring | Least frequent (set once, rarely changed) |

The page indicator dots shift accordingly: dot 1 (leftmost) = Brightness, dot 2 = Power, dot 3 = Presets.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Quick Dim | Dedicated page with 3 quick-dim levels (25%, 50%, 75%) as large buttons | v1-expanded |

### Interaction Model

Knob behavior changes to match the new page order:

| Input | Brightness Page (default) | Power Page | Presets Page |
|-------|--------------------------|-----------|-------------|
| Knob rotate | Adjust brightness ±5% | No action | No action |
| Knob press | Toggle power (shortcut) | Toggle power | Apply preset |
| Swipe left | Go to Power | Go to Presets | No action |
| Swipe right | No action (first page) | Go to Brightness | Go to Power |

The key interaction change is that knob press on the Brightness page toggles power rather than "returning to Power page." This makes the most common two actions (adjust brightness, toggle power) accessible without any page navigation. The user can wake the device, rotate the knob to adjust brightness, and press the knob to turn off — all without swiping.

### Sleep/Wake Behavior

Wake-only-first enforced. Display wakes to the Brightness page (page 0). The user's first conscious action after wake is brightness adjustment — the most natural "I just woke up and want to change the light" flow.

### LED Ring Behavior

Same as Concept 10. No page-specific LED behavior.

### Motion/Animation Concept

Same as Concept 10. The page order change does not affect animations.

### Dark-Room Usability

Excellent. Waking to the Brightness page means the user immediately sees the current light level — the most useful information at 3 AM.

### Daylight Readability

Same as Concept 10.

### LVGL Implementation Feasibility

**Very Easy.** This is a one-line change in the LVGL page configuration: swap the page order so Brightness is `page_id: 0`. All widgets, styles, and interactions remain the same.

### Expected Performance Risk

**Very Low.** Identical to Concept 10.

### Required Assets

Same as Concept 10.

### What Makes It Premium

Brightness-First UI is premium because it optimizes for the user's most common need rather than the designer's conceptual hierarchy. Premium products anticipate behavior; this concept anticipates that bedroom users adjust brightness more often than they toggle power.

### What Makes It Unique

Most smart light controllers default to a power toggle or a dashboard. Defaulting to brightness is a subtle but meaningful UX decision that signals "we understand how you actually use this device."

### What Could Go Wrong

The page reorder changes the mental model. Users who expect "page 1 = power" may be confused. The knob press on Brightness page toggling power (instead of returning to Power page) is a non-obvious mapping that needs clear onboarding. If Hardik prefers the Power-first hierarchy, this concept is simply rejected.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | Page reorder only — requires Hardik approval to change default page |
| **v1-expanded** | Quick Dim page, knob press = toggle shortcut on Brightness page |
| **v2** | User-configurable default page, context-aware default (time-of-day) |

---

## Concept 12 — Door Switch Replacement

### Visual Identity

Door Switch Replacement treats VelaDial as a direct replacement for a traditional wall light switch. The UI is optimized for the "walk into room, hit the switch" use case rather than the "sit in bed and adjust" use case. The Power page dominates: a massive touch target fills the entire display, with a clear ON/OFF state. Brightness and Presets are accessible but secondary — the device's primary job is to be the best light switch in the world.

The visual language is bold and immediate. When lights are on, the entire display glows warm amber — not just an icon or a ring, but the full 240x240 circle. When lights are off, the display is dark with a thin white border circle indicating "touch here." The transition between states is a dramatic full-screen fill/drain that is visible from across the room.

### Screen Architecture

| Page | Layout | Visual State: ON | Visual State: OFF |
|------|--------|-----------------|-------------------|
| Power | Full-screen touch target | Entire display amber with white "ON" text, 64pt | Dark display with thin white border circle, dim "OFF" text, 48pt |
| Brightness | Standard arc + value | Same as Concept 10 | Same as Concept 10 |
| Presets | Standard 2x2 grid | Same as Concept 10 | Same as Concept 10 |

The Power page is the hero. The full-screen amber fill when lights are on creates a "the switch itself is a light" effect — the device glows in sympathy with the room lights. This is visible from the doorway, confirming the light state before the user even reaches the switch.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Scene | Quick scene triggers for "Entering" / "Leaving" / "Goodnight" | v2 |

### Interaction Model

The Power page is touch-optimized: any touch anywhere on the display toggles power. No need to find a button — the entire screen is the button. Knob press also toggles. On Brightness and Presets pages, behavior is standard v1 spec.

### Sleep/Wake Behavior

Wake-only-first enforced, but with a modification: the wake timeout on the Power page is shorter (30 seconds instead of 60) because the door-side use case is transient — the user walks in, toggles, and walks away.

### LED Ring Behavior

Aggressive: all 5 LEDs at full amber when lights on, creating a visible "switch is on" indicator from across the room. All off when lights off. Higher brightness than other concepts because the door-side location is not next to a sleeping person.

### Motion/Animation Concept

The hero animation is the full-screen fill on power toggle. When toggling on: amber fills from center outward (radial wipe, 300ms). When toggling off: amber drains from edges inward (reverse radial wipe, 300ms). This is the most dramatic animation in the matrix and is designed to be satisfying every single time.

### Dark-Room Usability

Good for the Power page (large touch target, visible state). Less relevant for Brightness/Presets since the door-side use case is primarily toggle-and-go.

### Daylight Readability

Excellent. The full-screen amber fill is impossible to miss in any lighting condition.

### LVGL Implementation Feasibility

**Easy-Medium.** The full-screen color fill is a simple `lv_obj` with background color animation. The radial wipe effect is harder — it may need to be simplified to a fade or a vertical wipe. The "any touch toggles" behavior is a full-screen `lv_obj` click handler.

### Expected Performance Risk

**Low.** The full-screen fill is a single rectangle redraw. The radial wipe, if implemented, is more expensive but fires only on toggle (not continuously).

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| Roboto font, 2 sizes (48pt, 64pt) | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

The full-screen amber glow transforms the device from a controller into a light fixture. The device itself becomes part of the room's lighting design. This is the kind of detail that makes guests ask "what is that?" — which is the definition of a premium product moment.

### What Makes It Unique

No smart switch uses the entire display as a state indicator. Traditional smart switches have a small LED or a small screen; VelaDial's round display glowing amber is a completely different visual presence on a wall.

### What Could Go Wrong

The full-screen amber fill may be too bright in a dark room, especially at the door-side location where someone might be sleeping nearby. The "any touch toggles" behavior on the Power page conflicts with swipe navigation — the system must distinguish between a tap (toggle) and a swipe (navigate). The Brightness and Presets pages may feel like afterthoughts compared to the dramatic Power page.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | Large power toggle, standard Brightness/Presets pages, wake-only-first |
| **v1-expanded** | Full-screen amber fill, radial wipe animation, shorter sleep timeout |
| **v2** | Scene page, "Leaving" auto-off trigger, motion-sensor wake |

---

## Concept 13 — Lunar Phase Visualization

### Visual Identity

This is the first of the high-novelty (9/10) concepts. Lunar Phase Visualization maps the brightness level to the phase of the moon. At 0% brightness (lights off), the display shows a new moon — a dark circle with a faint limb. At 100% brightness, the display shows a full moon — a luminous white-amber disc filling the round display. Intermediate brightness levels show the corresponding lunar phase: crescent at 25%, half moon at 50%, gibbous at 75%.

The metaphor is poetic: controlling bedroom light is like controlling the moon. The round display is perfectly shaped for this — it IS a moon. The physical form factor and the visual metaphor are one and the same. No rectangular display could achieve this effect.

The visual execution must be painterly rather than photographic. A photorealistic moon image would look like a screensaver; a stylized, slightly abstract lunar surface with subtle crater textures and a soft terminator line (the boundary between light and shadow) would look like art. The amber color palette (warm white for the lit surface, deep charcoal for the shadow) ties the lunar metaphor to the VelaDial brand.

### Screen Architecture

| Page | Visualization | Center Overlay | Knob Behavior |
|------|--------------|----------------|---------------|
| Power | Full moon (on) / new moon (off) | Small power icon, semi-transparent | Press = toggle |
| Brightness | Moon phase proportional to brightness % | Small `%` value, semi-transparent | Rotate = change phase, Press = return |
| Presets | Four small moons at quadrants, each showing preset's default brightness as phase | Active preset moon is brighter | Press = apply |

The Brightness page is the hero. As the user rotates the knob, the moon waxes and wanes in real-time. The terminator line sweeps across the display, revealing or concealing the lunar surface. The center `%` value is overlaid semi-transparently so it does not break the visual metaphor.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Actual Moon | Show the real current lunar phase (calculated from date) as ambient display | v1-expanded |
| Tidal | Abstract visualization of "energy" in the room based on time of day | v2 |

### Interaction Model

Same as locked v1 spec. The knob drives the moon phase on the Brightness page. The visual metaphor does not change the interaction — it only changes the visual feedback.

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, the moon fades in from black (new moon state) and then transitions to the current brightness phase over 500ms. On sleep, the moon wanes to new moon (500ms) then the display goes dark. This creates a poetic "moonrise" and "moonset" wake/sleep cycle.

### LED Ring Behavior

The LED ring shows a soft white-amber glow proportional to the moon phase. At full moon (100%), all 5 LEDs glow. At new moon (0%), all LEDs are off. The ring becomes a "moonlight" source that illuminates the wall around the device.

### Motion/Animation Concept

The primary motion is the terminator sweep — the boundary between the lit and shadowed halves of the moon moving across the display as brightness changes. This is a smooth, continuous animation driven by knob rotation. Secondary motion: subtle "crater" texture parallax as the terminator moves (the surface texture shifts slightly, creating depth). The parallax is a v1-expanded feature; v1 locked uses a static texture.

### Dark-Room Usability

**Exceptional.** The lunar metaphor is specifically designed for dark rooms. A dim crescent moon on a dark display is beautiful, calming, and provides just enough visual information to read the brightness level. This is the most bedroom-appropriate high-novelty concept.

### Daylight Readability

Marginal. The subtle lunar surface textures may wash out in bright light. The semi-transparent `%` overlay provides a fallback readability layer.

### LVGL Implementation Feasibility

**Medium-Hard.** The moon phase visualization requires either pre-rendered image assets (8-12 phase images) or real-time canvas drawing of the terminator arc. Pre-rendered images are easier but consume flash storage (approximately 50-100KB per image at 240x240). Canvas drawing is more flexible but requires custom C++ or complex LVGL canvas operations. The Presets page with four small moons is the hardest element — four simultaneous image renders or four canvas draws.

### Expected Performance Risk

**Medium.** Image-based approach: low risk (image decode is fast). Canvas-based approach: medium risk (custom drawing on every knob tick). The Presets page with four moons is the bottleneck.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| 8-12 moon phase images, 240x240 | PNG with alpha, amber palette | AI-generated or hand-painted |
| 4 small moon phase images, 80x80 | PNG with alpha, for Presets page | Downscaled from above |
| Roboto font, 2 sizes | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

The lunar metaphor elevates a light dimmer into a piece of ambient art. The device on the wall looks like a moon — not a gadget. Guests will notice it, ask about it, and remember it. This is the concept most likely to appear in a design magazine or an Instagram post.

### What Makes It Unique

No smart-home device uses a lunar phase metaphor. The concept is completely original to VelaDial. The round display form factor makes it possible; no competitor with a rectangular display can replicate it.

### What Could Go Wrong

The moon phase images must be high quality. Low-resolution or poorly rendered moons would look cheap rather than premium. The terminator sweep must be smooth — any stuttering destroys the illusion. The concept may feel "too artistic" for users who want a straightforward utility device. The flash storage cost of 8-12 high-quality images may be significant on the ESP32-S3.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | 8-phase moon images for Brightness page, simple power moon, 2x2 preset grid (not moon-based) |
| **v1-expanded** | 12-phase smooth moon, four small moons on Presets page, moonrise/moonset wake/sleep |
| **v2** | Real lunar phase display, parallax texture, tidal ambient page |

---

## Concept 14 — Sundial Shadow UI

### Visual Identity

The Sundial Shadow UI maps brightness to the position and length of a shadow cast by a virtual gnomon (the vertical element of a sundial) at the center of the round display. At 100% brightness, the shadow is short and the display is bright — high noon. At 0% brightness, the shadow is long and the display is dark — sunset/twilight. The round display becomes a sundial face, and the brightness knob controls the "time of day."

The visual palette shifts with the shadow: high brightness uses warm white and amber (midday sun), medium brightness uses golden amber and soft orange (golden hour), low brightness uses deep amber and charcoal (twilight). The shadow itself is a soft gradient, not a hard edge, creating a naturalistic light-and-shadow effect.

### Screen Architecture

| Page | Visualization | Center Element | Knob Behavior |
|------|--------------|----------------|---------------|
| Power | Sundial at noon (on) / sundial at night (off) | Small gnomon icon | Press = toggle |
| Brightness | Shadow length proportional to brightness (inverted: bright = short shadow) | `%` value overlaid on sundial face | Rotate = move shadow, Press = return |
| Presets | Four sundial quadrants, each showing preset's color temperature as shadow color | Active quadrant highlighted | Press = apply |

### Interaction Model

Same as locked v1 spec. The knob rotation on the Brightness page moves the shadow around the sundial face. Clockwise rotation = shorter shadow (brighter). Counter-clockwise = longer shadow (dimmer). The shadow direction could optionally track the actual time of day (v2 feature).

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, the sundial "sunrise" animation plays: shadow sweeps from long (edge of display) to the current brightness position over 500ms. On sleep, "sunset": shadow extends to fill the display, then fades to black.

### LED Ring Behavior

The LED ring color matches the sundial's current "time of day" palette: warm amber at high brightness, deep gold at medium, very dim amber at low. This creates a "golden hour" ambient effect on the wall.

### Motion/Animation Concept

The shadow sweep is the primary motion — smooth, continuous tracking of knob rotation. The shadow is a soft gradient that rotates around the center gnomon. The color palette shifts smoothly as the shadow moves. The sunrise/sunset wake/sleep animations are the secondary motions.

### Dark-Room Usability

Good. The twilight/sunset palette at low brightness is warm and non-stimulating. The shadow metaphor naturally produces dark visuals at low brightness levels.

### Daylight Readability

Good. The bright "noon" state at high brightness has good contrast. The `%` overlay provides fallback readability.

### LVGL Implementation Feasibility

**Medium-Hard.** The shadow gradient requires either pre-rendered images (similar to Concept 13) or custom canvas drawing. The color palette shift adds complexity — each brightness level needs a different background color and shadow color. The Presets page with four sundial quadrants is complex.

### Expected Performance Risk

**Medium.** Similar to Concept 13 — image-based is lower risk, canvas-based is higher risk.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| 8-12 sundial state images, 240x240 | PNG, warm palette | AI-generated |
| Gnomon icon | Simple line art | SVG or text-based |
| Roboto font, 2 sizes | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

The sundial metaphor connects electric light to natural light — a philosophical statement embedded in a UI. The color palette shift from noon to twilight creates a rich, warm visual experience that changes with every brightness adjustment. The device feels like a window into a miniature world.

### What Makes It Unique

No smart-home device uses a sundial metaphor. The concept is deeply tied to the round form factor and the idea of "controlling light." It is the most conceptually coherent metaphor in the matrix.

### What Could Go Wrong

The shadow direction and length must be intuitively mapped to brightness. If the mapping feels arbitrary, the metaphor breaks. The color palette shift must be smooth — any banding or stepping is visible. The concept may be too abstract for users who just want to see "75%."

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | 8-state sundial images for Brightness, simple power state, standard presets |
| **v1-expanded** | 12-state smooth sundial, color palette shift, sunrise/sunset wake/sleep |
| **v2** | Real time-of-day shadow tracking, seasonal shadow angle, ambient page |

---

## Concept 15 — Tree Ring Growth Pattern

### Visual Identity

Tree Ring Growth Pattern maps brightness to the number of visible concentric rings on the display, like the cross-section of a tree trunk. At 0% brightness, the display shows a single dark heartwood circle at the center. At 100%, the display is filled with concentric rings extending to the bezel — a mature tree with many growth rings. Each ring is a slightly different shade of amber/brown, creating an organic, warm texture.

The metaphor connects light to growth, warmth, and natural cycles. The round display is the perfect canvas — it IS a tree cross-section. The concentric rings exploit the circular geometry in a way that no rectangular display could achieve.

### Screen Architecture

| Page | Visualization | Center Element | Knob Behavior |
|------|--------------|----------------|---------------|
| Power | Full rings (on) / single heartwood (off) | Small power icon at center | Press = toggle |
| Brightness | Ring count proportional to brightness | `%` value at center, overlaid on heartwood | Rotate = add/remove rings, Press = return |
| Presets | Four quadrants of the tree cross-section, each with different ring density/color | Active quadrant highlighted | Press = apply |

### Interaction Model

Same as locked v1 spec. Knob rotation on the Brightness page adds or removes rings from the outside in. Clockwise = add rings (brighter). Counter-clockwise = remove rings (dimmer). Each encoder detent adds/removes approximately one ring, creating a satisfying "growth" feel.

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, rings grow outward from the heartwood to the current brightness level (500ms). On sleep, rings retract inward to the heartwood (500ms), then fade to black.

### LED Ring Behavior

The LED ring glows warm amber, with brightness proportional to the number of visible rings. At full rings (100%), all 5 LEDs at moderate amber. At heartwood only (0%), all LEDs off.

### Motion/Animation Concept

The ring growth/retraction is the primary motion. Each ring appears with a subtle "pop" — expanding from zero width to full width over approximately 50ms. The cumulative effect of multiple rings growing in sequence creates a satisfying organic animation. The wake animation plays the full growth sequence; the sleep animation plays the full retraction.

### Dark-Room Usability

Good. The warm amber/brown palette is non-stimulating. The heartwood at low brightness is a small, dim circle — minimal light pollution.

### Daylight Readability

Good. The concentric rings have clear contrast against each other. The `%` overlay provides fallback readability.

### LVGL Implementation Feasibility

**Medium.** The concentric rings can be implemented as multiple `lv_arc` widgets with increasing radii and full 360° sweep, each with a slightly different color. Alternatively, pre-rendered images for 8-12 ring states. The ring growth animation requires timed sequential arc appearance. The Presets page with four quadrants is complex.

### Expected Performance Risk

**Medium.** Multiple simultaneous full-circle arcs (up to 10-12 for fine granularity) create a heavier rendering load than a single arc. Pre-rendered images reduce this risk.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| 8-12 tree ring state images, 240x240 | PNG, amber/brown palette | AI-generated |
| Roboto font, 2 sizes | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

The organic, natural aesthetic sets VelaDial apart from every other smart-home device on the market. The tree ring metaphor communicates warmth, growth, and natural cycles — values that align with a premium bedroom product. The texture and color variation in the rings create visual richness that simple geometric UIs cannot match.

### What Makes It Unique

No electronic device uses tree rings as a UI metaphor. The concept is completely original and deeply tied to the round form factor. It transforms a light controller into a piece of nature-inspired art.

### What Could Go Wrong

The tree ring texture must be high quality — poorly rendered rings look like a bullseye target rather than wood. The ring growth animation must be smooth — any stuttering breaks the organic feel. The concept may feel too "decorative" for users who want a utilitarian device. The amber/brown palette may not provide enough contrast for the `%` overlay.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | 8-state tree ring images for Brightness, simple power state, standard presets |
| **v1-expanded** | 12-state smooth rings, growth/retraction animation, quadrant presets |
| **v2** | Seasonal color variation (spring green, autumn gold), real growth tracking over time |


---

## Concept 16 — Topographic Contour Map

### Visual Identity

Topographic Contour Map renders the round display as a terrain elevation map, where brightness level corresponds to elevation. At 0% brightness, the display shows a flat plain — a single contour line at the center. At 100%, the display shows a mountain peak — dense concentric contour lines filling the entire circle, with the "summit" at the center. The contour lines are thin, precise, and closely spaced at high brightness (steep terrain) and widely spaced at low brightness (gentle slopes).

The color palette is cartographic: amber contour lines on a dark charcoal background, with subtle elevation shading between contours. The aesthetic reference is a USGS topographic map or a hiking trail map — technical, beautiful, and information-dense in a way that rewards close inspection. The round display becomes a "window" looking down at a terrain from above.

### Screen Architecture

| Page | Visualization | Center Element | Knob Behavior |
|------|--------------|----------------|---------------|
| Power | Peak (on) / flat plain (off) | Small summit marker icon | Press = toggle |
| Brightness | Contour density proportional to brightness | `%` value at summit, small text | Rotate = raise/lower terrain, Press = return |
| Presets | Four terrain "regions" with different contour colors per preset | Active region highlighted | Press = apply |

The Brightness page is the hero. As the user rotates the knob clockwise, new contour lines appear from the center outward, as if the terrain is rising. Counter-clockwise rotation removes contour lines from the outside in, as if the terrain is sinking. The effect is hypnotic — watching contour lines emerge and disappear is visually compelling.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Terrain Map | Full topographic map with color-coded elevation zones | v1-expanded |
| Weather | Contour lines represent barometric pressure (SHT45 data) | v2 |

### Interaction Model

Same as locked v1 spec. The knob drives contour density on the Brightness page. The mapping is intuitive: more contours = higher elevation = more brightness. The physical rotation of the knob feels like "raising the mountain."

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, contour lines emerge from center outward to the current brightness level (500ms). On sleep, contour lines retract inward (500ms), then fade to black. The animation resembles a terrain "growing" from flat ground.

### LED Ring Behavior

The LED ring shows elevation-mapped color: warm amber at high brightness (sunlit peak), dim amber at medium (treeline), very dim at low (valley). All off when lights off.

### Motion/Animation Concept

The contour emergence/retraction is the primary motion. Each contour line appears as a thin ring expanding from center, then settling at its final radius. Multiple contours emerging in rapid sequence create a "ripple" effect. The wake animation plays the full emergence; the sleep animation plays the full retraction.

### Dark-Room Usability

Good. The thin contour lines on dark background produce minimal light. At low brightness, only a few widely-spaced lines are visible — very subtle.

### Daylight Readability

Good. The contour lines have clear contrast against the dark background. The density variation is readable in any lighting.

### LVGL Implementation Feasibility

**Medium.** Contour lines can be implemented as multiple `lv_arc` widgets with full 360° sweep at different radii, or as pre-rendered images. The emergence animation requires timed sequential arc appearance. Pre-rendered images (8-12 states) are the practical approach for v1.

### Expected Performance Risk

**Medium.** Similar to Concept 15. Multiple simultaneous arcs or image decoding. Pre-rendered images reduce risk.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| 8-12 contour state images, 240x240 | PNG, amber/charcoal palette | AI-generated or procedurally generated |
| Roboto font, 2 sizes | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

The cartographic aesthetic is sophisticated and unusual in consumer electronics. Topographic maps are associated with exploration, precision, and the outdoors — values that elevate a bedroom gadget into something with character. The dense contour lines at high brightness create visual complexity that rewards close inspection.

### What Makes It Unique

No electronic device uses topographic contours as a UI metaphor. The concept is original and deeply tied to the round form factor (contour maps are inherently circular when viewed from above). It transforms brightness control into terrain exploration.

### What Could Go Wrong

The contour lines must be precisely rendered — uneven spacing or inconsistent line weight would look like a rendering error rather than a design choice. The metaphor (brightness = elevation) is less intuitive than some others (brightness = moon phase, brightness = shadow length). Users may need a moment to understand the mapping.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | 8-state contour images for Brightness, simple power state, standard presets |
| **v1-expanded** | 12-state smooth contours, emergence animation, color-coded elevation zones |
| **v2** | Weather contour page, real terrain data, 3D perspective view |

---

## Concept 17 — Iris Aperture Mechanism

### Visual Identity

The Iris Aperture Mechanism maps brightness to the opening of a camera iris (diaphragm). At 0% brightness, the iris is fully closed — overlapping blades form a tight polygon at the center, blocking all "light." At 100%, the iris is fully open — the blades retract to the edges, revealing a bright amber field behind them. The round display is the perfect canvas for this metaphor: the iris aperture is inherently circular, and the display bezel becomes the lens housing.

The visual execution uses metallic gray iris blades against a warm amber "light" behind them. The blades have subtle texture (brushed metal effect) and cast soft shadows on each other, creating depth. The aperture opening is a polygon (typically 6-8 sides) rather than a perfect circle, which is mechanically accurate and visually distinctive — the polygon shape is the signature of real camera lenses.

### Screen Architecture

| Page | Visualization | Center Element | Knob Behavior |
|------|--------------|----------------|---------------|
| Power | Open iris (on) / closed iris (off) | Power icon visible through aperture | Press = toggle |
| Brightness | Iris opening proportional to brightness | `%` value visible through aperture | Rotate = open/close iris, Press = return |
| Presets | Four small iris apertures at quadrants, each at preset's default brightness | Active iris highlighted | Press = apply |

The Brightness page is the hero. As the user rotates the knob clockwise, the iris blades retract and the amber light behind them is revealed. Counter-clockwise rotation closes the iris. The physical rotation of the knob maps directly to the mechanical opening of the iris — turning the knob feels like turning a camera lens ring.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Focus | Iris + a "depth of field" blur effect on the center content | v2 |
| Exposure | Full camera exposure triangle visualization (aperture + shutter + ISO) | v2 |

### Interaction Model

Same as locked v1 spec. The knob drives the iris opening on the Brightness page. The mapping is deeply intuitive for anyone who has used a camera: open = more light = brighter. The physical knob rotation mimics the mechanical action of a real aperture ring.

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, the iris opens from closed to the current brightness level (400ms). On sleep, the iris closes (400ms), then the display goes dark. The closing animation is particularly satisfying — the blades converge to a point, like a camera shutter closing.

### LED Ring Behavior

The LED ring brightness maps to the iris opening: fully open = all 5 LEDs at moderate amber, fully closed = all LEDs off. The ring becomes a "light leak" around the lens housing.

### Motion/Animation Concept

The iris open/close is the primary motion. The blades move simultaneously, converging toward or diverging from the center. The motion is smooth and mechanical — like a real iris mechanism. The blade overlap creates subtle shadow changes during the animation. This is the most mechanically satisfying animation in the matrix.

### Dark-Room Usability

Good. A nearly-closed iris shows only a tiny polygon of amber light at the center — minimal light pollution. The metallic blade texture is visible at low backlight, providing visual interest without brightness.

### Daylight Readability

Good. The high contrast between metallic blades and amber light field ensures readability. The `%` value is visible through the aperture opening.

### LVGL Implementation Feasibility

**Medium-Hard.** The iris blades require either pre-rendered images (8-12 aperture states) or custom canvas drawing of polygons. Pre-rendered images are the practical approach — each image shows the iris at a different opening. The blade animation between states requires smooth image crossfading or a sequence of images. Custom canvas drawing of 6-8 moving polygons with shadows is possible but complex.

### Expected Performance Risk

**Medium.** Image-based: low-medium risk (image crossfade between states). Canvas-based: high risk (real-time polygon rendering with shadows). Pre-rendered images are strongly recommended.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| 8-12 iris aperture state images, 240x240 | PNG with alpha, metallic/amber palette | AI-generated or 3D-rendered |
| 4 small iris images, 80x80 | PNG, for Presets page | Downscaled from above |
| Roboto font, 2 sizes | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

The camera iris metaphor is associated with precision optics, professional photography, and high-end equipment. The mechanical animation — blades converging and diverging — creates a tactile, satisfying interaction that feels like using a luxury instrument. The metallic texture and shadow depth add visual richness.

### What Makes It Unique

The iris aperture metaphor for brightness control is a perfect conceptual match (aperture literally controls light) and a perfect form-factor match (iris is circular, display is circular). No smart-home device uses this metaphor. It is the most mechanically coherent concept in the matrix.

### What Could Go Wrong

The iris blade images must be high quality — the metallic texture and shadow depth are critical to the premium feel. Low-quality renders would look like a cheap camera app filter. The polygon aperture shape (hexagonal or octagonal) must be consistent across all states. The animation between states must be smooth — any jumping or flickering breaks the mechanical illusion.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | 8-state iris images for Brightness, simple power iris (open/closed), standard presets |
| **v1-expanded** | 12-state smooth iris, blade animation, four small iris presets |
| **v2** | Focus/blur effect, exposure triangle, real-time blade rendering |

---

## Concept 18 — Radar Sweep Animation

### Visual Identity

Radar Sweep Animation turns the round display into a radar scope. A bright amber sweep line rotates around the center like a radar beam, and "blips" appear on the scope to represent system state. The sweep line's rotation speed maps to brightness: at 100%, the sweep rotates quickly (one revolution per 2 seconds); at low brightness, it rotates slowly (one revolution per 10 seconds); at 0% (lights off), the sweep stops and the scope goes dark.

The visual palette is classic radar green-on-black, adapted to VelaDial's amber palette: amber sweep line, amber blips, dark charcoal background with faint concentric range rings. The aesthetic reference is a military radar display or an air traffic control scope — technical, atmospheric, and hypnotic.

### Screen Architecture

| Page | Visualization | Blips/Elements | Knob Behavior |
|------|--------------|----------------|---------------|
| Power | Sweep active (on) / sweep stopped (off) | Single large blip at center = power state | Press = toggle |
| Brightness | Sweep speed proportional to brightness | Center value `%`, faint range rings | Rotate = change sweep speed, Press = return |
| Presets | Four blips at quadrants, each representing a preset | Active blip pulses brighter | Press = apply |

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Sensor Scope | Blips represent sensor readings (lux, temp, humidity) at different positions | v2 |
| Network | Blips represent connected HA devices | v2 |

### Interaction Model

Same as locked v1 spec. The knob drives sweep speed on the Brightness page. The mapping is less intuitive than some concepts (sweep speed ≠ brightness in any natural sense), but the visual effect is compelling enough to justify the abstraction.

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, the sweep line appears and begins rotating from the 12 o'clock position. On sleep, the sweep line completes its current revolution, then fades to a stop.

### LED Ring Behavior

The LED ring could show a "rotating" pattern that follows the sweep line: one LED at a time lights up in sequence, creating a physical radar sweep around the device. This is a v1-expanded feature — v1 locked uses standard on/off behavior.

### Motion/Animation Concept

The radar sweep is a **continuous animation** — the sweep line rotates constantly while the display is awake. This is the only concept in the matrix with continuous motion. The sweep line leaves a fading trail (phosphor decay effect) that creates the classic radar "afterglow." Blips pulse when the sweep line passes over them.

**Performance warning:** Continuous animation is the most expensive rendering pattern on ESP32-S3 + LVGL. The sweep line redraw on every frame (approximately 30 FPS) is a significant load. This concept requires careful optimization or frame rate reduction.

### Dark-Room Usability

Good for atmosphere, marginal for information. The rotating sweep is mesmerizing but may be distracting in a dark bedroom. The continuous motion conflicts with the "calm, quiet bedroom device" philosophy. The sweep brightness must be very low in Night Mode to avoid being a disturbance.

### Daylight Readability

Marginal. The faint sweep line and afterglow may be hard to see in bright light. The range rings and blips provide structural readability.

### LVGL Implementation Feasibility

**Hard.** The rotating sweep line requires either a custom canvas draw (line at angle, redrawn every frame) or a rotating image. The phosphor decay trail requires either multiple fading line segments or a custom shader-like effect. The continuous animation at 30 FPS is the hardest rendering requirement in the matrix. LVGL's `lv_canvas` can draw lines, but doing so 30 times per second with fading trails is pushing the limits.

### Expected Performance Risk

**High.** Continuous animation is the most demanding pattern. The ESP32-S3 may not maintain 30 FPS with a full sweep + trail + blips. Frame rate reduction to 15 FPS or 10 FPS may be necessary, which makes the sweep look choppy. This is the highest-risk concept for performance.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| Range ring background image, 240x240 | PNG, faint amber circles on black | Procedurally generated |
| Roboto font, 2 sizes | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

The radar scope aesthetic is deeply atmospheric. It transforms a bedroom device into something that feels like it belongs in a submarine control room or an air traffic control tower. The continuous sweep creates a sense of the device being "alive" and "watching" — which is either premium or creepy, depending on the user.

### What Makes It Unique

No smart-home device uses a radar metaphor. The continuous sweep animation is visually distinctive and immediately recognizable. The round display is the perfect form factor for a radar scope.

### What Could Go Wrong

The continuous animation may be too distracting for a bedroom device. The sweep speed mapping to brightness is unintuitive. The performance risk is the highest in the matrix. The "military/surveillance" aesthetic may feel inappropriate for a bedroom. The phosphor decay trail is hard to implement well — a bad implementation looks like a rendering bug rather than a design choice.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | NOT RECOMMENDED for v1 — continuous animation risk too high, bedroom-inappropriate |
| **v1-expanded** | Simplified sweep (no trail, lower frame rate) as an optional "fun mode" |
| **v2** | Full radar scope with trail, sensor blips, network visualization |

---

## Concept 19 — Vinyl DJ Crossfader

### Visual Identity

The Vinyl DJ Crossfader maps the round display to a vinyl record. The display shows concentric grooves like a vinyl LP, with a "tonearm" indicator showing the current position. Brightness maps to the playback position: at 0%, the tonearm is at the outer edge (start of record); at 100%, the tonearm is at the center (end of record). The knob rotation mimics "scratching" the record — rotating the knob moves the tonearm across the grooves.

The visual palette is warm and retro: dark charcoal grooves with subtle reflective highlights (simulating the light-catching property of vinyl), an amber tonearm indicator, and a warm amber label at the center (like a vinyl record's center label). The aesthetic reference is a high-end turntable — Technics SL-1200, Rega Planar, or Pro-Ject Debut.

### Screen Architecture

| Page | Visualization | Center Element | Knob Behavior |
|------|--------------|----------------|---------------|
| Power | Record spinning (on) / record stopped (off) | Center label with power icon | Press = toggle |
| Brightness | Tonearm position proportional to brightness | `%` value on center label | Rotate = move tonearm, Press = return |
| Presets | Four "tracks" marked on the record surface, tonearm at active track | Track/preset name on center label | Press = apply (drop needle) |

The Presets page reimagines presets as "tracks" on a vinyl record. Each preset is a marked position on the record surface (like track markers on a vinyl label). The knob rotates the tonearm to the desired track, and pressing the knob "drops the needle" (applies the preset). This is a deeply satisfying metaphor for anyone who has used a turntable.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Playlist | Multiple "records" (scenes) that can be swapped | v2 |
| Mixer | Crossfader between two presets (blend between color temperatures) | v2 |

### Interaction Model

Same as locked v1 spec, with the turntable metaphor layered on top. Knob rotation on the Brightness page moves the tonearm. The physical rotation of the knob mimics the physical rotation of a turntable platter — the mapping is natural and satisfying. On the Presets page, knob rotation moves the tonearm between track markers.

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, the record "starts spinning" — a subtle rotation animation begins (the groove highlights shift slightly, creating the illusion of rotation). On sleep, the record "stops spinning" — the rotation slows and stops, then the display fades to black.

### LED Ring Behavior

The LED ring could show a "spinning" pattern — LEDs light up in sequence to simulate rotation, synchronized with the on-screen record spin. This is a v1-expanded feature. V1 locked uses standard on/off behavior.

### Motion/Animation Concept

The record spin is a subtle continuous animation — the groove highlights shift by 1-2 pixels per frame, creating the illusion of rotation without actually rotating the entire image. This is much cheaper than the Radar Sweep's full line rotation. The tonearm movement is smooth and deliberate, like a real tonearm tracking across a record. The "needle drop" on preset apply is a quick downward motion of the tonearm indicator (100ms).

### Dark-Room Usability

Good. The dark vinyl surface with subtle groove highlights is non-intrusive. The amber tonearm and center label provide minimal but sufficient information.

### Daylight Readability

Good. The groove texture and tonearm have sufficient contrast. The center label provides clear text readability.

### LVGL Implementation Feasibility

**Medium.** The vinyl grooves can be a pre-rendered background image. The tonearm is an `lv_line` or `lv_img` widget that rotates around the center. The groove highlight animation (simulating spin) requires either a rotating image or a subtle pixel-shift effect. Pre-rendered images for different tonearm positions (8-12 states) are the practical approach.

### Expected Performance Risk

**Low-Medium.** The spin animation is subtle (1-2 pixel shift) and cheap. The tonearm position change is a simple widget move. Pre-rendered images keep the rendering cost low.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| Vinyl record background, 240x240 | PNG, dark charcoal with groove texture | AI-generated |
| Tonearm overlay, various positions | PNG with alpha, or rotatable widget | AI-generated or drawn |
| Center label design | PNG or LVGL widgets | Designed |
| Roboto font, 2 sizes | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

The turntable metaphor is associated with audiophile culture, vinyl collecting, and analog warmth — all premium associations. The tactile connection between the physical knob and the virtual tonearm creates the same "precision instrument" feel as Concept 2 (SmartKnob Arc), but with more personality. The vinyl aesthetic is warm, nostalgic, and distinctive.

### What Makes It Unique

No smart-home device uses a turntable metaphor. The concept is playful but sophisticated — it does not take itself too seriously, which is refreshing in a market full of sterile, corporate smart-home UIs. The "drop the needle" preset application is a delightful interaction detail.

### What Could Go Wrong

The vinyl/turntable metaphor may feel forced — the connection between "playing a record" and "controlling bedroom lights" is tenuous. Users who do not have a relationship with vinyl records may not understand the metaphor. The groove texture must be high quality — a flat gray circle with lines does not read as "vinyl." The spin animation must be subtle — too fast and it is distracting, too slow and it is invisible.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | Vinyl background image, tonearm position for brightness, standard presets |
| **v1-expanded** | Spin animation, track markers for presets, needle drop animation, LED spin |
| **v2** | Playlist page, mixer/crossfader, multiple record designs |

---

## Concept 20 — Eclipse Corona

### Visual Identity

Eclipse Corona is the most visually dramatic concept in the matrix. The round display shows a solar eclipse: a dark disc (the moon) at the center, surrounded by a luminous corona (the sun's atmosphere) that extends to the edges of the display. Brightness maps to the corona's intensity: at 100%, the corona is a blazing amber-white halo with visible streamers and prominences. At 0%, the corona is extinguished and only the dark disc remains — a total eclipse in darkness.

The visual execution must be breathtaking. The corona is not a simple gradient — it has structure: radial streamers of varying length and brightness, a bright inner corona close to the disc edge, and a fainter outer corona extending to the bezel. The color palette transitions from white-hot at the inner edge through amber to deep red at the outer edge. The dark central disc has a subtle blue-black color (like the actual lunar disc during an eclipse).

### Screen Architecture

| Page | Visualization | Center Element | Knob Behavior |
|------|--------------|----------------|---------------|
| Power | Bright corona (on) / dark disc only (off) | Power icon on dark disc | Press = toggle |
| Brightness | Corona intensity proportional to brightness | `%` value on dark disc | Rotate = intensify/diminish corona, Press = return |
| Presets | Four corona "flares" at quadrants, each with preset's color temperature | Active flare brightest | Press = apply |

The Brightness page is the hero. As the user rotates the knob clockwise, the corona intensifies — streamers lengthen, the inner corona brightens, and the outer corona extends further toward the bezel. Counter-clockwise rotation diminishes the corona. The effect is like watching a solar eclipse transition from totality to partial — the sun's atmosphere gradually reveals itself.

**Potential expanded pages (require approval):**

| Page | Purpose | Classification |
|------|---------|---------------|
| Solar Cycle | Corona activity varies with a long-term cycle (decorative) | v2 |
| Prominence | Individual corona streamers represent different data sources | v2 |

### Interaction Model

Same as locked v1 spec. The knob drives corona intensity on the Brightness page. The mapping is intuitive: more corona = more light = brighter. The physical rotation of the knob feels like "controlling the sun."

### Sleep/Wake Behavior

Wake-only-first enforced. On wake, the corona ignites from the dark disc outward (500ms) — streamers extend, inner corona brightens, outer corona fades in. On sleep, the corona extinguishes inward (500ms) — streamers retract, corona dims, leaving only the dark disc, then black. This is the most dramatic wake/sleep animation in the matrix.

### LED Ring Behavior

The LED ring becomes the physical corona. All 5 LEDs glow in a warm amber-white that matches the on-screen corona intensity. At full brightness, the LEDs are at their brightest warm white. At low brightness, they glow faintly. At lights off, they are dark. The LED ring extending the corona effect onto the physical wall creates a stunning halo effect around the device.

### Motion/Animation Concept

The corona has subtle continuous motion — streamers slowly undulate (1-2 pixel shift per second), creating a "living" corona effect. This is much subtler than the Radar Sweep's rotation and costs significantly less in rendering. The intensity change on knob rotation is smooth and continuous. The wake/sleep ignition/extinction is the hero animation.

**Performance note:** The streamer undulation can be achieved with a slow image offset or a two-frame alternation, keeping the rendering cost very low despite being technically "continuous."

### Dark-Room Usability

**Exceptional at low brightness.** A faint corona around a dark disc is one of the most beautiful things in nature, and it translates perfectly to a dark bedroom. The warm amber glow is calming and non-stimulating. At high brightness, the corona may be too bright for a dark room — the adaptive backlight must reduce intensity aggressively in Night Mode.

### Daylight Readability

Marginal. The corona's subtle gradients and streamers may wash out in bright light. The `%` value on the dark disc provides fallback readability. The concept is optimized for dim/dark environments.

### LVGL Implementation Feasibility

**Medium-Hard.** The corona requires high-quality pre-rendered images (8-12 intensity states). Each image must include the structured corona with streamers — this cannot be generated with simple LVGL gradients. The streamer undulation requires either two alternating images or a subtle image offset animation. The Presets page with four corona flares is complex. Flash storage for 8-12 high-quality 240x240 images is approximately 100-200KB total (compressed PNG).

### Expected Performance Risk

**Medium.** Image-based approach with slow alternation: low risk. The streamer undulation at 1 FPS is negligible. The wake/sleep ignition animation (crossfade between images) is a brief burst of rendering. The Presets page with four flares is the bottleneck.

### Required Assets

| Asset | Type | Source |
|-------|------|--------|
| 8-12 corona intensity images, 240x240 | PNG, amber/white/red palette with structured streamers | AI-generated, high quality critical |
| 2 alternating corona frames for undulation | PNG, subtle variation | AI-generated |
| 4 small corona flare images, 80x80 | PNG, for Presets page | AI-generated |
| Roboto font, 2 sizes | TTF/OTF to LVGL font | Google Fonts |

### What Makes It Premium

Eclipse Corona is the most visually stunning concept in the matrix. A solar eclipse is one of nature's most awe-inspiring phenomena, and rendering it on a round display creates a genuinely beautiful object. The device on the wall looks like a captured eclipse — art, not gadgetry. The corona extending onto the physical wall via the LED ring creates an immersive effect that no other smart-home device can match.

### What Makes It Unique

No electronic device uses a solar eclipse as a UI metaphor. The concept is completely original, visually unforgettable, and perfectly suited to the round form factor. It is the concept most likely to make VelaDial iconic — the device people photograph and share.

### What Could Go Wrong

The corona images must be exceptional quality. A poorly rendered corona looks like a blurry gradient ring — the streamers, prominences, and color transitions are what make it look real. The concept is optimized for dark rooms and may feel underwhelming in daylight. The flash storage cost is the highest in the matrix. The concept may feel "over the top" for users who want a simple light switch.

### v1/v1-Expanded/v2 Classification

| Scope | What's Included |
|-------|----------------|
| **v1 locked** | 8-state corona images for Brightness, simple power corona (on/off), standard presets |
| **v1-expanded** | 12-state smooth corona, streamer undulation, four flare presets, LED ring corona echo |
| **v2** | Solar cycle variation, prominence data visualization, real-time generative corona |

---

## Summary Verdict Table

| # | Concept | v1 Verdict | Key Strength | Key Risk |
|---:|---------|-----------|-------------|----------|
| 1 | Minimal Thermostat | **v1 candidate** | Safest, most proven pattern | May feel generic |
| 2 | SmartKnob-Inspired Arc | **v1 candidate** | Best knob-to-screen mapping | Arc smoothness critical |
| 3 | Large Center Power Button | **v1 candidate** | Most intuitive power toggle | Other pages feel secondary |
| 4 | Single-Page Simple Mode | **Reject (breaks 3-page lock)** | Maximum simplicity | Violates locked scope |
| 5 | Preset Ring UI | **v1 candidate with adaptation** | Best round-display preset layout | Four arcs may be complex |
| 6 | Night Mode Ultra-Minimal | **v1 candidate (layer)** | Essential bedroom feature | Not a standalone concept |
| 7 | Text-First Utility | **v1 candidate** | Maximum readability | May feel "cheap" if poorly typeset |
| 8 | Apple Watch Complications | **Reject (breaks scope)** | Information density | Too complex for 240x240 |
| 9 | LED-Ring Status-First | **v1 candidate with adaptation (layer)** | Ambient room awareness | 5 LEDs is limited canvas |
| 10 | Three-Screen Tab Carousel | **v1 candidate (baseline)** | Already implemented | Not distinctive |
| 11 | Brightness-First UI | **v1 candidate with adaptation** | Optimizes for common action | Changes mental model |
| 12 | Door Switch Replacement | **v1 candidate with adaptation** | Best power toggle experience | Other pages feel secondary |
| 13 | Lunar Phase | **v1 candidate with adaptation** | Most poetic metaphor | Image quality critical |
| 14 | Sundial Shadow | **v1 candidate with adaptation** | Most conceptually coherent | Shadow mapping unintuitive |
| 15 | Tree Ring Growth | **v1 candidate with adaptation** | Most organic/natural | Texture quality critical |
| 16 | Topographic Contour | **v1 candidate with adaptation** | Most intellectually interesting | Metaphor less intuitive |
| 17 | Iris Aperture | **v1 candidate with adaptation** | Best mechanical metaphor | Image quality critical |
| 18 | Radar Sweep | **Reject for v1 (performance)** | Most atmospheric | Continuous animation too expensive |
| 19 | Vinyl DJ Crossfader | **v1 candidate with adaptation** | Most playful/personality | Metaphor may feel forced |
| 20 | Eclipse Corona | **v1 candidate with adaptation** | Most visually stunning | Image quality critical, flash cost |

