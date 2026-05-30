# VelaDial Single-Source Test Guide

This guide is the **single source of truth** for testing the VelaDial door-side rotary display firmware. It supersedes older LocalTuya-specific guides.

## 1. Hardware Status
- **Display**: ✅ Working (PR #61)
- **Touch**: ⏳ Pending validation (PR #62)
- **Knob**: ⏳ Pending validation (PR #62)
- **Home Assistant Control**: ⏳ Pending validation (PR #62)

## 2. Flashing the Firmware
```bash
# 1. Connect the ESP32-S3 via USB
# 2. Compile and flash
esphome run esphome/door_side_rotary.yaml
```

## 3. Home Assistant Integration (Surplife)
VelaDial controls `light.bedroom_group`. You must create this group in Home Assistant.

### Step 3.1: Add Surplife Lamps to HA
1. Install the **Surplife** integration in Home Assistant (via HACS or core if available).
2. Authenticate with your Surplife account.
3. Verify you can control the lamps from the HA dashboard.

### Step 3.2: Create the Light Group
Add this to your `configuration.yaml` (or via the HA UI Helpers):
```yaml
light:
  - platform: group
    name: "Bedroom Group"
    entities:
      - light.surplife_lamp_1
      - light.surplife_lamp_2
      - light.surplife_lamp_3
      - light.surplife_lamp_4
      - light.surplife_lamp_5
```
*Note: Replace `light.surplife_lamp_X` with your actual entity IDs.*

## 4. Testing Checklist

### A. Display & Boot
- [ ] Screen turns on immediately (no black screen delay)
- [ ] Boot splash shows "VELADIAL" and "Booting..."
- [ ] After 3 seconds, transitions to Power Page (shows "OFF" or "ON")
- [ ] Theme name at bottom shows "T01 Minimal"

### B. Touch & Navigation
- [ ] Swipe left/right changes pages (Power ↔ Brightness ↔ Presets)
- [ ] Tapping the screen wakes it from sleep

### C. Knob & Theme Selector
- [ ] Rotate knob on Brightness page changes value
- [ ] Rotate knob on Presets page changes highlight
- [ ] Long-press knob (>1.5s) opens Theme Selector
- [ ] Theme Selector shows large color circle and "01/20"
- [ ] Rotate knob in Theme Selector changes colors and names
- [ ] Short-press knob in Theme Selector applies theme (shows "APPLIED" in green)

### D. Home Assistant Control
- [ ] Short-press knob on Power page toggles `light.bedroom_group`
- [ ] Changing brightness on Brightness page updates lamps
- [ ] Selecting a preset on Presets page updates lamps
- [ ] When lamps are turned on via HA app, VelaDial screen updates to "ON"

## 5. Troubleshooting
If the screen is black:
- Check the ribbon cable connection.
- Verify you flashed `door_side_rotary.yaml` and not a debug file.

If touch doesn't work:
- Check ESPHome logs for `I2C Bus Error` or `CST816` errors.
- The CST816 goes to sleep after 2 seconds; try tapping repeatedly.

If HA control fails:
- Check ESPHome logs for `POWER TOGGLE: HA API NOT connected`.
- Verify `light.bedroom_group` exists in HA Developer Tools -> States.
