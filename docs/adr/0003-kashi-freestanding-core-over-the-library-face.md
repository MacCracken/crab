# 0003 — Take kashi's freestanding core, not its library face

**Status**: Accepted
**Date**: 2026-08-17 (recorded here 2026-09-02, lifted out of `cyrius.cyml`)

## Context

crab draws all default text (`font = 0`) with kashi's CP437 8×16 bitmap glyphs, blitted by dhancha's
`dh_draw_text`. kashi ships two consumable surfaces:

- **the freestanding core**, `src/font_data.cyr` — glyph data plus `kashi_font_init` and
  `kashi_glyph_row`, and nothing else;
- **the library face**, `dist/kashi.cyr` — the core plus PSF/BDF/PCF import and a runtime font
  registry.

kashi's own ADR 0001 says stdlib-linked consumers should take the **library face**. crab links the
full stdlib, so on the face of it crab is exactly the consumer that guidance names, and taking the
core looks like a shortcut.

Two facts make it a real choice rather than a default:

1. **Until kashi 1.0.6 the library face was literally unconsumable.** `src/lib.cyr` opened with
   `include "src/font_data.cyr"` — a kashi-root-relative path that cannot resolve once the file is
   vendored into a consumer's `lib/`: `error: cannot open include file: src/font_data.cyr`.
   kashi 1.0.6 added `dist/kashi.cyr`, which fixes that. So before 1.0.6 there was no decision to
   make; after it, there is.
2. **The face costs measurably more, and dead-code elimination does not reclaim it.** Measured on
   dhancha: core **364,640 B** vs full face **548,000 B** — **+183,360 B (+50 %)** for import paths
   and a registry nothing in this stack calls.

## Decision

**crab declares `modules = ["src/font_data.cyr"]` — the freestanding core.**

crab calls exactly two kashi functions, `kashi_font_init` and `kashi_glyph_row`. It does not load
fonts at runtime and has no surface on which to ask for one.

⛔ **The expiry is written and specific: switch to `modules = ["dist/kashi.cyr"]` the day crab needs
runtime font LOADING.** Not the day proportional text lands — that is rekha's TrueType path, a
different dependency — but the day crab must read a font file the user chose.

## Consequences

- **Positive** — 183 KB that buys crab nothing stays out of a binary this project measures at every
  cut. crab's size sensitivity is not theoretical: the chitra adoption was an operator ruling taken
  with +115 % in hand, and kashi's own library face was refused at +50 %.
- **Negative** — crab diverges from kashi's stated guidance for stdlib-linked consumers, so a reader
  who finds only kashi's ADR 0001 will think crab is wrong. That is the cost this ADR exists to pay
  down.
- **Neutral** — the choice must be revisited when the trigger above fires, and nothing gates it. It
  is written at the declaration in `cyrius.cyml` as a one-line pointer here.

## Alternatives considered

- **The library face (`dist/kashi.cyr`), per kashi ADR 0001.** Rejected on the measurement: +50 % of
  the dependency's footprint for PSF/BDF/PCF import and a runtime registry crab never calls, with
  `CYRIUS_DCE=1` reclaiming none of it. Reconsider at the written expiry, not before.
- **Vendoring the glyph table into crab directly.** Rejected: it makes crab the owner of a system
  font it does not maintain, and a kashi glyph fix would then need transcribing rather than
  re-resolving. crab already vendors `lib/kashi_font_data.cyr`, but as a *dependency artifact* the
  toolchain manages, not as crab source.
