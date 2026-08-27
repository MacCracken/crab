# 001 — Every frame allocates, and nothing is ever freed

**Measured**: 2026-08-26, against crab 0.5.0 and dhancha 0.9.12.
**Re-measured**: 2026-08-26, against dhancha **0.9.13** — step 1 below is DONE.

> ⭐ **Step 1 has landed upstream (dhancha 0.9.13) and the per-frame cost is down 44.8 %:
> 746,440 B → 412,040 B.** `dh_surface_new` went from 334,440 B to **40 B**. Steps 2 and 3 are
> untouched, so the gate is *narrowed, not closed* — 412 KB per keypress is still unreclaimed, and
> the "do not add a continuously-repainting element" rule below still stands (24 MB/s at 60 Hz).
> ⚠ Step 1 was **not** the one-line change this document called it — see the correction under
> "What the fix looks like".

## The invariant

crab rebuilds its **entire widget tree on every render**, and the allocator underneath it
(`lib/alloc.cyr` / `lib/alloc_agnos.cyr`) is a **bump allocator with no `free()`**. `alloc()` moves a
pointer through a chain of mmap'd 2 MB chunks; old chunks are never unmapped. `alloc_reset()` exists
but rewinds the *whole* arena, which would invalidate the pane paths, the readdir buffers and the
compositor client struct — so crab cannot call it.

**Therefore: every byte a render touches is retained for the life of the process.**

This is not a bug in crab, and it is not a bug in dhancha alone. It is the shape of the substrate,
and it constrains every feature on the roadmap. A reader cannot derive it from `crab_render`, which
looks like ordinary immediate-mode UI code.

## The number

Measured with a host probe calling the production `crab_render` at 380×220 with 114 entries per pane
(the real iron count for `/`, recorded in `src/main.cyr`):

```
frame delta bytes: 412152
frame delta bytes: 412040
frame delta bytes: 412040
frame delta bytes: 412040
frame delta bytes: 412040
total over 5 frames: 2060312
```

**412,040 B per frame**, against **746,440 B** before dhancha 0.9.13. Where it goes now:

| item | bytes | note |
|---|---:|---|
| `dh_surface_new` pixel buffer | ~~334,440~~ **40** | ✅ **fixed in dhancha 0.9.13** — deferred to `dh_surface_pixels`, which nothing calls |
| `sd_surface_new` inside `dh_surface_render` | 334,432 | the real render target — **step 2, still open** |
| ~236 widget records @ 248 B | 58,528 | root, panes, columns, headers, lists, 228 rows — **step 3, still open** |
| crab's own row/status scratch | ~19,000 | `disp`, size strings, `crab_u2s` temporaries |

⚠ **The original figure in this document was 749,704 B and the reproduction measured 746,440 B.** The
3,264 B gap is fixture-dependent — entry-name lengths and the file/directory mix change how much row
and status scratch each frame allocates — not a change in the code. **The two pixel buffers reproduce
to the byte** (334,440 and 334,432), which is the part the conclusion rests on. The probe is
`alloc_used()` deltas around five back-to-back `crab_render` calls at 380x220, 114 entries per pane.

⛔ **`sd_surface_new` is now 81 % of what remains**, so step 2 is no longer a secondary item — it is
almost the whole problem.

**Before 0.9.13, 89 % of it was two full-size pixel buffers, and one of them was pure waste.**
`dh_surface_new` (`lib/dhancha.cyr:2167`) did `alloc(40)` then `alloc(w*h*4)` and stored the buffer at
`DH_S_PIXELS` — but `dh_surface_render` (`:2468`) ignores it entirely and allocates its own
`sd_surface_new(w, h)` to draw into. Nothing ever touched the first buffer. **That half is gone**;
the second buffer remains, and is now 81 % of the frame on its own.

## Why it has not bitten yet, and when it will

crab renders **on input**, not continuously. A navigation session of a few hundred keypresses costs a
few hundred megabytes and the process exits before anyone notices. Until 0.5.0 the event loop also
terminated itself after about two seconds of spinning, which capped the damage by accident.

Two roadmap items remove that accident:

- **M2's self-redraw** — the idle mascot line, transfer progress and index progress all repaint
  *without* input. At 60 Hz, 412,040 B/frame is **24 MB/s** (it was 45 MB/s before 0.9.13).
- **M4's transfer tray** — repaints continuously for the duration of a copy.

There is a second, smaller leak on the same footing: `dh_setu_poll_event` (`lib/dhancha.cyr:2646`)
calls `setu_msg_new()` **before** it knows whether anything is pending, so every idle poll leaks
~80 B. ⛔ **0.5.0's stopgap is `sys_pause` (#14), NOT a ~16 ms sleep, and this document said sleep.**
A `sys_sleep_ms` draft `preempt_disable()`d the machine and the compositor never presented at all —
see the ⛔ block at the call site in `src/main.cyr`. `pause` yields to a ready proc and then waits on
an interrupt, which bounds the same leak without stopping the scheduler.

## What the fix looks like, and where it lives

**Mostly upstream.** dhancha allocates through plain `alloc()` at 19 sites and offers no arena hook,
so crab cannot redirect the widget-tree allocations no matter how it is written. ⚠ **The render
target is the exception** — see step 2: that one needs a change on *both* sides. Three
steps, roughly in value order:

1. ✅ **DONE (dhancha 0.9.13) — stop allocating the unused buffer in `dh_surface_new`.** Measured
   746,440 → 412,040 B/frame, a 44.8 % cut; `dh_surface_new(380,220)` is 334,440 B → 40 B.
   ⛔ **AND IT WAS NOT "one line", WHICH THIS DOCUMENT ASSERTED TWICE.** Simply storing 0 breaks
   dhancha's `programs/event_test.cyr` S): `dh_surface_present` returns `_NO_SURFACE` when pixels are
   0 and `_UNSUPPORTED` otherwise, so a permanently-zero field silently downgrades 0.9.5's
   "refuse loudly and diagnosably" contract into "bad surface". The shipped fix **defers** the
   allocation into `dh_surface_pixels` (allocate on first call, cache) so the published accessor's
   contract is unchanged for any external holder while every caller that never asks pays nothing.
   Mutation-verified: the naive variant fails `event_test` with exit 1.
   ⚠ A "one-line upstream fix" that a test in the upstream repo rejects is the shape of estimate that
   gets made from reading the caller and not the callee's suite.
2. **Let `dh_surface_render` reuse a surface** rather than allocating a new `sd_surface` per call.
   ⛔ **THIS IS NOT PURELY UPSTREAM, CONTRARY TO THE HEADING ABOVE.** A reused render target has to
   hang off something that outlives the frame, and the natural home is the `DhSurface` — but
   **`crab_render` builds a fresh `DhSurface` every frame** (`src/ui.cyr`, `dh_surface_new(w, h)`
   just before the layout pass), so a dhancha-side cache keyed on the surface would save crab
   **nothing**. Closing step 2 needs a matching crab change: hold one `DhSurface` across frames and
   render into it. It also changes `dh_surface_render`'s contract from *returns a fresh surface* to
   *may return the same surface as last time*, which a caller comparing two renders can observe —
   crab's own `src/render_test.cyr` renders twice and dumps the first. ⇒ Operator decision, not a
   drive-by.
3. **Give dhancha an arena hook.** The stdlib already ships `arena_new` / `arena_alloc` /
   `arena_reset` (`lib/alloc.cyr:370-540`) documented for exactly this *"per-frame / per-emission
   reuse"* pattern — `arena_reset` is a bump-back-to-base with no free list. A widget tree built into
   a per-frame arena and reset each frame costs nothing to reclaim.

crab's own share (~19 KB/frame) is fixable locally by hoisting the row and status scratch buffers out
of the render path, but it is under 5 % of the problem and doing it first would be optimising the
wrong end.

## What a reader should take from this

- **Do not add a continuously-repainting element** — an animation, a progress bar, a clock — until
  the dhancha gate is closed. It will work in QEMU and exhaust memory on iron. ⚠ **Step 1 landing
  does not lift this.** 412,040 B/frame at 60 Hz is still 24 MB/s; the rule was never about the
  factor, it was about the absence of `free()`.
- **Do not "fix" this in crab** by caching the widget tree. Rebuilding it every frame is what makes
  the scroll-offset round-trip and the toolkit-owned selection work; the 0.4.10 port depends on it.
- The gate is tracked in [`../development/roadmap.md`](../development/roadmap.md) M2 and blocks every
  milestone after it.
