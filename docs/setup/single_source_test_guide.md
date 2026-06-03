# VelaDial Door-Side Rotary — Single-Source Test Guide

> **Firmware branch**: `firmware/fix-input-lockout-ha-verification-themeos-truth`
> **Primary file**: `esphome/door_side_rotary.yaml`
> **Target entity**: `light.bedroom_group` (5 Surplife RGBCW bulbs)

---

## Prerequisites (ONE-TIME Setup)

### 1. Enable "Allow device to perform Home Assistant actions"

1. Go to **Settings → Devices & Services → ESPHome**
2. Click **CONFIGURE** next to `veladial-door-rotary`
3. Check **"Allow the device to perform Home Assistant actions"**
4. Click **Submit**

> Without this, the device can send commands but HA silently ignores them.
> This is the #1 cause of "command dispatched but nothing happens."

### 2. Verify `light.bedroom_group` exists

1. Go to **Developer Tools → States**
2. Search for `light.bedroom_group`
3. If missing: **Settings → Helpers → Create → Light Group → "Bedroom Group"** → add all 5 Surplife bulbs
4. Ensure Entity ID is exactly `light.bedroom_group`

### 3. Test manually in HA first

Go to **Developer Tools → Actions** and run:

```yaml
action: light.turn_on
target:
  entity_id: light.bedroom_group
data:
  brightness_pct: 50
```

Then:

```yaml
action: light.turn_off
target:
  entity_id: light.bedroom_group
```

Both must physically toggle bulbs before testing VelaDial.

---

## Flash Command

```bash
esphome run esphome/door_side_rotary.yaml
```

---

## 20-Step Test Sequence

### Phase 1: Boot & Display (Steps 1-3)

| Step | Action | Expected Result | Log Message |
|------|--------|-----------------|-------------|
| 1 | Power on / flash | "VELADIAL" splash in ice-blue | `BOOT: Backlight ON` |
| 2 | Wait 3 seconds | Splash hides, Power page appears | `Boot: splash done, hiding overlay` |
| 3 | Observe Power page | Large "OFF" (roboto48), "Bedroom Group" label, "T01 Minimal" in theme color, "Press/Tap" hint | `BOOT: Rendering theme and UI` |

### Phase 2: HA Diagnostic Buttons (Steps 4-8)

These buttons bypass ALL input handling. They test ONLY the HA action dispatch path.

| Step | Action | Expected Result | Log Message |
|------|--------|-----------------|-------------|
| 4 | HA: press "VelaDial Turn ON Bedroom Group" | Bulbs physically turn ON | `TURN ON BEDROOM GROUP: requested` |
| 5 | Observe VelaDial screen | "ON" in theme color, "Bedroom Group ON" in green | State import fires |
| 6 | HA: press "VelaDial Turn OFF Bedroom Group" | Bulbs physically turn OFF | `TURN OFF BEDROOM GROUP: requested` |
| 7 | Observe VelaDial screen | "OFF" in gray, "Bedroom Group OFF" | State import fires |
| 8 | HA: press "VelaDial Brightness 50" | Bulbs turn on at 50% | `SET BRIGHTNESS 50: requested` |

**If Steps 4-8 ALL fail**: "Allow actions" permission is NOT enabled. Go back to Prerequisites Step 1.
**If Steps 4-8 work**: HA communication is confirmed. Proceed to Phase 3.

### Phase 3: Knob Deterministic Power Control (Steps 9-12)

| Step | Action | Expected Result | Log Message |
|------|--------|-----------------|-------------|
| 9 | Short-press knob (bulbs ON) | "Turning OFF..." → bulbs OFF → "Bedroom Group OFF" | `POWER CMD seq=1: lights_on=1 -> sending light.turn_off` |
| 10 | Short-press knob (bulbs OFF) | "Turning ON..." → bulbs ON → "Bedroom Group ON" | `POWER CMD seq=2: lights_on=0 -> sending light.turn_on` |
| 11 | Rapid double-press (<800ms) | ONLY first press fires. Second shows lockout | `INPUT IGNORED: duplicate within lockout (knob press)` |
| 12 | Wait 1s after any toggle | Status label shows verified state | `VERIFY seq=N: state_now=X` |

### Phase 4: Touch Deterministic Power Control (Steps 13-14)

| Step | Action | Expected Result | Log Message |
|------|--------|-----------------|-------------|
| 13 | Tap center of screen | Same as knob: deterministic ON or OFF | `INPUT ACCEPTED: touch power tap (seq=N)` |
| 14 | Rapid double-tap (<800ms) | Only first tap fires | `INPUT IGNORED: duplicate within lockout (touch tap)` |

### Phase 5: Theme Selector (Steps 15-18)

| Step | Action | Expected Result | Log Message |
|------|--------|-----------------|-------------|
| 15 | Long-press knob (>1.5s) | Theme Selector opens: color circle, "01/20", name, ring | `Theme selector: OPENED` |
| 16 | Rotate knob clockwise | Circle/ring color changes, index increments, name changes | `Theme selector preview: X - Name` |
| 17 | Short-press to apply | "APPLIED" green → returns to Power page | `Theme applied: X` |
| 18 | Observe Power page | New theme name, colors, motif, arc style | `Theme rendered: X` |

**Critical**: Pressing knob in Theme Selector must NOT also fire power toggle (fall-through bug was fixed with early-return lambda).

### Phase 6: Page Navigation (Steps 19-20)

| Step | Action | Expected Result | Log Message |
|------|--------|-----------------|-------------|
| 19 | Swipe left on screen | Navigates to Brightness page | — |
| 20 | Swipe left again | Navigates to Presets page | — |

---

## Troubleshooting Decision Tree

```
Step 4 fails (HA button doesn't toggle bulbs)?
├─ Check: "Allow device to perform HA actions" enabled?
│  ├─ NO → Enable it (Prerequisites Step 1) — this is the fix
│  └─ YES → Check: light.bedroom_group exists and works manually?
│     ├─ NO → Create it (Prerequisites Step 2-3)
│     └─ YES → Check ESPHome logs for connection errors

Step 9 fails (knob doesn't toggle) but Step 4 works?
├─ Check logs for "INPUT ACCEPTED" or "INPUT IGNORED"
│  ├─ "INPUT IGNORED" → Wait 800ms and try again (lockout working correctly)
│  ├─ "INPUT ACCEPTED" but no toggle → HA permission or state import issue
│  └─ No log at all → Knob hardware issue (check encoder wiring)

Step 11 shows double-toggle (both presses fire)?
├─ This should NOT happen with this firmware
├─ Check: are you on the correct firmware version? (must be this PR)
└─ Check logs: should see "INPUT IGNORED" for second press

Screen shows "ENABLE HA ACTIONS" in red?
├─ Either: HA API not connected, OR permission not granted
├─ Check: ESPHome device online in HA? (Settings → Devices)
├─ Check: "Allow device to perform HA actions" enabled?
└─ Restart HA if recently changed permissions
```

---

## Key Log Messages Reference

| Message | Meaning |
|---------|---------|
| `INPUT ACCEPTED: knob short press (seq=N)` | Knob press passed 800ms lockout, sending command |
| `INPUT ACCEPTED: touch power tap (seq=N)` | Touch tap passed 800ms lockout, sending command |
| `INPUT IGNORED: duplicate within lockout` | Second input within 800ms — correctly blocked |
| `POWER CMD seq=N: lights_on=X -> sending Y` | Deterministic command (turn_on or turn_off) dispatched |
| `VERIFY seq=N: state_now=X` | State verification 1000ms after command |
| `SHORT PRESS: applying theme from selector (no fall-through)` | Theme applied, power toggle NOT fired |
| `HA API CONNECTED` | API connection established |
| `HA API DISCONNECTED` | API connection lost |

---

## What Each Diagnostic Button Tests

| Button Name in HA | HA Action Sent | Isolates |
|-------------------|---------------|----------|
| VelaDial Turn ON Bedroom Group | `light.turn_on` | HA permission + entity existence |
| VelaDial Turn OFF Bedroom Group | `light.turn_off` | Same |
| VelaDial Test Toggle Bedroom Group | `light.toggle` | Toggle ambiguity |
| VelaDial Brightness 50 Bedroom Group | `light.turn_on` + `brightness_pct: 50` | Data parameter passing |

**If ALL 4 buttons fail** → HA permission not enabled (100% certain).
**If buttons work but knob/touch don't** → Input routing bug (report logs).
**If buttons AND knob/touch work** → Full success. Merge PR.

---

## Honest Limitations of This Firmware

1. **Does NOT recreate all 20 rich concept prototypes** — themes differentiate via accent color, motif label, LED ring color, arc visibility/width, and 5 layout groups (structural variants)
2. **Does NOT use `target:` syntax** — ESPHome only supports `data: entity_id:` (confirmed in docs)
3. **Does NOT claim physical validation** — awaiting Hardik's hardware test
4. **Does NOT auto-detect entity** — hardcoded to `light.bedroom_group` via substitution
5. **A dedicated visual-port PR is needed later** to bring each concept's full unique layout to production
