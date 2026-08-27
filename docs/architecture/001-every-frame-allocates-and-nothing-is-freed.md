# 001 — Every frame allocated, and nothing was ever freed

**Measured**: 2026-08-26 against crab 0.5.0 / dhancha 0.9.12.
**Closed**: 2026-08-27 against dhancha 0.9.15 + crab's frame arena.

> ⭐ **A rendered frame now costs the global heap ZERO bytes.** Measured at both 114 entries per pane
> (the real iron count for `/`) and 256 (the `CRAB_MAX_ENTRIES` ceiling).
>
> | | per steady-state frame |
> |---|---:|
> | baseline (dhancha 0.9.12) | 746,440 B |
> | + step 1 (dhancha 0.9.13) — `dh_surface_new`'s dead pixel buffer, deferred | 412,040 B |
> | + step 2 (dhancha 0.9.14 + crab) — the sadish render target, reused | 77,568 B |
> | + step 3 (dhancha 0.9.15 + crab) — the widget tree, arena'd | **0 B** |
>
> ⚠ **Zero is per-frame, not total.** crab pays a **one-time ~597 KB**: the 334,432 B render target
> plus the 262,144 B arena chunk, both allocated on the first frame and reused for the life of the
> process. That is the whole point — a fixed cost instead of a per-keypress one.
> ⛔ **This document has been wrong twice in ways worth keeping**: step 1 was not "one line", and step
> 2 was not purely upstream. Both corrections are below, with what made them wrong.

## The invariant — still true about the substrate

`lib/alloc.cyr` / `lib/alloc_agnos.cyr` is a **bump allocator with no `free()`**. `alloc()` moves a
pointer through a chain of mmap'd 2 MB chunks; old chunks are never unmapped. `alloc_reset()` exists
but rewinds the *whole* arena, which would invalidate the pane paths, the readdir buffers and the
compositor client struct — so crab cannot call it.

**Therefore: every byte a render touches with plain `alloc()` is retained for the life of the
process.** That has not changed and will not. What changed is that crab's render path no longer uses
plain `alloc()` — dhancha 0.9.15 lets a caller supply a per-frame arena, and `arena_reset` is a bump
back to base with no free list.

⛔ **The escape is opt-in and narrow.** dhancha routes exactly two things onto the arena — widget
records (`dh_widget_new`) and layout's measure scratch — because those are the only allocations that
die with the frame by construction. Surfaces, the setu client's shared buffer, event records and
queues, textinput buffers and canvas surfaces all outlive a frame and stay on the global allocator.
Routing `dh_surface_new` would free crab's session surface out from under it on the first rewind, and
the bump allocator would then hand that memory back out — corrupting silently rather than faulting.

## The number

A host probe takes `alloc_used()` deltas around back-to-back `crab_render` calls at 380×220, into one
surface and one arena — the lifetimes `src/main.cyr` uses:

```
frame delta bytes: 334544      <- frame 1 pays the one-time render target
frame delta bytes: 0
frame delta bytes: 0
frame delta bytes: 0
arena high-water bytes: 262144
```

Identical at 114 and at 256 entries per pane: the 256 KiB initial chunk absorbs both, so the arena
never even has to chain.

| item | before | now |
|---|---:|---:|
| `dh_surface_new` pixel buffer | 334,440 | **0** — deferred (0.9.13); nothing calls the accessor |
| `sd_surface_new` render target | 334,432 | **0/frame** — cached on the DhSurface (0.9.14); 334,432 B once |
| ~236 widget records @ 248 B | 58,528 | **0** — arena'd, rewound each frame (0.9.15) |
| crab's row/status scratch | ~19,000 | **0** — same arena, via `dh_falloc` |

⚠ **The original figure in this document was 749,704 B; the reproduction measured 746,440 B.** The
3,264 B gap is fixture-dependent — name lengths and the file/directory mix change how much row and
status scratch a frame allocates — not a change in the code. The two pixel buffers reproduced to the
byte, which is the part the conclusion rested on.

## What it took, and the two things this document got wrong

1. ✅ **Step 1 (dhancha 0.9.13) — stop allocating the buffer nothing reads.** `dh_surface_new`
   allocated `w*h*4` bytes that `dh_surface_render` ignored in favour of its own `sd_surface_new`.
   Nothing in the stack ever wrote or read it.
   ⛔ **It was NOT "one line", which this document asserted twice.** Storing 0 breaks dhancha's
   `programs/event_test.cyr`: `dh_surface_present` returns `_NO_SURFACE` when pixels are 0 and
   `_UNSUPPORTED` otherwise, so a permanently-zero field silently downgrades 0.9.5's "refuse loudly
   and diagnosably" contract to "bad surface". The shipped fix **defers** the allocation into
   `dh_surface_pixels` so the published accessor's contract is unchanged for any external holder.
   ⚠ A "one-line upstream fix" that a test in the upstream repo rejects is the shape of estimate made
   from reading the caller and not the callee's suite.

2. ✅ **Step 2 (dhancha 0.9.14 + crab) — reuse the render target.** Cached on the `DhSurface`
   (`DH_S_SDS` at +40; the struct grew 40 → 48 B).
   ⛔ **It was NOT purely upstream, which this document asserted.** A cached target must outlive the
   frame, and `crab_render` was calling `dh_surface_new` **itself, once per call** — so a
   per-DhSurface cache would have been discarded every frame and saved crab nothing. crab now holds
   one surface for the session and takes it as a parameter, reading `w`/`h` back off it.
   ⚠ **It is a contract change**: `dh_surface_render` may return the same surface twice. A caller
   wanting two frames at once needs two `DhSurface`s — `src/render_test.cyr` is one, and guards it
   with `check(sds2 == sds, 0)`.

3. ✅ **Step 3 (dhancha 0.9.15 + crab) — arena the widget tree.** `dh_frame_arena_set` installs it,
   `dh_falloc` draws from it, `dh_frame_begin` rewinds it. crab installs a **growable** arena in
   `src/main.cyr` and `crab_render` calls `dh_frame_begin` at the top, so every caller is correct
   without remembering.
   ⭐ **Growable is load-bearing.** `arena_reset` on a GROW arena rewinds to the first chunk and
   **keeps the chain**, so the loop converges on its high-water mark and then allocates nothing at
   all. A fixed arena would have to guess a capacity, and guessing low means `dh_falloc` spilling
   back to the global heap — the leak, quietly restored.
   ⛔ **`dh_frame_begin` does two things and they cannot be separated.** `_dh_focus`, `_dh_hover`,
   `_dh_press` and `_dh_drag_src` are raw widget pointers dhancha holds across calls; after a rewind
   they address memory the arena is about to hand out again. Resetting without clearing them turns a
   memory saving into silent misbehaviour. **Never call `arena_reset` on a frame arena directly.**
   ⚠ It follows that **an app using a frame arena must re-establish focus every frame** — crab does,
   in `crab_pane`. Cross-frame widget identity and a per-frame arena are mutually exclusive by
   construction, not by policy.
   ⚠ **crab's own scratch had to move too, and not as an optimisation.** `dh_widget_set_text` stores
   the pointer and does not copy, so a row's display string and the status line's buffer must share
   the widget's lifetime. Leaving those on the global allocator while the widgets moved would have
   been the one lifetime mismatch this design can produce.

## What a reader should take from this

- ⭐ **The "do not add a continuously-repainting element" rule is LIFTED for the frame itself.** It
  stood for three releases and it was right: at 746,440 B a frame, 60 Hz was 45 MB/s into an
  allocator with no `free()`. A rendered frame now costs zero. The idle mascot line (deferral #29),
  a transfer tray (M4) and index progress (M7) are no longer blocked by *this*.
- ⛔ **But a NEW gate takes its place, and it is not hypothetical.** `dh_setu_poll_event` calls
  `setu_msg_new()` **before** it knows whether anything is pending, so every idle poll allocates —
  ~80 B, on the global heap, never reclaimed. A continuously-repainting element implies continuous
  polling, so at 60 Hz that is ~4.8 KB/s of permanent growth. Slower than what it replaces by four
  orders of magnitude, and still unbounded. **Gate: dhancha** — hoist the buffer or accept a
  caller-owned one. Roadmap M2, *deferral #09*.
- ⚠ **Anything added to the render path must allocate through `dh_falloc`, not `alloc`.** A single
  plain `alloc()` in `crab_render` reintroduces a per-frame leak, and the suite will say so:
  `tests/crab.tcyr` asserts twenty rendered frames move the global heap by exactly 0 bytes.
- **Do not "fix" the widget tree by caching it.** Rebuilding it every frame is what makes the
  scroll-offset round-trip and the toolkit-owned selection work; the 0.4.10 port depends on it. The
  arena is what makes rebuilding free — the tree is still rebuilt.
- ⚠ **`lib/` is not what compiles.** Measured 2026-08-27: appending garbage to crab's
  `lib/dhancha.cyr` leaves the build green, while appending it to `../dhancha/dist/dhancha.cyr` fails
  it — the `path` override compiles the sibling's `dist/` directly. Appending garbage to
  `lib/alloc.cyr` is also harmless: the stdlib comes from the installed toolchain. ⇒ **The stdlib's
  arena internals cannot be mutation-tested from this repo**, so the arena assertions here pin the
  observable contract (chain grew, global heap flat, chain stable) rather than the implementation.
