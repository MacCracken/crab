# dhancha's drag API and dhancha's frame-arena API are mutually exclusive

**Status:** ✅ **FIXED upstream in dhancha 0.9.21 (2026-08-30)** — `dh_drag_progress` now refuses to begin a drag it cannot finish, and `dh_drag_available()` exposes the capability. ⛔ **This does NOT unblock crab's drag**: drag is synthesized inside `dh_dispatch`, and crab does not use `dh_dispatch` (operator ruling 2026-08-27, `src/app.cyr:215`). The remaining choice is crab-side — see the roadmap's M4 entry.
Not a crab defect and **not workaroundable in crab** without giving up the frame arena, which is the
thing four releases were spent building. crab is the first consumer of either API to want both.

**Date**: 2026-08-30
**From**: crab 0.7.1 (AGNOS file manager) — M4's *drag between panes*, which the roadmap lists as
gated on nothing because dhancha ships `DRAG_START`/`DRAG_MOVE`/`DRAG_DROP`/`DRAG_END` and
`dh_widget_set_draggable` / `dh_widget_set_drop_target`.
**Affects**: `dhancha/src/` → `dist/dhancha.cyr` — `dh_reset_input`, `dh_frame_begin`, and the drag
synthesis in the pointer path.
**Severity**: P1 for any app that uses both. Today that is crab and nothing else.

## Summary

A drag is inherently **multi-frame**: press, one or more moves, release. dhancha holds the drag
source as a raw widget pointer in `_dh_drag_src`, and synthesizes `DRAG_MOVE` / `DRAG_DROP` /
`DRAG_END` only while it is non-zero.

`dh_frame_begin` — which every arena-using app calls at the top of every render — does this:

```
fn dh_frame_begin(): i64 {
    var a = dh_frame_arena_get();
    if (a == 0) { return DHANCHA_OK; }
    arena_reset(a);
    dh_reset_input();          # <- clears _dh_focus, _dh_hover, _dh_press, _dh_drag_src
    return DHANCHA_OK;
}
```

⇒ **`_dh_drag_src` is zeroed at every frame boundary.** A drag that lasts longer than one frame —
i.e. every real drag — loses its source before the second pointer event arrives. `DRAG_START` can
fire; `DRAG_MOVE`, `DRAG_DROP` and `DRAG_END` cannot.

## ⛔ This is not a bug in `dh_reset_input`, and that is what makes it awkward

dhancha's own comment on `dh_frame_begin` is **correct** and should not be weakened:

> `_dh_focus`, `_dh_hover`, `_dh_press` and `_dh_drag_src` are raw widget pointers held across calls.
> After a reset they address memory the arena is about to hand out again … Resetting the arena
> without clearing them converts a memory saving into silent misbehaviour.

So the clear is load-bearing. The conflict is structural: **drag state is identity that must outlive
a frame, stored as a pointer that cannot.** Removing the clear would hand `DRAG_DROP` a dangling
widget from a recycled arena — worse than the current failure, because it would appear to work.

## Why crab cannot work around it

crab already owns its pointer state rather than routing through `dh_dispatch`, for a reason recorded
in `src/main.cyr`: *"The last-click cells are crab's OWN model — pane index and row index, never
widget pointers. A widget pointer would not survive `dh_frame_begin`'s arena rewind between the two
clicks of a double-click; a row index does."* crab hit the same wall at double-click and solved it by
not using widget identity at all.

⇒ crab **can** implement drag in its own (pane, row) model, reusing `crab_hit`. But that puts drag
semantics in the app — the exact "the rule lived in three apps" duplication that the LIST port,
`on-accent` and the column spec were all moved into dhancha to end.

⛔⛔ **AND THE TOOLKIT CANNOT OWN IT FOR crab EITHER, WHICH WAS MISSED WHEN THIS WAS FILED.** Drag is
synthesized inside `dh_dispatch`, and **crab does not use `dh_dispatch`** — operator ruling
2026-08-27 (`src/app.cyr:215`), taken for this same reason: `dh_dispatch` tracks a press as a widget
pointer, and crab rebuilds its tree every frame. So no dhancha drag fix, of any shape, delivers
events to crab. The original filing's "the toolkit should own this" was wrong on that point.

## What was done — dhancha 0.9.21

**Not** the payload API suggested below. That API is real and has **no consumer**: `dh_frame_begin`
is a no-op without an arena, so the defect reaches only apps using both — and dhancha's own position
(the 0.9.15 note) is that you pick one. Building it to serve nobody grows surface that cannot be
tested.

⇒ dhancha took the honest option instead: **`dh_drag_progress` refuses to begin a drag it cannot
finish.** Under a frame arena a press on a draggable widget is simply a click — no `DRAG_START`, and
`ACTIVATE` still synthesizes on release. `dh_drag_available()` exposes the capability up front.
Mutation-proven in `programs/event_test.cyr`.

⇒ **crab's remaining choice is crab-side**: implement drag in its own `(pane, row)` model — what it
already does for double-click and the wheel, working today with no dependency — or drop drag from
M4. See the roadmap's M4 entry.

## The options considered and declined (kept for the next reader)

1. **App-supplied opaque drag payload.** `dh_widget_set_drag_data(wgt, u64)` at build time;
   `dh_emit_drag` carries the u64 rather than the pointer. Survives a rewind by construction, and is
   what an app actually wants at the drop. ⚠ Declined for want of a consumer, not because it is
   wrong — if a future app uses `dh_dispatch` **and** an arena, this is the fix.
2. **Exempt drag state from `dh_reset_input` and re-resolve at the drop** by hit-testing the current
   tree. Loses the source's identity across a re-layout — fine when rows do not move mid-drag, wrong
   when the list scrolls during one.
3. **Split the reset.** Does not work as stated: `_dh_drag_src` is exactly what a rewind
   invalidates, so it needs option 1 underneath it anyway.
