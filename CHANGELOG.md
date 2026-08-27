# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.5.0] - 2026-08-26 — a P-1 sweep, and a roadmap that finally exists

Four things, all of them structural: a P-1 audit of the whole codebase with its repairs, the
deferral tail bubbled out of comment prose into a roadmap, the design canvas turned into a sequenced
plan to 1.0, and the two decisions that plan rests on written down as ADRs.

⚠ **No new user-visible features.** 0.5.0 is the release that stops building on a floor with holes in
it. What it buys is that M2 onward can be built without re-discovering these.

### Fixed — P1: the path helpers had no bounds, and ordinary navigation overflowed the heap

`crab_strcpy` and `crab_join` took **no destination size and performed no length check of any kind**,
while every destination was a fixed `alloc(256)`. Descent depth is unbounded, so a pane path grew by
`1 + strlen(name)` on every Enter with nothing anywhere that stopped it.

⛔ **It needed no hostile input — only Enter.** The bump allocator hands out adjacent blocks in call
order, so the overrun landed on identifiable live objects:

- `lpath + 256` **is** `rpath` — an overlong LEFT path silently rewrote the RIGHT pane's path string,
  which the right pane then readdir'd. The operator sees one directory and gets another.
- `rpath + 256` **is** `lbuf` — an overlong RIGHT path overwrote the other pane's 64-byte readdir
  records **including their type bytes**, so entries changed name and flipped between file and
  directory.
- past `pathscr + 256` lie `statbuf` and then the `SetuClient` struct — the compositor fd and surface
  id.

Every write was a `store8`. There is no bounds checking on `store8`. All of it was silent.

⚠ **Listing alone reached it, not just descending.** `crab_stat_all` joins path + `/` + name into the
same 256-byte scratch **once per entry**, so a deep-but-legal directory overflowed on a plain
readdir. Reproduced: a 252-char path joined with a 62-char name wrote **315 bytes** into a 256-byte
buffer.

⇒ `crab_strcpy_n` / `crab_join_n` take a capacity, stop at `cap - 1`, always NUL-terminate **inside**
the allocation even when truncating, and return −1 when the source did not fit. The unbounded
originals are **deleted** — there is no primitive left to misuse. `CRAB_PATH_MAX` sizes
`lpath`/`rpath`/`pathscr` and bounds the helpers, the same one-constant-derived-everywhere discipline
the readdir cap already had.

⭐ **The return value is the fix, not the truncation.** A truncated path is a *different* path.
`crab_stat_all` now reports the entry as unstattable rather than statting a truncated path and
showing another file's size against this row; `crab_descend` refuses and says so rather than
readdir'ing somewhere the operator did not ask to go.

### Fixed — P1: the event loop was a spin count, and it ended sessions after about two seconds

`while (frame < 2000000)` incremented once per **poll**, and nothing in the body blocks:
`dh_client_poll_event` returns immediately on an idle channel and `sys_sched_yield` returns at once.
So `frame` counted neither frames presented nor seconds nor user actions — it counted how fast the
CPU could spin. At roughly a microsecond an iteration, **crab closed its own window after about two
seconds**, mid-session, while focused and in use. The exit path printed nothing, so from the
operator's side it was indistinguishable from a crash. It also burned one core at 100 % for its whole
short life.

The loop now ends on the two things that actually mean stop — EOF and `WINDOW_CLOSE`. Both were
already handled; neither was allowed to be the reason the loop ended.

⛔ **And the sleep is load-bearing, not politeness.** `dh_setu_poll_event` calls `setu_msg_new()`
*before* it knows whether anything is pending, so every idle poll leaks ~80 B into an allocator with
no `free`. Removing the frame cap without slowing the poll would have turned a bounded 152 MB leak
into an **unbounded** one — roughly 80 MB/s of idle growth. Sleeping ~16 ms between **empty** polls
takes that to ~5 KB/s; a burst of events still drains at full speed. The upstream fix is a dhancha
gate (roadmap M2).

⚠ Not `dh_client_next_event`, which blocks: crab must be able to redraw without input for the idle
mascot line, transfer progress and index progress. A blocking read forecloses all three.

### Fixed — the window is no longer repainted after the compositor destroys it

The render + 334 KB shm write at the bottom of the event branch ran **unconditionally**, including on
the `WINDOW_CLOSE` and EOF paths — drawing one last frame into a surface that had just been
destroyed, and leaking a full render to do it.

### Fixed — the size ladder rounded wrong, stopped at M, and overflowed

Three defects in `crab_size_str`, each reproduced with a compiled probe against the real function:

| input | was | now |
|---|---|---|
| `1048575` | `1024K` | **`1M`** |
| `1073741824` | `1024M` | **`1G`** |
| `107374182400` | `102400M` | **`100G`** |
| `i64` max | **`K`** — a bare unit, no digits | **`8388608T`** |
| `-1` | `""` (empty) | **`-`** |

The first is a comparison hazard in the one application whose job is comparing files: a size that
reads `1024K` when the next byte reads `1M`. The last two share a cause — `(val + 512)` **overflows**
near `i64` max and wraps negative, the ladder exits on its first pass, and `crab_u2s` writes nothing
at all for a negative. `st_size` is a `u64` read straight out of a kernel stat buffer, so a top-bit-set
value is a corrupt-filesystem question, not an impossible one. Computing quotient and remainder
separately cannot overflow at any input. `crab_u2s` now renders `-` rather than the empty string,
which is what let a bare unit letter through.

⚠ An unstattable file now renders `-` in the **pane** as well as the status line. It used to be blank
in one and a dash in the other — one file, two renderings.

### Fixed — a truncated name now says it was truncated

The name column is 13 characters and names run to 62, so `A001_0812_R1.dng` and `A001_0812_R8.dng` —
the design canvas's own fixture set — rendered as one identical row with no indication either was
cut. An operator selects, descends and acts on rows. Truncated names now end in `~`.
⚠ `~`, not `…`: the kashi system font is CP437 8×16 and has no ellipsis glyph.

### Fixed — the date formatters clamp instead of writing an embedded NUL

`crab_pad2`/`crab_pad4` write a fixed width, so an out-of-range value could not overflow — but
`48 + t` for `t` outside `[0,9]` emits a non-digit, and for a large enough value emits **0**, an
embedded NUL that truncates the whole status line at that point. The mtime they format is
`load64(statbuf + STAT_MTIME)` — kernel data crab does not control.

### Changed — `src/path.cyr`, so the P1 repair can be tested at all

⛔ **`tests/crab.tcyr` includes `src/ui.cyr`, never `src/main.cyr`** — `main.cyr` ends in `_entry()`,
so including it would *run the app*. Nothing defined in `main.cyr` was reachable from the suite. That
was tolerable while it held transport glue; it stopped being tolerable when the sweep found a heap
overflow in exactly the two functions living there. **A memory-safety repair that cannot be asserted
on is a repair held on trust.**

The record layout and the bounded helpers now live in `src/path.cyr`, which `ui.cyr` includes — so
`main.cyr`, `render_test.cyr` and the suite all see one declaration.

### Changed — named constants where the code had duplicated literals

- `STAT_BUFSZ` / `STAT_SIZE` / `STAT_MTIME` replace `alloc(48)` and the bare offsets `16` and `40`.
  `STAT_BUFSZ` is 48 on agnos and **144** on the Linux/Windows peers, and the allocation sits outside
  the agnos `#ifdef` while the reads sit inside it — so the buffer is now sized by whichever peer the
  build actually included.
- `CRAB_REC_SZ` / `CRAB_REC_TYPE` replace the literals `64` and `63` at six sites across two files.
  ⚠ These describe the **#81 syscall's** record geometry, which crab reports and does not choose;
  naming them is about having one declaration, not about crab owning the layout.
- `crab_readdir_into` clamps the returned count to `CRAB_MAX_ENTRIES`. ⚠ Defence in depth, not a
  known defect — the syscall contract already bounds it. The clamp is there because `n` is a loop
  bound over a buffer sized from the same constant.

### Fixed — two silent exits, and an unchecked render

The `attach_buf` and `commit` sends returned 1 in silence **after** the compositor had already minted
a surface, so the one process that knew the failure said nothing. `crab_render`'s return is now
checked at both call sites.

### Fixed — the test suite had two defects of its own

- ⛔ `crab_render` was called with **`rmt` as the left pane's mtimes**, so `lmt` was allocated,
  filled, and never read — both panes rendered from one array. Both arrays hold the same value in the
  fixture, which is exactly why it survived: the fixture could not tell the bug from the fix.
- The suite header claimed the premultiplied flag was *"armed by `CRAB_PREMUL=1`"*. `CRAB_PREMUL` has
  never existed anywhere in the tree; `crab_surface_flags` is a bare unconditional return. A reader
  looking for the off switch would have found nothing and had no way to tell whether they were
  misreading the code or the comment.

### Added — 11 → 37 assertions, mutation-proven

New groups cover the bounded path helpers (including the exact 252-char-path + 62-char-name shape
that used to write 315 bytes into 256), the size ladder at every boundary, the date-formatter clamps,
and the truncation marker. Reference coverage **23 % → 53 %**.

⛔ **The truncation test was rewritten because its first draft could not fail.** It reimplemented
`crab_row`'s name-column loop in the harness and asserted on the copy — so deleting the marker from
the production function left the suite green. A test that mirrors the code under test is measuring
itself. It now drives the real `crab_row` through a real `dh_list` and reads the resulting widget
text back. This is the same defect 0.4.14 records, where counting per-entry trace lines made the
diagnostic the thing being measured.

Every new assertion is mutation-proven: reverting each bound, the overflow fix and the truncation
marker each produce a named failure.

### Added — `docs/development/roadmap.md`, and the deferral tail that fills it

⛔ **The roadmap was the `cyrius init` template — `### M1 — _Title_ (v0.2.0)` — through fifteen
releases.** So every deferral crab accumulated had nowhere to be sequenced, and lived as ⛔/⚠ prose
scattered across `src/`, the CHANGELOG, `state.md` and `cyrius.cyml`. The only way to find them was
to read all of it.

**39 deferrals** were harvested and folded into eight milestones (M1 hardening → M8 assisted search),
each carrying its **named upstream gate** rather than discovering it when the milestone starts:

- **dhancha** — per-frame allocation (blocks everything after M2), the allocating idle poll, TABLE /
  GRID / COLUMNS / TREE / MENU / PROGRESS / context menu / modal sheet
- **rupa** — an `on-accent` token; without it a selected row cannot carry guaranteed-legible text
- **setu** — `SETU_SURF_FULL_KEYS`, so key *release* and held keys exist
- **agnos** — resumable readdir (a pane cannot represent more than 256 entries; the canvas draws 812),
  and the write syscalls M4 needs
- **rekha** — proportional text; crab passes `font = 0` and calls no `rekha_*` function today
- **daimon** — the vector store the whole AI arc rests on. ⛔ crab's package description, its `[deps]`
  comment and its README all promise it, and `cyrius.cyml` declares no daimon dependency.

### Added — ADRs for the two decisions the roadmap rests on

- **[ADR 0001](docs/adr/0001-compositor-owns-theming.md)** — the compositor owns theming; crab ships
  no palette and no theme UI. The canvas's light and dark shells are two **compositor states**, not
  two crab settings. ⚠ Recorded because the pressure to reverse it is predictable: "add a dark mode
  toggle" looks like a small local change and is architecturally excluded.
- **[ADR 0002](docs/adr/0002-semantic-find-is-a-mode.md)** — semantic find is a **mode over any
  view**, not a view of its own. The canvas raises this as its own open question and notes *"cheapest
  to build as a view; better to use as a mode"*. Deciding factor: the ranked-result affordances
  belong in **browse** too — the canvas draws suggested tags in 1a's browse pane and dupe grouping is
  useful in a plain listing. One result model that list, grid, columns and gallery all render.
  ⚠ Consequence, stated up front: the entry record must carry optional match metadata from M3, not
  M7 — the readdir record is the syscall's fixed 64 bytes and cannot hold it.

### Added — `docs/architecture/001-every-frame-allocates-and-nothing-is-freed.md`

**Measured: `crab_render` costs 749,704 B per frame at 114 entries per pane, and none of it is ever
reclaimed.** 89 % of that is two full-size pixel buffers — and one of them, `dh_surface_new`'s
`alloc(w*h*4)`, is **never written and never read**, because `dh_surface_render` allocates its own
`sd_surface_new` to draw into.

It has not bitten because crab repaints on input and used to exit after two seconds. Both of those
accidents are now gone, and M2's self-redraw makes it **45 MB/s at 60 Hz**. The note records the
measurement, the breakdown, the three-step upstream fix, and the rule it implies: **do not add a
continuously-repainting element until the dhancha gate is closed** — it will work in QEMU and exhaust
memory on iron.

### Verified

`cyrius build` OK on x86_64 (381,544 B) and `--agnos` (381,600 B) · `cyrius tests` **37 / 0** ·
`fuzz` PASS · `bench` PASS · `vet` 1 dep, 0 untrusted, 0 missing · `deny` 0 violations ·
`fmt --check` clean · coverage **53 %** (was 23 %).

⚠ Binary grew 377,288 → 381,544 B (**+4,256**), which is the bounds checks. That is the price and it
is worth stating rather than burying.

⚠ **Not run on agnos or iron.** Every repair here is host-verified — compiled probes against the real
functions plus mutation-proven assertions — and the two most consequential (the loop lifetime and the
idle sleep) are in `#ifdef CYRIUS_TARGET_AGNOS` regions that **no host test executes**. They need a
burn before they are claimed.

⚠ `cyrius build --win` still fails, unchanged and for the reason 0.4.15 recorded: `sys_socket` /
`sys_connect` are absent from the Windows syscall table. Nothing in crab causes it.

## [0.4.15] - 2026-08-26 — seven toolchain releases, same size, two-thirds new bytes

### Changed — cyrius pin 6.5.28 -> **6.5.35**

Seven releases. `cyrius build` had been printing `toolchain drift` on every invocation since the
installed `cycc` moved to 6.5.35 — the pin is documentation, not enforcement, so crab was compiling
with .35 while *declaring* .28. ⛔ **The declaration is the half that matters**: CI installs the
toolchain by `grep '^cyrius = ' cyrius.cyml`, so a cold build got .28 and a local build got .35, and
nothing in either run said so.

⭐ **6.5.35's register-allocator rework rewrote two thirds of crab and changed its size by zero
bytes.** Measured, both compilers run over the same tree, `build/crab` x86_64:

|                  | 6.5.28    | 6.5.35              |
|------------------|-----------|---------------------|
| size             | 377,288 B | **377,288 B** (±0)  |
| bytes differing  | —         | **240,284 (63.7 %)** |

.35 replaced the "every live interval ends at the function end" stub with real loop-aware liveness
and lifted the lifetime cap on register picks, so straight-line regions time-share registers for the
first time. On crab that redistributes the code without shrinking it. ⚠ **No runtime claim is made
here.** crab's cost is a `readdir` and a blit, neither of which this touches, and cyrius's own release
notes say the runtime win was not demonstrable on their corpus either — their first A/B showing −11 %
collapsed to noise at best-of-25.

### Changed — `lib/` re-vendored from the 6.5.35 snapshot

`cyrius lib sync` rewrote **29** stdlib leaves. **Two changed content**, and `cyrius.lock` moves
exactly two hashes:

- **`lib/fmt.cyr`** — 6.5.30's `fmt_float` carry fix. When the rounded fraction reached a full unit
  the carry had nowhere to go, because the integer part had already been written to the buffer, so
  the raw `10^decimals` was emitted verbatim as the fraction field: `3 - 1e-7` printed `2.1000000`.
  ⚠ crab calls no `fmt_float` and no `f64_*` — this is correctness crab now carries but does not
  exercise.
- **`lib/syscalls_windows.cyr`** — 6.5.30's `Stat` enum. A compile-time contract only: `xstat`
  returns −1 unconditionally on `CYRIUS_TARGET_WIN`, so no byte at those offsets is ever read.

Everything else in `lib/` was already byte-identical to the .35 snapshot.

⚠ **`lib sync` covers 29 of the 30 stdlib leaves crab actually vendors — `lib/atomic.cyr` is not one
of them.** It is a transitive leaf (`alloc`, `io`, `fmt` and three `syscalls_*` files include it), it
is not in `[deps].stdlib`, and the sync walks the *declared* set. Checked by hand this cut and it is
already byte-identical to the .35 snapshot, so nothing is stale today — but it is the one file a
toolchain bump can silently leave behind, and `cyrius.lock` would happily lock the old hash.

### Unchanged — all six deps, verified rather than assumed

sadish 0.5.2 · rupa 0.1.4 · rekha 0.3.5 · kashi 1.0.6 · dhancha 0.9.12 · setu 0.8.7.

⛔ **This is a checked result, not a skipped step.** Four of the six carry `path` alongside `tag`
(rupa, kashi, dhancha, setu — sadish and rekha are tag-only), and **`path` wins**, so a green build is
not evidence that the declared graph resolves. That is the drift 0.4.13 caught and closed. Each tag
was checked three ways: against the sibling's `VERSION`, against `git rev-parse <tag> == HEAD` in the
sibling working tree, and against the newest tag actually published on GitHub (`git ls-remote`,
sorted `-V`). All six agree, and every vendored bundle in `lib/` is byte-identical to that sibling's
`dist/` output. There was nothing to bump.

### Fixed — four claims in `cyrius.cyml` that were false about the graph they describe

The manifest's comment blocks are load-bearing — they are where the ⛔ rules live — so a stale one is
worse than none. All four re-derived from the live tree:

- **"`net` stays until setu moves off TCP"** — setu moved off TCP in **0.8.4 (2026-08-07)**; crab has
  pinned past it since 0.4.5. Measured: with `net` deleted from `[deps].stdlib` *and* `lib/net.cyr`
  removed, `cyrius deps` re-creates the leaf (30,092 B) from setu's `dist/setu.deps` sidecar and the
  build is OK at the same 377,288 B. The declaration is **redundant**, not load-bearing. ⚠ It is
  **not dropped here** — that is a separate change, not a version bump.
- **"the agnos socket (`anu`)"** — the codename is **retired**, operator ruling 2026-08-05: *a name is
  a distribution fact*, and a band that only appears as a prefix inside one kernel has no repo boundary
  to cross. It is `#97 chan_op` / `VFS_CHAN = 11` / `chan_*`. `anu` resolves to nothing.
- **"Every other dep in this stack carries both"**, in **four** places — false. **Four of six** carry
  `path` (rupa, kashi, dhancha, setu); **sadish and rekha are tag-only** and always have been. A
  reader trusting that comment would look for a `path` that is not there.
- **"path=../rupa until tagged"** — rupa has been tagged since 0.1.3, and this manifest pins
  `tag = "0.1.4"` ten lines below the claim.

### Docs — `docs/development/state.md` rewritten; it had rotted for eleven releases

⛔ **The file's own ⛔ warning about going stale came true a second time, and the second time was
worse** — rot in a detailed file rather than neglect of an empty one. Untouched from 2026-08-07 to
2026-08-26 (0.4.4 → 0.4.15), it asserted a `6.5.5` pin, **all six** dep versions wrong, a `Next` item
that shipped in 0.4.6, the retired `anu` name, and — flatly false — **"Never run on iron"**, when crab
has burned on iron twice and both burns found real defects (2026-08-08: orphaned alive holding one of
16 system-wide `#86` shm slots, fixed in 0.4.6 · 2026-08-19: 114 entries at `/`, 32 listed, fixed in
0.4.13/0.4.14).

Two live gates in it were also **already cleared** and still read as open: the "re-establish crab's
agnos standing before anything is called proven again" gate (cleared 0.4.5 — crab is the second client
in agnos 1.56.40's ipc bite 7), and "`-smp 4` fault-kills" (that same run proved under `-smp 4`). The
⛔ retraction blocks are kept as records; only the live claims were touched.

Added a **Known gaps** section, which the file had never had. Its first entry: ⛔ a **stale `crab`
binary is tracked at the repo root** (319,040 B, 2026-07-23, 58,248 B smaller than today's build).
`.gitignore` covers `/build/` but not `/crab`, and `release.yml` publishes `git archive HEAD` with no
`.gitattributes` `export-ignore` — so **every source tarball ships a months-old binary**, and that
tarball is what `cyrius deps` fetches. Left for the operator: it is a git change.

### Verified

`cyrius build` OK on x86_64 (377,288 B) and `--agnos` (377,312 B) · `cyrius test` **11 / 0** ·
`cyrius tests` 1 / 0 · `fuzz` PASS · `bench` PASS · `vet` 1 dep, 0 untrusted, 0 missing · `deny`
0 violations · `fmt --check` clean.

⚠ **`cyrius build --win` still fails, and it is not this release.** `error: refusing to emit binary
with 2 reachable undefined function(s)` — `sys_socket` / `sys_connect`. Verified pre-existing by
rebuilding the 0.4.14 tree with the 6.5.28 toolchain: identical failure, identical two symbols.

⛔ **The obvious attribution is wrong, and this entry shipped it wrong in draft.** It is *not* the
retired `net` TCP transport: `lib/net.cyr` names neither symbol — it reaches BSD sockets through the
generic form (`syscall(NSYS_SOCKET, AF_INET, …)` at `net.cyr:197`), and a generic `syscall()` emits no
named reference, so `net` cannot produce an undefined *name*. The only callers in the tree are
`lib/setu.cyr:971` / `:973` / `:1022`, and they are **AF_UNIX / SOCK_SEQPACKET** — i.e. they exist
*because* setu already moved off TCP (setu 0.8.4, 2026-08-07). The real cause is target-table
coverage: `sys_socket` / `sys_connect` are defined in `lib/syscalls_linux_common.cyr:470` / `:515`,
`lib/syscalls.cyr` routes `CYRIUS_TARGET_WIN` to `lib/syscalls_windows.cyr` instead, and that file
defines `sys_socketpair` but neither of these two. Windows is not a declared crab target, so this is
recorded rather than fixed.

## [0.4.14] - 2026-08-19 — narrating the listing WAS the cost

### Fixed — crab got slower the moment its entry cap grew

Iron 2026-08-19: "crab shows way more entries than before but is responding slower from inputs."
`crab_stat_all` printed `crab: stat <name> <size>` for **every entry** and called `alloc(24)` for
every entry — on every readdir, i.e. every descend and every ascend. At the old 32-entry cap that
was 32 console writes; at 256 on a real `/` (114 entries on iron) it is 114 writes to a console
**three processes share unserialised**, plus 114 leaked scratch allocations (this allocator has no
free). The listing was not the cost. Narrating it was.

Per-entry tracing is now **off by default** behind `CRAB_STAT_TRACE=1`, and its scratch buffer is
allocated once.

### Added — one summary line per listing

`crab: listed <n> entries in <path>`, always on. ⛔ This is the honest oracle for "did the pane see
everything": a COUNT, stated once. `crab-listing-cap-test.py` previously inferred it by counting
per-entry lines — which made the diagnostic the thing being measured, and is why the fix for the cap
shipped with a performance regression attached to its own instrumentation.


## [0.4.13] - 2026-08-19 — the pane showed 32 of 114

### Fixed — a directory with more than 32 entries was silently truncated

`crab_readdir_into` called `sys_readdir(path, buf, 32)`. Iron 2026-08-19: `/` held **114** entries
(burn outputs land at the root), so the pane listed 32 and dropped 82 with no indication. A file the
shell's `ls` showed was simply absent from crab — which reads as a filesystem or staging fault, not
a cap. The cap is now **`CRAB_MAX_ENTRIES` = 256** (records are 64 B, so a 16 KB pane buffer).

⛔ The `32` was a literal in **seven** places — the call plus six `alloc` sites (`lbuf`/`rbuf`,
`lsizes`/`rsizes`, `lmtimes`/`rmtimes`). Raising the call without every allocation writes past the
buffer; raising an allocation without the call changes nothing. Both now derive from the constant.

### Added — the pane reports its own truncation

`sys_readdir` returns what fit, so `n == max` is indistinguishable from "there were more" — it is
the only truncation signal available. Hitting the cap now prints
`crab: WARNING listing truncated at the entry cap`. A pane that stops silently makes missing files
look nonexistent.

⚠ Display was never the limit: `CRAB_ROWS_CAP` was removed in 0.4.10 for a scrolling `dh_list`, so
256 entries scroll.

### Changed — dhancha tag 0.9.11 -> **0.9.12**, matching the vendored bundle

`cyrius build` re-vendored `lib/dhancha.cyr` from `path = "../dhancha"` (0.9.12, adding
`WINDOW_CONFIGURE`), which moved the file and its `cyrius.lock` hash while the manifest still
declared `tag = "0.9.11"`. ⛔ **The path WINS over the tag**, so the build was green against a
library the declared graph did not name — CI clones by tag and would have built 0.9.11. crab uses
no 0.9.12 API; the tag is corrected so the declaration matches what was actually compiled and
staged. Verified released: `0.9.12` is tagged in dhancha.

### Changed — cyrius pin 6.5.27 -> 6.5.28

⚠ `cyrius test` stayed **11/0 while `src/main.cyr` did not compile** — the suite does not build the
binary, so a green run here is not evidence the program links. Build both targets explicitly.


## [0.4.12] - 2026-08-17 — input goes through the toolkit

### Changed — the hand-rolled input loop is gone

crab polled `setu_poll_input` and switched on raw setu message kinds while using dhancha for pixels —
the same split M7-D closed for its panes. Input now runs on **`dh_client_poll_event`** (dhancha
0.9.11), and `setu_client_close` became `dh_client_close` for symmetry with the existing
`dh_client_connect`.

⭐ **Adopting it fixes a latent bug crab could not see.** setu documents `setu_poll_input` as
"decodes only the FIRST frame of each recv; coalesced or split frames are dropped ... kept for API
compat" — and a dropped tail **loses key-RELEASE events**. dhancha wraps `setu_client_poll_input`,
which reassembles the stream. crab never noticed because it only reads presses; the next consumer
wanting held keys would have.

⚠ **Non-blocking was the requirement, not a preference.** A file manager repaints on its own —
selection moves, panes scroll — so a blocking read would stall the render loop on an idle connection.
That is why dhancha grew both shapes rather than crab keeping its own loop.

### Added — EOF now exits

`dh_client_poll_event` returns -6 when the connection is gone. crab treats that as terminal instead
of spinning 2M frames rendering into a surface nobody reads while holding its `#86` shm slot — the
same class of leak `SETU_CLOSE` handling already guarded against, on the other failure path.

### Unchanged — the present path, deliberately

crab writes a LIVE shared buffer with no per-frame protocol traffic; `dh_client_present` sends
ATTACH + COMMIT every frame. Different models, and swapping them is not a rename.

⚠ The old note here claimed `dh_client_present` "re-sends pixels every call" — that is **stale**.
`setu_client_present` has used a cached shared buffer (create once, rewrite in place, recreate on
resize) for some time. The reason to stay put is the per-frame commit, not the pixel copy.

### Changed — `[deps.dhancha]` 0.9.10 -> **0.9.11**

## [0.4.11] - 2026-08-17 — toolchain pin to 6.5.27

### Changed — `cyrius = "6.5.21"` -> **6.5.27**

Stack-wide sweep so every repo in the desktop stack declares one toolchain. Pins had drifted across
three lines (6.5.5 / 6.5.20 / 6.5.21) while the installed wrapper was 6.5.27, so every build ran with
a drift warning and the declared graph did not describe what was actually compiled.

⚠ **Measured byte-identical**: 6.5.21 and 6.5.27 produce the same artifact for this repo, so the bump carries no codegen risk here. Recorded because a pin that is assumed to be cosmetic is how a real change gets waved through later.

⚠ The vendored `lib/` was re-synced to the 6.5.27 bundled set, which clears the
`./lib/ shadows version-pinned` warning. Tests re-run green after both changes.

## [0.4.10] - 2026-08-17 — the panes stop being hand-rolled: onto dhancha's LIST

### Fixed — ⛔ ENTRIES PAST THE 7th WERE UNREACHABLE, NOT MERELY UNDRAWN

`CRAB_ROWS_CAP = 7` capped the rows a pane BUILT, and `crab_maxsel` clamped the down-arrow to the same
number. Those two together were not a display limitation: in a directory of 40 files, **33 could not be
selected, entered, or acted on at all**. A scrollbar was not missing — reachability was.

A pane is now a **`dh_list`** (dhancha 0.9.8) holding every entry, and the cap is deleted rather than
raised. `crab_maxsel` returns `count - 1`.

### Changed — the selection is dhancha's to paint now

`crab_row` used to set its own background: accent when the pane was active, a muted line-tint when it
was not. dhancha 0.9.8 draws exactly that from the LIST's own selection and focus, so the row now sets
**no background at all** and `active` no longer reaches rows.

⚠ **A row that keeps a background paints OVER the toolkit's highlight** and the selection silently
stops showing. That is the trap in this port, and it is covered by a mutation check.

⛔ This is the deletion that makes the port worth doing. The rule *"accent when focused, muted
otherwise"* — which is the only thing telling the operator which pane the arrow keys drive — existed in
crab, in puka, and in aethersafha's launcher, three times, separately. It now exists once.

### Changed — scroll offsets are app state, deliberately

crab rebuilds its whole widget tree every frame, so the LIST that owns a scroll offset is destroyed and
recreated each time. Two heap cells carry the offsets across frames and `crab_render` writes back the
values the toolkit settled on.

⚠ **Without them the app would still be correct but would feel wrong**: `dh_list_scroll_to_sel` moves
by the MINIMUM distance that makes the selection visible, and with the offset reset to 0 every frame
that minimum is always "put it at the top", so each downward step past the viewport snaps instead of
scrolling by one row.

⚠ The order in `crab_render` is load-bearing: layout, then follow the selection (it needs the list's
laid-out height to know what "visible" means), then layout again (the new offset moves the rows).

### Changed — `[deps.rupa]` 0.1.2 -> **0.1.3** (a hard requirement, not a freshness bump)

⛔ **crab did not build against 0.1.2 at all.** dhancha 0.9.6 added per-widget motion, so
`dist/dhancha.cyr` references `RupaMotion` / `RupaEase` / `rupa_motion_duration`, all of which arrived
in rupa 0.1.3. The failure is `undefined variable 'RupaMotion'` from a file crab does not own, and it
had gone unnoticed because crab had not been rebuilt since dhancha 0.9.6 shipped.

⚠ `path = "../rupa"` added to match every other dep in this stack — without it a local rupa change
cannot be exercised until it is pushed.

### Testing — `src/render_test.cyr` is now a self-checking pixel proof

It used to render a frame, dump BGRA, and assert nothing — and it dumped to a **hardcoded scratchpad
path from an unrelated session**, so on any other machine the write silently failed. It now returns a
failure count, writes to `build/crab-render.bin`, and checks on pixels:

- the ACTIVE pane's selection is accented and the inactive pane's is muted (⛔ two accented rows cannot
  answer "which pane do my arrows drive" — the one question a two-pane manager must answer on screen)
- a 12-entry pane in a ~6-row viewport scrolls to reveal the last entry, with a nonzero offset
- nothing paints below the pane onto the status line (dhancha 0.9.7's clip, seen from the app)
- `crab_maxsel` reaches the last entry at 12 and at 40

Mutation-tested: restoring the 7-row cap, restoring the row background, dropping `dh_focus_set` on the
active pane, and dropping `scroll_to_sel` are each caught. `cyrius test` still passes 11/11, including
the `#92` premultiplied-alpha contract — which matters here because the highlight is a new painted rect.

## [0.4.9] - 2026-08-17 — desktop-stack catch-up: dhancha 0.9.5, setu 0.8.6, one language version

### Changed — `[deps.dhancha]` 0.9.4 -> **0.9.5**

Picks up two toolkit fixes crab is a direct consumer of: `dh_hit_test` now CLIPS to the parent, so a
widget can no longer answer clicks at coordinates where it is not drawn, and `dh_surface_present`
refuses (`DHANCHA_ERR_UNSUPPORTED`) instead of returning `DHANCHA_OK` without presenting.
⚠ The clipping change is behaviour crab inherits in its dual-pane UI (30 dhancha calls in `src/ui.cyr`);
its own 11 + 1 assertions stay green.

### Changed — `[deps.setu]` 0.8.5 -> **0.8.6**

`present_probe` honours `SETU_CLOSE`. Not crab's own defect — crab has always exited on close — but it
shared the 16-slot `#86` budget with the probe, which leaked one slot per desktop launch (measured on
iron: 16 → 15 → 14 → 13).

### Changed — cyrius pin 6.5.9 -> **6.5.21**, matching agnos, aethersafha and dhancha

One language version across the desktop stack.

### Fixed — `[deps.kashi]` gains a `path` override

Every other dep in this stack carries `git` + `path`; kashi did not, so a local kashi change could not
be built against. ⛔ `path` WINS over `tag`, so a green build here is not evidence the declared graph
resolves — re-verify the tag against kashi's `VERSION` at each cut.

⚠ **Unblocked a hard resolution failure, whose cause was NOT here.** `cyrius deps` was failing with
*"dep dhancha requires 'kashi_font_data' but it is not in the cyrius stdlib"*. kashi is VENDORED, so it
belongs to neither list; dhancha's `dist/dhancha.deps` sidecar wrongly declared it a stdlib leaf.
Fixed in dhancha 0.9.5 (a generator script + CI gate, ported from setu's, for the same open cyrius
defect: `setu docs/development/issues/2026-08-07-distlib-deps-sidecar-under-reports.md`).

**Verified**: `--agnos` build OK, 356,360 B; host suites **11 + 1** green.

## [0.4.8] - 2026-08-16 — the connection goes through dhancha, not around it

### Changed — `dh_client_connect` / `dh_client_fd`, and `[deps.dhancha]` 0.9.3 -> 0.9.4 with a `path`

crab already built its UI on dhancha (30 API calls in `src/ui.cyr`) while opening its own setu
connection — depending on the toolkit for pixels and bypassing it for the transport.

⛔ **THE BYPASS WAS FORCED, NOT CHOSEN.** dhancha's dist bundle shipped only
`error / widget / layout / event / theme / surface`; its client layer was never published, so
`dh_client_connect` did not exist downstream. Fixed in dhancha 0.9.4; this is the consumer half.

⚠ The PRESENT path stays hand-rolled: crab uses a LIVE shared buffer (`setu_buf_create` once,
`setu_buf_write` per frame, no re-present) while `dh_client_present` re-sends pixels every call. Swapping
those is not a rename.

⚠ `path = "../dhancha"` added, matching every other dep in this stack — without it crab could only build
against a PUBLISHED tag, so a local toolkit fix could not be exercised until pushed, which is how a burn
ends up testing last week's library. ⛔ `path` WINS over `tag`: a green build here is not evidence the
declared graph resolves.

**Verified**: 352,176 B, host suites 11 + 1 green, and in QEMU alongside puka —
`presented: 2`, `exit 95` (`agnos/scripts/harness/puka-terminal-test.py`).

## [0.4.7] - 2026-08-12 — one rendezvous, named by setu

⭐ Passes **0** to setu instead of hardcoding `"/tmp/aethersafha-setu.sock"`, so the socket is named in
one place — `setu_un_path` (setu **0.8.5**), which resolves an explicit path, then `$SETU_SOCKET`, then
`SETU_UNIX_PATH`. Four repos each carried that literal; they agreed, but all four had to be edited in
step for that to stay true.
⚠ `[deps.setu]` gains `path = "../setu"` alongside its tag — it was the one dep here declaring a tag
with no path override, so a local setu change could not be built against at all. Verified: with the old
vendored 0.8.4 this client silently ignored `$SETU_SOCKET` and `--clients` answered 94.
⛔ Also corrects a comment asserting the path was *"advisory and always was — setu ignores it"*, false
since setu 0.8.4.

## [0.4.6] - 2026-08-08 — `AE-6`: crab's surface is PREMULTIPLIED, and crab EXITS when its window is closed

⭐⭐ **crab now declares `SETU_SURF_PREMULTIPLIED`, unconditionally.** That routes it through agnos
`gpu_shader_op #92` op 0x01 — a real per-pixel `out = src + dst * (1 - src_a)` on the compute units —
instead of `#87`, a byte mover with no ALU that can only copy. The blend is the one operation CP-DMA
structurally cannot do, and until now it had **no client at all**.

**crab satisfies the op's precondition by construction.** sadish's `sd_alpha_of()` maps a bare `sd_rgb`
value's 0 byte to 255 and `draw.cyr` stores that, so every pixel crab produces is **alpha 255** — trivially
premultiplied, since at a == 255 premultiplied and straight alpha are the same bytes.

⚠ **No flag, no arm, no env var.** The client states what its pixels ARE; the compositor decides how to draw
them. A boot whose GPU has no shader envelope falls back to `#87` **per window** in
`ae_gpu_present_frame` — exact at alpha 255 — instead of costing the frame its hardware compositing.

### Added — `tests/crab.tcyr`: the premultiplied contract, per pixel, on the production render

crab's entire test coverage was `assert(1, "true is true")` and `1 + 1 == 2`, while the arc's docs carried
"crab is alpha-255 clean throughout" as a load-bearing fact. It was true by inheritance from sadish, with
nothing pinning it. The suite now renders the **production `crab_render`** and asserts `a == 255` **and**
`c <= a` — the op's actual contract, not a proxy — on all 83,600 pixels, so if crab ever gains a translucent
element the gate fails and names `sd_premul()` as the fix. **11/11**, with negative controls for each check.

⛔ **The negative controls caught a bug in themselves first.** They seeded `pix + 4*3` for "pixel 4, byte 3"
— a pixel is **4 bytes**, so that wrote pixel 3's blue channel, left every alpha at 255, and the scan
correctly found nothing. Without them the suite would have read green while testing nothing.

⚠ `src/test.cyr` is now deliberately empty with a warning in it: bare `cyrius test` auto-discovers
`tests/*.tcyr` and does **not** run the `[build].test` hook, so a gate written there never executes.

### Fixed — crab EXITS when the compositor closes its window

⛔⛔ **crab used to ignore being closed, and stayed running for the rest of the boot.** aethersafha's F4
removed the window from its own vector and told nobody, so on the 2026-08-08 iron burn the process was
left **orphaned alive** — still holding its `#97` channel end and its `#86` GPU-visible shm slot. There
are only **16 of those slots system-wide**, so a handful of closes would starve hardware compositing for
everything on the system. The operator saw it as *"not closing properly"*.

⭐ **`SETU_CLOSE` (kind 7) has been in the protocol from the start** (`lib/setu.cyr:140`) — it was simply
never sent by any compositor and never handled by any client. crab now handles it in the same dispatch
that already reads `SETU_INPUT_KEY`, leaves the frame loop, and falls into the existing
`setu_client_close` teardown.

⚠ **crab's EXIT is the release mechanism, not a courtesy.** The kernel reclaims the channel end and the
shm slot when the process dies, so a client that acknowledges the message and keeps running still leaks.

⭐ Verified in QEMU: the compositor sent `SETU_CLOSE`, crab printed
`crab: compositor closed the window -- exiting`, and the screendump shows the window gone with clean
background where it stood — no ghost, no doubling.

## [0.4.5] - 2026-08-07 — the setu handshake fails out loud

### Changed — setu > 0.8.1: crab presents on the agnos channel band

⭐ crab is the **second** client in agnos 1.56.40's ipc bite 7: the compositor mints a channel, endows
one end, and spawns crab already connected — crab dials nothing and `setu_connect` reads `AGNOS_CHAN`.
Proven under QEMU `-smp 4` alongside `present_probe`: both present, framebuffer-confirmed.

Pinned to **setu 0.8.4**, no override. That tag also removes TCP from setu's Linux arm (AF_UNIX /
SOCK_SEQPACKET), so crab speaks a record transport on both targets and its handshake cannot be correct
on one and broken on the other for a framing reason.

⚠ **`chrono` added to `[deps] stdlib`.** crab was missing it and building clean anyway: setu's
`dist/setu.deps` under-reported 8 of 12 leaves, which switched OFF `cyrius deps`' own validation. setu
0.8.4 corrects its sidecar, the check fires again, and this gap became a hard error. Requires
**agnos >= 1.56.40**.

### Added — the three silent `return 1`s in the handshake now say what failed

The CREATE_SURFACE send, the SURFACE_CREATED read, and the reply-kind check each returned bare `1`. A
client that exits 1 with no output is indistinguishable from a client that never ran, and that
ambiguity cost real time: the compositor's serial log showed the reply being *sent*, and only crab
could say whether it arrived. Each now prints before returning.

## [0.4.4] - 2026-08-02

### Changed — setu 0.7.2: crab can actually connect to the compositor on agnos

> ⛔ **SUPERSEDED 2026-08-03 — the transport this entry repairs is RETIRED as the wrong primitive.**
> TCP-on-loopback was the WRONG PRIMITIVE for a local display protocol: it was picked because a TCP
> stack happened to exist, was never put to the operator, and accumulated six accommodations (the
> `sys_net_ip()` dial below being the last of them). A local display protocol has nothing to route,
> nothing to checksum, no window to negotiate, and no business owning a port. The desktop transport is
> now the agnos socket (`anu`) — agnos `docs/development/planning/ipc.md` §9/§10.
>
> ⚠ **What this entry claims is nevertheless TRUE and is NOT withdrawn.** With `net_src_for`
> (agnos 1.56.34) the handshake completes on an ordinary boot, and on 2026-08-02 the honest harness
> `agnos/scripts/harness/aethersafha-clients-test.py` — which byte-scans `build/agnos` and hard-exits if
> the kernel carries any selftest hook — reached **`connected: 2, presented: 2`** with **crab** and
> setu's `present_probe` (`/bin/puka`) as the two clients. Scope it honestly: QEMU at `-smp 1`, never
> shown on iron, `-smp 4` fault-kills. Retirement is architectural — do not restore a TCP dial, but do
> not record this result as a false green either.

Picks up setu **0.7.2**, whose `setu_connect` dials `sys_net_ip()` instead of `127.0.0.1`.

⛔ **Before this, crab could never present on a real agnos boot.** agnos puts `net_ip` in an outbound
SYN's source field, so a connect to 127.0.0.1 produced a SYN-ACK the client's own conn could not
match on the 4-tuple; `sock_connect` #47 returned -1 instantly and crab printed
`crab: setu connect failed`. It looked like a crab fault and was not — the identical failure hit
setu's own `present_probe`. The `AETHERSAFHA_SETU_SELFTEST` kernel hook had been masking it by
assigning `net_ip = 0x7F000001`, which is why the desktop smoke passed while no ordinary boot worked.

No source change here — this is a dependency bump and a rebuild.

⚠ **crab consumes setu's `dist/` bundle**, not its `src/`, so a setu fix only reaches this client
after `cyrius distlib` runs in setu. A local `path` override alone is not enough.

### Verified

Composited as a live window on agnos in QEMU, launched **in the foreground** (`aethersafha`, no `&`)
from the agnsh prompt: the dual-pane file manager rendering real `/bin` contents — checked on the
**framebuffer**, not just the serial log.

### Note — corrected 2026-08-02

This entry originally said *"0.4.3 was never released — `VERSION` read 0.4.3 with no CHANGELOG section."*
That was wrong: a 0.4.3 section existed, misfiled **below** 0.4.2 behind an empty `[Unreleased]` heading,
which is why a search for it came up empty. The ordering is repaired and 0.4.3 now sits in sequence; no
content was invented or dropped.

## [0.4.3] - 2026-08-02

### Changed — cyrius pin 6.4.71 -> 6.5.5; dhancha 0.9.3, sadish 0.5.1, rupa 0.1.2, rekha 0.3.4, kashi 1.0.4, setu 0.7.1

Part of the whole-desktop-stack toolchain catch-up cut on this date.

⭐ crab picks up **setu 0.7.1** and is, as of today, the only setu client in the stack whose surface
is already alpha-255 clean throughout — which makes it the natural first candidate when the
compositor's premultiplied `#92` blend path gets a real consumer.

### Verification

Host + `--agnos` builds green; 1 suite passes.

## [0.4.2] - 2026-07-23

### Changed — setu 0.7.0 (`SETU_SURF_PREMULTIPLIED`) + dep refresh

No behaviour change: the flag is opt-in and this client does not set it, so its surface is still composited
with the opaque `gpu_blit_shm` #87 path.

## [0.4.1] - 2026-07-23

### Changed — setu 0.6.0: client buffers are GPU-visible on agnos

Picks up `setu` **0.6.0**, whose `setu_buf_create` now asks for `shm_create_gpu` **#86** before falling back
to `shm_create` **#71**.

⚠ **Why this matters beyond a version number.** `#71` allocates **system RAM**, which the agnos GPU cannot
reach at all — bus-master is off by design and the engines see only the framebuffer aperture. The kernel
rejects a `#71` slot at both GPU entry points (`gpu_blit_shm` #87: `src_mc == 0 ⇒ the GPU cannot read it`;
`gpu_shader_op` #92: `GPO_E_BADSLOT`). Every shared surface in the desktop was allocated that way, so the
whole iron-proven ring-3 GPU band had **no reachable consumer**. Buffers from this release are eligible for
a hardware blit.

No API change and no call-site change here — the buffer id behaves identically, and `#86` falls back to
`#71` automatically on a machine with no GPU carveout (every QEMU boot).

### Changed — cyrius pin → 6.4.71

## [0.4.0] - 2026-07-12 — follows the shared desktop theme (rupa)

crab now colours itself from the active desktop theme instead of a hardcoded dark
palette, so the file manager matches the compositor chrome and every other dhancha app.
Switch the whole desktop's look with `rupa_theme_set_active_name("shanta-dark")` and crab
re-colours with it — two themes, each dark + light: MUDRA (the seal, default) and SHANTA
(stillness).

### Changed

- **Panes, rows, headers, and the status line draw with `dh_theme_*`** (dhancha's theme
  helpers over the shared **rupa** token core), replacing the hardcoded `sd_rgb(...)`
  literals in `src/ui.cyr`: root → `dh_theme_bg`, pane column → `dh_theme_panel`, list rows
  → `dh_theme_widget`, header + status → `dh_theme_panel`. The **selected** row / active
  header is now the theme **accent** when its pane is focused, and a muted `dh_theme_line`
  tint when it isn't (was a fixed blue). This makes crab legible under the light themes.
- **`[deps.dhancha]` 0.8.0 → 0.9.0** (the `dh_theme_*` API) + new **`[deps.rupa]` 0.1.0**
  (the shared theme tokens). Builds green against published rupa@0.1.0; `test` + `render_test`
  unaffected.

## [0.3.2] - 2026-07-10 — file mtime (status line)

Each entry's modification time joins its size: a Midnight-Commander-style status bar along
the bottom shows the active pane's selected entry — name, size, and mtime date.

### Added

- **mtime status line** — a `BOX_V` root now wraps the panes row plus a bottom status
  `LABEL` (`crab_status_str`) rendering the active selection's `<name>  <size|<dir>>
  <YYYY-MM-DD HH:MM>`. The date comes from the same `stat` syscall (#33 — `st_mtime` @ +40,
  unix seconds), formatted via the civil days→(y,m,d) algorithm (verified against the host
  `datetime` across leap-year boundaries). crab now stats **every** entry (files AND dirs)
  into parallel `sizes[]` + `mtimes[]` per pane. Proven on agnos: the status bar shows
  `aethersafha  14M  2026-07-10 19:18` from the real inode mtime, composited by aethersafha.

  > ⛔ **RETRACTED 2026-08-03 — "Proven on agnos" here is a FALSE GREEN.** The observation came from a
  > setu smoke run against `build/ae-setu-smoke/agnos-ae.img`, the image staged by the now-deleted
  > `aethersafha-setu-smoke.sh`, which built the kernel with `AETHERSAFHA_SETU_SELFTEST=1`. That hook
  > assigned `net_ip = 0x7F000001` before launching the compositor, which is the ONLY reason the
  > loopback handshake completed **in this era** — before `net_src_for` (agnos 1.56.34) an ordinary
  > boot could not. The `stat`/`mtime` work itself may well be sound; what is retracted is the claim
  > that *this run* proved it on agnos. TCP is retired as the desktop transport — as the wrong
  > primitive, not because it never worked (post-`net_src_for` crab connected and presented un-rigged
  > in QEMU at `-smp 1`, 2026-08-02). See agnos `docs/development/planning/ipc.md` §9/§10.

## [0.3.1] - 2026-07-10 — directory navigation + per-entry size

The read-only listing from 0.3.0 becomes a navigable, informative browser: keyboard
directory traversal (Enter/Backspace) and each file's size shown alongside its name.

### Added

- **Directory navigation** — Enter descends into the selected directory (re-`readdir`s the
  child and resets the selection to the top); Backspace ascends to the parent. Pane paths
  are now mutable buffers, so each pane's header tracks the current directory. Path helpers
  (`crab_strcpy`/`crab_join`/`crab_descend`/`crab_ascend`) are host-safe — `sys_readdir` stays
  behind `#ifdef CYRIUS_TARGET_AGNOS`. Proven on agnos via the setu-descend smoke: focus the
  right pane, descend into `/lost+found`, ascend back to `/` (Up/Down + Left/Right unregressed).
- **Navigation serial log** — a successful descend/ascend emits `crab: cd <path>` to serial
  (the smoke's dispositive gate, alongside `key received`).
- **Per-entry file size (richer listing)** — each file's byte size renders right-gapped after
  a 13-char name column, human-readable (`14M` / `299K` / `512`); directories keep the `/`
  marker. Sizes come from the agnos `stat` syscall (#33 — `sys_stat`, `st_size` @ +16 of the
  §4.1 struct): crab stats each listed entry on listing/navigation into a parallel `sizes[]`
  per pane. No new kernel/cyrius work — stat #33 shipped in agnos 1.41.3 and `sys_stat` is in
  cyrius 6.4.43. Proven on agnos (setu-stat smoke): `/bin` lists `aethersafha 14M` / `crab 299K`
  / `puka 79K` from real `stat` sizes, composited by aethersafha. `st_mtime` (@ +40) is
  available from the same syscall but not yet rendered — the ~187px panes don't fit both cleanly.

> ⛔ **RETRACTED 2026-08-03 — both "Proven on agnos" claims in this release are FALSE GREENS.** The
> setu-descend and setu-stat smokes both ran on `build/ae-setu-smoke/agnos-ae.img`, staged by the
> now-deleted `aethersafha-setu-smoke.sh` with `AETHERSAFHA_SETU_SELFTEST=1`. That hook assigned
> `net_ip = 0x7F000001` in the kernel, making a loopback SYN's source and destination agree by
> accident; **in that era (before `net_src_for`, agnos 1.56.34) the compositor↔client handshake could
> not complete on an ordinary boot.** The navigation and `stat` code may be correct — what is retracted
> is that *these smokes* proved it on agnos. TCP is no longer the desktop transport: it is retired as
> the wrong primitive, not as a thing that never worked — after `net_src_for` crab did connect and
> present un-rigged (QEMU, `-smp 1`, 2026-08-02). See agnos `docs/development/planning/ipc.md` §9/§10.

## [0.3.0] - 2026-07-10 — real filesystem listing (the `readdir` syscall)

crab's panes now show the **actual on-disk contents** of a directory on agnos rather than a
hardcoded name list. Each pane calls the ring-3 `readdir` syscall (#81) — landed in **agnos
1.53.13** and exposed as the named, agnos-gated `sys_readdir(path, buf, max)` stdlib wrapper in
**cyrius 6.4.43** — and renders the live entries in the kashi system font, directories suffixed
`/`. Left pane lists `/bin` (`aethersafha` / `crab` / `puka`), right pane lists `/` (`bin/` /
`lost+found/`). QEMU-proven end-to-end: the compositor spawns crab, both panes list their real
directories, and Up/Down selection + Left/Right pane-switch navigate the live entries.

> ⛔ **RETRACTED 2026-08-03 — "QEMU-proven end-to-end" is a FALSE GREEN.** The end-to-end path ran under
> the `AETHERSAFHA_SETU_SELFTEST` kernel hook, which assigned `net_ip = 0x7F000001` so the setu
> loopback handshake would close; **before `net_src_for` (agnos 1.56.34)** no ordinary boot could
> complete it. The hook and its smoke are deleted, and TCP-on-loopback is retired as the desktop
> transport — retired as the **wrong primitive** for local display IPC, not because it never worked:
> after `net_src_for`, crab connected and presented un-rigged (QEMU, `-smp 1`, 2026-08-02). See agnos
> `docs/development/planning/ipc.md` §9/§10. The `readdir` work stands on its own; *this* end-to-end
> proof does not.

### Added

- **Real directory listing via `sys_readdir`** (`src/main.cyr`, `src/ui.cyr`) — each pane
  `readdir`'s its path into a buffer of fixed 64-byte records (name at `+0`, type at `+63`:
  `1` = dir, `0` = file) and renders the live entries; directories get a trailing `/`. The
  selection clamps to the real entry count. Replaces the 0.2.0 hardcoded name list.

### Changed

- **cyrius pin 6.4.34 → 6.4.43** — picks up the agnos-gated `sys_readdir` wrapper; crab now
  calls `sys_readdir(path, buf, max)` instead of the raw `syscall(81, …)` under its own
  `#ifdef CYRIUS_TARGET_AGNOS`.

### Dependencies

- (unchanged) sadish 0.4.1 + rekha 0.3.1 + kashi 1.0.2 + dhancha 0.8.0 + setu 0.4.0.

## [0.2.0] - 2026-07-10 — a real dual-pane file manager on the sovereign desktop

crab graduates from scaffold to a working file manager: a **dhancha widget client** that
presents a **dual-pane** UI over **setu** and is composited by **aethersafha on agnos** — the
first standalone dhancha app to build `--agnos`. Norton-Commander / Dolphin style, navigable
by keyboard, drawn in the kashi system font.

> ⛔ **RETRACTED 2026-08-03 — "composited by aethersafha on agnos" was, *in this arc*, only true under a
> rigged kernel.** Every agnos compositing run in this arc went through the `AETHERSAFHA_SETU_SELFTEST`
> hook, which assigned `net_ip = 0x7F000001` so setu's loopback TCP handshake would match; before
> `net_src_for` (agnos 1.56.34) an ordinary boot could not. The "connects on loopback:7700" transport
> below is RETIRED — as the **wrong primitive** for a local display protocol (nothing to route,
> checksum, or negotiate; no business owning a port), *not* as something that never worked: after
> `net_src_for`, crab connected and presented un-rigged (QEMU, `-smp 1`, 2026-08-02). The replacement
> is the agnos socket (`anu`). See agnos `docs/development/planning/ipc.md` §9/§10.

### Added

- **Dual-pane UI** (`src/ui.cyr`) — two file-list columns (`BOX_H` → two `BOX_V`), each a
  header + rows, rendered by the dhancha toolkit via sadish (2D vector). The active pane's
  header + selection are bright, the inactive pane's dim.
- **Keyboard navigation** — Left/Right (or `h`/`l`) switch the active pane; Up/Down (or
  `j`/`k`) move the selection within it. Routed over setu as `SETU_INPUT_KEY`.
- **setu client transport** (`src/main.cyr`) — connects on loopback:7700, presents a
  shared-buffer surface (CREATE → ATTACH-by-buf → COMMIT), and re-renders live on forwarded
  focus / key input.
- **Real file names** in the kashi system font (full CP437, lowercase) — the same font the
  compositor chrome uses.
- Host layout harness (`src/render_test.cyr`) — dumps the rendered UI for fast iteration.

### Dependencies

- sadish 0.4.1 + rekha 0.3.1 + **kashi 1.0.2** (system font) + **dhancha 0.8.0** (toolkit,
  kashi text path) + setu 0.4.0 (transport). No mabda — on agnos the present goes over setu's
  shared buffer, not a GPU ioctl.

## [0.1.0]

### Added
- Initial project scaffold
