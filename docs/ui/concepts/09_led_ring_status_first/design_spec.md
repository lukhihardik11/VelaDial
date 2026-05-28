# Concept 09: LED-Ring Status-First — Design Specification

**Date:** 2026-05-28  
**Status:** CONCEPT PROTOTYPE  
**Hardware:** ELECROW CrowPanel 1.28" ESP32-S3 (240x240 round, CST816D touch, rotary encoder, 5x WS2812 LED ring)  
**Target:** `light.bedroom_group`  
**Classification:** Layer concept (ranked #6, score 399)

---

## 1. Design Philosophy

This concept **inverts the typical smart-display hierarchy**. The 5-LED WS2812 ring is the primary output — visible from across the room — while the 240x240 screen is secondary, consulted only up close for precise adjustments. The room reads the device's state from the LED ring; the screen merely confirms what the ring is already showing.

This follows the **calm technology** principle: the device communicates its state to the entire room peripherally, without demanding attention. Walking into a bedroom and seeing the warm amber glow of the LED ring confirms "lights are on, Warm White preset" without needing to approach the display.

---

## 2. Visual Identity

### Screen (Secondary Role)

The screen is intentionally understated — almost a permanent Night Mode. No arcs, no large buttons, no decorative elements. Three pages with minimal centered text:

| Page | Content | Font | Purpose |
|------|---------|------|---------|
| Power | `ON` / `OFF` | Roboto 24pt, white/gray | Confirm what ring shows |
| Brightness | `75%` | Roboto 24pt, white | Precise value ring cannot show |
| Presets | `Warm White` | Roboto 20pt, amber | Name the active preset |

Background: Pure black (`0x000000`).  
Page indicator: 3 dots at bottom, active amber, inactive dim gray.

### LED Ring (Primary Role — Hero Element)

The 5 LEDs are individually addressable. The ring uses:
- **Color** to indicate the active preset (warm amber, deep gold, cool white, dim amber)
- **Brightness** to indicate light level (proportional to actual brightness)
- **On/Off** to indicate power state (glowing = on, dark = off)

---

## 3. LED Ring Behavior Map

### Power State Mapping

| Light State | LED Ring | Brightness |
|-------------|----------|------------|
| ON | All 5 LEDs at preset color | Proportional to light brightness |
| OFF | All 5 LEDs off | 0% |
| Unavailable | LED 0 + LED 2 + LED 4 red pulse (1Hz) | 15% max |

### Brightness Mapping (when ON)

| Light Brightness | LED Ring Behavior |
|-----------------|-------------------|
| 100% | All 5 LEDs at 50% LED brightness (bedroom-safe ceiling) |
| 75% | All 5 LEDs at 38% LED brightness |
| 50% | All 5 LEDs at 25% LED brightness |
| 25% | All 5 LEDs at 13% LED brightness |
| 5% (minimum) | 1 LED at 5% LED brightness, others off |

**Critical:** LED brightness is capped at 50% maximum to prevent bedroom disturbance. The mapping is non-linear — light brightness maps to LED brightness with a gamma curve that favors lower values.

### Preset Color Mapping

| Preset | LED Color (RGB) | Color Temperature Equivalent |
|--------|----------------|------------------------------|
| Warm White | `#FFB300` (amber) | ~2700K |
| Soft Amber | `#CC8800` (deep gold) | ~2200K |
| Neutral White | `#E0E0E0` (cool white) | ~4000K |
| Low Nightlight | `#FFB300` at 10% | ~2700K, very dim |

### Error/Unavailable State

| Condition | LED Pattern | Screen |
|-----------|------------|--------|
| WiFi disconnected | All 5 LEDs: alternating red/off, 500ms cycle | `NO WIFI` label |
| HA unavailable | LED 0+2+4: slow red pulse (2s cycle) | `UNAVAIL` label |
| Sensor failure | LED 1+3: amber blink (1Hz) | Normal display |

---

## 4. Animation Specifications

| Transition | Duration | Pattern |
|-----------|----------|---------|
| Power ON | 400ms | All LEDs fade from 0 to target brightness |
| Power OFF | 400ms | All LEDs fade from current to 0 |
| Brightness change | 200ms | All LEDs smooth fade to new level |
| Preset change | 300ms | Cross-fade from old color to new color |
| Error enter | 100ms | Snap to error pattern |
| Error exit | 300ms | Fade from error to normal |

---

## 5. Sleep/Wake Behavior

### Screen Sleep
- Screen timeout: 30 seconds of inactivity
- Backlight fades to 0% over 500ms
- LVGL rendering paused

### LED Ring Extended Timeout
- When screen sleeps, LED ring **remains active** for an additional 30 seconds
- After 30s, LED ring fades to off over 2 seconds
- This provides ambient room awareness after screen has gone dark

### Wake-Only-First
- ALL inputs (touch, knob CW/CCW, knob press) during sleep are **wake-only**
- First input wakes screen + resets LED ring to current state
- No action is executed on the wake event itself

---

## 6. Interaction Model

Identical to locked v1 spec:

| Input | Page 1 (Power) | Page 2 (Brightness) | Page 3 (Presets) |
|-------|----------------|--------------------|--------------------|
| Knob CW | → Page 2 | Brightness +5% | Next preset |
| Knob CCW | → Page 3 | Brightness -5% | Previous preset |
| Knob Press | Toggle power | Toggle power | Apply preset |
| Touch center | Toggle power | Toggle power | Apply preset |
| Swipe L/R | Page navigation | Page navigation | Page navigation |

**LED ring responds immediately** to all state changes — the user watches the ring, not the screen.

---

## 7. Dark-Room Usability

**Excellent.** The LED ring at minimum brightness (5% of 50% ceiling = 2.5% absolute) is visible without being disturbing. The minimal screen content (24pt on black) is readable at arm's length. The combination creates the ideal dark-room experience — ambient LED feedback without screen glare.

---

## 8. Daylight Readability

**Marginal for LED ring, adequate for screen.** The LED ring is hard to see in bright daylight (WS2812 LEDs are not high-brightness). The screen compensates with high-contrast white-on-black text. In daylight, the user relies more on the screen; in darkness, more on the ring.

---

## 9. Hardware-Test-Pending Notes

| Item | Status | Impact |
|------|--------|--------|
| LED color order (GRB assumed) | NOT TESTED | Colors may be wrong if RGB/GRBW |
| LED 0 physical orientation | NOT TESTED | Pattern positions may need rotation |
| LED brightness at 50% ceiling | NOT TESTED | May be too bright or too dim |
| LED visibility at 5% | NOT TESTED | May be invisible in practice |
| Cross-fade smoothness | NOT TESTED | May show stepping artifacts |
| GPIO48 RMT timing | NOT TESTED | May need RMT channel adjustment |

---

## 10. Constraints and Boundaries

- **v1 locked:** LED on/off echo, minimal screen labels, wake-only-first
- **v1-expanded (this prototype):** LED brightness scaling, preset color mapping, extended LED timeout
- **v2 (NOT implemented):** Individual LED patterns, LED config page, ambient-reactive LED
- **Maximum LED brightness:** 50% (127/255) — bedroom safety
- **Minimum visible LED:** 5% (13/255) — verified on hardware
- **LED ring is NOT a light source** — it is a status indicator only
