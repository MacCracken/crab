# 001 — Every frame allocates, and nothing is ever freed

**Measured**: 2026-08-26, against crab 0.5.0 and dhancha 0.9.12.
**Re-measured**: 2026-08-26, against dhancha **0.9.14** + crab's hoisted surface — steps 1 and 2 are DONE.

> ⭐ **Steps 1 and 2 have landed and the per-frame cost is down 89.6 %: 746,440 B → 77,568 B.**
>
> | | per frame | note |
> |---|---:|---|
> | baseline (dhancha 0.9.12) | 746,440 B | |
> | after step 1 (dhancha 0.9.13) | 412,040 B | `dh_surface_new`'s dead pixel buffer, deferred |
> | after step 2 (dhancha 0.9.14 + crab) | **77,568 B** | the sadish render target, reused |
>
> ⚠ **The gate is still not closed.** What remains is the widget tree — ~236 records rebuilt every
> frame — and crab's own row/status scratch, and the allocator still has **no `free()`**, so 77,568 B
> per frame is still permanent growth. At 60 Hz it is **4.7 MB/s**, down from 45 MB/s. Step 3 (an
> arena hook in dhancha) is what closes it.
> ⚠ Step 2 also cost a **one-time 334,544 B**: the first frame still allocates the render target. It
> is paid once per surface for the life of the process, not once per frame.
> ⛔ Two claims this document made and got wrong are corrected below: step 1 was **not** "one line",
> and step 2 was **not** purely upstream.

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
frame delta bytes: 412112      <- frame 1 pays the one-time render target
frame delta bytes: 77568
frame delta bytes: 77568
frame delta bytes: 77568
frame delta bytes: 77568
```

**77,568 B per steady-state frame**, against **746,440 B** before dhancha 0.9.13. Where it goes now:

| item | bytes | note |
|---|---:|---|
| `dh_surface_new` pixel buffer | ~~334,440~~ **0** | ✅ **fixed, dhancha 0.9.13** — deferred to `dh_surface_pixels`, which nothing calls |
| `sd_surface_new` inside `dh_surface_render` | ~~334,432~~ **0 per frame** | ✅ **fixed, dhancha 0.9.14 + crab** — cached on the DhSurface, which crab now holds for the session. Still 334,432 B **once**. |
| ~236 widget records @ 248 B | 58,528 | **step 3, still open** — 75 % of what is left |
| crab's own row/status scratch | ~19,000 | `disp`, size strings, `crab_u2s` temporaries |

⚠ **The original figure in this document was 749,704 B and the reproduction measured 746,440 B.** The
3,264 B gap is fixture-dependent — entry-name lengths and the file/directory mix change how much row
and status scratch each frame allocates — not a change in the code. **The two pixel buffers reproduce
to the byte** (334,440 and 334,432), which is the part the conclusion rests on. The probe is
`alloc_used()` deltas around five back-to-back `crab_render` calls at 380x220, 114 entries per pane,
into **one** surface — the same lifetime `src/main.cyr` uses.

⛔ **The widget tree is now 75 % of the frame**, so step 3 is what the remaining work is.

**Before 0.9.13, 89 % of it was two full-size pixel buffers, and one of them was pure waste.**
`dh_surface_new` did `alloc(40)` then `alloc(w*h*4)` and stored the buffer at `DH_S_PIXELS` — but
`dh_surface_render` ignored it entirely and allocated its own `sd_surface_new(w, h)` to draw into.
Nothing ever touched the first buffer. **Both are now gone from the per-frame path**: the first is
allocated only if someone asks for it (0.9.13), the second is allocated once per DhSurface and reused
(0.9.14). Together they were the 89 %, and the measurement bears that out — 746,440 → 77,568 B.

## Why it has not bitten yet, and when it will

crab renders **on input**, not continuously. A navigation session of a few hundred keypresses costs a
few hundred megabytes and the process exits before anyone notices. Until 0.5.0 the event loop also
terminated itself after about two seconds of spinning, which capped the damage by accident.

Two roadmap items remove that accident:

- **M2's self-redraw** — the idle mascot line, transfer progress and index progress all repaint
  *without* input. At 60 Hz, 77,568 B/frame is **4.7 MB/s** — down from 45 MB/s, and still unbounded.
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
2. ✅ **DONE (dhancha 0.9.14 + crab) — `dh_surface_render` reuses its render target.** Measured
   412,040 → **77,568 B/frame**. The target is cached on the `DhSurface` (`DH_S_SDS` at +40; the
   struct grew 40 → 48 bytes) and reused whenever the dimensions still match.
   ⛔ **AND IT WAS NOT PURELY UPSTREAM, WHICH THIS DOCUMENT ASSERTED.** A cached target has to hang
   off something that outlives the frame, and `crab_render` was calling `dh_surface_new` **itself,
   once per call** — so a cache on the DhSurface would have been thrown away every frame and saved
   crab exactly nothing. The dhancha change is inert without the crab half. crab now creates **one
   surface for the session** in `src/main.cyr` and passes it in; `crab_render` takes `surf` and reads
   `w`/`h` back off it, so the signature got *shorter* (18 → 17 parameters) for gaining one.
   ⚠ **It is a real contract change**: `dh_surface_render` used to return a fresh surface every call
   and now returns the same one. A caller wanting two frames at once needs two `DhSurface`s —
   `src/render_test.cyr` is exactly that caller and now creates two, with a `check(sds2 == sds, 0)`
   guard so a future regression to a shared target cannot pass silently.
   ⚠ Resize abandons the old target rather than freeing it (there is nothing to free with), so a
   resize costs one screenful once. That is the right trade against one per frame.

3. **Give dhancha an arena hook.** The stdlib already ships `arena_new` / `arena_alloc` /
   `arena_reset` (`lib/alloc.cyr:370-540`) documented for exactly this *"per-frame / per-emission
   reuse"* pattern — `arena_reset` is a bump-back-to-base with no free list. A widget tree built into
   a per-frame arena and reset each frame costs nothing to reclaim.

crab's own share (~19 KB/frame) is fixable locally by hoisting the row and status scratch buffers out
of the render path. ⚠ **It was under 5 % of the problem and is now roughly a quarter of it** — the two
pixel buffers were so much larger that everything else rounded to noise. It is still second to the
widget tree (75 %), but it is no longer negligible.

## What a reader should take from this

- **Do not add a continuously-repainting element** — an animation, a progress bar, a clock — until
  the dhancha gate is closed. It will work in QEMU and exhaust memory on iron. ⚠ **Steps 1 and 2
  landing do not lift this.** 77,568 B/frame at 60 Hz is 4.7 MB/s, and the allocator still has no
  `free()`, so the growth is still unbounded — the rule was never about the factor. ⭐ What HAS
  changed is the margin: an element that repaints a few times a minute is now clearly affordable,
  where at 750 KB a frame it was not.
- **Do not "fix" this in crab** by caching the widget tree. Rebuilding it every frame is what makes
  the scroll-offset round-trip and the toolkit-owned selection work; the 0.4.10 port depends on it.
  ⚠ Note this is about the **widget tree**, not the surface — hoisting the *surface* out of the frame
  is exactly what step 2 did, and it is safe precisely because a surface carries no per-frame state.
- The gate is tracked in [`../development/roadmap.md`](../development/roadmap.md) M2 and blocks every
  milestone after it.
