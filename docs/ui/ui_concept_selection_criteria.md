# VelaDial — UI Concept Selection Criteria & Scoring Framework

**Companion to:** [`ui_concept_direction_matrix.md`](ui_concept_direction_matrix.md), [`ui_concept_shortlist.md`](ui_concept_shortlist.md), [`ui_concept_notes.md`](ui_concept_notes.md).
**Status:** Planning / direction. **No firmware in this PR.**

This document defines the criteria used to evaluate the 20 concepts in the matrix and provides a numeric scoring framework that the implementation PR (later) can reuse to defend the final pick.

---

## 1. Why a scoring framework

Picking a UI concept by gut alone is fine when there are two options. With 20, the conversation degenerates without a structure. The framework here is meant to:

- Force a *consistent* evaluation across all 20 concepts.
- Make the rejection of bold concepts (e.g. #4 Single-Page Simple) defensible by criterion, not vibes.
- Give Hardik a predictable lens through which any later "what about concept X?" question can be answered quickly.

The framework is also small on purpose. Eleven criteria max. No 40-row Likert-scale grids that nobody actually fills out.

## 2. Hard gate: v1 scope preservation

Before any concept is scored numerically, it passes a single binary gate:

> **Does adopting this concept require changing the locked v1 scope?**
>
> Locked v1 includes: exactly 3 pages (Power, Brightness, Presets), exactly 4 presets, no Environment page, wake-only-first behaviour, knob-first interaction, 3-dot indicator with amber active dot, SHT45 as secondary/future diagnostic only, APDS-only bedside, no sensor fusion, no VL53L4CD/VL53L0X firmware.

If the answer is **yes**, the concept is `REJECT UNLESS SCOPE CHANGES` regardless of how it would score otherwise. No numeric calculation runs. This is the gate that disqualifies #4 (Single-Page Simple) and #8 (Apple Watch Complications) in the matrix.

If the answer is **no**, the concept proceeds to numeric scoring.

## 3. Scoring criteria

Ten weighted criteria. Each scored 0 — 3 (0 = poor, 1 = acceptable, 2 = good, 3 = excellent). Final score = sum of `score × weight` divided by `sum of weights × 3` (i.e. normalised 0 — 1, displayed as a percentage).

| # | Criterion | Weight | What 3 looks like | What 0 looks like |
| ---: | --- | ---: | --- | --- |
| 1 | **Round-display fit (240×240)** | 3 | Geometry is natively circular; nothing is cropped by the round bezel. | Layout is fundamentally rectangular and the round bezel cuts off content. |
| 2 | **LVGL / compile safety** | 3 | Uses only stock LVGL widgets documented at `esphome.io/components/lvgl/widgets/`. CI green. | Requires custom C++ components, novel LVGL drivers, or undocumented features. CI risk. |
| 3 | **Wake-only-first preserved** | 3 | Same dual-layer `display_awake && !touch_woke_this_cycle` guard works without modification. | Concept inherently competes with the wake latch (e.g. continuous animation must run while asleep). |
| 4 | **Dark-bedroom readability** | 2 | Mostly black at idle; selective amber glow; no large white fills. | Requires large bright fills, white backgrounds, or aggressive flashing. |
| 5 | **Knob-first interaction** | 2 | Knob rotation drives the primary value cleanly; touch is supplementary. | Knob is awkward or irrelevant; concept assumes a touch-primary model. |
| 6 | **Animation / performance cost** | 2 | Static at idle; minimal redraw during interaction; <20% CPU during drag on ESP32-S3. | Continuous animation at idle; expected to dominate CPU; risks dropped touch events. |
| 7 | **Visually distinctive** | 1 | Looks unlike any other smart-home thermostat or wall switch. | Indistinguishable from a generic Material smart-home UI. |
| 8 | **Asset cost** | 1 | No new fonts, no new sprites, no new image assets beyond what is already in the repo. | Requires a sprite atlas, custom font, or generated-art pipeline at build time. |
| 9 | **PRD / UI brief drift** | 1 | Implements `docs/01_PRD.md` and `docs/04_UI_UX_Design_Brief.md` as written. | Requires PRD or UI brief revisions to justify. |
| 10 | **Realistic to test on hardware** | 1 | A user with the ELECROW board, an HA install, and a router can validate end-to-end in under an hour. | Requires hardware Hardik does not have, or test rigs the repo does not include. |

**Maximum weighted raw score:** `(3×3) + (3×3) + (3×3) + (2×3) + (2×3) + (2×3) + (1×3) + (1×3) + (1×3) + (1×3)` = `9 + 9 + 9 + 6 + 6 + 6 + 3 + 3 + 3 + 3` = **57**.
**Normalised score range:** 0 % — 100 %.

## 4. Threshold guidance

| Normalised score | Recommendation |
| ---: | --- |
| ≥ 80 % | **Strong v1 candidate.** Adopt or shortlist for primary direction. |
| 60 — 79 % | **v1 candidate with adaptation.** Worth a small POC PR if its visual upside is unique. |
| 40 — 59 % | **v2 / future.** Record the idea; do not block v1 on it. |
| < 40 % | **Reject for v1.** Either out of scope (gate failure) or net-negative even after considering visual upside. |

These thresholds are calibrated against the worked example in §5 so that the existing baseline (current YAML, roughly #2 + #3 + #7 + #10) scores in the ≥ 80 % band.

## 5. Worked examples

These two examples show the framework in action — scoring the strongest v1 candidate against the most novel one. Numbers are illustrative; later PRs may revise them based on POC findings.

### 5a. Concept #2 — SmartKnob-Inspired Arc

Gate: v1 scope preserved? **Yes.** Already implemented in `esphome/door_side_rotary.yaml`.

| Criterion | Weight | Score | Weighted |
| --- | ---: | ---: | ---: |
| Round-display fit | 3 | 3 | 9 |
| LVGL / compile safety | 3 | 3 | 9 |
| Wake-only-first preserved | 3 | 3 | 9 |
| Dark-bedroom readability | 2 | 3 | 6 |
| Knob-first interaction | 2 | 3 | 6 |
| Animation / performance cost | 2 | 3 | 6 |
| Visually distinctive | 1 | 2 | 2 |
| Asset cost | 1 | 3 | 3 |
| PRD drift | 1 | 3 | 3 |
| Testability | 1 | 3 | 3 |
| **Total** | | | **56 / 57 = 98 %** |

**Verdict:** Strong v1 candidate. Matches reality: this concept is already shipped.

### 5b. Concept #17 — Iris Aperture Mechanism

Gate: v1 scope preserved? **Yes** (used only on Brightness page; does not add a 4th page).

| Criterion | Weight | Score | Weighted |
| --- | ---: | ---: | ---: |
| Round-display fit | 3 | 3 | 9 |
| LVGL / compile safety | 3 | 1 | 3 (custom canvas drawing required) |
| Wake-only-first preserved | 3 | 2 | 6 (inline guard must be added carefully — animations can fire on programmatic updates) |
| Dark-bedroom readability | 2 | 2 | 4 (depends on palette) |
| Knob-first interaction | 2 | 3 | 6 |
| Animation / performance cost | 2 | 1 | 2 (continuous animation when value changes) |
| Visually distinctive | 1 | 3 | 3 |
| Asset cost | 1 | 1 | 1 (sprite atlas or custom widget) |
| PRD drift | 1 | 2 | 2 (still 3 pages, but UI brief does not mention iris imagery) |
| Testability | 1 | 2 | 2 (POC compile is straightforward; visual judgement is the hard part) |
| **Total** | | | **38 / 57 = 67 %** |

**Verdict:** v1 candidate with adaptation. Below the 80 % threshold for adoption-without-POC, but above the 60 % threshold for a small POC PR. Matches the shortlist's "stretch" placement.

### 5c. Concept #4 — Single-Page Simple Mode

Gate: v1 scope preserved? **No** — directly violates the "exactly 3 pages" lock.

**Verdict:** `REJECT UNLESS SCOPE CHANGES`. No numeric scoring runs. Matches the matrix.

## 6. How to use this when a new concept appears later

If Hardik (or anyone) suggests a 21st concept after this PR merges, the workflow is:

1. **Apply the gate.** Does the concept require changing the locked v1 scope? If yes, write a short note explaining what would need to change and stop here.
2. **If the gate passes,** score each of the 10 criteria with 0 — 3. Aim for honesty over optimism — overscoring is the most common failure mode of this kind of framework.
3. **Normalise to 0 — 100 %.** Use the threshold table in §4 to decide what to do with the result.
4. **Write the verdict and one paragraph of justification.** Append to the matrix (`ui_concept_direction_matrix.md`) and the shortlist if the verdict places it in the top 3.
5. **Do not jump straight to implementation.** Open a docs PR with the new entry first, get Hardik's sign-off, then write the implementation PR.

## 7. What this framework does **not** replace

- **Hardik's judgement.** A 78 % concept that Hardik genuinely loves can outrank an 82 % concept that nobody on the team is excited about. The score is decision support, not the decision.
- **Hardware validation.** A concept may score 95 % on paper and then look terrible on the real GC9A01A panel under bedroom lighting. The framework cannot substitute for `esphome run` + flashlight test.
- **The locked rules.** No combination of high scores authorises adding a 4th page, a 5th preset, an Environment view, sensor fusion, VL53L4CD firmware, or VL53L0X firmware. Those would require a separate process Hardik runs explicitly, not a score crossing a threshold.

## 8. What this PR explicitly does **not** do

- ❌ Does **not** implement any UI change.
- ❌ Does **not** change firmware behaviour.
- ❌ Does **not** modify any ESPHome YAML.
- ❌ Does **not** claim physical PASS.
- ❌ Does **not** lift any approval gate.
