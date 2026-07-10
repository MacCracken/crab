# crab — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures
> (durable); this file is **state** (volatile).

## Version

**0.1.0** — scaffolded 2026-07-10 via `cyrius init`. No releases yet.

## Toolchain

- **Cyrius pin**: `6.4.34` (in `cyrius.cyml [package].cyrius`)

## Source

Initial scaffold only.

## Tests

- `tests/crab.tcyr` — primary suite (smoke + math; passes on `cyrius test`)
- `tests/crab.bcyr` — benchmark stub (no-op)
- `tests/crab.fcyr` — fuzz stub

## Dependencies

Direct (declared in `cyrius.cyml`):

- stdlib — string, fmt, alloc, io, vec, str, syscalls, assert

## Consumers

_None yet._

## Next

See [`roadmap.md`](roadmap.md).
