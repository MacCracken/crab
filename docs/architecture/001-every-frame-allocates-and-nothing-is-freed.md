# 001 — Every frame allocates, and nothing is ever freed

**Measured**: 2026-08-26, against crab 0.5.0 and dhancha 0.9.12.

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
frame delta bytes: 749816
frame delta bytes: 749704
frame delta bytes: 749704
frame delta bytes: 749704
frame delta bytes: 749704
total over 5 frames: 3748752
```

**749,704 B per frame.** Where it goes:

| item | bytes | note |
|---|---:|---|
| `dh_surface_new` pixel buffer | 334,440 | ⛔ **never written, never read** — see below |
| `sd_surface_new` inside `dh_surface_render` | 334,432 | the real render target |
| ~236 widget records @ 248 B | 58,528 | root, panes, columns, headers, lists, 228 rows |
| crab's own row/status scratch | ~22,300 | `disp`, size strings, `crab_u2s` temporaries |

**89 % of it is two full-size pixel buffers, and one of them is pure waste.** `dh_surface_new`
(`lib/dhancha.cyr:2149`) does `alloc(40)` then `alloc(w*h*4)` and stores the buffer at
`DH_S_PIXELS` — but `dh_surface_render` (`:2433`) ignores it entirely and allocates its own
`sd_surface_new(w, h)` to draw into. Nothing ever touches the first buffer.

## Why it has not bitten yet, and when it will

crab renders **on input**, not continuously. A navigation session of a few hundred keypresses costs a
few hundred megabytes and the process exits before anyone notices. Until 0.5.0 the event loop also
terminated itself after about two seconds of spinning, which capped the damage by accident.

Two roadmap items remove that accident:

- **M2's self-redraw** — the idle mascot line, transfer progress and index progress all repaint
  *without* input. At 60 Hz, 749,704 B/frame is **45 MB/s**.
- **M4's transfer tray** — repaints continuously for the duration of a copy.

There is a second, smaller leak on the same footing: `dh_setu_poll_event` (`lib/dhancha.cyr:2611`)
calls `setu_msg_new()` **before** it knows whether anything is pending, so every idle poll leaks
~80 B. 0.5.0's stopgap is to sleep ~16 ms between empty polls, taking that from ~80 MB/s to ~5 KB/s.

## What the fix looks like, and where it lives

**It is upstream.** dhancha allocates through plain `alloc()` at 19 sites and offers no arena hook,
so crab cannot redirect the widget-tree or surface allocations no matter how it is written. Three
steps, roughly in value order:

1. **Stop allocating the unused buffer in `dh_surface_new`** — one line, halves crab's per-frame cost.
2. **Let `dh_surface_render` reuse a surface** rather than allocating a new `sd_surface` per call.
3. **Give dhancha an arena hook.** The stdlib already ships `arena_new` / `arena_alloc` /
   `arena_reset` (`lib/alloc.cyr:370-540`) documented for exactly this *"per-frame / per-emission
   reuse"* pattern — `arena_reset` is a bump-back-to-base with no free list. A widget tree built into
   a per-frame arena and reset each frame costs nothing to reclaim.

crab's own share (~22 KB/frame) is fixable locally by hoisting the row and status scratch buffers out
of the render path, but it is 3 % of the problem and doing it first would be optimising the wrong end.

## What a reader should take from this

- **Do not add a continuously-repainting element** — an animation, a progress bar, a clock — until
  the dhancha gate is closed. It will work in QEMU and exhaust memory on iron.
- **Do not "fix" this in crab** by caching the widget tree. Rebuilding it every frame is what makes
  the scroll-offset round-trip and the toolkit-owned selection work; the 0.4.10 port depends on it.
- The gate is tracked in [`../development/roadmap.md`](../development/roadmap.md) M2 and blocks every
  milestone after it.
