# VelaDial — UI Concept Notes (Clustering, Borrowing, Moodboard)

**Companion to:** [`ui_concept_direction_matrix.md`](ui_concept_direction_matrix.md), [`ui_concept_shortlist.md`](ui_concept_shortlist.md), [`ui_concept_selection_criteria.md`](ui_concept_selection_criteria.md).
**Status:** Planning / direction. **Optional reading.** No firmware in this PR.

This is the soft-form companion to the harder analysis in the matrix and shortlist. It groups the 20 concepts into clusters, identifies which can borrow elements from each other, and flags which would benefit most from a future image-generation moodboard pass.

---

## 1. Concept clusters

The 20 concepts naturally fall into six visual clusters. Concepts inside a cluster share aesthetic DNA — same mental category, same likely asset palette, similar performance envelope.

### Cluster A — Smart-home conservative

Concepts: **#1 Minimal Thermostat**, **#3 Large Center Power Button**, **#4 Single-Page Simple Mode**, **#7 Text-First Utility**, **#12 Door Switch Replacement**.

What they have in common: Nest / Hue / generic-smart-home visual register. Familiar; safe; ages well. Low novelty by design. All Easy feasibility. None require custom drawing or asset atlases. The risk with this whole cluster is *sameness* — the device ends up looking like every other smart-home object. The reward is *frictionless adoption* — every user immediately knows what to do.

### Cluster B — Round-display native

Concepts: **#2 SmartKnob-Inspired Arc**, **#5 Preset Ring UI**, **#14 Sundial Shadow UI**, **#17 Iris Aperture**, **#19 Vinyl DJ Crossfader**, **#20 Eclipse Corona**.

What they have in common: they treat the round bezel as a *feature* rather than a constraint. Circles, arcs, rings, rotational symmetry. These concepts feel "of" the GC9A01A in a way that the Cluster A concepts (which would work equally well on a square display) do not. This is the cluster the recommended Top 1 primary direction lives in.

### Cluster C — Astronomical / atmospheric

Concepts: **#13 Lunar Phase Visualization**, **#14 Sundial Shadow UI**, **#18 Radar Sweep Animation**, **#20 Eclipse Corona**.

What they have in common: sky imagery, slow motion, soft glow, contemplative pacing. Bedroom-appropriate by construction. These are the concepts that would feel right *at 3 AM* — they don't shout at you. Note the overlap with Cluster B (round-native) for #14 and #20.

### Cluster D — Pattern / generative

Concepts: **#15 Tree Ring Growth Pattern**, **#16 Topographic Contour Map**.

What they have in common: organic, generative, pattern-based. Strong novelty. Both share the same Achilles heel: legibility at a glance. The user has to *read* the pattern to extract the value. Bedroom UIs that require reading are usually bad UIs. Both are 9/10 novelty but score lower on the scoring framework because of this.

### Cluster E — Hardware ambient

Concepts: **#9 LED-Ring Status-First**.

The only member. Carries state via the 5-LED WS2812 ring rather than the screen. Compatible with any Cluster A / B / C / D screen concept as a *layer*. Worth treating as a horizontal capability rather than a competing direction.

### Cluster F — Motion / energy

Concepts: **#17 Iris Aperture**, **#18 Radar Sweep**, **#19 Vinyl DJ Crossfader**.

What they have in common: continuous or near-continuous animation. Highest visual reward in the matrix; highest performance risk. Whatever wins from this cluster must be POC'd before being adopted — animation cost on ESP32-S3 is the wildcard.

## 2. Element borrowing

Even if only one concept wins as the primary direction, elements from others can be borrowed and layered on top. Worth recording so the implementation PR can pick up "easy wins" from rejected concepts.

| Element | From concept | Could be borrowed by | Why it works |
| --- | --- | --- | --- |
| Ambient outer ring around the disc | #1 Minimal Thermostat | All of Cluster B (#2, #5, #14, #17, #19, #20) | A subtle status ring around the bezel works for almost any centre-content concept and reinforces "on / off / current" state at a glance. |
| Centre `%` glyph | #7 Text-First | All Brightness-page concepts (#2, #5, #13, #14, #17, #20) | Even the most atmospheric Brightness visual benefits from a numeric anchor for unambiguous reading. |
| Phase / state language for presets | #13 Lunar | #2 + #5 Top-1 baseline | The four presets could be *named* in lunar terms (Harvest / Blood / Full / Crescent) without changing the actual `light.turn_on` calls. Pure UI labelling. |
| Sun / shadow imagery for Night preset | #14 Sundial | All preset concepts | The "Low Nightlight" preset could visually depict a sun below the horizon, regardless of how the other three presets are rendered. |
| LED ring state colour | #9 LED-Ring | Any winning screen concept | The screen and the ring can carry different fidelities — screen shows the number, ring shows the room-from-across-the-room glance. No conflict. |
| Iris-shaped power button | #17 Iris | #3 Large Center Power Button | The big centre power button could *be* an iris that opens on toggle, even if no other iris animation appears elsewhere. Single-event animation, much lower cost than continuous. |

The point: even if Top 1 wins, the implementation PR can layer in *one* of these for visual punch without breaking scope. Recommend exactly one borrow at a time — too many and the design becomes incoherent.

## 3. Concepts that would benefit most from a moodboard image batch

A future image-generation pass (e.g. via Midjourney / DALL-E / Stable Diffusion / Nano-Banana with carefully scoped prompts and a single visual style) would help on these concepts most:

| # | Concept | Why moodboard helps |
| ---: | --- | --- |
| 14 | Sundial Shadow UI | Hard to imagine "what does a brightness shadow look like at 30% vs 70%" without seeing it. |
| 15 | Tree Ring Growth | Generative pattern; needs visual reference to judge legibility. |
| 16 | Topographic Contour Map | Same — abstract pattern, hard to picture from words. |
| 17 | Iris Aperture | The aperture *frames* matter visually — single still image cannot capture motion, so a batch of stills at different apertures helps. |
| 20 | Eclipse Corona | Heavily visual; corona styling has many possible interpretations (sharp / soft / pulsing / steady). |
| 13 | Lunar Phase Visualization | Lower priority — moon phases are universally recognised, but a moodboard helps pick the right *style* (woodcut / photoreal / minimalist / scientific). |

Concepts that **do not** need a moodboard pass: all of Cluster A (the smart-home conservative cluster) — those are already well-represented in stock thermostat / smart-switch imagery and reference is everywhere. Also #2 (SmartKnob) — Scott Bezek's project has extensive existing visuals.

## 4. Visual similarity warnings

Three concept pairs are visually similar enough that they should *not* be combined within a single device — the user would not be able to distinguish them at a glance:

- **#1 (Minimal Thermostat) + #3 (Large Centre Power Button)** — both are "big circle with a value in the middle". Use one *or* the other; combining loses the distinction.
- **#13 (Lunar Phase) + #20 (Eclipse Corona)** — both are circular-celestial imagery. Combining them muddies which is which (is that the moon or the sun?).
- **#14 (Sundial Shadow) + #18 (Radar Sweep)** — both feature a single rotating line on a circle. Combining looks like the device is undecided about its visual language.

These pairs *can* alternate between modes (e.g. lunar at night, sundial by day) but cannot share the same screen at the same time without confusion.

## 5. Notes for the implementation PR (future)

When the implementation PR is opened (not this PR), it should:

- Implement *one* primary direction from the shortlist's Top 1 / Top 2 / Top 3.
- Layer in at most one element-borrow from §2 above.
- Avoid combining concepts from the visually-similar pairs in §4.
- Run the ESPHome compile CI before requesting review.
- Preserve every locked rule the matrix enumerates.
- Not lift the Step 15B physical validation gate; flashing happens after merge, in Hardik's hands.

The implementation PR is **out of scope** for this concept-direction package. The job of this package is choice, not code.

## 6. What this PR explicitly does **not** do

- ❌ Does **not** implement any UI change.
- ❌ Does **not** change firmware behaviour.
- ❌ Does **not** modify any ESPHome YAML.
- ❌ Does **not** generate any actual moodboard images (§3 records candidates only).
- ❌ Does **not** claim physical PASS.
