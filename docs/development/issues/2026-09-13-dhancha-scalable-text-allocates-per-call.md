# dhancha's scalable text path allocates a full-surface canvas per call, outside the frame arena

> ⚠ **This is crab's COPY. The canonical filing is in dhancha**, at
> `dhancha/docs/development/issues/2026-09-13-scalable-text-allocates-per-call-outside-the-frame-arena.md`
> — an issue about another repository that lives only here is one nobody who could act on it will
> ever read.
> ⛔ **What it means for crab, in one line**: the moment crab passes a real font, **"a rendered frame
> costs the global heap ZERO bytes" — crab's M1.5 headline — stops being true**, and crab's own gate
> would not have noticed, because it measures the other branch.

**Status:** ✅ **CLOSED by dhancha 0.10.0 (2026-09-13) — and VERIFIED from crab's side 2026-09-14.**
Three moves: **sadish 0.5.5** put every per-call `alloc(` behind an `sd_alloc(n)` / `sd_alloc_set(fp)`
hook and added `sd_canvas_blit_at`; **rekha 0.3.10** draws its outline scratch from the same seam;
**dhancha 0.10.0** installs `dh_falloc` as that hook for the duration of one `dh_draw_text_ink`
(restoring the previous hook after) and sizes the canvas to clip ∩ surface ∩ run rather than to the
surface. ⭐ **crab's expiry assertion fired exactly as written and is now INVERTED**: a warm frame
under a real face costs the global heap **exactly 0**.
⚠ **dhancha's hand-off predicted crab's failure by name and to the byte**, from measurements taken
against crab 0.8.10 — not `scost > 0` but `arena_capacity_total(farena) == cap0`, *got 468,040,
expected 16,384*. It was right. Three things crab had to do, all of them from that note: pin
**sadish >= 0.5.5** and **rekha >= 0.3.10** (without them the build is REFUSED —
`2 reachable undefined function(s)`: `sd_alloc_set`, `sd_canvas_blit_at`); render **one warm-up face
frame** on the same surface and width before measuring, because a cold face frame chains ~452 KB of
arena chunks (glyph paths ~4.3 KB each) — the arena GROWING, which `arena_reset` then reuses, not a
leak; and open the face **outside** a draw, since `rekha_font_open` follows the scoped hook and would
die at the arena's first reset.
**Original status:** 🔴 OPEN — a real defect in dhancha's `font != 0` path. MEASURED, not read.
**Severity:** High for any dhancha consumer that adopts a scalable face and has an allocation
discipline. It is latent for everyone today only because **nothing in the stack passes a non-zero
font** — dhancha's own demo client is host-only.

## What was found

`dh_draw_text_ink`'s scalable branch opens with:

```
var cv = sd_canvas_new(sd_surface_width(sds), sd_surface_height(sds));
```

A **full-surface canvas, per call** — that is per LABEL, per FRAME — plus `rekha_char_to_sdpath`
allocating a sadish path **per glyph**. All of it from the global bump allocator, which has no
`free()`, and **none of it through dhancha's own per-frame arena** (`dh_falloc`), which every other
draw path in the toolkit already uses and which exists for exactly this.

The bitmap branch (`font == 0`) returns before reaching it, which is why nothing has noticed.

⚠ **The clipping the canvas is there for is right, and is not what is in question.** Its comment
earns it — *"a glyph straddling the viewport edge is CUT, not dropped… Dropping the whole glyph was
the tempting shortcut and it reads as text that vanishes a row early while scrolling."* The defect is
**where the memory comes from**, not what it is for.

## Why crab cares more than most

crab spent **four releases** (dhancha 0.9.13–0.9.15, crab 0.6.0) driving per-frame allocation from
**746,440 bytes to zero**, and it is M1.5 — a milestone of its own. A crab pane at 760×300 draws on
the order of a dozen labels per frame; at a full-surface canvas each, a single frame would allocate
megabytes that are never returned, and a session would exhaust the heap rather than merely slow down.

## ⛔⛆ And crab's gate would not have caught it — the blind spot is the interesting part

crab's zero-allocation assertion renders **twenty frames and asserts the heap cost is exactly 0**.
Every one of those renders passes `font = 0`. The scalable path is a different function body, so the
gate was measuring the branch that was not running — *"a gate that covers one state proves one
state"*, which crab's own roadmap already warns about in those words, about a different blind spot,
for the same reason.

⭐ **crab 0.8.10 closed that blind spot from its own side.** The suite builds a synthetic proportional
face, renders one frame with it, and asserts the cost is non-zero. ⚠ **That assertion carries its own
expiry, stated at the assertion:** it documents a defect, and **the day dhancha routes this canvas
through the frame arena it will FAIL and must be inverted**. That is deliberate — it is how a blocker
in a sibling repo gets a gate in this one, instead of being a sentence in a document that goes stale.

## What would close it

Route the canvas and the per-glyph paths through `dh_falloc` — the per-frame arena dhancha already
owns and already rewinds at `dh_frame_begin` — so the scalable path costs what the bitmap path costs:
nothing that survives the frame.

⚠ **It is not crab's to fix**, and crab should not work around it: a consumer-side workaround here
would mean crab reaching into dhancha's draw path, which is the toolkit's job by construction.

## Related

- [`2026-09-13-no-proportional-face-on-the-target.md`](2026-09-13-no-proportional-face-on-the-target.md)
  — the **other** blocker on `0.9.0 · A real face`, owned by agnos (canonical:
  `agnos/docs/development/issues/2026-09-13-no-proportional-face-on-the-target.md`). Both must clear, and they are
  independent: a face arriving without this fix would ship a memory leak, and this fix landing
  without a face would change nothing observable.
