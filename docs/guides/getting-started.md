# Getting started with crab

## Build

```sh
cyrius deps                              # resolve dependencies
cyrius build src/main.cyr build/crab    # compile
cyrius test                              # auto-discovers tests/*.tcyr — [build].test is NEVER run
```

## Layout

- `src/main.cyr` — entry point. Top-level `var r = main(); syscall(SYS_EXIT, r);`.
- `tests/crab.tcyr` — **the suite.** Every unit case goes here; `cyrius test` auto-discovers it.
  ⚠ `cyrius.cyml`'s `[build].test` now points at `tests/`, but **it is inert either way** — proven
  by pointing it at a file that exits 1 and watching `cyrius test` still return 0. Do not treat it
  as a gate. The gate is `cyrius test` itself, plus the steps in `.github/workflows/ci.yml`.
- `tests/crab.tcyr` — primary test suite (`cyrius test` auto-discovers).
- `tests/crab.bcyr` — benchmarks (`cyrius bench`).
- `tests/crab.fcyr` — fuzz harness (`cyrius fuzz`).

## Adding a feature

1. Edit `src/main.cyr` (or add a new module and `include` it).
2. Add a test case to `tests/crab.tcyr`.
3. Run `cyrius test`.
4. Bump `VERSION` and add a CHANGELOG entry before tagging.

See [`../adr/template.md`](../adr/template.md) when a non-trivial design choice deserves an ADR.
