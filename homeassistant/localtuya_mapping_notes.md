# VelaDial — LocalTuya DP Mapping Notes

**Status:** Placeholder reference only. **USER SETUP PENDING / NOT TESTED.**
**Purpose:** Help Hardik map Tuya bulb Data Points (DPs) to Home Assistant light entity features correctly during LocalTuya setup, without committing real values.

---

## What a "DP" is

Tuya devices speak a protocol built around numbered **Data Points** (DPs). Each DP is a small typed field — a boolean for power, an integer for brightness, a string for a colour mode, and so on. LocalTuya's job is to bridge those DPs to Home Assistant entity attributes (`brightness`, `color_temp`, `rgb_color`, etc.).

DP numbers and value ranges are **not standardised**. They are firmware- and product-specific. A bulb from one Tuya OEM may use DP `1` for power and DP `3` for brightness; a different OEM bulb may use entirely different numbers. Always look at the actual device, never assume.

## Typical DPs for Tuya RGB+CCT bulbs

This is a **typical, not authoritative** layout that most Tuya WiFi RGB+CCT bulbs follow. Your exact bulb may differ. The values shown are documentary only.

| DP # | Typical role | Typical type | Typical range |
| ---: | --- | --- | --- |
| 1 | Power on/off | boolean | `true` / `false` |
| 2 | Work mode | enum (string) | `white`, `colour`, `scene`, `music` |
| 3 | Brightness (white mode) | integer | 10 — 1000 (some firmwares 0 — 255) |
| 4 | Colour temperature (white mode) | integer | 0 — 1000 (0 = warmest, 1000 = coolest) |
| 5 | Colour (HSV in colour mode) | string | `"00ff00ff00ff"`-style hex packed HSV |
| 21 | Scene / preset | string | scene code or JSON payload |
| 22 | Countdown timer | integer | seconds |

The exact numbers and ranges for **your** bulbs must be confirmed against either:
- The bulb's entry on the Tuya IoT Platform (`developer.tuya.com`) after you have linked it to your developer account — this is the most reliable source.
- A device dump from the LocalTuya **Add → Get device status** flow, which prints the live DP values the device is currently reporting.
- Community reports for your specific bulb model (search the LocalTuya GitHub issues by model number — note these are not always current with firmware revisions).

## Finding DPs on your bulbs

There are three practical paths, in increasing order of effort:

1. **LocalTuya's built-in scan.** After installing LocalTuya and entering a device's IP, ID, and `local_key`, LocalTuya will read live DP values and present them as a list. This is usually enough.
2. **Tuya IoT Platform query.** Log into `developer.tuya.com`, link your Smart Life / Tuya app account, find the device, open its DP definitions panel. Most authoritative source.
3. **Manual sniff.** `tuyapi` / `tinytuya` Python tools can dump DP values from the LAN directly. Useful when LocalTuya cannot connect for some reason. Not required for normal setup.

## Mapping DPs to LocalTuya light entity fields

During the LocalTuya UI **Add → Light** wizard you will be asked to assign DPs to these light-entity features. Use this mapping as a checklist:

| LocalTuya Light field | DP role | Typical DP # | Required for VelaDial? |
| --- | --- | ---: | --- |
| **Switch DP** | Power on/off | 1 | **Yes** (required for `light.turn_on` / `light.turn_off`) |
| **Brightness DP** | Brightness | 3 | **Yes** (required for the door-side brightness arc + knob) |
| **Brightness lower / upper** | Brightness range bounds | (range of DP 3) | **Yes** — wrong bounds cause percentage scaling bugs |
| **Color temp DP** | Color temperature | 4 | **Yes** (required for Warm White / Soft Amber / Neutral White / Low Nightlight presets) |
| **Color temp lower / upper** | Color temp range bounds | (range of DP 4) | **Yes** — wrong bounds map presets to the wrong temperatures |
| **Color DP** | RGB / HSV | 5 | **Optional** (v1 presets are color_temp-based; RGB is v2 / future) |
| **Color mode DP** | Work mode | 2 | **Yes** if the bulb requires explicit mode switching before color/temperature DPs are honoured |
| **Scene DP** | Scenes | 21 | **No** — not used by v1 |
| **Music DP** | Music mode | (varies) | **No** — not used by v1 |

If your bulb uses different numbers, substitute accordingly. The role-to-field mapping is what matters; the specific DP numbers are device-specific.

## Brightness range — the most common mis-map

VelaDial's door-side firmware sends `brightness_pct: "1"` to `"100"` (per Home Assistant convention; see `esphome/door_side_rotary.yaml` preset/arc handlers). Home Assistant scales that 1–100 percentage to its internal 0–255 brightness model, then LocalTuya scales 0–255 to whatever the bulb's brightness DP expects.

Two common failure modes:

- **Range too narrow.** If you set the brightness lower bound too high (e.g. lower=10, upper=1000), the bulb may never get fully off via brightness alone, and the minimum brightness will feel surprisingly bright on the "Low Nightlight" preset.
- **Range too wide.** If you tell LocalTuya the range is 0–255 when the firmware actually expects 10–1000, mid-range commands will appear dim-or-off because LocalTuya is sending small values the bulb interprets as near-zero.

When in doubt, set the brightness DP via LocalTuya's "Set device value" debug tool to the bulb's actual maximum reported value first, observe the result, and back off from there.

## Color temp range — the second most common mis-map

Tuya bulbs typically use a 0 — 1000 range where 0 is the warmest temperature the bulb supports and 1000 is the coolest. Home Assistant's `color_temp` field is in **mireds** (typical range: 153 = ~6500 K = cool, 500 = ~2000 K = very warm), and LocalTuya maps the two ranges.

VelaDial's presets target these mireds:

| Preset | Target mireds | Approx. K | Notes |
| --- | ---: | ---: | --- |
| Warm White | 370 | ~2700 K | Standard warm-white incandescent feel |
| Soft Amber | 454 | ~2200 K | Sunset / candlelight feel |
| Neutral White | 250 | ~4000 K | Office daylight feel |
| Low Nightlight | 454 | ~2200 K | Warm and at 5 % brightness |

If your bulbs cap out at, say, ~3500 K on the cool side or ~2700 K on the warm side, the Soft Amber and Low Nightlight presets may not visibly differ from Warm White — the bulb is being asked to go warmer than its hardware can produce. There is no firmware workaround; you would simply need warmer-capable bulbs. Confirm your bulb's actual mired range before tuning the presets.

## What the repo does NOT contain (and why)

| Item | Where it lives | Why not in repo |
| --- | --- | --- |
| Tuya `local_key` for each bulb | Home Assistant's `/config/.storage/` (encrypted at rest) | Per-device secret; leak = full local control by anyone on your LAN |
| Tuya Cloud developer credentials | Your personal Tuya account on `developer.tuya.com` | Personal account credential |
| Bulb IP addresses | Router DHCP table + LocalTuya UI | Reveals LAN structure |
| Bulb device IDs | LocalTuya UI | Used in combination with `local_key` to identify devices to anyone on the network |
| Wi-Fi SSID / password | ESPHome `secrets.yaml` (gitignored) and your router | Standard secret hygiene |

The repository contains **no** real values for any of the above. Every entity ID you see in `homeassistant/packages/veladial_example.yaml` is a placeholder.

## Status

| Item | Status |
| --- | --- |
| LocalTuya install on Raspberry Pi | USER SETUP PENDING / NOT TESTED |
| Tuya bulb adoption | USER SETUP PENDING / NOT TESTED |
| DP discovery for the actual bulbs | USER SETUP PENDING / NOT TESTED |
| Brightness range mapped correctly | USER SETUP PENDING / NOT TESTED |
| Color temp range mapped correctly | USER SETUP PENDING / NOT TESTED |
| `light.bedroom_group` controls all five bulbs locally | USER SETUP PENDING / NOT TESTED |
| End-to-end command path from VelaDial firmware to bulbs | USER SETUP PENDING / NOT TESTED |

Nothing in this document has been validated against real hardware. The Step 15B physical validation gate remains in force. No physical PASS is claimed by these notes.

## Further reading

These are the upstream sources to keep next to you during LocalTuya setup. They are also listed in `docs/REFERENCE_RESOURCES.md`:

- LocalTuya GitHub: https://github.com/rospogrigio/localtuya
- LocalTuya alternative fork docs: https://xzetsubou.github.io/hass-localtuya/
- Tuya IoT Platform overview: https://developer.tuya.com/en/docs/iot/Home-assistant-tuya-intergration?id=Kb0eqjig0utdd
- HACS (for installing LocalTuya): https://hacs.xyz/

LocalTuya is a community-maintained integration. Confirm the upstream is still active and your version is current before relying on it long-term.
