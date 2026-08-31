# dhancha `dh_surface_render` retains a full-resolution pixel buffer per distinct window extent

**Status:** 🟠 **OPEN — upstream (dhancha), verified from crab 0.8.0-dev against dhancha 0.9.20.**
Not a crab defect and **not fixable in crab**: crab must honour resize, and the retention is on the
toolkit side of `dh_surface_render`. Filed here because **crab is the client that reaches it** and
because crab's own resize path is what makes it live.

**Date**: 2026-08-30
**From**: crab (the AGNOS file manager) — first dhancha client that both accepts `WINDOW_CONFIGURE`
and runs at arbitrary compositor-chosen extents.
**Affects**: `dhancha/src/` → `dist/dhancha.cyr`, `dh_surface_render` / `dh_surface_resize`.
**Severity**: P1. Bounded and non-corrupting, but ~14.7 MB per full-screen extent on a bump
allocator with no `free()`.
**Related**: dhancha 0.9.13–0.9.16 (the allocation-gate arc this is the residue of);
crab `docs/development/handoff.md` (the zero-bytes-per-frame gate, which this does **not** break).

## Summary

`dh_surface_render` caches its sadish render target in `DH_S_SDS` and reuses it while the extent is
unchanged — this is the 0.9.14 repair that took crab's per-frame cost from 412,040 B to 77,568 B.
The reuse check is a dimension compare:

```
var sds = load64(surf + DH_S_SDS);
if (sds != 0) { if (sd_surface_width(sds)  != w) { sds = 0; } }
if (sds != 0) { if (sd_surface_height(sds) != h) { sds = 0; } }
if (sds == 0) {
    sds = sd_surface_new(w, h);        # <- the previous buffer is dropped here, never freed
    if (sds == 0) { return 0; }
    store64(surf + DH_S_SDS, sds);
}
```

On an extent change the cached pointer is **overwritten**, and `alloc` is a bump allocator with no
`free()`. So the old buffer is retained for the life of the process. `dh_surface_resize` sets
`DH_S_PIXELS` to 0 for the same reason (a stale-sized cache is an overflow) and is correct in
itself; the retention is purely in the render path's re-allocation.

⇒ **Every distinct window extent the surface is ever rendered at permanently retains one
full-resolution BGRA buffer.**

## Cost

| extent | bytes retained |
|---|---:|
| 380×220 (crab's compile-time default) | 334,400 |
| 1280×800 | 4,096,000 |
| 2560×1440 (the 2026-08-30 iron display) | **14,745,600** |

A maximize / restore / maximize cycle retains **~30 MB**, because the restore does not recover the
buffer it had before — that one was already dropped on the way up.

## Why it has not bitten yet

⛔ **Only because no `WINDOW_CONFIGURE` has ever reached crab.** aethersafha emits `SETU_CONFIGURE`
from exactly two call sites — F5 maximize (`src/input.cyr`, `win_notify_resize`) and a pointer drag
on a resize grip. The 2026-08-30 iron burn traced HID usages 43/81/79/40/82/41; F5 is 62 and no
pointer event reached crab at all, so the window stayed 380×220 for the whole run and the surface
was rendered at exactly one extent.

⚠ **crab will accept the resize when it is offered.** `crab_resize_wanted` admits 2560×1440
(14,745,600 B < `CRAB_SURFACE_BYTES_MAX` 33,554,432; both axes < `CRAB_DIM_MAX` 4096), so any F5 at
a large extent is the trigger — reachable under QEMU, no hardware required.

## What this is NOT

⭐ **It does not reopen the zero-allocation gate.** That gate is bytes-per-*steady-state*-frame, and
this allocates only when the extent CHANGES — never per frame at a fixed size. crab's four-release
arc to 0 B/frame stands. This is a one-time-per-extent cost, not a per-frame one.

⚠ **crab's own shm side is already correct** and is not part of this: `src/main.cyr` creates the new
setu buffer before closing the old one on resize, which is the ordering that avoids handing the
compositor a dead fd.

## Suggested fix (dhancha's call)

Either is sufficient; the second is cheaper and needs no allocator change.

1. **Free or reuse the old target on extent change.** Needs a real `free` for sadish surfaces, which
   the bump allocator does not have — so this likely means an explicit surface pool rather than
   `alloc`.
2. ⭐ **Do not re-allocate for an extent already seen.** Keep a small ring of (w, h, sds) entries —
   two or three covers restore/maximize/restore, which is the whole realistic pattern — and hand
   back the matching one. Bounded, no allocator work, and it makes the common cycle allocate twice
   total instead of once per transition.

⚠ **Whatever is chosen, `sd_clear` must keep filling every pixel** — it is what makes a reused
target pixel-identical to a fresh one, and the reuse in the current code already depends on it.
