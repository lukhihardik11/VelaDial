# Concept 20: Eclipse Corona — Validation Notes

## 1. Compile Status

**COMPILE STATUS: PASS**
- ESPHome Version: 2026.5.0
- Framework: ESP-IDF 5.5.4
- Errors: 0
- Build Time: 165.61s (fresh build)
- RAM: 16.0% (52,292 / 327,680 bytes)
- Flash: 61.1% (1,121,055 / 1,835,008 bytes)

## 2. Physical Hardware Validation Plan (NOT TESTED)

This concept relies heavily on the illusion of glow and depth. It must be validated on the physical Elecrow GC9A01 display to ensure the effect is convincing.

### A. The "Banding" Test
- **Risk:** The GC9A01 is a 16-bit (RGB565) display. Smooth gradients often exhibit visible "banding" (stair-stepping of colors) because there aren't enough color steps available.
- **Test:** Observe the concentric glow rings at 50% and 100% brightness.
- **Success Criteria:** The transition between the concentric rings must look reasonably smooth, not like distinct, hard-edged circles.

### B. The "Black Level" Test
- **Risk:** IPS LCDs cannot produce true black (unlike OLEDs). The backlight bleeds through, making black look dark gray.
- **Test:** Observe the central "moon disk" in a completely dark room.
- **Success Criteria:** The contrast between the glowing corona and the dark moon disk must be strong enough to maintain the eclipse illusion. If the moon disk looks too gray, the illusion fails.

### C. The "Wall Projection" Test
- **Risk:** The LED ring might not align visually with the on-screen corona.
- **Test:** Mount the device on a wall. Turn brightness to 100%.
- **Success Criteria:** The on-screen amber glow and the physical LED amber glow on the wall should feel like a single, continuous phenomenon.

### D. Performance Test
- **Risk:** Blending 6-8 semi-transparent objects might cause frame drops when rotating the knob quickly.
- **Test:** Rapidly rotate the knob from 0% to 100% and back.
- **Success Criteria:** The UI must remain responsive, and the corona must expand/contract smoothly without tearing or significant lag.

## 3. Metaphor Comprehension

- **Risk:** Users might not understand that the glowing ring represents brightness, or they might find the dark center confusing.
- **Test:** Ask a user to "turn up the lights" without explaining the UI.
- **Success Criteria:** The user intuitively turns the knob, sees the corona expand, and understands the relationship between the visual size of the glow and the brightness of the room.
