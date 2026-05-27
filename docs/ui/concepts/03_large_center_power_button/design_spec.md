# Concept 03: Large Center Power Button — Design Specification

## Visual Identity

The Large Center Power Button concept places a single, oversized circular button at the absolute center of the 240x240 round display. The button is approximately 140px in diameter, dominating the visual field and creating a concentric-circle harmony with the physical bezel. When the light is on, the button glows amber with a soft shadow halo that fades into the pure black background, creating a "lamp" effect where the button itself appears to be a light source. When the light is off, the button becomes a dark circle with a thin white border — a ghost outline barely visible in darkness, waiting to be activated.

The design metaphor is the most primal one in lighting: a light switch. But instead of a cheap plastic rocker, this is a luminous disc floating in darkness. The round display and the round button create concentric circles that feel harmonious and intentional. The bezel is the outer ring, the button is the inner ring, and the glow bridges them.

This concept is distinct from Concept 01 (Minimal Thermostat) in that it centers the UI around a **touch-first power button** rather than a knob-first arc. The button is the hero element — not the brightness value, not the arc. The power state is the primary information communicated.

## Screen/Page Architecture

This concept uses the locked **3-page architecture** (Power, Brightness, Presets) with horizontal swipe navigation and a 3-dot page indicator at the bottom.

| Page | Center Content | Peripheral Content | Knob Behavior | Touch Behavior |
|------|---------------|-------------------|---------------|----------------|
| Power | 140px glowing circle with "ON"/"OFF" text inside | Small state label below button, 3-dot indicator at bottom | Press = toggle power | Tap button = toggle power |
| Brightness | 100px circle showing percentage value, surrounded by thin arc | Thin arc ring near display edge | Rotate = adjust brightness, Press = return to Power | Swipe to navigate |
| Presets | 4 smaller circles (55px each) in diamond arrangement | Active preset circle glows amber, others dim | Rotate = highlight, Press = apply highlighted | Tap circle to select preset |

The design language is **circles within circles**. Every interactive element is circular, reinforcing the round display form factor. The Presets page uses a diamond arrangement (top, left, right, bottom) rather than a 2x2 grid to better fit the circular viewport — the corners of a 2x2 grid get clipped by the round bezel, but a diamond arrangement keeps all four elements within the safe circular area (approximately 200px diameter inscribed circle).

## Interaction Model

### Power Page (Page 1)

The Power page is touch-first. The large 140px button is an irresistible touch target — users will instinctively tap it. At 140px diameter on a 240px display, the button occupies approximately 58% of the display width, making it nearly impossible to miss even in complete darkness. The knob press serves as a redundant toggle for users who prefer physical input.

When on, the button displays "ON" in white text centered within the amber-glowing circle. When off, the button displays "OFF" in dim gray text within the dark ghost circle.

### Brightness Page (Page 2)

On the Brightness page, the knob takes over as primary input. The center circle shrinks to 100px to make room for a surrounding thin arc that indicates the current brightness level. The center displays the percentage value (e.g., "75%"). Rotating the knob adjusts brightness in 5% steps. Knob press returns to the Power page.

### Presets Page (Page 3)

Four preset circles are arranged in a diamond pattern:
- **Top:** Warm White
- **Left:** Soft Amber
- **Right:** Neutral White
- **Bottom:** Low Nightlight

Each circle is 55px diameter. The currently active preset glows amber; inactive presets show a dim outline. Rotating the knob highlights presets sequentially (top → right → bottom → left). Knob press applies the highlighted preset. Touch tap on any circle selects and applies it immediately.

## Sleep/Wake Behavior

Wake-only-first is strictly enforced on all input paths. On wake, the center button fades in first (the hero element appears immediately), then peripheral elements (labels, dots) appear after a short delay. This creates a "spotlight" effect where the most important element appears first. On sleep, the display dims to 10% backlight after 60 seconds of inactivity.

## LED Ring Behavior

The LED ring mirrors the center button state: all 5 LEDs glow amber when the light is on, all LEDs are off when the light is off. This creates a unified "the whole device lights up" moment during power toggle.

## Dark-Room Behavior

Excellent dark-room usability. The glowing center button is specifically designed to be visible in complete darkness without being bright enough to disturb sleep. The amber glow is warm and non-stimulating (no blue light). The large 140px touch target is easy to hit even when half-asleep — no precision required.

## Daylight Readability

Good. The large button and high contrast (amber on black, white text on amber) ensure readability in ambient light. The glow effect is less visible in daylight, but the button outline, text, and color state remain clear.

## Typography and Spacing

| Element | Font | Size | Color |
|---------|------|------|-------|
| Power button text ("ON"/"OFF") | Roboto Bold | 32px | White (on) / Gray (off) |
| Brightness percentage | Roboto | 48px | White |
| Preset labels | Roboto | 16px | White (active) / Gray (inactive) |
| Page indicator dots | N/A | 8px circles | Amber (active) / Dark gray (inactive) |

## Color Palette

| Element | On State | Off State |
|---------|----------|-----------|
| Power button background | 0xFFA500 (amber) | 0x1A1A1A (near-black) |
| Power button border | None (glow replaces border) | 0x555555 (dim gray) |
| Power button shadow/glow | 0xFFA500 at 40% opacity, 20px spread | None |
| Arc indicator | 0xFFA500 (amber) | 0x333333 (dark gray) |
| Background | 0x000000 (pure black) | 0x000000 (pure black) |
| Text primary | 0xFFFFFF (white) | 0x888888 (gray) |

## Motion/Animation Concept

The hero animation is the power toggle. This is a single-event animation (not continuous) that fires once per toggle:

**Toggle ON:** The center button transitions from dark ghost to amber glow over 200ms. The shadow/glow appears simultaneously. LED ring turns amber in sync.

**Toggle OFF:** The amber glow fades to dark over 300ms. Shadow disappears. LED ring turns off in sync.

No continuous animations are used — the device is visually still when untouched, per the VelaDial design philosophy ("animate on interaction, not on idle").

## Why It Feels Premium

The oversized glowing button creates an emotional response — it feels like pressing a luxury car's ignition button or tapping a high-end audio system's power control. The concentric-circle design language (bezel → glow halo → button → text) feels cohesive and intentional. The single-purpose hero element communicates confidence: "this device does one thing beautifully."

## What Is v1-Compatible

- Controls `light.bedroom_group` exclusively
- 3-page architecture (Power, Brightness, Presets)
- 4 locked presets (Warm White, Soft Amber, Neutral White, Low Nightlight)
- Wake-only-first enforced on all input paths
- Knob-first operable (touch is enhancement, not requirement)
- Pure black background, amber accents
- No Environment page, no sensor fusion

## What Is Concept-Only (Requires Approval)

- The 140px oversized button as hero element (production uses a different layout)
- Diamond preset arrangement (production uses a different preset layout)
- Shadow/glow effect on the button (may require testing for performance)
- LED ring synchronized toggle animation (production may use simpler LED logic)

## Hardik Approval Required

- Adopting the large center power button as the primary visual identity for the Power page
- Diamond arrangement for presets (vs. the current production preset layout)
- Shadow/glow styling on the power button (performance impact to be validated)
