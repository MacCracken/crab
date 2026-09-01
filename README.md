# crab 🦀

**Sovereign Cyrius-native file manager for AGNOS.** A dual-pane GUI file browser
on the native desktop stack.

⚠ The AI-native organization crab is *designed* around — semantic find, auto-tag and dedup on the
same sovereign memory substrate the rest of the desktop rides — is a **design target and is not
shipped**. Nothing in this README's § Architecture is a capability claim; § Status is.

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
  flexbox layout, events) drawing via `sadish` (2D vector), presented through
  `setu` to the `aethersafha` compositor. Same seam `puka` (the terminal) uses —
  crab is another resident, not new plumbing.
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
  > The replacement is the agnos **channel band** — syscall **`#97 chan_op`**, `VFS_CHAN = 11`,
  > kernel prefix `chan_*` (agnos `docs/development/planning/ipc.md` §9/§10). crab dials nothing:
  > the compositor mints a channel, endows one end and spawns crab already connected, and
  > `setu_connect` reads `AGNOS_CHAN`.
  > ⛔ **This paragraph said "the agnos socket (`anu`)" until 2026-08-26. THE CODENAME IS RETIRED** —
  > operator ruling 2026-08-05, *"a name is a distribution fact"*: a band that only ever appears as a
  > prefix inside one kernel has no repo boundary to cross. **`anu` resolves to nothing in the
  > kernel**, so a reader who searched for it found nothing and had no way to tell whether they were
  > misreading the code or the document.

  ⛔ **And separately, corrected 2026-08-26 — the text crab draws is `kashi`'s CP437 8×16 BITMAP
  font, not `rekha` TrueType.** The bullet above named rekha as something crab drew with; it never
  did. crab passes `font = 0` to `dh_draw_text`, which is dhancha's kashi bitmap path, and calls
  **no `rekha_*` function anywhere**. ⚠ rekha remains a declared dependency (dhancha's `dist` bundle
  references its symbols), and proportional text is a real roadmap item — **M5, gated on rekha plus
  dhancha font plumbing**. It is not what ships.
- **AI-native organization via `daimon`'s vector store** — semantic file finding
  (RAG), duplicate detection, auto-tagging by content, predictive organization.
  This is the **same substrate `mneme` rides**: crab is *files-as-memory* to
  mneme's *notes-as-memory*. It reads the shared index; it does not silo its own.
- **Local-first, no external service** — the whole point of a sovereign store.

## Status

**Shipping. It browses, it writes, and it previews.**

⛔⛔ **THIS SECTION HAS NOW BEEN WRONG IN BOTH DIRECTIONS, AND BOTH TIMES FOR THE SAME REASON: it
was written once, correctly, and never re-read at a cut.** It said **"Scaffold."** until 2026-08-26 —
false since 0.2.0, fifteen releases earlier. It was then rewritten to **"Shipping, and read-only"**,
which was true that day and was falsified by M4 five releases later, while still quoting a retired
256-entry cap and a stat figure the cap bump had quadrupled. Both are called out rather than quietly
overwritten, because a status line that under-claims misleads exactly as much as one that over-claims.
⇒ **Re-read this section at every release.** Nothing gates it — the same absence that let
`state.md` rot twice.

⭐ **Version, binary sizes, test counts and dependency versions are deliberately NOT here** — inlined
state rots, and this repo has watched it happen twice. They live in
[`docs/development/state.md`](docs/development/state.md), refreshed every release. **Picking the work
up cold? Read [`docs/development/handoff.md`](docs/development/handoff.md) first** — what is verified
versus merely built, the open items, and the hazards.

**What works today**: two panes over real `readdir` + `stat` — the stat sweep is **deferred off the
keystroke path** and fills in from the idle loop, so descending costs no syscalls (it cost ~1.1 ms per
entry under QEMU, so ~1.1 s at the 1024-entry cap it would otherwise pay on a keypress); keyboard navigation — switch pane,
move the selection, Enter to descend, Backspace to ascend — on a scrolling `dh_list`, plus **pointer
input, a mouse wheel, held-key repeat and compositor resize** (M2). `s` cycles four **sort** modes
(name · size · modified · kind, directories always first) across both panes, and the selection
**follows** you: Backspace lands on the directory you just left, and re-sorting keeps you on the same
entry. Panes start at `/bin` and `/` unless you name them: `crab [LEFT] [RIGHT]`. Rows are
**real columns with headers** — NAME · SIZE · MODIFIED — sharing one width spec so the header and
every row line up. NAME is the **remainder** column, so it grows with the pane; a truncated name is
still marked with `~`. ⚠ **Columns that do not fit are dropped, not squeezed**, and below 600 px the
second pane leaves entirely in favour of an A/B switcher — both thresholds derived from what a pane
can honestly show rather than copied from a mockup. The **status line** carries the active
selection's name, size and modified date. ⛔ **Dotfiles are never hidden**, deliberately: hiding them
is only safe where there is a way to reveal them, and crab has no such affordance. crab connects to the
`aethersafha` compositor through `setu` and
presents on a real agnos kernel, observed under QEMU at `-smp 4`.

⚠ **crab has also run on iron, and both times it found a defect QEMU never reproduced** — an orphaned
window holding one of only sixteen system-wide GPU buffer slots, and a directory listing that dropped
82 of 114 entries in silence. QEMU is not a control for timing- or pressure-dependent behaviour.

**Writing and acting on files** (M4): copy, move, **recursive** copy and delete, rename, batch
rename, mkdir, and open. Multi-select with Space, a context menu that is a *discovery* surface rather
than a second set of verbs — every entry maps to a key binding that already exists — and a transfer
tray with a progress bar, a rate and an ETA. ⛔ **Long transfers are stepped off the idle tick and
Esc cancels them**, so a recursive copy never freezes the window.

**Two views** (M5): `g` toggles both panes between the list and a **grid** of names — same
information a list row's NAME column carries, three columns instead of one. ⚠ In grid mode the arrow
keys navigate the grid and `h`/`l` switch panes; in list mode nothing changes.

**The preview column** (M5): `p` opens a right-hand inspector for the selected entry — NAME · KIND ·
SIZE · MODIFIED · DIMENSIONS · CAMERA · SHOT, and a **thumbnail** for PNG, JPEG, GIF and BMP.
Camera and shutter time come from a JPEG's EXIF, in either byte order. ⭐ Dimensions are read
straight out of the file header with no decoder at all; thumbnails go through `chitra` and are
**decoded off the idle tick**, so neither ever lands on the arrow-key path. ⚠ **The width rule is
derived, not drawn**: the preview may not cost a pane its SIZE column, so it needs a window of
303 px and refuses out loud below that.
⛔ **Thumbnails are budgeted, and the budgets are the feature.** The decoder's memory is never
reclaimed on this stack, so crab refuses an image larger than ~1024×1024 *from its header* — before
allocating — and stops decoding altogether once a session has spent 32 MB. It says which of those
happened rather than showing a blank square.

**What does not**:

- **Thumbnails stop at about 1024×1024**, and larger images say so instead of showing one. That is
  not a rendering limit — it is the decoder's memory, which this stack cannot reclaim. Recorded with
  its measurements in [`docs/development/roadmap.md`](docs/development/roadmap.md).
- **No grid, columns or gallery view, and no sidebar.** Genuinely gated on `dhancha` widgets that do
  not exist yet. (Roadmap M5–M6.)
- **No column sorting from the headers.** The headers are labels, not buttons — `s` cycles the sort
  mode. Clicking a header does nothing, deliberately: it is not wired, rather than wired and silent.
- **A pane shows at most 1024 entries**, a compile-time ceiling. Beyond it the listing is truncated —
  but crab says by how much (`showing 1024 of 1200`), because agnos's resumable `readdir_at` lets it
  keep counting past its own buffer.
- **Text is a bitmap font, not proportional.** crab draws with `kashi`'s CP437 8x16 glyphs; the
  `rekha` TrueType path is not wired. (Roadmap M5.)
- **None of the AI arc exists.** No index, no tags, no semantic find, no dedup — and `cyrius.cyml`
  declares no daimon dependency. (Roadmap M7–M8.)

Roadmap **Priority 1 — ship before beta** (agnosticos
`docs/development/planning/roadmap.md` § File Manager). Retires the third-party
interim (`yazi` TUI + Thunar GTK) that was never a first-party project. The eight milestones from
here to 1.0 are sequenced in [`docs/development/roadmap.md`](docs/development/roadmap.md).

## Build

```sh
cyrius deps                                          # resolve stdlib + sibling deps
cyrius build src/main.cyr build/crab                 # compile for the host
cyrius build --agnos src/main.cyr build/crab_agnos   # for AGNOS — the real target
cyrius test                                          # run [build].test + tests/*.tcyr
```

```sh
crab /usr/local /home            # start the panes where you want them; defaults are /bin and /
```

⚠ **A start path must be absolute and must fit `CRAB_PATH_MAX`.** One that is not is **refused and
said so on the console**, and the default is used — it is never silently substituted, because an
empty pane with no message reads as a broken filesystem rather than a rejected argument.
⚠ **On the AGNOS desktop the defaults are what you get**: the compositor spawns crab through the
launcher with a path and no arguments.

⚠ **The host build is the convenience target, not the product.** Every `#ifdef CYRIUS_TARGET_AGNOS`
region — the whole event loop included — is uncompiled without `--agnos`, and no host test can see
it.

## License

GPL-3.0-only
