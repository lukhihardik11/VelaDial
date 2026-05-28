# Concept 09: LED-Ring Status-First — Validation Notes

**Date:** 2026-05-28  
**Status:** COMPILE ONLY — NOT TESTED ON HARDWARE  
**Physical Validation:** PENDING HARDIK

---

## 1. Compile Validation

| Check | Result |
|-------|--------|
| ESPHome config validation | PENDING |
| Full firmware compile | PENDING |
| Zero errors | PENDING |
| Zero warnings (critical) | PENDING |
| Binary size within flash limits | PENDING |

---

## 2. LED Ring Hardware Tests (CRITICAL — ALL PENDING)

These tests MUST be performed on physical hardware before this concept can be considered validated:

### Test L1: Color Order Verification
**Purpose:** Confirm GRB color order assumption  
**Method:** Set LED 0 to pure red (255, 0, 0). If LED shows green, order is RGB. If LED shows red, order is GRB (correct).  
**Expected:** LED 0 shows RED  
**Result:** NOT TESTED

### Test L2: LED 0 Physical Position
**Purpose:** Determine which physical LED is index 0  
**Method:** Set only LED 0 to white, all others off. Note physical position (top/bottom/left/right).  
**Expected:** Document LED 0 position for pattern design  
**Result:** NOT TESTED

### Test L3: Brightness Ceiling Comfort
**Purpose:** Verify 50% brightness ceiling is appropriate for bedroom  
**Method:** Set all 5 LEDs to amber at 50% brightness. Observe in dark room from bed position (2-3 meters).  
**Expected:** Visible but not disturbing, no eye discomfort  
**Result:** NOT TESTED

### Test L4: Minimum Visible Brightness
**Purpose:** Verify 5% brightness is visible in dark room  
**Method:** Set 1 LED to amber at 5% brightness. Observe from 2-3 meters in complete darkness.  
**Expected:** Barely visible, confirms device is powered  
**Result:** NOT TESTED

### Test L5: Color Temperature Distinction
**Purpose:** Verify preset colors are distinguishable on WS2812  
**Method:** Cycle through all 4 preset colors at 50% brightness. Ask observer to identify which is active.  
**Expected:** At least 3 of 4 presets clearly distinguishable  
**Result:** NOT TESTED

### Test L6: Transition Smoothness
**Purpose:** Verify 200ms/300ms/400ms transitions look smooth  
**Method:** Toggle power (400ms), change brightness (200ms), change preset (300ms). Observe for stepping.  
**Expected:** Smooth fades without visible steps  
**Result:** NOT TESTED

### Test L7: Extended Timeout Behavior
**Purpose:** Verify LED ring stays on 30s after screen sleep, then fades  
**Method:** Let screen sleep. Observe LED ring for 30 seconds. Verify 2-second fade to off.  
**Expected:** Ring visible for 30s, smooth 2s fadeout  
**Result:** NOT TESTED

### Test L8: Error Pattern Visibility
**Purpose:** Verify red pulse error pattern is noticeable but not alarming  
**Method:** Simulate HA unavailable state. Observe red pulse from across room.  
**Expected:** Noticeable attention-getter without being startling  
**Result:** NOT TESTED

### Test L9: RMT Channel Conflict
**Purpose:** Verify GPIO48 WS2812 does not conflict with display SPI  
**Method:** Run full UI with LED ring active. Check for display glitches or LED artifacts.  
**Expected:** No interference between display and LED ring  
**Result:** NOT TESTED

### Test L10: Power Consumption
**Purpose:** Measure LED ring power draw at various brightness levels  
**Method:** Measure current at 0%, 25%, 50% LED brightness with all 5 LEDs amber.  
**Expected:** < 150mA at 50% (5 LEDs × 60mA max × 50%)  
**Result:** NOT TESTED

---

## 3. Screen UI Tests (Standard)

### Test S1: Label Readability
**Purpose:** Verify 24pt text is readable on 240x240 at arm's length  
**Method:** Display "ON", "75%", "Warm White" on respective pages. Read from 50cm.  
**Expected:** All text clearly readable  
**Result:** NOT TESTED

### Test S2: Page Navigation
**Purpose:** Verify swipe and knob page changes work  
**Method:** Swipe left/right, rotate knob on Power page  
**Expected:** Smooth page transitions  
**Result:** NOT TESTED

### Test S3: Wake-Only-First
**Purpose:** Verify first input only wakes, does not act  
**Method:** Let screen sleep. Touch screen. Verify no power toggle occurs.  
**Expected:** Screen wakes, no state change  
**Result:** NOT TESTED

---

## 4. Integration Tests

### Test I1: LED + Screen Sync
**Purpose:** Verify LED ring and screen show consistent state  
**Method:** Toggle power via HA. Verify both LED ring and screen update.  
**Expected:** Both update within 500ms of each other  
**Result:** NOT TESTED

### Test I2: LED Ring During Screen Sleep
**Purpose:** Verify LED ring shows correct state while screen is off  
**Method:** Change brightness via HA while screen is sleeping. Observe LED ring.  
**Expected:** LED ring updates even with screen off (during 30s window)  
**Result:** NOT TESTED

### Test I3: Simultaneous LED + Display Rendering
**Purpose:** Verify no frame drops or glitches when both are active  
**Method:** Rapidly change brightness while observing both LED ring and screen  
**Expected:** No visual artifacts on either output  
**Result:** NOT TESTED

---

## 5. Acceptance Criteria

| Criterion | Required | Status |
|-----------|----------|--------|
| Compile PASS | Yes | PENDING |
| LED color order correct | Yes | NOT TESTED |
| LED brightness safe for bedroom | Yes | NOT TESTED |
| All 4 presets distinguishable | Yes | NOT TESTED |
| Error pattern visible | Yes | NOT TESTED |
| No RMT conflicts | Yes | NOT TESTED |
| Wake-only-first works | Yes | NOT TESTED |
| Screen + LED sync | Yes | NOT TESTED |

**Overall Status: NOT VALIDATED — COMPILE ONLY**
