# Architecture notes

Non-obvious constraints, quirks, and invariants that a reader cannot derive from the code alone. Numbered chronologically — never renumber.

Not decisions (those live in [`../adr/`](../adr/)) and not guides (those live in [`../guides/`](../guides/)). An item here describes *how the world is*, not *what we chose* or *how to do something*.

## Items

| Note | Constraint |
|------|-----------|
| [001](001-every-frame-allocates-and-nothing-is-freed.md) | The allocator still has no `free()`, but the per-frame bound is **CLOSED** (2026-08-27, dhancha 0.9.15 + crab's frame arena): a frame once cost ~750 KB and now costs the global heap **zero**. ⛔ Anything added to the render path must allocate through `dh_falloc`, not `alloc` |
