# Architecture notes

Non-obvious constraints, quirks, and invariants that a reader cannot derive from the code alone. Numbered chronologically — never renumber.

Not decisions (those live in [`../adr/`](../adr/)) and not guides (those live in [`../guides/`](../guides/)). An item here describes *how the world is*, not *what we chose* or *how to do something*.

## Items

| Note | Constraint |
|------|-----------|
| [001](001-every-frame-allocates-and-nothing-is-freed.md) | Every frame allocates ~750 KB and the allocator has no `free()` — this bounds what crab may repaint |
