# Getting started with crab

## Build

```sh
cyrius deps                              # resolve dependencies
cyrius build src/main.cyr build/crab    # compile
cyrius test                              # auto-discovers tests/*.tcyr — [build].test is NEVER run
```

## Layout

- `src/main.cyr` — entry point. Top-level `var r = main(); syscall(SYS_EXIT, r);`.
- `src/test.cyr` — the INERT `[build].test` hook (`cyrius.cyml`). ⛔ **Never add a case here.**
  `cyrius test` auto-discovers `tests/*.tcyr` and does not compile this file at all, so a case
  written here never runs while the suite's pass count still reads green. See the file's own
  header for what that cost once. Every unit case goes in `tests/crab.tcyr`.
- `tests/crab.tcyr` — primary test suite (`cyrius test` auto-discovers).
- `tests/crab.bcyr` — benchmarks (`cyrius bench`).
- `tests/crab.fcyr` — fuzz harness (`cyrius fuzz`).

## Adding a feature

1. Edit `src/main.cyr` (or add a new module and `include` it).
2. Add a test case to `tests/crab.tcyr`.
3. Run `cyrius test`.
4. Bump `VERSION` and add a CHANGELOG entry before tagging.

See [`../adr/template.md`](../adr/template.md) when a non-trivial design choice deserves an ADR.
