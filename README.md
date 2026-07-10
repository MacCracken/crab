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

## Architecture (design target)

crab is another **view onto the sovereign memory layer**, not app-with-its-own-AI:

- **Dual-pane GUI** on the native desktop stack — `dhancha` (toolkit: widgets,
  flexbox layout, events) drawing via `sadish` (2D vector) + `rekha` (TrueType
  text), presented through `setu` to the `aethersafha` compositor. Same seam
  `puka` (the terminal) proved — crab is another resident, not new plumbing.
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
