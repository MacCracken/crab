# crab — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures
> (durable); this file is **state** (volatile).
>
> ⛔ **This file described an empty scaffold ("0.1.0, no releases yet, initial scaffold only") until
> 2026-08-02, five releases after that stopped being true.** A state file that is wrong is worse than a
> missing one, because it is what a cold start reads first.

## Version

**0.4.4** (2026-08-02) — see [`../../CHANGELOG.md`](../../CHANGELOG.md).

## Toolchain

- **Cyrius pin**: `6.5.5` (in `cyrius.cyml [package].cyrius`)
- ⚠ The pin is documentation, not enforcement — `cyrius build` compiles with the **installed** `cycc`,
  warns `toolchain drift`, and carries on.

## Source

- `src/main.cyr` — entry, setu client lifecycle, event loop.
- `src/ui.cyr` — dual-pane file browser (215 lines): pane model, directory listing, selection, status line.
- `src/render_test.cyr`, `src/test.cyr` — in-tree test entries.

## Proven

⛔ **The transport every agnos claim rested on is RETIRED (2026-08-03) — as the WRONG PRIMITIVE for
local display IPC, not as something that never worked.** A local display protocol has nothing to route,
nothing to checksum, no window to negotiate, and no business owning a port. The replacement is the agnos
socket (`anu`), agnos `docs/development/planning/ipc.md` §9/§10. crab's agnos standing must be
re-established on `anu` before anything is called proven again.

⛔ **Retracted — the "proven on agnos" lineage before 2026-08-02 is a FALSE GREEN.** The
setu-descend / setu-stat / readdir end-to-end observations all ran on the image staged by the deleted
`aethersafha-setu-smoke.sh`, which built the kernel with `AETHERSAFHA_SETU_SELFTEST=1` — that hook
assigned `net_ip = 0x7F000001`, the only reason the loopback handshake closed in that era. Before
`net_src_for` (agnos 1.56.34) an ordinary boot could not complete it.

✅ **Real, un-rigged, and NOT retracted** (2026-08-02, QEMU `-smp 1`, agnos 1.56.34+): with
`net_src_for` in the kernel, the honest harness `agnos/scripts/harness/aethersafha-clients-test.py` —
which byte-scans `build/agnos` and hard-exits if the kernel carries any selftest hook — reached
**`connected: 2, presented: 2`**: **crab** and setu's `present_probe` (`/bin/puka`) both connected and
presented. That harness is what CAUGHT the earlier rigging. crab was also observed as a composited
window launched in the foreground (`aethersafha`, no `&`) from the agnsh prompt, showing real ext2
directory contents on the framebuffer. ⚠ Scope: QEMU at `-smp 1` only — never shown on iron, and
`-smp 4` fault-kills. It rode the now-retired TCP path, so it does not carry forward as a *current*
capability claim on `anu` — but it is evidence that the retirement was architectural, not a
failure verdict.

⚠ Never run on iron.

## Dependencies

Declared in `cyrius.cyml` — sadish 0.5.1 · rupa 0.1.2 · rekha 0.3.4 · kashi 1.0.4 · dhancha 0.9.3 ·
setu 0.7.2, plus the stdlib set (incl. `net`, which setu's ⛔ retired TCP transport needs — the `net`
dep goes away with it when crab moves to the agnos socket).

⚠ **crab consumes setu's `dist/` bundle, not its `src/`.** A setu fix only reaches this client after
`cyrius distlib` runs in setu — a local `path` override alone is not enough.

## Tests

- `tests/crab.tcyr` — primary suite (passes on `cyrius test`)
- `tests/crab.bcyr` — benchmark stub · `tests/crab.fcyr` — fuzz stub

## Consumers

_None — top-level application._

## Next

See [`roadmap.md`](roadmap.md). Nearest: crab is the stack's only setu client already alpha-255 clean
throughout, making it the natural first consumer of the compositor's premultiplied `#92` blend path —
which has never run on any target.
