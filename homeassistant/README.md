# VelaDial — Home Assistant Configuration Examples

**Status:** Placeholder example only. **USER SETUP PENDING / NOT TESTED.**
**Purpose:** Give Hardik (and any future installer) a starting point for the Home Assistant side of VelaDial so the firmware's hardcoded targets (`light.bedroom_group`) and optional comfort/occupancy helpers can be wired up consistently.

---

## What this directory contains

```text
homeassistant/
├── README.md                       (this file)
├── packages/
│   └── veladial_example.yaml       Example package: light group + optional helpers
└── localtuya_mapping_notes.md      LocalTuya DP-mapping cheat sheet (placeholders only)
```

All values in every file under `homeassistant/` are **placeholders**. No real Wi-Fi credentials, no real Tuya local keys, no real Home Assistant long-lived access tokens, no real device IDs are present. The example files are tracked in the repository; populated copies on your Home Assistant install must **never** be committed.

## What this directory is NOT

- Not a fully working Home Assistant configuration. You will edit placeholder entity IDs to match your real bulbs after LocalTuya discovers them.
- Not a substitute for `docs/setup/raspberry_pi_home_assistant_setup.md` — that walkthrough is the source of truth for OS install, ESPHome add-on, HACS, and LocalTuya setup.
- Not validated. Until you (Hardik) actually run Home Assistant on a Raspberry Pi, adopt the bulbs through LocalTuya, copy the example package in, and load it, the `light.bedroom_group` path remains **NOT TESTED**. No row in `hardware/validation_results.md` is changed by the existence of these files.

## How packages work in Home Assistant

Home Assistant's *packages* feature lets you split your configuration into per-feature YAML files instead of one monolithic `configuration.yaml`. Each file under a `packages/` directory becomes one logical bundle of config — domains like `light:`, `template:`, `automation:` all live at the top level of the file.

To enable packages, add this once to your main `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

After that, every `*.yaml` file you drop into `/config/packages/` will be loaded as a package when Home Assistant starts (or when you reload YAML).

## How to use these files

1. **Copy** `packages/veladial_example.yaml` from this repo to your Home Assistant config at `/config/packages/veladial.yaml` (the destination filename is up to you — `veladial.yaml` is fine).
2. **Edit** the placeholder entity IDs in that file to match your real Tuya bulb entity IDs as LocalTuya assigns them. The placeholders are `light.bedroom_bulb_1` through `light.bedroom_bulb_5`; replace with whatever LocalTuya created (e.g. `light.tuya_bulb_4dxxxxxxxxxxxxxxxx`).
3. **Decide** whether you want the *optional* template helpers (comfort index, bedroom occupied) and uncomment those sections if so. They are commented-out by default to keep the minimal install footprint as small as possible.
4. **Enable** packages in your `configuration.yaml` as shown above (only required once).
5. **Reload** Home Assistant via **Developer Tools → YAML → Reload Configuration** or restart Home Assistant.
6. **Verify** `light.bedroom_group` appears in **Settings → Devices & Services → Entities** with all five members listed.
7. **Read** `localtuya_mapping_notes.md` for guidance on the LocalTuya side — Data Point (DP) numbering varies by bulb firmware and the cheat sheet helps you avoid common mis-mappings.

For the broader install sequence — Raspberry Pi flashing, Home Assistant OS onboarding, ESPHome add-on, HACS, LocalTuya — follow `docs/setup/raspberry_pi_home_assistant_setup.md`. This directory is a supplement to that guide, not a replacement.

## Helpers UI vs. YAML packages

Section 7 of the setup guide tells you to create `light.bedroom_group` via the Helpers UI (**Settings → Devices & Services → Helpers → Create Helper → Group → Light group**). That path remains supported and is the easiest first-time setup. The YAML package here is an alternative for users who prefer config-as-code, want to keep the group definition in source control, or want to add the optional template helpers in the same file.

You can use either approach — not both. If you use the YAML package, do not also create the group via the UI: you would end up with two competing definitions of the same entity.

## Redaction rules — everything in this directory

These rules apply to anything you ever consider committing under `homeassistant/`:

- No real Wi-Fi SSIDs or passwords.
- No real Tuya `local_key` values. LocalTuya keys live in Home Assistant's encrypted storage on the Pi (`/config/.storage/`), not in YAML.
- No real Home Assistant long-lived access tokens.
- No real device IDs from the Tuya cloud.
- No real internal LAN IP addresses, MAC addresses, hostnames, or DHCP reservation tables.
- No real LocalTuya device entries (those are configured through the LocalTuya UI integration flow on your Pi — they are not file-based config).
- No screenshots that contain any of the above.

The `.gitignore` rules from PR #25 catch `*.local.yaml`, `*.secret`, `*.key`, and any file literally named `secrets.yaml`, but **do not rely on `.gitignore` as your only protection**. Run `git diff --staged` before any commit that touches `homeassistant/` and visually confirm nothing real slipped in.

## Status of the path this directory supports

| Item | Status |
| --- | --- |
| HA OS install on Raspberry Pi | USER SETUP PENDING / NOT TESTED |
| ESPHome add-on install + device adoption | USER SETUP PENDING / NOT TESTED |
| HACS install | USER SETUP PENDING / NOT TESTED |
| LocalTuya install + bulb adoption | USER SETUP PENDING / NOT TESTED |
| `light.bedroom_group` exists and controls bulbs | USER SETUP PENDING / NOT TESTED |
| Optional template helpers load without errors | USER SETUP PENDING / NOT TESTED |
| End-to-end local-only command path | USER SETUP PENDING / NOT TESTED |

Nothing in this directory has been validated on real hardware. All rows in `hardware/validation_results.md` remain `NOT TESTED`. The Step 15B physical validation gate is not lifted by anything here.
