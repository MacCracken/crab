# crab 🦀

**Sovereign Cyrius-native file manager for AGNOS.** A dual-pane GUI file browser
on the native desktop stack, with AI-native organization backed by the same
sovereign memory substrate the rest of the desktop rides.

Written in [Cyrius](https://github.com/MacCracken/cyrius).

## Why a crab

The hermit crab **carries its home** on its back and **reaches out** with its
claws — which is precisely what a file manager does: it carries your `$HOME` and
reaches out to grab, move, and keep your files. It's also a deliberate bit of
attitude:

- **Sea-arthropod parity on the Dolphin** — KDE went sea-*mammal*; AGNOS goes
  sea-*arthropod*. Same ocean, cheekier phylum.
- **Wearing Rust's own mascot** — 🦀 is Ferris, the Rust mascot. crab is the
  sovereign Cyrius replacement for the **Rust** file-manager interim (`yazi`).
  Naming the replacement after the replaced language's mascot is not a port —
  it's a trophy. *psst… what you gonna do.*

## Mascot

The crab has a name: **Bueller** 🦀 — Ferris (Rust's crab) → Ferris Bueller →
"Bueller." It implies Ferris without ever saying it, and hands the crab a deadpan
voice. The full rationale (don't flatten the layers) + the planned easter egg —
an idle-pane / `crab --about` deadpan roll-call, *"Bueller?… Bueller?…"* — live in
[`docs/development/mascot.md`](docs/development/mascot.md).

## Architecture (design target)

crab is another **view onto the sovereign memory layer**, not app-with-its-own-AI:

- **Dual-pane GUI** on the native desktop stack — `dhancha` (toolkit: widgets,
  flexbox layout, events) drawing via `sadish` (2D vector) + `rekha` (TrueType
  text), presented through `setu` to the `aethersafha` compositor. Same seam
  `puka` (the terminal) uses — crab is another resident, not new plumbing.
  > ⛔ **Corrected 2026-08-03 — "the seam puka PROVED" is withdrawn, and the TCP transport is
  > RETIRED.** Two separate things, kept separate:
  > - **The withdrawal.** setu's transport was TCP on loopback:7700. **Before `net_src_for`
  >   (agnos 1.56.34)** it could not complete a compositor↔client handshake on an ordinary boot —
  >   agnos stamped `net_ip` as the source of outbound segments, so the SYN-ACK returned on a
  >   4-tuple the client's own conn could not match. The only test that passed in that era ran
  >   under the since-deleted `AETHERSAFHA_SETU_SELFTEST` kernel hook (`net_ip = 0x7F000001`),
  >   which made src and dst agree by accident. Claims resting on that smoke are FALSE GREENS.
  > - **What DID happen, un-rigged.** After `net_src_for`, on 2026-08-02, the honest harness
  >   `agnos/scripts/harness/aethersafha-clients-test.py` — which byte-scans the kernel and
  >   hard-exits if it carries any selftest hook — reached **`connected: 2, presented: 2`**:
  >   **crab itself** and setu's `present_probe` (staged as `/bin/puka`) both connected and
  >   presented. ⚠ Scope: QEMU at `-smp 1` only; never shown on iron, and `-smp 4` fault-kills.
  >
  > The retirement is **architectural, not a failure verdict**: a local display protocol has
  > nothing to route, nothing to checksum, no window to negotiate, and no business owning a port.
  > Replacement is the agnos socket (`anu`), agnos `docs/development/planning/ipc.md` §9/§10.
- **AI-native organization via `daimon`'s vector store** — semantic file finding
  (RAG), duplicate detection, auto-tagging by content, predictive organization.
  This is the **same substrate `mneme` rides**: crab is *files-as-memory* to
  mneme's *notes-as-memory*. It reads the shared index; it does not silo its own.
- **Local-first, no external service** — the whole point of a sovereign store.

## Status

**Scaffold.** Roadmap **Priority 1 — ship before beta** (agnosticos
`docs/development/planning/roadmap.md` § File Manager). Retires the third-party
interim (`yazi` TUI + Thunar GTK) that was never a first-party project.

Planned scope: dual-pane GUI, thumbnail preview, batch rename, AI via the daimon
vector store.

## Build

```sh
cyrius deps                              # resolve stdlib deps
cyrius build src/main.cyr build/crab     # compile
cyrius test                              # run [build].test + tests/*.tcyr
```

## License

GPL-3.0-only
