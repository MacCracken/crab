# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.10.4] — unreleased — daimon 2.4.3, and M7's gate re-derived: the build closed, the surface was never there

> **Not cut.** `VERSION` stays `0.10.3`; the number on this heading and the cut are the operator's.
>
> **A pin release, and no byte of crab moved** — daimon is declared, not linked. What moved is what
> crab's documents say about daimon, because re-deriving the gate at this pin found that most of it
> was wrong.

### Changed — daimon `2.1.4` → `2.4.3`

- **Verified three ways before the manifest moved**: the highest tag on the remote is `2.4.3`
  (`e5537f45`), the sibling checkout sits on it, and it is the only one of the eight declared deps
  that moved — the other seven are on their highest tags.
- ⭐ **No byte moved.** Both DCE targets are **byte-identical** to the 0.10.3 cut — host
  `f6a223f9…` (608,040 B), agnos `8102a82e…` (878,376 B) — so 0.10.3's QEMU face PASS stands for
  these exact bytes. `cyrius.lock` unchanged; check 4 (every `path` off) byte-identical.
- ✅ **daimon builds AND runs on agnos — re-derived, not read off its changelog.** `cyrius build
  --agnos` in a scratch copy of the 2.4.3 tag: **0 errors** (one unreachable `sys_dup2` warning,
  from nein's bundle), where 2.1.4 gave 53. And the 2.4.3 commit's CI job *"AGNOS guest test — Guest
  test on the released agnos kernel"* passed, read from the run record.
- The blocker 0.10.2 recorded closed where 0.10.2 said it would, upstream: **cyrius 6.6.6's
  `distlib` dropped `syscalls_linux_common` from bote's sidecar** when bote 3.3.10 regenerated it,
  daimon 2.1.5 pinned that bote, and daimon's own three sites went with its 2.1.7 agnos port.
  ⚠ **0.10.2's handoff said *"6.6.5/6.6.6 changed neither"* — wrong**: 6.6.6 is exactly what fixed
  it, once the sibling between re-ran `distlib`. 0.10.2 measured daimon 2.1.4, which still pinned
  bote 3.3.9. A toolchain fix reaches a consumer only through the sibling that regenerates.
- ⛔ **The transport crab's manifest named NO LONGER EXISTS.** `agent_ipc_new` → `<dir>/<agent>.sock`,
  the AF_UNIX socket crab was going to talk to, was **removed at daimon 2.3.3** — it had no caller and
  its `SO_PEERCRED` check failed open. Agents daimon starts now get a channel on fd 3 (`chan_op`#97
  on agnos), which is for daimon's children. daimon's road for crab is its **HTTP API** — its 2.3.0
  entry says so in as many words, *"crab reaches it over HTTP"*: 127.0.0.1:8090 on Linux; on agnos
  the NIC's own address, because `sock_listen`#56 takes none and TCP to 127.0.0.1 is dropped — so a
  local client dials `sys_net_ip()` and still sends `Host: 127.0.0.1`, as daimon's own
  `tests/agnos/http_client.cyr` does.
- ⛔⛆ **NOTHING IN THE TOOLCHAIN VERIFIES THE DAIMON TAG.** With no `modules`, `cyrius deps` never
  fetches daimon. Measured: `tag = "9.9.9"` — which does not exist — with every `path` disabled:
  "7 deps resolved", `deps --verify` 49/0, `~/.cyrius/deps/daimon` never created. A phantom daimon
  pin passes every gate crab has, which is the 2026-08-28 phantom-tag failure in a new place. It is
  checked by `git ls-remote --tags` at the cut and by nothing else; written into the manifest and
  into the roadmap's *Still to automate*.

### Found — ⛔⛆ the eighth false gate: the agnos build was never M7's only one

From 0.10.0 the roadmap's gate table called the index, tags and smart folders **UNGATED** because
daimon was *declared*; from 0.10.1 to 0.10.3 it called them blocked on daimon's *build* — and the
manifest said M7 *"waits on it"*. Neither ever checked whether daimon offered anything M7 could use.
**It does not, on any target, at 2.4.3 or before:**

- none of the **25 path branches** in daimon's `http_route` (`src/router.cyr`) is about files, tags
  or an index;
- its per-agent **memory store has no route** and no tag field — its own KNOWN GAP note says tag
  lookup is *"a SUBSTRING search over the whole record"*;
- its **RAG keeps nothing on disk** (no write in `vector_store.cyr`, `fed_vector_store.cyr` or
  `rag.cyr`), "embeds" into **32 slots indexed by the sum of a token's bytes** (`stop`, `pots` and
  `tops` are one token), and answers a query with a **prompt template**, not with files.

⇒ Wrong about *which half was missing*, the way proportional text was: the build now exists; the
surface never did. **Filed where it can be acted on** —
`daimon/docs/development/issues/2026-09-23-crab-needs-a-file-index-tags-and-ranked-search.md`, with
crab's copy in `docs/development/issues/`. It states the need from crab's canvas — a persisted,
background, battery-aware disk-wide index; per-file tags (exact, with counts and suggestions) and
ratings; smart folders as queries with counts; ranked file results with *why* for M8 — and the
constraints the answer must fit (crab's loop never blocks; five open agnos filings bear on any
local TCP client; 8 TCP slots on the machine; three kinds of nothing to tell apart). It designs no
route, and it tells daimon crab has no attachment to HTTP.
⛔ **crab ships nothing for M7's remaining half meanwhile**: no crab-side index (it would silo the
shared index crab's design says it reads) and no tag or smart-folder rows that could only ever say
"unavailable" (the VOLUMES rule). ⚠ **Someone was working in daimon's tree during this** (its README
and two guides modified — a doc refresh, no new routes); the filing is the only file this session
put there.

### Fixed — stale text, cut rather than annotated

- `cyrius.cyml` `[deps.daimon]`: the AF_UNIX transport, and the *"DOES NOT BUILD FOR AGNOS"* block
  0.10.2 added — both false at this pin.
- `docs/development/roadmap.md`: the daimon-ruling row (*"crab talks to its AF_UNIX socket. 0.10.0 and
  0.11.0 are unblocked"*), the index and search rows' gates, the paragraph under the ladder (*"what
  remains is the daimon ruling"* — ruled 2026-09-14), M7's and M8's gate lines, and the gate table's
  four daimon rows, which said **UNGATED** (the closed duplicates row is dropped, as the table's own
  rule says). Below the table, *"Next is `0.9.0`"* and *"`cyrius.cyml` declares [daimon] nowhere, and
  daimon 2.1.2 exists locally"* — both false since 0.10.0 — are cut. *Small, cheap, unblocked* still
  listed dropping the `net` declaration *"still in `cyrius.cyml`"*: 0.10.0 dropped it; cut.
- `README.md`, `docs/development/state.md`, `docs/development/handoff.md`: every *"daimon does not
  build for agnos"* and every AF_UNIX transport claim; the dependency table's daimon row; the Known-gaps
  item; *Next*. The handoff's old READ FIRST block is replaced rather than amended.

### Tests

No source line changed, and every gate was run anyway: `cyrius test` **2,543 / 0**; `render_test` **55
/ 0**, built explicitly with its log grepped for `undefined function` (none); `cyrius fuzz` 100,000
rounds; `fmt --check` clean on all eight files; coverage **87 %**; `vet`, `deny`, `deps --verify`
49 / 0. Both targets built under `CYRIUS_DCE=1`, as CI and the release build them.

## [0.10.3] — 2026-09-21 — names are measured by character, and the release ships under DCE

> Cut on operator direction; the commit, the tag and the push are the operator's.
>
> Two things, both decided at the 0.10.2 cut and done here: crab measures text the way dhancha
> 0.10.4 draws it — one glyph per **character** — and the release build carries `CYRIUS_DCE=1`.

### Fixed — ⛔⛆ crab measured per BYTE while the toolkit drew per CHARACTER

dhancha 0.10.4 made every text walk decode UTF-8. crab's two measuring loops — `crab_text_w` and
`crab_name_cell_px` in `src/ui.cyr` — still handed `load8(s + i)` to `dh_text_advance` one byte at
a time, so from 0.10.2 the two disagreed about anything non-ASCII:

```
                            drawn (dhancha 0.10.4)   measured (crab 0.10.2)
`Über.txt`  (9 bytes)       8 cells                  9 cells        -> a `~` on a name that fitted
`—`         (E2 80 94)      1 cell                   3 cells
a cut with room for `Ü`'s lead but not its continuation:            -> `C3 7E` — a raw cell, then `~`
```

- ⭐ **One reader, shared with the draw**: `crab_char_adv(s, at, n, lenp)` — `dh_text_decode` (the
  toolkit's own decoder) plus the per-arm advance the draw uses: the bitmap arm one cell for every
  character with a scalar (`?` and a raw byte included) and nothing for a stray continuation byte;
  the scalable arm `dh_text_advance` of the **scalar**, nothing for a stray. Both loops walk by it.
- ⛔ **The cut lands on a character boundary.** A character is accepted whole or not at all; `fit`
  counts BYTES written (what the caller is told, where the marker goes) while `j` walks CHARACTERS.
- ⛔ **The bound is the bound.** `dh_text_decode` reads a lead's continuation bytes until one is not
  a continuation byte, and crab's bound is a LENGTH (`CRAB_REC_TYPE`, `CRAB_TEXT_SCAN_MAX`), not a
  NUL — so within four bytes of it the decoder is fed a NUL-terminated copy of what remains, and a
  length that would cross it is clamped. A sequence the bound cuts is its raw lead byte, one byte
  long: a wrong width, never a walk, the rule the scans have kept since 0.5.0.
- ⚠ For ASCII nothing moved: the 640x220 `render_test` dump is **byte-identical** to 0.10.2's.
- ⚠ The bitmap arm's `n * CRAB_COL_CHARW` shortcut is gone — it was the bug for non-ASCII — so a
  label is now walked under both faces. Labels are short; the sweep that measured 4 µs/keystroke at
  0.9.4 is unaffected (nothing here is on the stat path).

### Fixed — stale text about the per-byte draw, cut

The 🦀-glyph gate was recorded in three places as closed at **three** independent levels, the
first being *"`dh_draw_text_ink` walks ONE BYTE per glyph in both branches"*. dhancha 0.10.4 closed
that one — a four-byte 🦀 now reaches the cmap as U+1F980 — and the other two stand (rekha's
format-4 cmap is BMP-only; Liberation Sans has no crab; kashi's page is CP437, so it draws `?`).
`crab_door`'s comment, the button-gate comment in `ui.cyr`, the `main.cyr` note beside the menu bar
and `crab_kind_mark`'s *"the two faces disagree about every byte >= 128"* all said the 0.9.0 thing;
each now says the 0.10.4 thing. The roadmap's *Absent affordances* row for the glyph likewise.

### Changed — ⭐ `CYRIUS_DCE=1` on the release build, and on CI's two target builds

Operator ruling 2026-09-21, taken with the 0.10.2 measurements in hand (2,149 unreachable
functions, 1,027,854 B, riding in the shipped artifact). `release.yml` builds both targets under
`CYRIUS_DCE=1`. ⛔ **And so does `ci.yml`** — deliberately, and said out loud rather than done
quietly: `release.yml` gates on `ci.yml`, the suite spawns `build/crab`, and the QEMU harnesses take
`build/crab_agnos`; a gate that tests a plain binary and a release that ships a DCE'd one is the
0.7.7 lesson (two compilers, one green) in a new coat. `render_test` stays plain — it is a harness,
not the artifact — and was measured under DCE too (55/0).

| | 0.10.2 (plain) | **0.10.3 (DCE)** |
|---|---:|---:|
| host | 1,636,136 | **608,040** (same size as 0.10.2's DCE probe, 94,327 bytes differ — `cmp`, never `ls -l`) |
| `--agnos` | 1,681,192 | **878,376** |

⚠ DCE NOPs the code of unreachable functions and keeps their `.bss`, so the `large static data
(143024 bytes)` advisory stays; the compiler's own hint says so.

### Tests

- **2,543 assertions** (2,503 → 2,543): `t_utf8_0103`, its own function per the rule. Under kashi:
  `Über.txt` measures eight cells not nine; `—` one; a stray continuation byte nothing; an invalid
  lead its own raw cell; a four-byte 🦀 one cell; an eight-cell column holds `Über.txt` **whole and
  unmarked** (0.10.2 wrote `Über.t~`); one cell of room holds `Ü` whole — `C3 9C 7E`, the
  continuation travelling with its lead; no room cuts BEFORE `Ü`, never inside it; a 63-byte name
  the kernel failed to terminate copies 63 bytes and **not byte 63**, planted here as a continuation
  byte (the one shape a type byte can never take). Under the fixture face: `nÄm` is 36 px not 44;
  `mÄmmm` with room for `m` and `Ä` cuts after the whole `Ä`; and **`Ü` measures the SCALAR's 12,
  not its lead byte's 8 and not the two bytes' 16**.
- ⭐ **The fixture face now maps `Ü` (U+00DC) onto the `n` glyph** — a second format-4 segment,
  with a reason: every non-ASCII codepoint was `.notdef` before, so the scalar and its lead byte
  priced the same (8) and *"decoded, then asked"* was indistinguishable from *"asked about a byte"*.
  It was: the mutation that hands `load8(s + at)` to `dh_text_advance` **survived** the suite until
  the segment was added, and fails two assertions after it. rekha's search wants segments sorted by
  endCode and ignores the searchRange hints; both are written correctly anyway.
- The 0.9.0 *Latin-1 limit* assertion — `é` at TWO `.notdef` glyphs, *"exactly the two dhancha would
  draw"* — was an expiry and it fired: INVERTED to one, kept with its history.
- ⛔ **Mutation-proven**: the 0.10.2 `ui.cyr` under the final suite fails **21**; the cut copying only
  the first byte of an accepted character, **7**; the bitmap arm pricing per byte of the character,
  **12**; a stray continuation byte priced as a cell, **1**; the scalable arm pricing the lead byte,
  **2** (0 before the fixture change). ⚠⚠ **Three SURVIVE and are recorded at the assertion**: the
  tail copy, the bound clamp, and both together. The whole-name path copies `n` bytes without the
  decoder, the greedy path can never accept the LAST character, and a lead cut at the bound prices
  as the character it would have become — so a read past the bound is not something a width can
  witness. Held by review and by the ⛔ at `crab_char_adv`.
- `render_test` 55 / 0, dump byte-identical to 0.10.2. Fuzz 100,000. fmt ×8. Coverage 322 / 370,
  87 %. vet, deny, `deps --verify` 49 / 0 — every gate run against the **DCE** binaries in `build/`,
  which is now CI's shape.
- ⭐⭐ **QEMU `crab-face-test.py` PASSES on the DCE'd agnos binary** — 878,376 B, the artifact shape
  the release now publishes: `crab: font /fonts/default.ttf 410820 bytes adv=9 upem=2048 i=4 m=13`,
  one `font` line, navigations 1, view changes 2, no fault, no allocator-failure text. DCE NOPed
  **1,771 unreachable functions, 806,806 bytes** (the compiler's own note), and the scalable path,
  the event loop and the frame arena of what was left ran on a real kernel. ⚠ Not on target: a UTF-8 file name — the harness rootfs has none, and the draw is
  dhancha's (pinned by its `text_utf8_test`, 126 checks); crab's half is the measure, pinned above.

## [0.10.2] — 2026-09-21 — toolchain 6.6.6, and the draw-path deps taken to their tags

> Cut on operator direction; the commit, the tag and the push are the operator's.
>
> **A pin-only release: no byte of `src/` moved.** The toolchain moves two patch releases and five
> of the eight deps go to the highest tag on their remote. What changed downstream is the **cost**
> of the binary (+49 %) and — inherited from dhancha 0.10.4 — how a non-ASCII name is drawn. The
> pixels of the production render did not: the 640x220 dump is byte-identical across the move.

### Changed — toolchain `6.6.4` → `6.6.6`

- **What 6.6.5 / 6.6.6 refuse or repair, checked against crab.** The nine new refusals need a shape
  crab does not have: no `#define` anywhere (`src/`, `tests/`: 0), no `struct` / `async fn` /
  `operator`, no duplicated top-level `var` name across the five files, no comment whose first word
  begins with an attribute or directive name (the 15 `#if…` hits are real `#ifdef CYRIUS_TARGET_AGNOS`
  lines), no `var` read after its top-level block. The Windows `O_APPEND` / `O_TRUNC` corruption has
  no PE target to act on; crab's one `AO_TRUNC` is the agnos arm of `crab_fs_open_w` (unchanged, and
  still the divergence *Known gaps* records). ⇒ The build was the gate, and it is green on both
  targets with zero diagnostics that name a `src/` line.
- ⭐ **The toolchain's own effect, MEASURED IN ISOLATION** (the 0.10.1 tree, its old pins resolved
  from their tags, compiled by 6.6.6): host **1,097,664 → 1,277,920 B (+180,256)**, agnos
  **1,146,920 → 1,327,080 B (+180,160)**. Not codegen — the **stdlib snapshot**: `lib/sankoch.cyr`
  moves **2.7.15 → 2.8.0** (665,879 → 945,063 B; Brotli decode for rekha's WOFF2), a leaf crab
  reaches only transitively through chitra's PNG inflate and links whole because crab builds
  without DCE. `lib/bench.cyr` doubles (20,668 → 42,805), `lib/io.cyr` +10 KB, every `syscalls_*`
  peer grows.
- ⚠ **New on every build: `warning: large static data (143024 bytes) — consider alloc()`.** The
  compiler's advisory threshold is 131,072 B of static `var` reservations
  (`cyrius/src/backend/x86/fixup.cyr`); crab was under it at 0.10.1 (no warning) and the 6.6.6
  stdlib alone puts it at 138,848 — the warning appears with the OLD deps too. crab's own sources
  declare **no** top-level array (`grep '^var .*\[' src/` = 0); the reservations are the bundles'
  (kashi's glyph tables, sankoch's Huffman tables, chitra's, dhancha 0.10.4's 128-slot CP437 map).
  Advisory, not an error; recorded so the next reader does not hunt for a crab buffer that is not
  there.
- **`lib/` re-vendored from scratch** (`rm -rf lib && cyrius deps` — CI's shape, not `lib sync`):
  **0 of 42 stdlib files drift** from `~/.cyrius/versions/6.6.6/lib` (every file `cmp`'d, not the
  declared subset — the 0.7.7 lesson). `lib/alloc_cx.cyr` arrives (new in 6.6.6); `lib/flags.cyr`
  and `lib/hashmap_fast.cyr` leave — nothing in the 6.6.6 resolution names them, and neither target
  misses a symbol. Lock 50 → **49 entries**, still 3 commit-pinned (the tag-only deps), and
  `deps --verify` reads **49 verified / 0 failed**.
- ⛔⛆ **THE COMMITTED 0.10.1 TREE NO LONGER BUILT ON THIS BOX, and nothing in crab had changed.**
  `cyrius build` on the untouched checkout: `refusing to emit binary with 3 reachable undefined
  function(s)` — `sd_flatten_op_begin`, `sd_flatten_op_end`, `sd_flatten_degraded`. `../dhancha`
  had moved to 0.10.4, whose dist calls those; `path` won over `tag = "0.10.0"`, rewrote
  `lib/dhancha.cyr`, and linked it against the still-pinned sadish 0.5.5. The manifest's hazard
  says *"`path` wins, so a green local build proves nothing about the tags"*; this is the same
  hazard facing the other way — **a sibling moving ahead breaks a green tree**. The baseline for
  every number above was therefore rebuilt from the TAGS (every `path` disabled), and it reproduces
  the artifacts the 0.10.1 cut left in `build/` **byte-identical** — sha256 `7ee03c10…` host,
  `a5ba0476…` agnos — which is also the first time those two numbers were re-derived rather than
  read.

### Changed — dependencies

| dep | was | now | | dep | was | now |
|---|---|---|---|---|---|---|
| sadish | 0.5.5 | **0.11.2** | | dhancha | 0.10.0 | **0.10.4** |
| rekha | 0.3.10 | **0.9.0** | | daimon | 2.1.3 | **2.1.4** |
| kashi | 1.0.8 | **1.0.10** | | rupa · setu · chitra | 0.1.7 · 0.8.9 · 1.0.3 | unchanged — already the highest tag |

- **Every one of the eight verified three ways** before the manifest moved: the highest tag on the
  GitHub remote (`git ls-remote --tags … | sort -V | tail -1`), the sibling checkout on that tag, and
  for the three tag-only deps the lock's commit equal to the remote's (sadish `d9f41f3`, rekha
  `47aabba`, chitra `833919c`). Every bundle in `lib/` `cmp`s byte-identical to its sibling's
  `dist/` (kashi: `src/font_data.cyr`).
- ⛔ **Check 4, re-run at the cut:** every `path` line disabled, `rm -rf lib && cyrius deps` resolved
  all seven from git (**7 commit-pinned**, dhancha at the peeled `79b7ad1` — ⚠ 0.10.4 is an
  ANNOTATED tag and `cyrius deps` prints `refs/tags/0.10.4 … is not a commit!` while pinning it
  correctly), and both targets rebuilt **byte-identical** to the override build — host
  `5f2a99e0…`, agnos `15e55f90…`. The paths went back afterwards; the tree and the lock `cmp` the
  same either way.
- **crab's own call surface into the moved deps is eight functions**, enumerated, every one at the
  same arity: `sd_surface_pixels` / `_width` / `_height` / `_pixel_at` and `sd_alpha_of`;
  `rekha_font_open(buf, len)`, `rekha_units_per_em(font)`, `rekha_char_advance_px(font, cp, px)` —
  rekha's three are `public fn` now. dhancha's 18 `sd_*` and 5 `rekha_*` are dhancha's to check,
  and its 0.10.1 did.
- ⭐ **dhancha 0.10.4 draws ONE GLYPH PER CHARACTER.** Through 0.10.3 every text walk took `load8` as
  the codepoint, so crab's notices — which carry `—` (E2 80 94) — drew three cells for one dash,
  and a name like `Über.txt` drew as `Ãœber.txt`; under the face each high byte was a `.notdef`.
  Now the dash is a dash and the umlaut an umlaut, in both arms, with no crab change.
  ⚠⚠ **AND crab STILL MEASURES PER BYTE — measure and draw now DISAGREE for a non-ASCII name.**
  `crab_text_w` and `crab_name_cell_px` (`src/ui.cyr`) pass `load8(s + i)` to `dh_text_advance`
  one byte at a time, so `Über` is summed as **five** advances (Ã, U+009C, b, e, r) where the draw
  makes **four**; a non-ASCII name is **over-measured** and cut a glyph or two earlier than it
  needs to be. Worse, the cut is byte-granular: it can land between C3 and 9C, leaving a lone lead
  byte that 0.10.4 draws as **its raw cell** before the `~`. Before this release the whole name was
  mojibake, so this is strictly less wrong — but it is a new *kind* of wrong, at the cut, and it is
  crab's to fix. **Not fixed here**: a pin bump is not the change that teaches the measure UTF-8.
  dhancha 0.10.4 exports `dh_text_decode(s, at, cpp)` — bytes consumed, scalar stored — for exactly
  this; the fix is the two loops walking by that and the cut landing on a character boundary.
  Recorded in the roadmap as the next item.
- **dhancha 0.10.2** opens ONE flatten operation around a label's glyph loop, so a hostile face
  cannot spend a budget per glyph; a legitimate label degrades to chords past ~1,889 glyphs at a
  32 px em. crab's longest label is a 64-byte name. **0.10.3** made dhancha's own `path` lines
  dormant so its lock pins commits — ⚠ crab has NOT adopted that convention; its five `path` lines
  are live, and the hazard above is the cost of that. Whether to follow dhancha is the operator's
  call; the block above is the evidence for the decision.
- **sadish 0.5.5 → 0.11.2** is six releases: styled strokes, gradient and pattern paint, exact
  coverage, inline `SdPath` points, the per-operation flatten budget, and **0.11.1's audit — four
  crashes, a heap overflow and a hang** in code that renders every glyph crab draws with a face.
  The bundle grows 85,815 → **471,512 B**. **rekha 0.3.10 → 0.9.0** pins sadish 0.11.2 exactly and
  its dist is cut against the inline-point port, so the two move together — the manifest says so
  now. 40,750 → **551,402 B**.
- **kashi 1.0.8 → 1.0.10** — no API change; `src/font_data.cyr` byte-identical. ⭐ But 1.0.10's own
  finding lands on crab's ADR 0003: **`dist/kashi.cyr` had never been published** — `dist/` was
  gitignored and no release attached it — so the ADR's written expiry (*switch to
  `modules = ["dist/kashi.cyr"]` the day crab needs runtime font loading*) would have FAILED to
  resolve against every kashi through 1.0.8. It is tracked, released and gated as of 1.0.10; the
  expiry is executable now. The manifest comment says so.
- ⛔ **daimon 2.1.3 → 2.1.4, and M7's index and tags STAY BLOCKED — with the root now known.** The
  handoff's *"likely root, NOT confirmed"* (a stale vendored snapshot; *re-run `cyrius deps` on a
  current pin*) is **REFUTED**: daimon 2.1.4 re-measured under 6.6.4 — identical 53 errors — and
  this release re-measured it under **6.6.6** in a scratch copy: still refused, **62 error lines,
  51 distinct undefined symbols** (the count grows with 6.6.6's larger
  `lib/syscalls_linux_common.cyr`, which is still being compiled for the agnos target). daimon's
  diagnosis: bote's `dist/bote.deps` sidecar names `syscalls_linux_common` as a stdlib leaf and
  `cyrius deps` prepends sidecar leaves **target-blind**. The fix is in cyrius distlib and/or bote's
  sidecar. ⇒ A pin move — this one included — is not it, and crab's `[deps.daimon]` block now says
  so beside the filing.

### Measured — the binary, and what DCE would return

| | 0.10.1 (6.6.4, old pins) | 6.6.6, old pins | **0.10.2** | Δ total | `CYRIUS_DCE=1` |
|---|---:|---:|---:|---:|---:|
| host | 1,097,664 | 1,277,920 | **1,636,136** | +538,472 (+49.1 %) | 608,040 |
| `--agnos` | 1,146,920 | 1,327,080 | **1,681,192** | +534,272 (+46.6 %) | 874,280 |

The toolchain is a third of the growth and the two draw bundles the rest; dhancha's 0.10.1 saw the
same shape (`smoke` +88 % plain, +9,632 B under DCE) and its advice is *build with `CYRIUS_DCE=1`*.
⚠ **crab's CI and `release.yml` build WITHOUT DCE**, and 2,149 unreachable functions (1,027,854 B)
now ride in the shipped agnos artifact. The DCE build was exercised, not just sized: `render_test`
under `CYRIUS_DCE=1` is **55 checks, 0 failed** at 546,352 B (1,566,256 plain). Turning it on for
the release is the operator's decision; the numbers are here so it is not made blind.

### Testing

- `cyrius test` **2,503 / 0**, unchanged. `render_test` **55 / 0** — built explicitly, its build
  log grepped for `undefined function` (none), because `cyrius test` never builds it. `cyrius fuzz`
  100,000 rounds ok. `fmt --check` clean on all eight files, one invocation each. `coverage`
  **322 / 371 fns, 86 %** ≥ 85. `vet`, `deny` (1 dep, 0 violations). `deps --verify` 49 / 0.
- ⭐⭐ **THE PRODUCTION RENDER IS PIXEL-IDENTICAL ACROSS THE MOVE.** `render_test` dumps the
  640x220 BGRA frame it asserts on; the dump from the 0.10.1 tree (6.6.4, its own pins, resolved
  from the tags) and the dump from this tree are **563,200 bytes, `cmp` clean**. A compiler two
  releases on and three draw-path deps — sadish six releases on, rekha five, dhancha four — changed
  **no pixel** of the bitmap-font frame. ⚠ That frame is `font = 0`; the face path is QEMU's.
- ⭐⭐ **QEMU `crab-face-test.py` PASSES on the new agnos binary** (2026-09-21, agnos 1.57.5,
  aethersafha 0.16.25): crab launched from the compositor, presented, and printed
  `crab: font /fonts/default.ttf 410820 bytes adv=9 upem=2048 i=4 m=13` — the face loaded through
  rekha 0.9.0, sadish 0.11.2 drew it, dhancha 0.10.4's frame arena carried it: one `font` line,
  navigations 1, view changes 2, **no fault, no allocator-failure text**. That is the
  `dh_frame_begin` arena contract the roadmap's bump note asked to see, seen.
- ⭐ **QEMU `crab-button-test.py` and `crab-pointer-test.py` — every crab arm that received a press
  answered correctly on the new binary; press DELIVERY at the aimed spot is the harness's coin.**
  Five runs of the new binary (face, pointer, button ×3) and one of the 0.10.1 binary for
  discrimination (agnos 1.57.5, aethersafha 0.16.25, TCG):
  - **button-test, new binary, run 3**: mask 4 → `crab: mark by middle click` (wire **3**, the
    compositor's one-shot spliced into crab's line), mask 1 → `crab: click`, mask 2 → `crab: context
    menu opened by pointer`; ARM 2 middle-on-pane **marks**; ARM 3 middle on a popup row **fires
    nothing**. Run 1 measured ARM 4 — the popup highlight followed the pointer across **6 rows**
    (`menu 1→2→3→4→5→0`) — and ARM 3's refusal as `crab: press btn 3 no action` in run 2.
  - **Runs 1 and 2 FAILED ARM 2** (a single middle press, no retry) and the verdict said so. The
    serial logs say why: the press was never forwarded — aethersafha's one-shot `a non-left button
    press had NO client content under the cursor` fired on the first probe and every later miss is
    silent by construction (`ae_btnx_nowin`, `src/main.cyr:659`). ⛔ **Discriminated before it was
    believed**: the **0.10.1 binary** run on the same kernel missed **all four** per-mask probes —
    left included — and landed ARM 2's one press. Same coin, different flip; the harness header
    records exactly this (*"seven runs went into aiming one"*). Not crab's, and not filed anywhere.
  - **pointer-test, one run**: ascended to `/`, left click resolved to a pane, `u` ×6 → 3 refreshes,
    `g` and `b` answered, **bare F10 opened the bar, View drove from the keyboard** (1 view change),
    bare Esc ×3 and Tab ×3 reached crab, Ctrl+Tab / Ctrl+F10 / Ctrl+Q went to the compositor,
    **no faults**. Its right-click arm was the same delivery miss (`NO client content under the
    cursor`, three tries) — UNMEASURED there, measured in button-test run 3 above.
  - ARM 4 (hover) read 6 rows in one run and nothing in two; the harness's own comment names that
    flake (*"3 rows once and 0 the next, from the same code"*). Measured once, on this binary.

### Fixed — stale text, cut rather than annotated

- `cyrius.cyml` — the rekha block opened *"crab does not draw with it yet (it passes `font = 0`)"*,
  false since 0.9.0. It names `crab_face` and the three calls now, and the sadish block carries the
  0.10.4 floor and the move-together rule.
- `README.md` — the 2026-08-26 correction block (*"crab … calls no `rekha_*` function anywhere …
  proportional text is M5, gated"*) contradicted the *Status* bullet twelve lines below it that
  said 0.9.0 closed exactly that; and *"No index, no tags, no semantic find, no dedup"* was written
  before `Shift+D` shipped in 0.10.0.
- `docs/development/state.md` — *Toolchain* still asserted a `6.6.2` pin against a manifest that
  read `6.6.4` (the 0.8.11 bump was never written into it — the file's own third rot, again); the
  dependency table listed seven deps at 0.8-era tags with sadish `0.5.4` and no daimon row; *Tests*
  said 2,012; *Known gaps* still carried *"the AI arc … declared nowhere"* and the `net` leaf (both
  closed at 0.10.0), the symlink *"refuse? report? recreate?"* question (answered by ADR 0004 at
  0.9.3) and the preview read *"on the selection path"* (moved to the tick at 0.9.4); and the
  *Version* section carried **117 lines of exact duplicate** (0.9.7 → 0.9.4 twice). Each rewritten
  to what is true, with the numbers above; the duplicate cut.
- `cyrius.cyml` said sadish and rekha were *"the only two"* tag-only deps; chitra has been the third
  since 2026-08-31. ADR 0003 gains a dated addendum for kashi 1.0.10's finding.

## [0.10.1] — 2026-09-14 — the sidebar can reach its own rows

> Cut on operator direction; the commit, the tag and the push are the operator's.
>
> ⚠ **Two defects found while designing M7's smart-folder section, and fixed as their own release
> rather than bundled into it.** Both are independently correct; one is a live wrong-on-screen bug on
> the shipped window size, and the other is currently unreachable but blocks the feature that would
> reach it.

### Fixed — ⛔⛆ four sidebar rows are below the fold, and nothing ever scrolled it

**MEASURED, not read.** `crab_sblst` appears **seven times** in `src/ui.cyr` and **not one is a scroll
call** — while the panes and the context column have been followed for releases
(`dh_list_scroll_to_sel(llst)`, `(rlst)`, `(crab_ctxlst)`).

```
an ordinary desktop: 7 places + 2 headers + 2 volumes  = 11 rows
the shipped 380x220: 174 px pane band / CRAB_SB_ROW_H 22 =  7 fit
                                                          --------
                                                           4 BELOW THE FOLD
```

⛔ **And the cursor walks there anyway.** `crab_sb_step` is bounded by `crab_sb_rows`, not by what is
visible — so the operator arrows onto a volume they cannot see highlighted, presses Enter, and the
pane goes somewhere they never saw selected. That is the wrong-on-screen class in the one surface
whose whole job is to say where you are about to go.

⚠ **Why it was easy to miss**: `crab_places_build` stat-checks each well-known directory and adds only
those that exist, so a headless box builds **three** places — 7 rows, which fits *exactly*. A developer
never sees it; a desktop with Documents, Downloads, Pictures and Music does.

⇒ One line, beside the context column's and carrying its reasoning: the sidebar is not scroll-driven,
so the only offset it can honestly have is the one that shows the cursor.

### Fixed — ⛔ a sidebar click that did nothing said nothing

The pointer road's `CRAB_PA_PLACE` arm had no `else`. Its **keyboard twin has answered "nothing to go
to" since 0.9.2**; the pointer arm just returned, leaving the operator unable to tell a click crab
ignored from one it never received — the exact ambiguity 0.9.5's `crab: press btn <n> no action` was
added to end, in the same surface.
⚠ Unreachable today, because every sidebar row has a path and `crab_sidebar_hit` refuses inert rows.
It becomes reachable the moment any row is pathless — which is what a smart-folder section is. Fixed
before the feature that would trip it, not after.

### Tests

**2,503 assertions** (2,490 → 2,503) and **render_test 53 → 55 checks**.

⛔⛆ **AND THE FIX WAS UNASSERTED WHEN IT FIRST LANDED — caught by mutation, not by luck.** Deleting
`dh_list_scroll_to_sel(crab_sblst)` left `cyrius test` at 2503/0 **and** render_test at 53/0. The
suite proves the *arithmetic* (eleven rows needed, seven fit, `crab_sb_step` walks past the fold) —
which is the motive, not the fix. Only a laid-out tree can show the list actually **moves**, and that
tree exists only in `render_test`. ⇒ A sidebar of 7 places in a deliberately short window, last row
selected, asserting a non-zero scroll offset. **Mutation-proven both ways: exit 1 without the fix,
exit 0 with it.**
⚠ The fixture uses a 140 px window rather than the shipped 220 px on purpose: at 220 an 8-row sidebar
overflows by two pixels, and a two-pixel margin is a poor thing to hang an assertion on. Same defect,
decisive arithmetic.

## [0.10.0] — 2026-09-14 — daimon is declared, the deferrals are swept, and duplicates are found

> Cut on operator direction; the commit, the tag and the push are the operator's.

### Decided — ⭐⭐ THE DAIMON RULING: DECLARED

`cyrius.cyml` carries `[deps.daimon]` pinned at **2.1.3**. `cyrius deps` resolves 7 deps; `--verify`
reports 50/50. **This was the oldest open item in the roadmap** and it gated M7 and M8 entirely.

⛔ **Declared, NOT linked, and that is the shape of the dependency.** daimon is a BINARY
(`[build] output = "build/daimon"`) and ships no `dist/` — there is no module to fold in, and there
must not be: linking an agent orchestrator into a file manager would put its HTTP server, its
scheduler and its federation code in crab's address space for the sake of a query. crab talks to the
**AF_UNIX socket daimon binds per agent** (`agent_ipc_new(agent_id, socket_dir)`), and agnos carries
the surface: `sock_connect` #47, `sock_listen` #56, `sock_accept` #57.
⚠ **No `modules` key**, deliberately — it would make `cyrius deps` fold a file that does not exist.
⛔ **And crab still runs without it.** The index is an enrichment, not a precondition: a box with no
daimon lists, copies, moves and deletes exactly as before, and the M7 surfaces must report that the
index is unavailable rather than failing — the rule the preview already follows for a file it cannot
decode: *say which kind of nothing this is.*

### Added — ⭐⭐ DUPLICATE DETECTION (M7), which crab does alone

The roadmap gated this on daimon and it never fully was — the gate row itself said *"daimon, **or a
content hash crab could do alone**"*. This is that half, and it is the half that still works on a box
with no daimon, which is exactly what the declaration commits crab to.

**`Shift+D`** scans the active listing and marks every duplicate **except the newest of each group**.

⛔ **IT MARKS. IT DOES NOT DELETE.** `Keep newest` names what *survives*; the rest get a mark and the
operator presses the verb — and the delete prompt then counts the set, names the first three and warns
about system directories, exactly as it does for a set marked by hand. A duplicate finder that deleted
would be the `/bin` incident with a better excuse.
⛔ **Not `d`.** `d` is delete, and putting a scan one un-shifted keypress from an irreversible verb is
the kind of adjacency that gets pressed by accident. The shifted row has existed since 0.8.9.

⭐ **The size pass is the pre-filter and it costs ZERO syscalls.** Two files of different lengths
cannot be byte-identical, and every entry is already `stat`ed by the deferred sweep — so only a size
*collision* is ever opened and read. Hashing all 1024 entries at the listing cap would be ~1.1 s of
I/O on the target.
⚠ **A size collision is a CANDIDATE, not a duplicate** — the hash decides. And **a hash match is not a
guarantee**: FNV-1a is 64 bits and not collision-resistant by design, so `crab_dup_same` requires the
size to match too (free, and independent of the hash), the read is bounded at 64 KiB, and the word
crab uses is **"duplicate"**, never "identical". The honest full answer is a byte-for-byte compare of
the candidates; the code says where it goes.

### Fixed — ⛆ the deferral sweep: `net` removed, four stale comments cut

- ⭐ **`net` is gone from `stdlib`.** setu dropped TCP at 0.8.4 and crab has pinned past it since
  0.4.5 — the leaf was dead for six minor versions while a comment said so and carried it anyway,
  "queued as its own change". This is that change.
  ⚠ **And the note was loose**: it said "measured clean"; the binaries are **not** byte-identical —
  both grew 16 bytes and 182,246 bytes differ, because dropping a leaf from the middle of the list
  changes the fold order. What is clean is what matters: no undefined symbol on either target, deps
  resolve, `deny` 0 violations, suite unchanged. *"Measured clean" and "byte-identical" are different
  claims and that comment blurred them.*
- ⛔⛆ **The A/B strip's comment said it "has no caller in `src/` and is NOT hit-tested"** — both halves
  false since 0.8.5, the release named two lines below it in the same comment. It has had its own hit
  test, its own `CRAB_PA_SWITCH` action and a press arm for five releases.
- ⛔ **`CRAB_COL_CHARW`'s "the day crab stops passing `font = 0`"** — that day was **0.9.0**, five
  releases back. The design held (every width followed `crab_char_w()` without being found); the
  comment just never noticed.
- ⛔⛆ **The 🦀 button's deferral named a gate that has since shipped without unblocking it.** It said
  *"it needs an icon path or proportional text, which is the M5 gate"* — 0.9.0 shipped proportional
  text and the button is still undrawable, for three reasons the old text did not name:
  `dh_draw_text_ink` walks one BYTE per glyph in **both** branches, `rekha_char_to_glyph` is BMP-only
  (U+1F980 is far past it), and Liberation Sans has no crab glyph. **The real gate is an icon path —
  a drawn shape, not a character.** Naming the wrong gate is worse than naming none: it made this look
  like it would fall out of work that has now been done twice.

### Tests

**2,490 assertions** (2,451 → 2,490). Three mutations planted and caught: the hash trusted without the
independent size check, unstattable entries grouping with each other, and `Keep newest` inverted.

⚠⚠ **TWO MUTATIONS SURVIVED AND ARE RECORDED AT THE ASSERTIONS, NOT HIDDEN.** Both are inside
`crab_dup_scan` and both survive for the same honest reason — they change what crab **opens**, not
what it **marks**:
1. Bypassing the size pre-filter (hash every entry) leaves the suite green. It is an optimisation, not
   a correctness property; the marks are identical either way.
2. Bypassing the `CRAB_KIND_FILE` guard leaves it green **even with two same-sized directories added
   to the fixture to catch it** — a directory opens and then fails to READ, so its hash stays 0 and it
   is never grouped. The guard is defence-in-depth; claiming otherwise would claim a proof the suite
   has not got.
⇒ Both predicates are pinned exhaustively; their *use* inside the scan is not. An I/O-count assertion
would need a syscall counter crab does not have.

## [0.9.7] — 2026-09-14 — H1: a cancelled copy deletes only what crab created

> Cut on operator direction; the commit, the tag and the push are the operator's.
>
> ⛔⛔⛔ **The oldest confirmed data-loss defect in this project, and its own code predicted it.**

### Fixed — ⛔⛔⛔ cancelling a MERGED copy deleted the folder it merged into, wholesale

`crab_walk_reroot_dtree` turns a cancelled tree copy into a **recursive delete** of the destination
root. Its own comment spelled out the invariant that made that safe — `crab_walk_begin` refused an
existing destination with `EEXIST`, so the root was **always one crab had just made** — and then
spelled out the consequence of losing it:

> *"If that guard in `crab_walk_begin` is ever relaxed to allow merging into an existing destination,
> THIS FUNCTION BECOMES A DATA-LOSS BUG and must be deleted in the same change."*

**0.8.7 relaxed exactly that guard** — to allow "copy this folder into one that already has a folder
of that name", the most ordinary thing an operator does with two panes — **and the delete was not
touched.** So cancelling a copy that merged into a folder the operator already had rerooted a
recursive delete onto **their** folder, taking every file crab never wrote.

**MEASURED ON IRON, BOTH WAYS** — `agnos/scripts/harness/crab-h1-test.py`, the image read back with
`debugfs` after shutdown, because asking crab whether crab destroyed something is not evidence:

```
                              SHIPPED                      FIXED
/zzkeep still on disk:        False                        True
operator's files surviving:   0/3  []                      3/3  [keep1, keep2, keep3]
cancel reported:              (nothing)                    crab: transfer cancelled rc 23
```

⇒ **The invariant is CHECKED now, not assumed.** `CRAB_OP_DMADE` is written at the only place that
knows — the `mkdir` in `crab_walk_begin` — and `crab_cancel_may_remove(dmade, root_len)` is the whole
decision, lifted into a pure function because everything that calls it is a walk step and walk steps
are not reachable from a host test.

⚠ **Only an explicit `1` authorises the removal, not any non-zero value.** A truthiness test would let
a stale record from some future field authorise deleting the operator's folder.

### Changed — a cancelled merge says what SURVIVED, instead of implying a clean undo

Every other cancelled tree copy takes its tree with it, so a bare *"transfer cancelled"* reads as a
clean undo of something that is half done — and the half that is done sits inside a folder full of the
operator's own files. `CRAB_FS_EMERGED` carries its own sentence: **"cancelled — what was already
copied into that folder is still there."**
⇒ crab does not delete that folder, and does not pretend it did. A partial merge the operator can see
and undo is the honest outcome; the alternative is deleting data they did not name, which is the
failure this project has already paid for once (`/bin`, 2026-09-03).

### Tests

**2,451 assertions** (2,440 → 2,451). Four mutations planted and caught: the shipped behaviour (any
root removed — H1 itself), truthiness instead of an explicit flag, the root-length check dropped, and
the cancel reporting `"done"` for a merge it left half-finished.

⭐⭐ **`crab-h1-test.py` is new and needs no pointer aiming at all** — crab's left pane opens on `/bin`
and sorts directories first, so the source folder is already selected and the whole scenario is two
keys: `c` then `Esc`. Run against the planted defect it reports the loss above; run against the fix it
passes.

## [0.9.6] — 2026-09-14 — a drag cannot outlive its listing

> Cut on operator direction; the commit, the tag and the push are the operator's.
>
> ⛔⛔ **Two data-loss defects, found in 0.9.5 and deliberately not bundled into it.** They share one
> root cause: **a drag is a claim about a ROW INDEX, and nothing kept the listing still.**
> `dragging` / `drag_pane` were touched in exactly three places — the motion arm, the press arm and
> the release arm — and **nothing in the key dispatch consulted drag state at all.**

### Fixed — ⛔⛆ a drop could move a file while a DELETE PROMPT was on screen

The release arm was gated on the LEFT BUTTON AND NOTHING ELSE. Neither `crab_pointer_blocked` nor
`crab_pointer_modal` was asked — though **the click has asked since 0.8.0 and the wheel since 0.8.5.**

```
press a row and hold → move 4 px (dragging = 1) → press `d`  (the prompt asks "delete <name>?")
                     → drag to the other pane  → release
```

The drop **moved a file, relisted BOTH panes, cleared every mark and CLAMPED both selections** — and
the `y` that followed answered a question about an entry that was no longer there. **The prompt named
one thing and the delete took another.** That is precisely the failure 0.8.0 closed for clicks and
0.8.5 closed for the wheel, arriving one input kind later and never revisited.

**MEASURED ON IRON, `crab-drop-test.py` ARM 1:**

```
crab: click
crab: prompt SYSTEM DIR! delete anuenue?  y = yes, any other key = no
crab: drop refused 2                       <- CRAB_DROP_BLOCKED
```

### Fixed — ⛔⛆ the keyboard could re-list the source pane mid-drag

Nothing gates the key dispatch on drag state, so **Enter or Backspace descends or ascends the pane
being dragged FROM while the button is still down.** `drag_row` then indexed a directory the operator
never dragged from — and the bounds check passed on any listing long enough.
⚠ **And the index is not identity even without navigating**: `crab_sort_entries` permutes in place, so
a sort key pressed mid-drag reorders the rows under the claim.

⇒ **THE NAME IS THE IDENTITY, NOT THE INDEX.** The press captures the row's name; the drop verifies
the row still holds it and refuses **out loud** when it does not. That is the discipline the delete
queue already states — *"Queuing by name removes that 'only safe because' entirely"* — and the one
`crab_goto` names when it clears marks: *"a mark that outlives its listing points at whatever now
occupies that index."* A drag outlives its listing more easily than either.

### Fixed — ⚠ a press on a pane HEADER left the previous drag armed

The arming sat inside `if (hr >= 0)` with no `else`, and `crab_hit` records **headers** (row `-1`) so
a header press focuses its pane. So a header press left the PREVIOUS drag armed, its row pointing
into a listing the operator had since moved on from. ⛔ And nothing else ever disarms one: the release
arm is the only clearer, and the compositor **drops a release delivered outside crab's content rect**
— so releasing on the titlebar leaves `dragging = 1` with no button down. The arm now disarms
unconditionally before it re-arms.

### Changed — the decision is one pure function, and the refusals differ

`crab_drop_ok(blocked, dragging, frompane, topane, row, count, name_ok)` answers `CRAB_DROP_GO` /
`_BLOCKED` / `_MOVED` / `_NONE`. Every line of the release arm is inside main.cyr's agnos `#ifdef`
where no host test reaches — the 0.9.3/0.9.4/0.9.5 lifting rule — so the decision moved out and the
suite drives it exhaustively.
⛔ **The question is asked BEFORE the target, and the order is the contract**: an operator with a
prompt on screen must not be told *"dropped nowhere"* and sent looking in the wrong place. ⚠ But
`dragging` outranks even that, so an ordinary click while a prompt is up stays **silent** rather than
raising a refusal notice.

### Tests

**2,440 assertions** (2,421 → 2,440). Five mutations planted and caught, each one the shipped
behaviour: no modal gate; trusting `drag_row` without the name; no bounds check; a header row allowed
to drop; and the target asked before the question.

⭐⭐ **`agnos/scripts/harness/crab-drop-test.py` is new.** ARM 1 — the data-loss path — **passes on
iron**, twice in separate runs.
⚠ **ARM 2 is honestly UNMEASURED and the harness says why**: its own Backspace re-lists the pane, so
every retry starts from a different layout and the loop does not converge. It reliably shows that
**nothing moved**; it does not reliably reach the name check. That half is proven in the suite by
mutation instead.
⛔ **And the drag had to be promoted with two generous moves, not one nudge.** A single 20 px move
armed the drag on some runs and not others from identical code: `crab_drag_started` needs only 4 px,
but the promotion happens in the POINTER_MOVE arm, so it needs a motion EVENT delivered while the
button is down — and aethersafha dedupes motion.

## [0.9.5] — 2026-09-14 — the middle button marks, and a popup's highlight follows the pointer

> Cut on operator direction; the commit, the tag and the push are the operator's.
>
> ⚠ **The roadmap's premise was wrong, and checking it was the first job.** The item read *"the
> middle mouse button does nothing"*. It has DISMISSED popups since 0.8.5 — both `CRAB_PA_DISMISS`
> returns are button-blind and never test `btn` — and three doc sites said otherwise in prose while
> no test pinned the one behaviour it had.

### Added — ⭐⭐ MIDDLE-CLICK MARKS THE ROW UNDER THE POINTER

**MEASURED ON IRON FIRST, because the whole half rested on a claim about five repos.** agnos captures
the HID bitmap, aethersafha 0.16.24 forwards bits 0–2 as `wire = kernel_bit + 1`, setu carries it
opaque — but `crab-pointer-test.py` had only ever proved button **2**. Button 3 had never been
observed anywhere in this stack. It arrives:

```
QEMU mask 4 -> crab: mark by middle click     (wire 3, middle)
QEMU mask 1 -> crab: click                    (wire 1, left)
QEMU mask 2 -> crab: context menu opened      (wire 2, right)
```

⛔ **It is an ALIAS OF `Space`, not a verb of its own.** `crab_pa_accel(CRAB_PA_MARK)` answers `0x2C`
and the arm synthesises that one key through the one binding table — so there is no second
`crab_mark_toggle` call site to drift, and *"middle is Space, not `d`"* is a **host assertion** rather
than a literal buried in main.cyr's agnos `#ifdef`.

⛔⛔ **And it is deliberately the safest gesture on the table.** The sibling that numbers these buttons
warns: *"a reader who assumes X11 puts **Delete** on the middle button."* X11 is 1/2/3 =
left/**middle**/right, so an operator with X11 muscle memory aims middle exactly where crab's **right**
lives — and `Delete` is literally a row in the menu right opens. A mark is self-inverse, moves nothing
on disk, and costs one press to undo. Middle stays **refused** on a popup row, a bar cell, the door,
the A/B strip and the sidebar — every surface where the thing underneath is a verb. Gated on iron:
*"ARM 3: middle inside the open popup fired a verb: False"*.

⚠ **The row is GUARDED, not clamped.** `crab_hit` records pane **headers**, which answer row `-1` with
`pane_hit` 1 — indistinguishable inside `crab_pointer_action`. The shipping MENU arm survives that
because opening a menu needs no row; this one ACTS, and `-1 < ln` is true, so a missing guard would
mark `buf + -1 * CRAB_REC_SZ`.

### Added — ⭐⭐ A POPUP'S HIGHLIGHT FOLLOWS THE POINTER

Until now a popup's highlight moved only for the arrow keys, so the operator aimed with the mouse and
committed with a key that was pointing somewhere else — the one gesture in crab where the two input
roads disagreed about what was selected.

⛔⛆ **IT DOES NOT ENGAGE UNTIL THE POINTER HAS MOVED, AND THAT IS NOT A REFINEMENT.** The popup is
placed **at** the pointer, and `dh_place_at_point` **FLIPS** it above the anchor when it would overhang
— which at crab's shipped 380x220 it usually does. So the cursor that opened the menu ends up sitting
in the MIDDLE of it, over a row nobody aimed at. A hover that engaged immediately would move `Enter`
onto that row before a muscle moved. `crab_hover_armed` uses the drag threshold already in this arm:
a pointer that has not travelled has not expressed an intent. Gated on iron — *"hover lines BEFORE the
pointer moved: none"*.

⛔⛆ **AND `-1` MEANS FOUR THINGS, OF WHICH ONLY TWO MEAN "HOLD".** `dh_list_index_at` answers `-1`
both for an INERT row (a separator, a greyed verb) and for a point OUTSIDE the list. Treating them
alike leaves the highlight on the last live row the pointer crossed — so sweeping off a menu to see
the pane underneath leaves `Enter` armed on whatever was passed last, and the natural exit from a menu
anchored at the pointer is down-and-right, straight through `Delete`.
⇒ **INSIDE on an inert row HOLDS** (what the keyboard already does — `crab_mb_item_move` steps over
disabled items). **OUTSIDE RESTORES** the open-time choice. Both visible in the iron trace:

```
crab: hover menu 3 → 2 → 1 → 0 → 1 → 2 → 3 → 4 → 5 → 0
```

the sweep up through a flipped menu, back down through all six rows, and the final `→ 0` as the
pointer leaves the popup and `Enter` means `Open` again.

### Fixed — ⛔⛆ the context menu opened on an EMPTY pane with NO HIGHLIGHT AT ALL

Both open sites set `menu_sel = 0`, which is `CRAB_MI_OPEN` — and on an empty pane `crab_menu_enabled`
refuses everything that needs a row (`if (count <= 0) { return 0; }`) before it reaches its OPEN
branch. So `crab_overlay` marked row 0 inert, `dh_list_select` refuses an inert row, and the menu drew
no highlight and `Enter` did nothing. **Measured**: `enabled(OPEN, count=0) = 0`, first enabled = `5`.
⇒ `crab_menu_first` — the twin `crab_mb_item_first` has given the menu BAR's drop-down since 0.8.6.
The context menu simply never got one. Fixed at **both** open sites, or the two roads would open the
same menu differently.

### Fixed — ⛆ four false claims in one comment, and one in the README

`src/ui.cyr`'s note on `crab_mlst` said `crab_menu_list` had no caller in `src/`, that no pointer path
to the menu existed, that `menu_open = 1` happened only under the Menu key, and that the `POINTER_BTN`
arm never read a button code. **All four were falsified by 0.8.5 and left standing for four releases.**
`crab_menu_list()` is the FIRST surface the press arm asks, and since this release it is what the hover
asks on every motion. Cut, not annotated. The README's *"the middle mouse button does nothing"* is
corrected to what it actually did and now does.

### Tests

**2,421 assertions** (2,384 → 2,421). Five mutations planted and caught: the `-1` collapse (leaving a
popup holds the last row swept), the hover engaging unarmed, `crab_pa_accel` mapping middle to `d`
instead of Space, `crab_menu_first` ignoring whether a row is enabled, and the popup-row arm firing on
any button. ⚠ **One assertion's expiry fired and was inverted, not deleted** — `"a pane, middle:
nothing yet"` had asserted `CRAB_PA_NONE` since 0.8.5 and is now `CRAB_PA_MARK`. That is what an
expiry is for.

⭐⭐ **`agnos/scripts/harness/crab-button-test.py` is new** and gates all four arms: which buttons
arrive and as what number, that middle reaches the mark arm, that middle fires no verb inside a popup,
and that the highlight follows the pointer only after it moves.

⛔ **Three harness lessons, each of which produced a WRONG answer first:**
1. **A `-2000` home and 0.3 s settles found nothing.** The proven aiming needs `-4000` and **0.8 s per
   move** — the kernel accumulates deltas between drains and the compositor clamps the NET move, so a
   pin and a walk folded into one drain land on the titlebar. On that evidence I would have filed a
   false cross-repo blocker saying middle does not arrive.
2. **Press ORDER was the whole test.** aethersafha's diagnostic is one-shot, so whichever non-left
   button arrives first is the only one it ever names; pressing right before middle spends it on wire
   2 and makes "dropped" indistinguishable from "forwarded silently".
3. **A popup left open by one probe decides the next one in the wrong branch.** A middle press with a
   menu up is `DISMISS`, not `MARK` — the harness measured crab correctly and the SCENARIO was wrong.
   And a one-directional sweep only works for one popup placement: the flip means the menu may lie
   entirely above the cursor, which is why ARM 4 now sweeps both ways.

## [0.9.4] — 2026-09-14 — the preview shows the SELECTION, and costs nothing to do it

> Cut on operator direction; the commit, the tag and the push are the operator's.
>
> ⚠ **The roadmap asked for one thing and the work found three.** The item was *"the 64 KiB
> dimension+EXIF read is on the idle tick, not the selection path"*. Moving it exposed that the
> preview column was reading **process-wide "last touched" state** rather than anything derived from
> the selection — and two of those readers were WRONG AND SHIPPING. They are one change, not three:
> the column is now a pure function of the selected path.

### Fixed — ⛔⛆ the preview's thumbnail lagged ONE FILE behind the selection, permanently

**MEASURED ON THE HOST before a line was changed**, two real decodable PNGs:

```
A: state 1  redraw fired: 1        <- CRAB_TH_OK, the tick redraws
B: state 1  redraw fired: 0        <- OK -> OK, the guard skips entirely
slot(A)=0 slot(B)=1 slot the keypress frame drew=0
```

Arrowing from image A to image B, the keypress frame asked `crab_thumb_pixels()` — *"what the last
STEP was about"* — which is still A. Then the idle tick stepped to B, but the gate was
`if (tafter != tbefore)` and **`OK -> OK` is not a change, so no frame was drawn at all**. A's picture
stayed under B's name until some unrelated event forced a repaint; while arrowing, the thumbnail is
permanently one file behind.

⚠ **This is 0.8.2's own bug, still live.** Its fix comment sits at the gallery site and names the
mechanism exactly — *"it PERSISTED: the preview's own redraw fires only when its state CHANGES, and
OK -> OK is not a change, so nothing corrected it. ⇒ Look the selection's slot up directly"* — and
**only 1 of 8 render sites obeyed it.** ⇒ `crab_pv_redraw_due` makes the SLOT part of the answer: a
different slot is a different file, whatever its state says.

### Fixed — ⛔⛆ `CAMERA: Canon EOS R5` stayed on screen under a TEXT FILE's name

Also measured before changing anything, with a real EXIF JPEG:

```
1. select shot.jpg   : CAMERA row drawn: YES -> "Canon EOS R5"
2. arrow to notes.txt: CAMERA row drawn: YES -> "Canon EOS R5"
```

`crab_preview_dims` returned at its `crab_is_image` gate **before** clearing the two process-wide EXIF
buffers, and the CAMERA/SHOT rows are drawn *outside* the column's `is_image` block — their only gate
is `load8(cam) != 0`. So the last photograph's camera sat under whatever was selected next.
⇒ EXIF now lives **in the cache slot beside the dimensions it was read with**, and `crab_pv_publish`
publishes `""` for a non-image, an empty pane or a closed preview. Not patched — unreachable.

### Changed — ⭐⭐ the read moved to the idle tick, and a MISS now costs what a HIT costs

```
SELECTION PATH (crab_pv_publish, warm cache): 4 us per keystroke
SELECTION PATH (cache EMPTY, all PENDING)   : 4 us per keystroke
IDLE TICK    (crab_pv_step, forced cold)    : 8 us per file, off the keystroke
```

⚠ **The invariance is the result, not the number.** Before, a keystroke onto an unread image cost an
`open` + `read(64 KiB)` + `close`; onto a read one, nothing. ⛔ **And the host figure understates the
target by two orders of magnitude** — crab's own recorded measurement for the much cheaper *stat*
sweep is **~1.1 ms per entry on agnos**, which is why that sweep was deferred in the first place. The
honest claim is not "12 µs became 4 µs"; it is **the selection path no longer opens files at all.**

⭐ **And arrowing BACK is now free.** The old memo held ONE entry, so reversing direction re-read
every file. The cache holds 128, keyed by full path — 56,320 bytes taken once.
⭐ **A preview the window is too narrow to draw now reads nothing**, which it did not before:
`crab_pv_should_step` gates on `crab_preview_fit`, the EFFECTIVE state, not the operator's want.

### Fixed — ⛆ a short read memoised a wrong negative

`crab_preview_dims` had a bare `sys_read` with no loop. A read may return fewer bytes than asked for
at any time, so a JPEG whose SOF sat past the returned prefix was recorded as *"no dimensions"* — and
that is a REMEMBERED refusal, never retried, so the file stayed dimensionless for the session.
`crab_pv_read_all` loops. ⚠ **Its test does not prove the loop and says so at the assertion** — a
regular local file never short-reads, so replacing the function with the old one-liner leaves the
suite green. Recorded rather than deleted; agnos, spanning ext2 blocks over NVMe, is where it matters.

### Fixed — ⛆ the mascot drew the preview from a record that could be minutes stale

It was the one render site of eight that published **no preview state at all**, and `crab_rs_reset` is
never called inside the loop — so it drew DIMENSIONS, the thumbnail and CAMERA from whatever was last
written, possibly about a different directory. It fires precisely when the operator has been idle long
enough to be reading the screen. Since the publish is now a lookup there is no cost argument against
it, which is why it was left out before.

### Fixed — ⛔⛆ THE LAYERING GATE WAS HALF A GATE, AND HAD BEEN ALL ALONG

`src/render_test.cyr` includes `ui.cyr` ALONE to enforce that the render path never calls up into
`app.cyr` — the rule the source records being broken five times. **Measured this release:**

| an up-call to an undefined… | compiler | build exit | render_test |
|---|---|---|---|
| **constant / enum** | `error:` | **1** | not reached |
| **function** | `warning:` | **0** | `53 checks, 0 failed` |

So the rule was enforced for enums and **not** for functions — the more likely mistake of the two, and
the one this release's design would have made. ⇒ `ci.yml` now fails on the warning. Mutation-proven
both ways: planting `crab_exif_raw_buf()` into `crab_tc_claim` gives exit 1, clean gives exit 0.
⛔ **And the step had a second hole found while fixing the first**: `cyrius build … | tee` hides the
build's exit status (a pipeline's status is the last stage's), so a *failed* build ran the **stale**
binary and reported its old green count. Caught doing exactly that. Both are fixed.

### Tests

**2,384 assertions** (2,345 → 2,384). Five mutations planted and caught, including the two shipping
bugs: `crab_pv_redraw_due` without the slot (the thumbnail lag), `crab_pvc_claim` not zeroing the
payload (the stale camera), `crab_pvd_row` drawing nothing for PENDING (the flicker),
`crab_pv_should_step` ignoring whether the column is on screen, and `crab_pv_state_for` never deriving
PENDING. ⚠ **A sixth mutation SURVIVED and is recorded at its assertion** — see the short read above.

⭐⭐ **`agnos/scripts/harness/crab-preview-test.py` is new, and MUTATION-PROVEN on iron.** Real PNGs of
two different known sizes go into the image; the new `crab: pv <name> state <s> <w>x<h>` oracle is
emitted by the DRAIN and by nothing else, so a line appearing at all proves the tick did the work —
which a screenshot cannot, since both designs end with numbers in the column.

```
ARM 1: zzpic1.png -> state 1, 137x42 (want 137x42)      the IDLE TICK read it
ARM 2: zzpic2.png -> state 1, 320x200 (want 320x200)    its OWN dimensions, not the previous file's
ARM 3: reads per file: {'zzpic1.png': 1, 'zzpic2.png': 1}
```

Every file read **exactly once**, and the other 48 entries in `/bin` produced no line at all — the
non-image gate costs nothing. Run against a planted re-read, ARM 3 reports the spin it exists to
catch: files read **up to 36 times each**, and both measurement arms unmeasured.

### What the operator sees

The DIMENSIONS row is drawn from the STATE, not the value. PENDING draws `DIMENSIONS: ?`, OK draws the
numbers, NONE draws no row — so while arrowing over images the row is **always present** and only the
value settles. ⛔ **A row has no reserved height: its presence IS the layout**, so a row that appeared
a tick after every selection would shove MODIFIED, CAMERA and SHOT down as it arrived. The `?` is not
a new affordance — SIZE and MODIFIED already show it on every listing whose stat sweep has not caught
up. The preview is already a surface that admits it does not know something yet.

## [0.9.3] — 2026-09-14 — crab can see a symlink, and the write layer stops guessing what one is

> Cut on operator direction; the commit, the tag and the push are the operator's.
>
> ⇒ Recorded as **[ADR 0004](docs/adr/0004-symlinks-are-shown-preserved-and-dereferenced-on-copy.md)**,
> because the roadmap carried this as a decision — *"refuse, report, or recreate"* — rather than a
> dependency.

### Added — ⭐⭐ the listing shows which entries are links, for no extra syscall

⛔ **readdir cannot tell crab a link is a link, and that is a kernel fact rather than an oversight.**
agnos's `ext2_readdir_at_sys` writes a 64-byte record and sets byte 63 with
`var t = 0; if (ftype == 2) { t = 1; }` — one bit, DIR or not — so `EXT2_FT_SYMLINK` arrives
indistinguishable from a regular file. crab has never had the information to show.

⭐ **But the stat sweep already visits every entry**, so the answer is free: `crab_stat_batch` walks
32 entries per idle tick until none is pending, whatever the sort mode — the synchronous storm is
only for a SIZE or MTIME sort, which cannot order anything without the data. It asks **`lstat`** now,
and the kind goes into the type byte crab already had (`0` file, `1` dir, now `2` link).

A link is marked `@` in the listing — ASCII, which is not a style choice: 0.9.0 made the two faces
disagree about every byte ≥ 128, so a non-ASCII marker would render one way on the host build and
another on the target. The KIND column says **Link**, and it outranks the extension: a symlink named
`cover.png` is a link, and calling it an Image in the one column whose job is to say what a thing IS
would be the `/` marker problem in words.

⛔ **`lstat` first, `stat` as the fallback, and the fallback is load-bearing**: `lstat`#102 is
**ext2-only** — FAT and exFAT cannot represent a symlink and the kernel returns -1 rather than
succeeding on a surface `stat`#33 does not have. crab lists FAT volumes, so an lstat-only sweep would
leave a whole disk unstatted. ⚠ And it loses nothing: a filesystem that cannot hold a link cannot
have one to miss.

### Fixed — ⛔⛔ deleting a link deleted what it POINTED AT, and that is older than this release

**MEASURED ON IRON, BOTH WAYS** — `agnos/scripts/harness/crab-symlink-test.py`, a real ext2 symlink
in the image, the image read back afterwards with `debugfs` rather than crab asked for its own
opinion. Against the pre-fix question, deleting `/bin/zzlink → zztarget/`:

```
ARM 3: crab: delete zzlink -> done          <- the operator is told it worked
ARM 3: /bin/zzlink still on disk: True      <- the link is still there
ARM 3: debugfs /bin/zztarget -> 0/3 files survived: []
```

**Three files the operator never pointed at, gone, on a reported success.** The single-entry delete
verb read `if (ddir != 0)` and handed anything non-zero to `crab_walk_begin(CRAB_OP_DTREE, …)`, which
**type-checks nothing**: it takes the name, builds a path, and the walk readdirs it — through the
link, into the target. Then the final `rmdir` of the root refused (the root is not a directory), so
the link survived its own deletion and `done` was reported anyway.

⚠ **It is NOT a regression from making links visible.** `crab_stat_one` used to call `stat`, which
FOLLOWS a link — so a link to a directory was already stored as type `1` and already took this
branch. Visibility did not open the hole; **the `!= 0` audit is what found it**, and it is the reason
that audit was worth doing properly rather than once.

⇒ The decision moved out of the `#ifdef` into **`crab_delete_plan(kind)`** — `CRAB_DEL_TREE` for a
directory, `CRAB_DEL_ONE` for everything else, forever. Same remedy as the planner below: a branch no
host test could reach became a function the suite drives exhaustively.

### Fixed — ⛆ the same `!= 0`, in six more readers, and the first audit found one of them

The type byte was two-valued and its readers disagreed about how to ask — some `== 1`, some `!= 0`.
The first sweep grepped `== 1` and `!= 1` and **missed the whole `!= 0` and `== 0` families**, which
is where the damage was. Every reader of `CRAB_REC_TYPE` was then enumerated rather than pattern-
matched — 22 sites, each traced to what it decides:

- ⛔⛔ **the delete verb** (above) — a link became the root of a recursive delete.
- ⛔⛔ **the delete prompt** — `!= 0` meant "directory", so deleting a link asked *"delete this
  **FOLDER** `<name>` and everything in it?"*. `crab_del_prompt` exists because five system binaries
  left an iron box on one confirm, and **the prompt names what dies**; naming a folder and its
  contents over a single link is that same failure with the words rearranged. A link now gets its own
  sentence — *"LINK `<name>`? the target is not touched"* — which says what **survives** as well as
  what goes.
- ⛔ **the transfer planner**, at both call sites — a link planned as `CTREE`/`MOVEDIR`, so
  `crab_walk_begin` would `mkdir` a destination FOLDER named after the link and walk a tree that is
  not there. `crab_transfer_plan` now takes the **kind** instead of a boolean each caller derived:
  two chances to derive it wrong, in code no host test can reach, became one classification the suite
  can drive.
- ⛔ **Enter / Open** — `!= 0` sent a link to `crab_descend`, which refuses anything that is not a
  directory, and the site dropped the `-1` with no else arm: the key **did nothing and said nothing**,
  which the operator cannot tell from a keypress crab never received.
- **two thumbnail readers** — gated on `== 0`, so a link was offered a preview it has no bytes for.
- **`crab_fs_delete`** — chose `rmdir` on `is_dir != 0`, so links would have been **undeletable**.

⚠ **Widening a field is only safe where every reader agrees how to ask** — and the way to know they
agree is to enumerate them, not to grep for the shapes you happen to remember writing. All of it now
goes through `crab_kind_mark`, `crab_delete_plan`, `crab_transfer_plan` and `== CRAB_KIND_DIR`.

⭐ **The wire records what the operator was asked.** `crab: prompt <text>` is new, because the prompt
is the last thing between a keypress and an irreversible verb and nothing outside the screen knew
what it said — so a prompt naming the **wrong thing** was invisible to every harness on the target.
One `crab_line_*` write, not four: the serial console is shared unserialised by three processes.

### ⛔⛔ And the hazard everyone expects is not real — the same one bit is why

A symlink pointing at an **ancestor** does not make the recursive walk loop forever, on either target:
the walk descends only where the type byte says `1`, and neither agnos (`ftype == 2` only) nor the
host path (`dt == 4` only) ever sets that for a link — and 0.9.3 writes `2`. **The one-bit type byte
that hid links is the same one that bounded the walk.** `CRAB_PATH_MAX` bounds it a second time:
`crab_join_n` refuses at depth 127 before `CRAB_WALK_DEPTH_MAX` can fire. Asserted, so it stays true.

### Decided — what each verb does with a link

- **Delete preserves it** — `unlink` removes the LINK, never its target. POSIX's rule, and agnos's
  `unlink`#30 follows it. ⛔ **This was the intent and not the behaviour** until `crab_delete_plan`
  landed; see *Fixed*, and the harness that now holds it.
- **Move preserves it** — `rename` moves the link itself.
- **Copy dereferences it** — `open` + `read` copies the target's bytes, which is what `cp -r` does. It
  is now a decision rather than an accident, and the operator can see that an entry is a link
  **before** acting on it, which they could not before.

⚠ **Recreate is possible and deliberately not done.** agnos has `symlink`#63 and `readlink`#70, both
with cyrius peers — but both are **ext2-only**, and crab's whole two-pane premise is copying between
volumes. A recreate that works on one side of a copy and fails on the other needs a decision about
what a link *becomes* when it lands somewhere that cannot hold one. Deferred with the primitives
confirmed present, rather than half-built.

### Tests

**2,345 assertions** (2,314 → 2,345). Six mutations planted and caught: a link marked `/` like a
directory; the kind test comparing the whole mode rather than its top nibble (which would answer
"file" for every real symlink on disk); the KIND column calling a link an Image; the delete prompt's
`!= 0` calling a link a FOLDER; the planner's boolean tree-copying a link; and `crab_delete_plan`'s
`!= 0` walking one.

⭐⭐ **And an iron arm, because the decision being right is not the same as the consequence being
right.** `agnos/scripts/harness/crab-symlink-test.py` is new: `mkfs.ext2 -d` carries a host symlink
into the image as a **real ext2 symlink**, crab is driven to it by keyboard, and the image is read
back with **`debugfs`** afterwards. It gates the prompt's wording, that Enter on a link answers out
loud, that the link is deleted, and — the arm that matters — that the target directory and all three
of its files are **still on disk**. Run against the planted defect it reports the data loss above; run
against the fix it PASSes. ⚠ `crab_stat_one`, `crab_readdir_into` and the whole delete verb are inside
the agnos `#ifdef` with no `#else`, so **no host test can reach any of it** — which is why the
decisions are lifted into pure functions the suite proves exhaustively *and* why the consequence
needed iron.

### Fixed — ⛆ the render gate stopped compiling, and the gate is what was supposed to catch this

`CRAB_KIND_FILE` / `_DIR` / `_LINK` were declared in `app.cyr` — which **includes** `ui.cyr`. So the
moment the render path started naming them (the `@` marker, the KIND column, the delete prompt) they
were invisible to it, and **`src/render_test.cyr` stopped building**. That file includes `ui.cyr`
ALONE for exactly this reason: `ui.cyr` sits BELOW `app.cyr` and the render path must never call up.
⇒ **The architectural gate worked. Nothing ran it** — `cyrius test` discovers `tests/*.tcyr` and does
not build `render_test.cyr`, so a green suite said nothing about it, and `ci.yml` would have caught it
on the push. The three values now live in `path.cyr`'s `CrabRec`, beside `CRAB_REC_TYPE` itself: the
byte's values next to the byte's offset, at the bottom layer every reader can see.
⚠ Both files already carried the rule in a comment — *"layering follows the include order, not the
other way round"* — which is the part worth keeping: a written-down rule is not a gate.

⛔ **The harness does not count keypresses.** A first run pressed Down 80 times to "clamp at the
bottom", landed on `whirl`, and deleted it: keys are lost between the compositor's per-frame drains,
so N presses are not N rows and a blind count silently aims at the wrong file. It homes on the answer
instead — press `d`, read the name out of `crab: prompt`, cancel, step, repeat — so it cannot confirm
a prompt naming the wrong entry.

## [0.9.2] — 2026-09-14 — `Go` is filled, and an open menu stops acting on the pane underneath it

> Cut on operator direction; the commit, the tag and the push are the operator's.

### Added — ⭐⭐ `Go` — a thin projection over the sidebar's destinations

`Go` has sat on the bar and **empty since 0.8.0**, because the canvas draws it and crab had no honest
way to fill it. Its rows are now every selectable sidebar destination, and picking one sends the
active pane there.

⛔ **THE MENU IS A DISCOVERY SURFACE, NOT A SECOND SET OF VERBS** — the rule every other bar menu
keeps by rewriting `u` to an existing key. `Go` cannot: its items are **paths**, and no key means
*"go to /home"*. So it reuses the other road that already exists — `crab_sb_path` and the same
send-the-pane code the sidebar's Enter runs. ⭐ **`synth_goto` mirrors `synth_u` exactly**: a pointer
pick becomes a *key* so one binding table answers it; a `Go` pick becomes a *path* so one navigator
does. The sidebar's Enter now files a destination too, so there is **one navigator with two callers**
rather than twenty duplicated lines that can drift.

⛔ **A flat list, not `crab_sb_rows`.** The sidebar interleaves two inert headers so a reader can tell
a disk from a directory; a menu of six rows cannot spend two on headings, and a header in a menu is a
row the keyboard steps over for no information.

⛔ **A volume is labelled by its PREFIX, not its name**, which closes a recorded defect: *"two FAT
volumes render as two identical rows."* `crab_vol_dedup` removes *aliases* of one filesystem, not two
filesystems that share a label. In a sidebar those rows at least sit under a heading and differ by
position; in a menu two identical rows are two identical choices. `/mnt/fat` and `/mnt/exfat` are
unique by construction.

### Fixed — ⛔⛆ an open drop-down let **every** key through, not just `d`

The roadmap recorded *"`d` is not consumed by the drop arm — so the delete prompt would draw
UNDERNEATH the menu."* Reading the arm, it was worse: it handled six keys — Esc, Up/k, Down/j,
Left/h, Right/l, Enter — each clearing `u`, and **every other key fell through to the main binding
table with `u` intact**. With a menu open, `c` and `m` started a transfer, `r` and `n` opened an edit
sheet, Backspace ascended and Space marked — all under a popup painted over them.

⛔⛔ **The delete case defeats a rule written after real loss.** `crab_del_prompt` exists because
*"THE PROMPT NAMES WHAT DIES"* — written after five system binaries left an iron box. A prompt drawn
under an opaque menu names what dies to **nobody**, and the next keystroke answers it.

⇒ `crab_mb_drop_key` — the same shape `crab_sb_key` already uses, lifted into a pure function the
suite can reach, eating the mutating set and refusing out loud. ⭐ **And the two lists are tied
together by an assertion**: every verb the sidebar eats, the drop-down eats too. Two surfaces that
borrow the keyboard from the panes must refuse the same verbs, or one of them is a hole.

### Fixed — the drop-down had no HEIGHT rule, and `Go` is the first menu that needed one

`crab_mb_drop_fit` checked width from 0.8.3 and never height, because every menu until now had three
to five rows and always fit. `Go`'s length is the **model's**.

⛔ **The number is smaller than anyone guesses.** At the shipped 380×220: 220 − the bar's 22 − the
status line's 22 − the placer's margin leaves 170 px, and a menu row is 26 — **six rows**. The
roadmap recorded an 11-to-17 row `Go` being *"clamped and flipped to cover BOTH the bar and the
status line"*; seventeen rows want 442 px, twice the window. ⚠ The status line is subtracted on
purpose: `dh_place_at_point` clamps against the *surface*, so without it a drop sits legally on top of
the line that says what is selected — and since 0.9.1 that line also holds the door that closes the
menu.

⚠ **Refused, not truncated.** A destination list silently cut at six hides places the operator has —
the same failure as a listing that drops files.

### Changed — `Go ▸ Parent` is absent, and not for the reason the roadmap gave

The condition read *"`Parent` is absent rather than dead at `/`"*, and the first draft of the model
carried a `hasparent` flag to hide the row at the root. **Wrong shape.** Parent is a **verb** — it is
Backspace, a key that already exists — and every other bar menu is a selection over the verb space. A
menu holding both verbs and destinations needs two pick paths in one list, which is exactly the
complexity that kept `Go` empty. ⇒ It is absent **everywhere**, not just at `/`, because it is not a
destination. If it is ever wanted on the bar it belongs beside `Open`, with its accelerator shown.

### Tests — ⭐ QEMU caught a bug the suite could not

**2,314 assertions** (2,257 → 2,314). Four mutations planted and caught: `d` leaking again; the height
cap removed; the status line not subtracted; and a volume indexing by row rather than within its kind.

⛔⛆ **The navigator sat ABOVE the dispatch first, and only the target showed it.** A `Go` pick sets
`synth_goto` from *inside* the key dispatch, so a navigator above it consumed nothing until the
**next** key arrived — the menu closed, the pane stayed put, and the destination fired later against
whatever was pressed next. On QEMU that read as `Go` doing nothing: `crab: go 3 destinations` and no
`crab: place`. Every line of it is inside the agnos `#ifdef`, so the suite was green throughout.

⭐ `agnos/scripts/harness/crab-go-test.py` — **PASS**. `F10 → Right ×2 → Enter → Enter` sent the pane
to `/` and listed **8 entries**; `d` ×3 with a drop-down open produced no delete activity at all.
⚠ **Driven by the keyboard deliberately** — 0.9.1 spent seven runs failing to aim a relative pointer
at a rect, and `F10 → Right → Enter` is a road `crab-columns-test.py` already drives.

⚠ **Two oracles were weak and are now dispositive**, both found by a harness that could not tell two
causes apart: `crab: go <n> destinations` (because "the menu did nothing" is either a broken pick or
an empty model, and nothing outside crab can distinguish them), and `crab: place <path> ok <n>
entries` (the line used to print whether or not the listing succeeded, and a refusal only ever
reached the status line).

## [0.9.1] — 2026-09-14 — the door: the menu row has a second way in, and `F10` stops being the only one

> Cut on operator direction; the commit, the tag and the push are the operator's.

### Added — ⭐⭐ a mark in the status line that reveals the menu row

`F10` was the only door — and for seven releases it was **no door at all**, because aethersafha
claimed that key until 0.16.25: the bar crab shipped in 0.8.0 was reachable by nobody. A surface with
one invisible door is a surface most operators never learn exists.

⛔ **It costs ZERO rows, and that constraint chose its home.** The bar is collapsed by default
because *"at the shipped 380×220 every row is contended"* — a permanent bar would spend a listing row
forever to show six words, and a permanent **button row** would spend one to show a mark. So the door
lives *inside* the status line: the line became a `BOX_H` of `[door][text]`, the same shape
`crab_pane` already uses when the A/B strip shares a header with the path. The text flexes; the mark
does not.

⛔⛆ **THE TOGGLE IS THE Z-ORDER, NOT A STATE BIT.** The door sits *below* the bar branch in
`crab_pointer_action`, so a press on it while the bar is already shown never reaches its own arm —
the bar branch sees a press that is not on a bar cell and answers `DISMISS`. Open when closed, close
when open, from one hit test and no flag of its own. Placing it any higher would make the door the
only piece of chrome that cannot put the bar away. *(The mutation that moves it up returns `DOOR`
where `DISMISS` is expected, which is the assertion that pins this.)*

### ⛔⛆ It is not a crab glyph, and the roadmap said it would be — that line was mine, from 0.9.0

The gate line always read *"needs an icon path **or** proportional text"*. 0.9.0 shipped proportional
text, so a line went in saying the face had unblocked this. **It had not, three times over:**

- `rekha_char_to_glyph` returns 0 for any codepoint above 65,535 — in its own code, with the comment
  *"format 4 is BMP-only"*. **U+1F980 is 128,896.**
- The shipped Liberation Sans carries **no format-12 subtable** (three cmap subtables, all format 4/6).
- `dh_draw_text_ink` walks **one byte per glyph in both branches**. There is no UTF-8 decode on the
  draw path at all.

Any one of the three is fatal on its own. ⇒ The icon path: **three filled boxes, no font**. ⭐ And the
road to an actual crab is recorded rather than guessed at — **CANVAS**, which crab already ships for
thumbnails (`dh_canvas_new(&crab_thumb_draw, pix)`). *An icon is a glyph with no font.*

⚠ **Using no font also sidesteps something 0.9.0 made live**: the same byte draws CP437 through kashi
and Latin-1 through rekha — byte 0xF0 is `≡` on the host build and `ð` on the target — so a mark made
of characters would render differently on the two. crab's drawable alphabet is printable ASCII.

### Fixed — ⛔⛆ a serial line must be ONE write, because the console is shared

Measured on QEMU, not reasoned. crab composed two oracles out of many small `crab_say` calls, and the
other two processes on the console spliced into both:

```
crab: font /fonts/default.ttf 0                      <- the byte count lost its digits
crab: door ptrscan: first sample handed to ring 3    <- the KERNEL, mid-line
```

The source already recorded one instance of this — *"the 2026-08-30 burn has crab's own exit line
spliced into the middle of aethersafha's frame banner"* — and 0.9.0 and 0.9.1 both walked into it. A
harness reading either line gets a parse failure **indistinguishable from the feature being broken**,
and three QEMU runs went on the door before that was the diagnosis.

⇒ `crab_line_reset` / `crab_line_s` / `crab_line_u` / `crab_line_say`: compose into a buffer, emit
once. ⚠ A single `crab_say` is still fine — it is the **sequence** that is not atomic.

### Tests

**2,257 assertions** (2,250 → 2,257 plus the door's own group). Four mutations planted and caught:
the door moved above the bar branch (the toggle dies); the fit rule stopping asking whether the text
keeps its floor; the stale-pointer clear removed; and the status text no longer flexing.

⭐ **On target, measured every run**: crab builds the door and reports it laid out at
`crab: door 0 196 22 22` under the real face.

⚠ **A pointer press ON it is NOT measured, and the harness says UNMEASURED rather than PASS or FAIL.**
Seven QEMU runs went into aiming one. The rect is in crab's *surface* coordinates; the monitor moves
a **relative** `usb-mouse` in *screen* coordinates; the compositor chooses the window origin; and
`mouse_move -4000 -4000` does not reliably home — an unchanged script proved delivery at (200,80),
(100,80), (200,130) and (200,180) across four runs, and the pane edge at y=154 in one run and y=104
in the next. Sweeping the mapped region in both axes delivered nothing, which means presses were
being **dropped**, not missing. ⇒ Reporting FAIL would assert the door is broken, which the harness
has no evidence for and the suite has nine assertions against. `crab-door-test.py` stays, with all
seven runs' findings in its header, for a session that can place an absolute pointer.

## [0.9.0] — 2026-09-14 — A REAL FACE: crab draws in Liberation Sans on the target

> Cut on operator direction; the commit, the tag and the push are the operator's.
>
> ⭐⭐ **The roadmap item that has been open since M5.** It was blocked by two things a day ago, both
> outside crab, both filed in the repo that owned them, and both closed within 24 hours. This is the
> crab half.

### Added — ⭐⭐ crab opens `/fonts/default.ttf` and measures in it

`crab_face()` opens the kernel-owned namespace agnos 1.57.2 added — **Liberation Sans Regular 2.1.5,
unmodified, 410,820 bytes, SIL OFL 1.1** (⚠ the licence text must travel with any redistribution) —
once, before the first frame, and hands the result to all eight `crab_render` call sites.

⛔ **Read front-to-back in one pass, because that is the contract and not a style.** `/fonts` is a
`VFS_MEMFILE`: `lseek` returns **-1**, so a reader that seeks — to size the file first, or to retry —
gets an error rather than a rewind.

⛔ **Opened once, outside any draw, and that ordering is load-bearing.** dhancha 0.10.0 installs
`dh_falloc` as sadish's allocation hook for the duration of one `dh_draw_text_ink`; `rekha_font_open`
follows that hook, so a face opened *inside* a draw would live on the frame arena and die at its
first reset. The call is spelled out before the first render rather than left to argument evaluation.

⚠ **A full buffer is treated as a truncation, not a success.** With no `lseek` there is no way to ask
how much is left, so filling the read cap is the only signal there is — and handing rekha a partial
font is how a face half-parses.

### ⛔⛆ The coincidence that would have fooled the obvious test

**Liberation Sans's `n` advances 1139/2048 em — at 16 px that is 8.9, which rounds to exactly 9: the
advance of kashi's CP437 8×16 cell.**

So every width crab derives — `crab_col_name_min()` and its six siblings, the two-pane threshold,
the column set — comes out **numerically identical to the bitmap face's**. A harness that asserted
"the advance changed" would have failed against a perfectly working face, and one that asserted
"the layout moved" would have failed too.

⇒ The oracle prints `i` and `m` as well: **`i=4 m=13`**. A monospace face cannot produce that. *The
columns did not move; what goes in them did.*

### Fixed — ⛔⛆ the truncation marker was still lying, and that was the whole point of the item

`crab_name_cell` truncated at a **character count**. A count describes a monospace column exactly and
a proportional one only on average — `crab_col_chars(120)` answers 10 for this face, and **ten `m`s
measure 160 px in a 120 px column.** The name would be clipped by the column while carrying a `~`
claiming it had been cut to fit: a cut name, marked, *at the wrong place*, still hiding the
difference between two files. That is precisely the 0.5.0 defect the marker exists to prevent —
`agnos-kernel-build.log` and `agnos-kernel-build.tmp` rendering as one identical row.

⇒ **`crab_name_cell_px` measures**, greedily, one glyph at a time, and the count form converts into
it. ⛔ The marker's own width is reserved **before any name byte is accepted** — filling the column
and then appending `~` overflows by exactly one glyph, and the mark saying "this was cut" would be
the thing clipped off the edge. ⚠ At `font = 0` every advance is `CRAB_COL_CHARW`, so this reduces
exactly to the arithmetic it replaced: the bitmap build is unchanged and the suite's numbers did not
move.

### Added — integer logging, because crab had none

`crab_say_u`. ⛔ **Arena-free, and that is why it is not `crab_u2s`**: that one formats through
`dh_falloc(24)` — the per-*frame* arena — and the caller that needs numbers here runs before the
first frame, where `dh_frame_arena_get()` is still 0.

### Tests — ⭐ QEMU, because `/fonts` exists on no host

**2,213 assertions** (2,203 → 2,213). Three mutations planted and caught: the count converting at
kashi's 9 instead of the face's advance; the marker's width not reserved; and — under the synthetic
proportional face — wide glyphs cut early, narrow ones left whole.

⚠ **One mutation PASSED and is recorded rather than deleted.** Deleting `crab_face`'s memo early
return leaves the host suite green: with no `/fonts` the load fails at its first step either way, so
"asked once" and "asked twice" give identical answers. ⇒ The memo's real gate is **ARM 2 of
`crab-face-test.py`** — exactly one `crab: font` line in a whole session. Unmemoised, crab would
`open`/`read`/parse a 410 KB file **on every frame** from an allocator with no `free()`.

⭐ **`agnos/scripts/harness/crab-face-test.py` — PASS.** The face loads as
`/fonts/default.ttf 410820 bytes adv=9 upem=2048 i=4 m=13`; exactly one load in the session; crab
navigates and switches views under it; no faults and no allocator failure — ⛔ **the last of which
dhancha 0.10.0 is what makes passable**: before it, every label allocated a full-surface canvas per
frame and a session in a face would have exhausted the heap.

⚠ **Two bugs of my own, caught by the suite and kept in the source as comments**, because both are
the kind that look right: exiting the greedy loop by assigning `j = n` ended the walk and destroyed
the count in the same statement (the marker was then placed by re-scanning for a NUL nothing had
written — SIGSEGV); and summing `dh_text_advance` *before* checking `crab_font == 0` dereferences a
null font, because the bitmap face is a different branch rather than a fallback value.

## [0.8.11] — 2026-09-14 — the 6.6.4 stack, and both blockers on a real face cleared within a day

> Cut on operator direction; the commit, the tag and the push are the operator's.

### Changed — the toolchain and five dependencies

`cyrius` **6.6.2 → 6.6.4**, and with it **sadish 0.5.4 → 0.5.5**, **rekha 0.3.7 → 0.3.10**, **kashi
1.0.7 → 1.0.8**, **dhancha 0.9.29 → 0.10.0**. rupa (0.1.7), setu (0.8.9) and chitra (1.0.3) had not
moved.

⛔ **sadish and rekha are not optional company for dhancha 0.10.0 — they are a floor.** dhancha's
hand-off measured it against crab 0.8.10 and the answer is a refusal, not a warning: without them the
build stops with `refusing to emit binary with 2 reachable undefined function(s)` — `sd_alloc_set` and
`sd_canvas_blit_at`.

⚠ **6.6.4 matters for a specific reason**, recorded because it is the kind of thing that looks like a
routine bump: **6.6.3 silently corrupts even-length string literals ≥ 64 KB**. agnos found it while
generating the embedded face, filed it in cyrius, and made its kernel hash-verify the font bytes
rather than trust them. crab is past it.

⭐ **6.6.4 records the toolchain IN THE LOCK** — `cyrius	6.6.4`, one new line, where previous locks
recorded none. The toolchain pin is now part of the locked, verifiable state rather than living only
in the manifest.
⚠ **The bump added no vendored leaf and dropped none**, checked explicitly because the roadmap's
toolchain-sync item records that a bump CAN add one untracked (6.5.41 brought `lib/hashseed.cyr` in
that way). Every `lib/` file the new graph produced is tracked; the lock's only structural change is
the toolchain line above.

✅ **Check four, re-run because the graph moved**: every `path =` override disabled, all seven deps
cloned at their declared tags — the lock went **3 → 7 commit-pinned**, which is the tell — and both
binaries came out **byte-identical** to the override build. The suite is 2,198 / 0 in that tree too.

### Fixed — ⭐⭐ the expiry fired, and the assertion is inverted: a face frame now costs ZERO

0.8.10 asserted a **defect in another repo** — that dhancha's scalable text path allocates a
full-surface canvas per label per frame — and wrote the assertion so that *"the day dhancha routes
that canvas through the frame arena, this test FAILS and must be inverted."*

**dhancha 0.10.0 did exactly that, and the assertion fired.** Three moves across three repos: sadish
0.5.5 put every per-call `alloc(` behind an `sd_alloc` / `sd_alloc_set` hook; rekha 0.3.10 draws its
outline scratch from the same seam; dhancha installs `dh_falloc` as that hook for the duration of one
`dh_draw_text_ink` and sizes the canvas to clip ∩ surface ∩ run instead of to the surface.

⇒ **A warm frame under a real proportional face now costs the global heap exactly 0** — the same
claim the bitmap loop has made since M1.5, on the branch that could not make it before.

⭐⭐ **dhancha's hand-off predicted crab's failure by name and to the byte**, from measurements taken
against crab 0.8.10 rather than from reading crab's source: not `scost > 0` but
`arena_capacity_total(farena) == cap0`, *got 468,040, expected 16,384*. It was right. **That is what a
filing with a gate behind it buys** — the fix arrived with crab's own test result already in it.

Three things that note taught, all now in the suite:

- ⛔ **A warm-up face frame is required before measuring**, on the same surface and width. A cold one
  chains ~452 KB of arena chunks (glyph paths at ~4.3 KB each) — that is the arena **growing**, which
  `arena_reset` then reuses forever, and it is not a leak. sadish also re-grows a per-row accumulator
  once for any canvas wider than one it has seen, so a narrow warm-up leaves that cost in the number.
- ⛔ **The face must be opened OUTSIDE a draw.** `rekha_font_open` follows the scoped hook, so a face
  opened inside `dh_draw_text_ink` would live on the frame arena and die at its first reset.
- ⚠ **The arm moved below the bitmap assertions, and that order is now load-bearing** — a face frame
  extends the chunk chain, which is precisely how it broke `cap0` above.

### ✅ And the other blocker closed too — agnos 1.57.2 ships a kernel-owned `/fonts`

Filed 2026-09-13; resolved the same day. ⭐ **The operator's ruling** — *"rekha is that thing... but
has yet to get Kernel support"* — named the answer, and agnos chose a mechanism the filing had not
imagined: the face is **embedded kashi-style**, not staged as an asset. rekha 0.3.8 generates it,
`kfont_init` assembles it at boot into a 2 MB direct-map region and **verifies it by FNV-1a-64 against
the generator's hash** before opening the namespace.

**Liberation Sans Regular 2.1.5, unmodified, 410,820 bytes, SIL OFL 1.1** — ⚠ the licence text must
travel with any redistribution. ⇒ **`/fonts/default.ttf`** is the stable contract to code against;
`/fonts/LiberationSans-Regular.ttf` is the same bytes under the provenance name. Read-only, a
`VFS_MEMFILE` fd, ⛔ **`lseek` is -1 — read it front-to-back in one pass.**

⭐⭐ **THE FILING'S POSTURE IS WHY THIS IS THE RIGHT MECHANISM.** It stated the need in one sentence
and explicitly declined to design agnos's answer, citing the VOLUMES precedent. agnos then picked
something better than anything crab would have asked for. *Declining to approximate is what got the
right primitive built* — twice now.

### ⇒ `0.9.0 · A real face` is unblocked

Nothing outside crab stands in the way. What remains is crab's own: open `/fonts/default.ttf`, hand
the face to `crab_render`, and prove a warm frame still costs zero — ⛔ **on QEMU**, because that file
exists on no host, and because the one existing template in the stack reads a host path and would
fall back silently on the target while looking finished.

### Tests

**2,198 assertions** (2,196 → 2,198) — and the two that changed are the inversion, not additions.
Every other gate green on the new stack: render_test 53 / 0, fuzz 100k, coverage 88 %, vet, deny,
`deps --verify` 50 / 0, and check four byte-identical.

## [0.8.10] — 2026-09-13 — every width is derived from the font, proved against a proportional face — and the two things that actually block a real one

> Cut on operator direction; the commit, the tag and the push are the operator's.
>
> ⚠ **THIS IS THE HALF OF `0.9.0 · A real face` THAT IS NOT BLOCKED, AND IT IS NUMBERED HONESTLY.**
> An operator cannot newly read crab in a proportional font — that needs a face crab has no way to
> obtain (below). What changed is that **crab is now correct under one, and the suite proves it.**
> Call it 0.9.0 if you would rather; the work is the same and the blockers are unaffected.

### Changed — ⭐⭐ the character count is the constant; the pixel width is derived

Seven widths read like this: `CRAB_COL_NAME_MIN = 90;  # 10 chars`. That is the whole defect in one
line — **the thing that governs is ten characters**, and 90 is what ten characters happen to measure
in the one face crab draws with. Nine pixels per character is a property of kashi's CP437 8×16 cell
and of nothing else.

⛔ **And none of them fails loudly under another face — they pick a WRONG LAYOUT.** A NAME column
narrower than ten characters cannot tell `agnos-kernel-build.log` from `agnos-kernel-build.tmp`,
which is the exact confusion `crab_name_cell`'s `~` was written in 0.5.0 to prevent.
`crab_two_panes_fit` splits a window that can no longer hold two honest listings. A grid cell clips
the name it exists to show.

⇒ The counts are written down (`CRAB_COL_NAME_CHARS = 10`) and the widths are computed —
`crab_col_name_min()`, `crab_col_size_w()`, `crab_col_mtime_w()`, `crab_pv_w()`, `crab_col_ctx_w()`,
`crab_grid_cell_w()`, `crab_gal_cell_w()`. ⚠ **Functions, not constants**: a Cyrius `enum` member must
be a literal, and a width that depends on the frame's font is not one. At `font = 0` each returns
precisely the literal it replaced (10×9 = 90, 6×9 = 54, 17×9 = 153) — `render_test`'s 53 pixel checks
pass unchanged, which is what "renders identically" means here.

### Added — ⭐⭐ a synthetic proportional face, which retires an admission 0.8.8 had to make

0.8.8 routed every width through `crab_char_w()` and then admitted, in its own test comments, that it
could not prove the central claim: at `font = 0` no host test can tell *"asks the font"* from
*"divides by the constant"*, because they are the same number, and building a face meant a TTF crab
has no way to reach.

There is still no TrueType face anywhere in the AGNOS stack. **So the suite builds one** — the way
`rekha/programs/hmtx_test.cyr` and `dhancha/programs/text_test.cyr` each build theirs: head, maxp,
hhea, hmtx and cmap, assembled byte by byte. ⚠ No `glyf`, and none is needed — a width question asks
rekha for **advances** and never rasterises a glyph.

⛔ **The advances are deliberately unequal**, because equal ones would prove only half of it. `m`
advances 16 px and `n` 12, so `"nn"` and `"nm"` are the same **length** and different **widths** — a
`length × advance` implementation cannot tell them apart, and every truncation decision rests on
telling them apart. Under that face the whole chain is watched to move: the advance becomes 12 rather
than 9, the NAME floor 120 rather than 90, the MODIFIED column 204, and `crab_cols_for_width(297)` —
the bitmap face's three-column width — drops to two columns.

### Fixed — ⛔⛆ the zero-allocation gate was measuring the branch that was not running

crab's M1.5 headline is *"a rendered frame costs the global heap ZERO bytes."* Every render in that
test passes `font = 0`, which takes dhancha's **bitmap** path. The **scalable** path is a different
function body and it allocates: `dh_draw_text_ink`'s `font != 0` branch opens with
`sd_canvas_new(sd_surface_width(sds), sd_surface_height(sds))` — a full-surface canvas per label, per
frame — plus a sadish path per glyph, all from the global bump allocator that has no `free()`.

⇒ The moment crab passed a real face the headline would have become false **and the gate would not
have noticed** — *"a gate that covers one state proves one state"*, exactly as the roadmap warns. The
fixture face makes that branch reachable from a host test for the first time, so the cost is now a
measured number in crab's own suite. ⚠ **The assertion says the scalable path COSTS heap, and it
carries its own expiry**: the day dhancha routes that canvas through the per-frame arena, this test
FAILS and must be inverted. That is how a blocker in a sibling repo gets a gate in this one.

### Documented — the Latin-1 limit, which is a DISPLAY limit and not crab's

The scalable path walks one **byte** per glyph: `load8(text + i)` is handed to rekha as a codepoint,
so byte 0xC3 draws as U+00C3 rather than as half of a two-byte sequence. There is no UTF-8 decode in
dhancha or rekha. `crab_text_w` matches it byte-for-byte **deliberately** — a measurer that decoded
UTF-8 while the drawer did not would put the caret and the truncation where the ink is not.

⚠ **crab cannot type such a byte** — `crab_key_char` tops out at `'z'` (122), asserted. ⛔ **But crab
can display one**: a name comes from a readdir record, and a UTF-8 filename on an ext2 volume is an
ordinary thing to find. Under the bitmap face the same byte indexes CP437 and draws a different wrong
glyph, so the face does not introduce this — it makes it legible.

### ⛔⛔ And what actually blocks `0.9.0 · A real face` — neither of them crab's

Found by asking where a face would come from **before** writing the code that loads one.

1. **There is no TrueType face in the stack, and nothing stages one onto the target.** Zero `*.ttf` /
   `*.otf` / `*.ttc` across every first-party repo; the `agnos` repo contains no occurrence of "ttf",
   "truetype" or "sfnt" in any script, manifest or doc; `agnos/build/rootfs` has no `/usr`, no
   `/share`, no font directory. ⚠ kashi is not the escape hatch — it owns AGNOS's *bitmap console*
   fonts by design and ADR 0003's written expiry is about bitmap loading, which has not fired.
   ⛔⛆ **The obvious template is a trap**: the one caller that feeds `rekha_font_open` real file bytes
   reads `/usr/share/fonts/liberation/LiberationSans-Regular.ttf` — a **host path that does not exist
   on AGNOS**. Copied into crab it would work on the host build, fall back silently on the target,
   and look finished. ⇒ **agnos** owns the staging; **the operator** owns which face, under what
   licence. ⚠ rekha 0.3.7 constrains the choice: `glyf` outlines only, format-4 BMP `cmap`.
2. **dhancha's scalable draw allocates per call, outside the frame arena** — the defect above.
   ⇒ **dhancha** owns it.

⭐⭐ **OPERATOR RULING, 2026-09-13, which sharpens (1) and is recorded because it changes what the ask
is**: *"rekha is that thing... but has yet to get Kernel support."* ⇒ **rekha IS the designated answer
for proportional text, and the gate is an agnos-side arc that has not been walked** — so this is
**not** a "choose a font, check the licence" question, and the framing above was mine rather than the
operator's. crab states the need and does not design agnos's answer, per the VOLUMES precedent where
crab filed, declined to approximate with a probe, and agnos minted `mountlist`#104.
⇒ **Both blockers are FILED**, one each, in `docs/development/issues/` — because an issue about
another repository that lives only in this one is an issue nobody who could act on it will read.

### Filed

Both **in the repo that owns the fix**, with a copy in crab beside each — an issue about another
repository that lives only in this one is an issue nobody who could act on it will read.

- **agnos** — `agnos/docs/development/issues/2026-09-13-no-proportional-face-on-the-target.md`
  ([crab's copy](docs/development/issues/2026-09-13-no-proportional-face-on-the-target.md)).
- **dhancha** — `dhancha/docs/development/issues/2026-09-13-scalable-text-allocates-per-call-outside-the-frame-arena.md`
  ([crab's copy](docs/development/issues/2026-09-13-dhancha-scalable-text-allocates-per-call.md)).

### Tests

**2,196 assertions** (2,158 → 2,196), three mutations planted and each caught: the NAME floor reverted
to a pixel literal, the preview column minting its own literal again, and `crab_text_w` pricing at
`length × advance` instead of measuring. ⚠ **No QEMU arm, and that is stated rather than skipped**:
nothing agnos-only changed, no new code entered the `#ifdef`, and `render_test`'s 53 pixel checks
passing unchanged is the proof that what ships still draws exactly as it did.

## [0.8.9] — 2026-09-13 — Shift: capital letters in names, and a sheet that can finally accept its own language

> Cut on operator direction; the commit, the tag and the push are the operator's.

### Added — ⭐⭐ a Shift latch, so a name crab writes can hold a capital letter

`crab_key_char` has taken a `shift` flag since the rename field was built, and mapped `a..z → A..Z`
for it. Its **one** production call site passed a hard-coded `0`, under a comment reading *"there is
no shift state on the wire yet."* That was true when written and had quietly stopped being true.

⛔ **The wire was never the problem, and half the written claim was wrong the whole time.** `mods`
really does carry only the press/release edge and nothing else — but a modifier's **own** edge
arrives as its own key event, and aethersafha 0.16.25 exempts those edges from the Ctrl-chord
swallow **on purpose**, saying so in its own source: *"a client that wants Shift state has no other
way to learn it."* crab has been receiving Shift since that release and throwing it away. 0.8.7's
`crab_key_is_modifier` — added to stop crab *acting* on those edges — already named this as the
sequel in its comment: *"Shift for capital letters would read this same usage as STATE, not as a
keystroke."* This is that reading.

⛔⛔ **A MASK, NOT A BOOLEAN, AND THE DIFFERENCE IS A SEQUENCE A TOUCH-TYPIST MAKES.** Hold
LeftShift, then hold RightShift, then release LeftShift: the operator is still holding a shift key.
A single flag cleared on that release and the next letters came out lower case mid-word, with both
hands on the keyboard and nothing visibly wrong. Two keys, two bits, held while either is down.

⛔ **The release-clear is a guarded subtraction, and an unpaired release is measured on this stack,
not hypothetical.** The 2026-09-13 QEMU investigation recorded that *"a claimed key's release is
forwarded while its press is not"* — the compositor swallows a press it acts on while `input_map`
returns `IA_NONE` for the release, so the release travels alone. Unguarded, one stray release turns
mask 1 into mask −1: `crab_shift_held` then answers yes forever, every letter is capitalised, and no
key is down.

⚠ **A lost release sticks, and that is stated rather than defended against.** The wire can drop an
edge; recovery is one more press-and-release of a shift key. A timeout would be a second answer to
*"is shift down"*, free to disagree with the wire — the lesson `crab_sb_focus_eff` already carries.

### Fixed — ⛔⛆ the batch-rename sheet could not accept the language it advertises

The sheet's label reads `Rename pattern:  # = number, * = old name`. **Neither character could be
typed into it.** `#` is Shift+3 and `*` is Shift+8, and `crab_key_char`'s entire shifted number row
returned 0 under a comment reading *"the shifted row is symbols crab does not need"* — while crab
needed two of them by name, in a label it puts on screen.

A surface that describes a language its own input cannot produce is worse than a missing feature:
the operator reads the instruction and then cannot follow it. It was invisible while the shift flag
was hard-coded to 0 — the row was unreachable either way — and would have become a live dead key the
moment the latch worked.

⇒ The row is filled, **all ten**, because half a row is an arbitrary line the operator cannot see and
would have to discover one dead key at a time. ⛔ **The suite pins `#` and `*` against
`crab_batch_name` itself**, not against two literals that look right: the assertion types the
character Shift+3 produces, runs it through the expander, and checks it expanded to the index. If
either operator ever moves, the key that types it must move with it or the test fails.

⚠ **Every character the row can type is a legal name** — `crab_name_ok` refuses exactly `/`, `.`,
`..`, empty and unterminated — and none is a shell hazard here, which is worth stating because it
usually is: crab never builds a command line. `sys_spawn_path(path, len)` and `sys_open(path, len,
…)` take a pointer and a length, and `sys_system` appears nowhere in `src/`.

### Added — the sheet says when it opens

`crab: edit open <label>`. It was the one interactive surface crab opened in silence — every other
one announces itself (`crab: cd`, `crab: view columns`, `crab: context menu opened by pointer`) and
this one printed nothing until it **committed**. That is a diagnostic gap on the exact surface where
a keystroke stops being a command and becomes text: a lost `n` leaves the operator typing a name into
the binding table, where `b` toggles the sidebar and `d` asks to delete something.

`crab: edit commit` now names what was written, too — `-> done` says a rename happened, not what it
was renamed to.

### Changed — the overwrite policy's second reason expired, and is corrected rather than dropped

0.8.7 justified arming *all* with `a` partly because **shift was not on the wire**, making `R` for
replace-all *"unreachable on this stack."* That reason is now false. ⇒ **The design does not change,
because the reason that survives is the one that was load-bearing anyway**: a capital is an
**invisible mode**. `R` and `r` differ by a key the operator is holding and the prompt cannot show,
and this is a prompt where the next keystroke can destroy a file. `a` toggles, `[all]` is visible,
and the answer after it applies to everything.

### Tests — ⭐ QEMU, because the ordering is invisible to the suite

**2,158 assertions** (2,104 → 2,158), every claim mutation-proven — including the two-key sequence, a
stray release that must not borrow, both batch operators against the expander, and `'0'` surviving a
range widened from `0x1E..0x26` to `0x1E..0x27` (the planted defect made it come out as `':'`).

⛔⛔ **What the suite cannot see, and what the harness is for.** In `src/main.cyr` the latch is fed
the **raw** edge three lines *before* `crab_key_is_modifier` zeroes a modifier's `kacts`. Track
first, suppress second. Get that backwards and every Shift **press** reaches the latch looking like a
**release**, the latch never sets — and **the suite stays completely green**, because all of it lives
inside `#ifdef CYRIUS_TARGET_AGNOS` with no `#else` and nothing includes `main.cyr`.

⭐ `agnos/scripts/harness/crab-shift-test.py` — **PASS**. `Shift+A Shift+B Shift+3 Shift+8` into New
folder committed exactly **`AB#*`**: the latch, the ordering, and both batch operators in one line.
`Shift+A` then `b c` committed exactly **`Abc`** — the press latched and the release cleared. Ten
shift edges arrived and crab acted on **zero** of them, so a modifier is still state and never a
keystroke. No faults.

⚠ **Its first run measured `nnnAB#*` and that was the harness, not crab.** The retry loop pressed `n`
until the sheet opened, could not tell that it had, and every extra press typed a literal `n` **into
the name**. That is what produced `crab: edit open` — the retry now reads crab's own line. A correct
latch reported as a failure by a broken harness is the same cost as the reverse.

## [0.8.8] — 2026-09-13 — the COLUMNS view, one reader for the 9 px advance, and a roadmap with an order

> Cut on operator direction; the commit, the tag and the push are the operator's.

### Fixed — ⛔⛆ the guard against half-wiring a fourth view was itself half-installed

`crab_view_is_grid` was added in 0.8.2 under a comment reading *"one predicate, asked in all three
places, so a fourth view cannot be added and half-wired."* It was asked in `src/main.cyr`'s three
ARROW sites and **nowhere else**: `crab_pane` and `crab_render`'s scroll round-trip each still tested
`view != CRAB_VIEW_LIST`. A fourth view id would have compiled and **rendered as a GRID, scrolled as
a GRID, and taken LIST arrow semantics** — precisely the half-wiring that function exists to make
impossible, in the file that introduced it, and the same defect 0.8.2 fixed, inverted.

⇒ **A negation is not a predicate.** `!= LIST` answers a question about one id and silently decides
for every id that comes after it. Found by asking what a new id would actually do **before** adding
one, which is the only time that question is cheap to answer.

⚠ **The two sites are not equally load-bearing, and the suite says which is which.** Reverting
`crab_pane` fails the pin. Reverting the scroll round-trip alone **changes nothing today** — on a
vertical LIST the grid branch reduces exactly to the list branch (`dh_grid_cols` clamps to 1, and
`dh_grid_cell_h` reads the same `DH_W_CELL_H` slot `dh_list_new` writes `row_h` into), so no test can
catch it. It is fixed anyway: depending on that coincidence is depending on dhancha's internals, and
a horizontal list would break it. **The mutation is how we know which claim we could make**, and the
test says so at the assertion rather than implying a proof it has not got.

### Added — ⭐⭐ COLUMNS: a fourth view, and the design question M5 left open

`g` now cycles a fourth time. The active pane grows a narrow **context column** to its left showing
the parent directory with the directory you are standing in marked.

⛔⛔ **IT IS A VIEW MODE OF ONE PANE, AND THAT IS A SAFETY DECISION RATHER THAN A LAYOUT ONE.** The
roadmap carried columns as *"gated on crab's own two-pane model — a design question, not a
dependency"* since M5. The question, stated properly: miller columns as N independently navigable
panes make `active_pane` something other than a 0/1 — and `active_pane` is a 0/1 that the **entire
M4 write layer** resolves every copy, move and delete against. A drag from column k into column k+1
would plan a `crab_fs_move` of a directory **into its own subtree**. That is a data-loss question
wearing a layout question's clothes, and no amount of renderer cleverness answers it. **K = 2 with
one driven column answers it by construction**, and the canvas agrees — it draws pane A as columns
with pane B as the preview. ⇒ *If miller ever goes N-deep, the gate is the write layer, not the
renderer.*

What follows from that, and is asserted:

- **The driven listing is unchanged** — the same LIST, the same selection, the same hit-test, the
  same verbs. The view adds a column; it does not add a mode to anything else.
- ⛔ **The context column never holds focus.** dhancha paints a selected row ACCENTED when its list
  has focus and MUTED when it does not, and that difference is the entire answer to *"which listing
  do my arrow keys drive"*. ⚠ Two assertions pinned the outcome and a mutation showed they were being
  carried by CALL ORDER — `crab_pane` runs afterwards and sets focus itself — so the test now asks
  `crab_col_ctx` directly, with focus parked somewhere known. **An order is not a contract.**
- ⛔ **A click on it lands nowhere.** It is not in `crab_hit`'s walk, for the reason the A/B switcher
  strip is not: a pointer path means an answer to *"which pane is this"*, and that is the 0/1 again.
- ⛔ **It is dropped, not squeezed.** `crab_cols_fit` derives its threshold the way every other fit
  rule here does — both sides must be honest, so both floors are `CRAB_COL_NAME_MIN` ⇒ 186 px. Below
  that the operator keeps the listing and loses only the context. A view that refused to render would
  be a mode you can enter and not leave.
- ⛔ **The view is solo by construction**, not by width: two context columns and two listings is three
  columns of names in a window `crab_two_panes_fit` only just agreed could hold two.
- ⛔ **`-1` is a real answer and survives.** The parent can genuinely not contain us — a mount point, a
  bind mount, a path reached through a symlink — and marking row 0 there puts *you are here* on a
  **sibling**. `crab_rs_reset` seeds the field to -1 for the same reason `MBDROP` and `SBROW` are
  seeded to -1: a zeroed record must not mean "row 0".
- ⚠ **At `/` there is no context column at all**, rather than an empty one. An empty strip where
  context should be reads as *"this directory has no siblings"* — a different, false statement.

⭐ **`crab_ascend` now truncates at `crab_parent_len` and nowhere else**, so the context column lists
exactly the directory Backspace lands in. Two derivations of one fact are how a UI starts lying.

⛔ **A readdir is not a render-path operation, and this view would have made it one.** `crab_cols_sync`
memoises the listing on the path it is for, so a frame that has not navigated costs no syscalls — and
the refusal at `/` is memoised too, or standing at the root would retry the listing on every keypress,
pointer move and idle tick forever. The memo is dropped in `crab_relist_keep_thumbs`, the one function
every write and every refresh passes through — the siting `crab_thumb_forget` earned.
⚠ **The 80 KiB of listing buffers is allocated on first use**, not at startup: `alloc` is a bump
allocator with no `free()`, and an operator who never opens this view must not pay for it.

⭐ **Proven on QEMU** — `agnos/scripts/harness/crab-columns-test.py`, PASS on its first run.
`crab_readdir_into`'s entire body is inside `#ifdef CYRIUS_TARGET_AGNOS` with no `#else`, so on the
host it returns 0 entries for **every** path, `/tmp` included. A first draft of the suite built a
real tree under `build/` and asserted the listing; every assertion failed against an empty listing
and a clean error code. ⇒ **The suite gates the render half and says so; the harness gates the
listing, the naming, and the memo** (redraws must not re-list — the arm that keeps a readdir off the
render path).

### Changed — ⭐ the 9 px advance now has exactly one reader

crab sized every column by dividing pixels by nine, in more than one place, and **nine is a property
of one font** — kashi's CP437 8×16 cell. This is not cosmetic: `crab_col_chars` decides how many
characters a NAME column holds, which drives `crab_name_cell`'s `~` marker, which exists so that two
different files never render as one identical row (`agnos-kernel-build.log` and
`agnos-kernel-build.tmp` did exactly that before 0.5.0). Under a proportional face `px / 9`
over-reports, the cut goes unmarked, and the operator selects, descends and **acts** on that row.

- `crab_char_w()` answers `CRAB_COL_CHARW` for the bitmap font and the **font's own** advance for
  anything else; every divide goes through it.
- `crab_text_w(s)` **measures** a string rather than pricing it at `length × advance` — for the bitmap
  font the two agree exactly, and for a proportional face they do not. ⛔ Its scan is bounded by
  `CRAB_REC_TYPE` because the strings it is asked about are readdir names: kernel data, untrusted by
  crab's own rule, and the exact shape that caused the 0.5.0 P-1 that made `src/path.cyr` a file.

⚠ **crab still passes `font = 0`, so this renders identically — which is exactly why it was done on
its own.** ⚠ And what the suite CANNOT say is said at the assertions: at `font = 0` no host test can
tell *"asks the font"* from *"divides by the constant"*, because they are the same number; deleting
`crab_font = font` from `crab_render` leaves the suite green, and the test records that rather than
implying a gate. **What is left of proportional text is passing a real face** — `0.9.0`.

### Changed — 🗺 the roadmap has an order

Added **[the ladder to 1.0](docs/development/roadmap.md)**: one table, every entry a VERSION plus one ruling, each
named by what an operator can newly do, with what blocks it and what closes it. The remaining work
was spread across six correct-but-unordered sections, so every slot began by re-deriving the sequence.

⛔ **The order is the commitment; the number is not.** This file has mapped milestones onto versions
wrong four times — M4 rode four patch numbers, M5 landed inside one, M6 spread across seven, and both
of the milestones closed in 0.8.7 and 0.8.8 did so *out of milestone order, after all of M6*. ⇒ *A
milestone is a grouping of features, not a window in time.* ⛔ **1.0.0 is not a feature release**: three
of its criteria are not code (a green iron burn, `docs/benchmarks.md`, `docs/examples/`), so it can be
blocked with every version above it shipped.

### Tests

2,104 assertions (2,016 → 2,104), every new claim mutation-proven — including one mutation that
**passed** and rewrote its own assertion (the focus grab carried by call order) and one that passed
and was left documented rather than deleted (the scroll round-trip, behaviour-identical today).

## [0.8.7] — 2026-09-13 — the REFRESH key, crab on a real kernel again, chrome keys on Ctrl, a flag surface, and a collision you can answer

> `git describe --tags` answered `0.8.6` exactly before a word of this was written — 0.8.6 is tagged
> `249279f`, on the remote — so `[0.8.6]` below is a record and is left alone. Cut on operator
> direction; the commit, the tag and the push are the operator's.

### Added — ⭐ `u` is REFRESH: both panes, PLACES and VOLUMES re-read from disk

Since 0.8.0 the comments above the PLACES and VOLUMES models deferred re-reading them to *"a refresh
key rather than on the frame"*, and no key existed — a `Downloads` created while crab ran, or a file
written by another program, was invisible until some navigation happened to relist. `u` relists both
panes and rebuilds both sidebar models, on the keypress, where the cost belongs.

⛔ **NOT F5.** The conventional refresh key is claimed by the compositor for MAXIMIZE and consumed
without being forwarded — as are Esc, Tab and F2–F10 (see *Measured* below, which is the larger
finding). `u` is the free letter closest to the word, and it follows every other crab binding's
shape: a letter, with a menu accelerator (`View ▸ Refresh`, the bar's fifth item).

⛔ **THE SELECTION IS A NAME; THE MARKS ARE INDICES.** Both rules are older than this key and neither
changed: `crab_refresh_sel` puts the cursor back on the entry the operator was looking at
(`crab_index_of`, the sort key's precedent) and, when that entry is gone, keeps the old INDEX clamped
so the eye stays where it was rather than jumping to the top; the marks are cleared, because a stale
mark set acts on whatever now occupies that row — the `/bin` rule. Both names are captured before
either relist, because `crab_relist` readdirs into the same buffer. **Refused out loud while a
transfer steps** — relisting a directory a walk is still writing into is the hazard the drop and
delete arms already guard.

⭐ **THE SIDEBAR CURSOR IS A ROW INTO A MODEL THAT JUST CHANGED — as stale as a mark.** Its path is
copied out first (`crab_sb_path` hands back a pointer INTO the buffer being rebuilt), the models are
rebuilt, and `crab_sb_row_for_path` re-seats it by **exact** path or drops it to -1. ⛔ Exact, not
containment: `crab_sb_here` would land a vanished row on its containing ancestor, and Enter would
then navigate UP from a cursor the operator never moved. Asserted against each other.

⛔⛔ **A REFRESH KEEPS THE THUMBNAIL CACHE, AND THAT WAS A REVIEW FINDING BEFORE IT SHIPPED.** The
first draft went through `crab_relist`, which wipes the 64-slot cache because a *write* can make a
name mean a different file — and every decode is permanent, charged to a 32 MB session ceiling that
nothing refunds. A refresh writes nothing; wiping on `u` re-decoded the selected image (or every
visible gallery cell) and charged it again per press: **four presses on a 1024×1024 image and the
session would never draw a thumbnail again.** An adversarial review (three lenses, two skeptics per
finding) found it; `crab_relist_keep_thumbs` is the refresh's relist — the preview memo (a 64 KiB
header read, no budget) is still forgotten, the cache is not — pinned by a test that claims a slot,
refreshes, and finds it, with the write-op relist as the control. ⚠ The trade, stated: an image
rewritten in place under the same path keeps its old thumbnail until that path leaves the cache,
while its DIMENSIONS/EXIF lines are re-read. Scroll offsets need nothing. The refresh scratch is
allocated once, not per keypress: the sort arm's per-press `alloc` is a small leak on a heap with no
`free()`, and this key did not copy it.

⛔ **The cursor re-seat carries its SECTION, also from review.** On agnos `/` is always two rows —
the Root place and the ext2 volume the kernel mounts first — and a path-only match hopped a cursor
from the volume row to the place row on every refresh. `crab_sb_row_for_path` takes the kind the
cursor was on and comes back to a row of that kind or not at all; the alias fixture is in the suite.
Two stale comments the review caught (the overlay guard's "unexercised defence", `t_menubar`'s
"where no bar menu reaches yet") now say what the code does.

⭐ **`View ▸ Refresh` makes the separator guard load-bearing at last.** View holds five items now, and
its fifth (index 4) is exactly the index `crab_menu_row` would shift; the bar's not-mapped guard was
"unexercised defence" for three cuts and is proven by the render assertion that selects item 4 and
expects row 4 — the mutation that applies the mapping fails with `got -1`, the shifted highlight
landing on nothing.

### Measured — ⭐⭐ QEMU, 2026-09-13, `agnos/scripts/harness/crab-pointer-test.py` (new): PASS

The first on-target run since 0.7.0, on agnos `build/agnos` (2026-09-11) with **aethersafha 0.16.24**
and this tree. Four runs; the first three taught the harness, the fourth is the verdict:

```
crab launched and presented: True | stray probe windows: 0
ascended to /: True
left click resolved to a pane at: (200, 180)
right click: compositor forwarded a non-left button: True wire number: 2 | crab opened the context menu: 1 time(s)
pick: a left press at offset (30,16) ran a menu entry | verb by pointer: True     (it ran `crab: copy tree bin` — row 1; refused, /bin exists)
dismiss: presses off the popup dismissed it 1 time(s); picks total 1
refresh: `u` x6 -> 4 refresh(es), 8 listing line(s)
display keys: g -> 1 view line(s); b -> 1 sidebar line(s)
Tab x6: compositor answered 2 time(s); crab ACTED on 0 key(s)
F10 x6: compositor answered 1 time(s); crab ACTED on 0 key(s)
Esc x4: compositor quit: True ; crab ACTED on 0 key(s)
faults: False
PASS — right-click menus 1, picks 1, dismisses 1, refreshes 4
```

⭐ **So the 0.8.5 pointer routes are real on the wire**: a right press arrives as button **2**
(aethersafha 0.16.24's numbering), crab opens the context menu, a left press on a row runs an entry
through the synthesised-key road, a press off the popup dismisses. The REFRESH key relists twice per
press, on a kernel whose `mountlist` path rebuilds VOLUMES for real. `g` and `b` answer.

⛔⛔ **AND THE COMPOSITOR CLAIMS Esc, Tab AND F4–F10 — CONSUMED, NEVER FORWARDED; Esc QUITS THE
DESKTOP.** Read in aethersafha's `input_map` and its *"CLAIMED KEYS ARE CONSUMED, NOT FORWARDED"*
block, then **measured**: crab acted on zero of them while the compositor answered every one. Which
means, on the real desktop: the `F10` menu bar (0.8.0) and everything under it including `View`
(0.8.6) is reachable by nobody; the `Tab` sidebar route (0.8.3) is unreachable; every `Esc` binding —
dismiss the menu, close a drop, cancel a transfer, abandon the sheet, leave sidebar focus — is
unreachable, and the key ends the session. crab already knew about F5/F6 and avoided them; Esc, Tab
and F10 were recorded nowhere. ⚠ `crab: key received` DOES move on Tab — by the *releases*, which
the compositor forwards because `input_map` returns nothing for a release; only the press edge is
gone, which is why the harness counts `crab: key press`. **Filed in aethersafha**
(`docs/development/issues/2026-09-13-claimed-keys-never-reach-a-client.md`; crab's copy at
[`docs/development/issues/2026-09-13-aethersafha-claims-esc-tab-f10.md`](docs/development/issues/2026-09-13-aethersafha-claims-esc-tab-f10.md))
with three shapes for the decision — a surface flag, modifiers on the wire, or rebinding — none of
which is crab's to pick. The bindings stay: they are correct the day the key arrives.

⚠ **What the first three runs taught, kept in the harness header:** (1) crab-resize-test's Enter ×8
launch burst leaks into crab — the launcher eats one and the rest are crab's Enter, which is OPEN on
the selected row, and `/bin`'s first row is `aethersafha`: run 1 put a **second compositor** on the
desktop and measured two of them sharing one mouse. (2) A DOWN burst is a coin flip — the launcher
wraps, so an even count lands back on puka. (3) Keys are lost because the boot-keyboard report is a
STATE; `sendkey <key> 400` holds the press across the compositor's per-frame drains and the launch
became deterministic. (4) A pointer pick's row is not knowable from the harness, so it reports which
verb ran instead of assuming; the pick arm ascends to `/` first so Open can only mean descend.
⚠ The same Enter burst is still in `crab-resize-test.py`; on this tree it can spawn a compositor.

⚠ **Still not on target**: the *you are here* marker (a pixel claim; the rule is pinned on the host),
the sidebar keyboard route (unreachable — see above), and iron.

### Added — ⭐⭐ an overwrite policy: a collision STOPS the walk and asks, per file

Until now a collision **ended the whole operation**. Copying a folder of 500 files into one that
already held **one** of them moved nothing, said *"something of that name is already there"*, and
named neither the file nor anything to do about it. Worse, the copy could not start at all if the
destination held a folder of the same name — the single most ordinary thing two panes are for.

⭐ **The policy is the operator's (2026-09-13): ASK, PER COLLISION.** The walk stops, the status line
names the file, and four answers are offered — **`r` replace · `s` skip · `k` keep both · `Esc`
stop** — with **`a`** arming an **all** that applies the next answer to every remaining collision.

⛔⛔ **"ALL" IS ARMED BEFORE THE VERB, NOT SPELLED BY A CAPITAL, because shift is not on the wire.**
`R` for replace-all is the obvious design and it is unreachable on this stack (`mods` carries the
press/release edge; `crab_key_char`'s shift flag is hard-coded to 0). ⇒ `a` toggles, the prompt shows
`[all]` while armed, and the next `r`/`s`/`k` applies to the rest. Two lowercase keystrokes, and the
armed state is **visible before it is acted on** — which a capital could not be.

⛔ **DIRECTORIES MERGE; ONLY FILES ASK.** Two folders of the same name are what a copy into an
existing tree means. Asking "replace this folder?" would mean either deleting a tree the operator
never asked to delete, or wrapping `docs (2)` around files that individually did not collide — both
worse than merging. The question is asked where the answer actually destroys something. ⚠ A
NON-directory in a directory's way still stops the run: crab cannot merge a tree into a file.

⛔ **KEEP BOTH PUTS THE SUFFIX BEFORE THE EXTENSION** — `report (2).txt`, never `report.txt (2)`,
because the extension is what every other program dispatches on. ⚠ A dotfile is a NAME, not an empty
stem: `.bashrc` → `.bashrc (2)`, not ` (2).bashrc`. ⚠ It tries (2), (3)… until a free name is found,
because `photo (2).png` can collide too and a keep-both that overwrote it would destroy a file while
answering the one thing that promises not to.

⛔ **REPLACE UNLINKS FIRST, and that is the two targets agreeing for once.** `crab_fs_open_w` is
`O_WRONLY|O_CREAT|O_EXCL` on the host and `AO_WRONLY|AO_CREAT|AO_TRUNC` on agnos, which has no
`AO_EXCL` — unlink-then-create is the one sequence that means the same on both. The suite proves it
by reading the bytes back on a host whose open is exclusive.

⛔ **A WAITING OPERATION DOES NOT STEP, AND A COLLISION PROMPT OWNS THE KEYBOARD AND THE POINTER.**
The walk advances on the idle tick, so the tick asks `crab_op_waiting()` before stepping — one that
stepped anyway would re-ask every 16 ms or walk past the entry in question. The key arm sits **above
every binding including Esc-cancels-a-transfer**, because the operation is not merely running, it is
stopped on a question; `r`/`s`/`k` are rename/sort/nothing elsewhere and must not reach them. And
`crab_pointer_blocked` takes the waiting state: it is `confirm_del`'s exact shape — a status-line
question with no overlay to prune clicks, whose next keystroke decides whether a file is destroyed.
⚠ The policy resets on **every run**: a `replace all` that survived would destroy without asking in
some later copy the operator armed nothing for.

### Fixed — ⛔⛆ a `DT_UNKNOWN` retry ended a delete and reported "done" on a half-emptied tree

`crab_walk_step` returns `1` for "more to do" and `CRAB_FS_OK` for "the whole tree is done" — and
**`CRAB_FS_OK` is 0**. The `DT_UNKNOWN` retry arm (an entry `readdir` typed as a file, whose `unlink`
failed, to be retried as a directory) did `return 0`. The idle tick treats anything that is not 1, 2
or 3 as terminal, so that arm **ended the operation and reported "done"** — and the retry never
happened. ⚠ Reaching it needs a filesystem answering `DT_UNKNOWN`, which the host's `getdents64`
does not, so no test could see it and the RETURN VALUE was never the thing under test. Found while
adding a second `return 0` beside it and asking what 0 meant. ⇒ Both return `1`, and the function's
contract now says: **never `return 0` for a non-terminal case.**

### Added — ⭐ a flag surface: `crab --help`, `crab --about`, and an unknown flag that says so

crab parsed positional paths and nothing else, so `crab --about` — which `docs/development/mascot.md`
asks for **by name** — could not be built, and `crab --anything` was refused as *"left path
unusable"*, which names the wrong problem entirely. `--help` (also `-h`) prints usage, the flags, and
**the key list**; `--about` closes on the Ben-Stein line; an unknown flag names itself, prints help
and exits **2**. Flags are scanned before any path is adopted and before any window exists.

⛔ **A FLAG IS NEVER AMBIGUOUS WITH A PATH, and that is not luck**: `crab_path_usable` requires an
ABSOLUTE path, so anything beginning with `-` cannot be one. That is what lets an unrecognised `-x`
be refused *as a flag crab does not know* rather than mis-diagnosed as a bad path — and why
`crab_flag_of` answers `UNKNOWN`, never `NONE`, for anything starting with a dash. ⚠ The paths are
the **positional** arguments now (`crab_positional`), not `argv(1)`/`argv(2)`, so a flag anywhere in
the line cannot shift which argument is the left pane.

⭐ **`--help` lists the KEYS, because nothing else does.** The menu bar shows six verbs behind `F10`
and the context menu the same six; the views, the sort, the sidebar, refresh and the sidebar's Tab
route were discoverable only by being told. It also says the compositor owns `Ctrl+Q` / `Ctrl+Tab` /
`Ctrl+F4–F10`, so an operator looking for the way out does not reach for Esc.

⚠ **NO `--version`, AND THE ABSENCE IS THE POINT.** `VERSION` at the repo root is the single source
of truth, the manifest interpolates it, and cyrius offers no build-time define that would reach a
source file — so a `--version` string would be a **second copy of the number**, drifting at the first
cut nobody remembered. That is the exact failure aethersafha's `version-bump.sh` carries a ⛔⛆ about.
`--version` is honestly `UNKNOWN` rather than a lie, and the suite asserts neither text carries a
version. It becomes possible the day the toolchain can inject the manifest's.

⚠ **On agnos the surface is unreachable today**, and that is recorded rather than pretended: the
launcher spawns `/bin/crab` with no arguments, and crab cannot start from a shell there at all (it
needs `AGNOS_CHAN`, which only the compositor mints). It exists for the host and for the day one of
those changes. ⚠ **What the suite can and cannot reach**: the DECISIONS are pure and pinned
(`crab_arg_is_flag`, `crab_flag_of`, both texts — 38 assertions, five mutations each caught, including
"the about text explains the joke" and "the roll-call is dropped"); the two functions that walk
`argv` are verified by running the binary (`--about`, `--help`, `-h`, `--nope`, and `crab /bin /`
unchanged), the same split every agnos-only arm in this codebase has.

⛔⛔ **`crab_about_text` CARRIES A JOKE AND THE JOKE IS INDIRECT — read `docs/development/mascot.md`
before touching it.** Rust's mascot is Ferris the crab; crab replaces the Rust interim; the mascot is
named **Bueller** — the surname — so it implies Ferris *without ever saying it*, and the roll-call is
the last line and is never explained. The suite asserts both halves: the line is present, and the
words "Ferris" and "Rust" are **absent**. The doc's own discipline: *"subtle and infrequent… the
whole thing dies if it's trying too hard."*

### Changed — the roadmap is 631 → 466 lines, and 42 stale claims are corrected

An audit of every claim in `docs/development/roadmap.md` against the code, the suite, the CHANGELOG
and the sibling repos (a workflow: four readers over disjoint sections, two skeptics per DONE/STALE
verdict, 142 agents) returned **124 findings — 48 open, 42 stale, 27 done-but-still-listed, 7 gated**.
**M5 and M6 are collapsed into the shipped table**, as M1–M4 were before them and for the same
reason: they had become 140 lines of shipped-feature narrative, some still carrying `(unreleased)`
labels that outlived their own tags. What a collapse keeps is what is NOT done and the lessons that
still govern. Corrections worth naming:

- *"It reads surface/text/accent from aethersafha via `dh_theme_*`"* — **there is no wire.** rupa's
  active theme is per-process and defaults to MUDRA dark; crab and aethersafha agree by sharing a
  default, not because one hands the other a theme. No setu kind carries one. The invariant crab
  actually keeps (it names no colour) is intact and now stated separately from the claim it did not.
- *"F1 is a change in the TRUST MODEL"* in the present tense — **F1 closed in 0.7.6**, in the release
  that raised it; the gallery walks only the visible range. What does not close is that ~22,500 lines
  of third-party parser still run in-process on attacker-chosen bytes.
- *"The keycode confusion… dhancha's `DhKey` constants are evdev"* — **they never were**; `DhKey` is
  puka's ASCII/Unicode sym space, and the word *evdev* appears nowhere in dhancha. The item is real
  (three key spaces across four repos) and its central claim was false for four releases.
- *"Two ADRs exist"* (three), *"three false gates"* (**seven**, wrong in four different ways),
  *"M3 … 0.7.0"* (three of its seven items closed their gates at 0.7.1), the M5 heading's `v0.9.0`
  (it shipped inside `[0.7.6]`), *"crab consumes none of it yet"* for the menu-bar strip (since
  0.8.0), *"83 over-long lines"* (**147**, and nearly all of the growth is in the suite, not the
  event loop), and the idle mascot line listed under *"the only parts of M1–M4 that are not done"*
  while shipped in M6.
- The **gates table** now lists only live gates; the closed ones are named once as a group. ⛔ The
  false-gate lesson is kept and sharpened: **a price is not a gate**, and a table of gates invites
  reading one as the other — which is exactly how thumbnails were read.

### Changed — ⭐⭐ the compositor's chrome keys are Ctrl chords now, so crab's Esc, Tab and F10 ARRIVE

**aethersafha 0.16.25** (prepared in the sibling on operator direction: *"it was easy for initial
testing of the desktop but now it's time to fix that right"*) moves every chrome key onto Ctrl —
**Ctrl+Q** quits, **Ctrl+Tab** cycles windows, **Ctrl+F4–F10** close/maximize/minimize/move — and
forwards the bare keys. The gate above is closed by the third shape the filing offered. Measured on
QEMU, `crab-pointer-test.py` with its ARM 7 rewritten to the new contract — **PASS, run 14**, and the
seven chord/bare lines below were identical in runs 6, 7, 9, 11, 12, 13 and 14 (the runs between
were the harness's own pointer timing, not this contract):

```
right click: compositor forwarded a non-left button: True wire number: 2 | crab opened the context menu: 1
pick: a left press ran a menu entry | verb by pointer: True | what ran: crab: cd /bin
dismiss: presses off the popup dismissed it 1 time(s)
refresh: `u` -> relists; display keys g/b answer
bare F10: crab opened the menu bar: True | compositor moved the window: 0
View via the keyboard: F10 → Right x3 → Enter → Enter -> 1 view change(s)
bare Esc x3: crab acted on 3 | compositor quit: False
bare Tab x3: crab acted on 3 | compositor claimed 0
Ctrl+Tab x3: compositor answered 1 | crab acted on 0
Ctrl+F10 x3: compositor answered 1 (one-shot) | crab acted on 0
Ctrl+Q x3: compositor quit: True | crab acted on 0
```

⚠ **And the harness learned three more things about driving a pointer under TCG**, all in its
header: the kernel ACCUMULATES deltas between drains, so a pin (−4000) and a walk folded into one
drain land the cursor at (0,0) — crab's titlebar — and the press "had NO client content under the
cursor"; where to press to MISS a popup is derived from `dh_place_at_point`'s flip, not guessed; and
a pick can open a SHEET (Rename…/New folder…) whose scrim blocks the pointer, so the arms reset
crab's modal state with bare Esc — which only works because Esc arrives now. A dismiss arm that
cannot re-open a menu reports UNMEASURED, not FAIL: an assertion over an empty set is not a pass.

⭐ **So the menu bar, `View`, the sidebar's keyboard route and every `Esc` cancel are live on the
real desktop for the first time** — `View` was driven from the keyboard on a real kernel in that run.
Nothing in crab's bindings changed; they were correct the day the key arrived, as the filing said.

⛔ **One crab change was REQUIRED by the new contract, and it closes an older defect: a modifier's own
edge is not a keystroke.** The compositor forwards Ctrl/Shift presses as usages `0xE0..0xE7` (bhumi
maps the boot report's modifier byte) and always did; crab's dispatch took that press like any other
— it **answered "no" to a pending delete prompt**, cleared a notice, cleared the held-key repeat. With
every chrome key now a chord, that edge precedes every Ctrl+Q. `crab_key_is_modifier` gates it before
the latch and before every dispatch gate; the harness's "crab acted on 0" for the chords is that gate
at work. ⚠ Shift for capital letters (a *Recorded as facts* item) would read the same usages as
STATE — the road is open now, not built.

⚠ The pointer harness's ARM 7 is a gate now, not an observation: bare F10 must open the bar, bare Esc
must not quit, Ctrl+Q must. Every other harness that sent a bare chrome key sends the chord
(`crab-resize-test` maximizes with `ctrl-f5`); `puka-terminal-test` expects its typed Tab to reach
puka. `docs/development/issues/2026-09-13-aethersafha-claims-esc-tab-f10.md` records the resolution.

### Verified

`cyrius test` **1838 → 2012 / 0** (+174: `t_refresh` — the selection rule, the exact re-seat against
the containment rule and against the `/` alias, the menu entry, the accelerator agreement, the
sidebar gate, the kept thumbnail with the write-op relist as control; and View's fifth row rendered)
; `crab_key_is_modifier` over the modifier row; `t_flags`; and `t_overwrite` + `t_overwrite_walk`,
the second of which drives the **shipping** `crab_op_step` against real directories on disk — the
lesson recursive copy cost this project — asserting the bytes after a replace, the `(2)` name after a
keep-both, the untouched destination after a skip, and that an armed answer asks **once**)
· render_test 53 / 0 · fmt clean · coverage
**89 %** · `vet`/`deny` 0 · `deps --verify` 50 / 0 · host **1,053,680 B** · `--agnos` **1,094,480 B**.
Sixteen mutations, each caught: a vanished name resetting
to 0; the re-seat by containment; the re-seat ignoring the section; the refresh relist wiping the
cache; the bar applying the separator mapping (`got -1`); Refresh on F5; an unknown flag answering NONE; a
prefix match on `--hel`; the about text explaining the joke; the about text dropping the roll-call; a
flag test that accepts a path; the skip returning 0 (the defect's own shape); a collision tearing the
walk down again; an unarmed answer sticking; replace forgetting to unlink; the root refusing instead
of merging. ⚠ One of those mutations **segfaulted out of the group before the verdict** (a null name
dereferenced by the test itself) — guarded, so a mutation now FAILS the assertion it should. ⭐ **Adversarial review**
(a workflow: three finder lenses over the diff, two refuters per finding, 23 agents): 10 findings
raised, 3 distinct defects survived both skeptics and are fixed above; the rest were refuted or were
the same defect seen from another lens.

## [0.8.6] — 2026-09-13 — `View` is filled: the last M6 interaction gap

> `git describe --tags` answered `0.8.5` exactly before a word of this was written — 0.8.5 is
> tagged `4344cb9`, on the remote, CI and Release both green — so `[0.8.5]` below is a record and is
> left alone. Cut on operator direction; the commit, the tag and the push are the operator's.

### Added — ⭐ the menu bar's `View` holds the four display switches the keyboard already has

Since 0.8.0 the bar shipped `View` as a label that opened on nothing (*"that menu has no items
yet"*). It now drops down **Cycle view · Cycle sort · Preview · Sidebar** — `g` · `s` · `p` · `b`,
with the real accelerators in the column — and each entry is the key with a label on it: the same
rewrite-and-fall-through the other menus use, so there is still exactly one implementation of every
command. The 0.8.3 hoist was the prerequisite: before it, `View ▸ Cycle view` would have been born
dead exactly as `Open` was, and the roadmap said so.

⛔ **THE IDS LIVE ABOVE `CRAB_MI_COUNT`, WHICH IS THE CONTEXT MENU'S BOUND — AND THAT IS THE DESIGN.**
`CRAB_MI_VIEW` … `CRAB_MI_SIDEBAR` (6..9) share the four maps (`crab_menu_label` / `_key` / `_accel`
/ `_enabled`) with the file verbs, and `CRAB_MI_ALL` (10) is the new bound for *is this an item at
all*; the context menu keeps walking `0..CRAB_MI_COUNT`. A right-click on a file offers what can be
done TO the file, not how to look at it. Pinned both ways: the render assertion that counts the
context menu's rows (seven for six items, separator included) fails at eleven if the display items
leak, and every View slot is asserted to be a display id and never a file verb.

⭐ **A display item is always live.** It acts on the VIEW, not on a row, so an empty pane is no
reason to grey it — and whether the window can HONOUR a toggle is the key handler's answer (`p` and
`b` flip the operator's want and refuse out loud when nothing fits). Greying the entry would be a
second, silent copy of that rule. ⚠ None of the four keys is one the sidebar-focus gate eats, so a
View pick acts even while the keys live on the sidebar — asserted as the property over all four.

⚠ `View`'s drop-down needs `crab_mb_cell_x(3) + CRAB_MENU_W + CRAB_MENU_MARGIN` — it fits at the
shipped 380 px, asserted, and one pixel under its rule it does not open, also asserted; the render
refuses a drop that would clamp under the wrong label, for the keyboard and the pointer alike.

⛔ **`Go` stays empty, with its reasons on record.** The roadmap refused it as a drop-down: an
11-to-17-row popup at 380×220 is clamped and flipped over both the bar and the status line, `d` is
not consumed by the drop arm so the delete prompt would draw underneath it, and `Go ▸ Parent` ships
dead at `/`. The sidebar's keyboard route is `Go`'s shape. The bar label stays because the canvas
draws it; opening it says so.

⚠ **A note that was wrong by one.** The overlay's *"unexercised defence"* note on the separator
mapping said it becomes load-bearing *"the moment a bar menu grows a fourth item"*. View grew one
and nothing moved: `crab_menu_row` shifts indices ABOVE `CRAB_MI_RENAME` (3), and a fourth item is
index 3. It is the FIFTH. The note, and its twin in the suite, now say so — and the suite asserts
that index 4 *would* be shifted, so the day it matters is already pinned.

### Verified — the cut checks

`cyrius test` **1790 → 1838 / 0** (+48: the View group, the id-space loops widened to
`CRAB_MI_ALL`, and the View drop-down rendered — four live rows under its own cell, the last one
highlighted unmapped) · render_test **53 / 0** · fuzz 100,000 rounds · `fmt --check` clean ·
coverage **88 %** (265/298) · `vet` + `deny` 0 · `deps --verify` 50 / 0 · host **1,049,568 B**
`ebe13334…` · `--agnos` **1,090,304 B** `59eeae79…` (+88 / +88 over 0.8.5). Five mutations, each
caught: View emptied; display items greyed on an empty pane; a file verb leaked into View; Sidebar
sending `c`; the context menu listing every id. ⭐ **Check four re-run at the cut**, all four `path`
overrides disabled: 7 deps / 0 errors, lock 3 → 7 commit-pinned, both binaries **byte-identical**,
1838 / 0 in the scratch copy. ⛔ **Not run on QEMU or iron**; the View pick rides the same
synthesised-key road as every other bar pick, which the host suite cannot reach past the maps.

⭐ **M6's six interaction gaps are closed — 2 in 0.8.3, 3 in 0.8.5, 1 here.** What M6 still carries
is gated, not deferred: the 🦀 chrome button (no crab glyph in CP437 — proportional text or an icon
path) and the held-key repeat number (agnos-runtime, needs the on-target harness).

## [0.8.5] — 2026-09-13 — the pointer reaches every surface, and the sidebar knows where you are

> ⛔ **`[0.8.4]` BELOW IS RELEASED — tagged `7929ae1`, on the remote, CI and Release both green —
> and its header still reads *"unreleased"*, as `[0.8.3]`'s does.** Both are records now and are left
> alone; that is the 0.7.2 rule. ⚠ `git describe --tags` answered `0.8.4-1-g8bdcbfe` when this
> heading was written: the first half of this section had already been committed as `8bdcbfe` with
> `VERSION` still 0.8.4, and the cut to **0.8.5** was made on operator direction on 2026-09-13.
> Nothing here is tagged or pushed by anyone but the operator.

### Changed — the whole dependency graph moved to the cyrius 6.6.2 siblings

Every one of crab's seven deps was released for 6.6.2 while crab still declared the previous tag,
and — `path` winning over `tag` — the local build was already compiling four of the new ones:
0.8.4's `lib/` carried **dhancha 0.9.29 · rupa 0.1.7 · setu 0.8.9** under a manifest naming
0.9.28 · 0.1.6 · 0.8.8, so CI and a local build compiled different code and neither said so.

| dep | from | to | what moved in the module crab consumes |
|---|---|---|---|
| `sadish` | 0.5.3 | **0.5.4** | version header only |
| `rupa` | 0.1.6 | **0.1.7** | version header only |
| `rekha` | 0.3.6 | **0.3.7** | version header only (a `_distprobe.cyr` landed at its root; not in the dist) |
| `kashi` | 1.0.6 | **1.0.7** | nothing — `src/font_data.cyr` is byte-identical; its tests moved |
| `dhancha` | 0.9.28 | **0.9.29** | version header only |
| `setu` | 0.8.8 | **0.8.9** | version header only |
| `chitra` | 1.0.1 | **1.0.3** | ⭐ **real content** — see below |

Every tag verified on its remote with `git ls-remote --tags` and on a clean sibling tree, and read
from the tag-to-tag diff of the consumed module rather than from the release note.

⭐ **chitra 1.0.3 is the one that matters, and it matters to crab specifically.** Its P-1 sweep
closed 19 defects, headed by *"a valid PNG could kill the process"*: `crc32_init_table()`'s OOM return
was discarded and the next `crc32()` read through a NULL table — **SIGSEGV on the first PNG a
memory-pressured process decodes**, no hostile file required. crab is exactly that process: it runs
chitra on a bump allocator with no `free()`, behind two budgets that exist because every decode is
permanent. Also in it: JFIF files whose colour space was overridable by their component ids, a
truncated JPEG scan reporting a clean close, and every RLE BMP whose width is not a multiple of 4
being refused. ⚠ None of it changes a crab test; the fuzz and thumbnail groups stayed green through
the bump, which is what a decoder fix should look like from the consumer's side.

⚠ **One stdlib leaf had been left behind by the 0.8.4 pin bump**: `lib/sankoch.cyr` was still 6.6.1's
2.7.14 while every other file matched the 6.6.2 snapshot — the transitive leaf `cyrius lib sync`
does not walk, again. `cyrius deps` corrected it to 2.7.15; the whole vendored tree now `cmp`s
byte-identical to `~/.cyrius/versions/6.6.2/lib`, file by file.

⭐ **Check four re-run against the new graph**: a scratch copy with all four `path` overrides disabled,
`cyrius deps` really cloning the tags — **7 deps / 0 errors**, lock **3 → 7 commit-pinned** (the tell
the overrides were really off), and both binaries **byte-identical** to the path-resolved ones —
host **1,045,296 B** `d75c35a9…`, `--agnos` **1,085,872 B** `9ca89ea3…`. *That equality is the
evidence.* `deps --verify` 50 / 0.

### Added — ⭐⭐ the pointer routes M6 shipped without: right-click, the popup, the bar, the switcher

Since 0.8.0 the context menu opened from the Menu key alone, the menu bar and its drop-downs from
`F10` and the arrows alone, and the A/B strip was *"a display, not a control"*. crab read only `b`
of `POINTER_BTN` — press or release — so a right-click was a left one. The information was never
on the wire: aethersafha forwarded a hardcoded `1`. **aethersafha 0.16.24** (prepared 2026-09-12 on
operator direction, in the sibling — see *Upstream*) forwards every button, and this is the
consumer.

- **Right-click on a pane opens the context menu there**, over the row under the point: the pane
  is focused and that row selected first, so the verbs act on what the operator pointed at rather
  than on what the arrows last touched. The Menu key still opens it at the tracked position.
- **A left press on a popup row runs that entry** — a context-menu verb or a bar drop-down item —
  by the same road Enter takes: the entry's accelerator is **synthesised** and falls through the
  one binding table. There is still exactly one implementation of every verb.
- **A press anywhere else while a popup or the bar is up dismisses it**, and is consumed. It never
  falls through to the surface underneath — that surface is one the operator is not looking at.
- **A left press on a bar cell** selects that menu and opens its drop-down onto the first enabled
  item, as Enter does; on the cell whose drop is already open it closes it; a neighbouring cell
  while a drop is open switches menus, as every menu bar does.
- **A left press on the A/B strip focuses that pane.** Its own hit, never `crab_hit`'s pane index.

⛔⛔ **ONE FUNCTION DECIDES, AND ITS ORDER IS THE DESIGN.** `crab_pointer_action` in `src/ui.cyr`
takes what every surface answered about the point and names the ONE arm that takes the press. The
first two lines of that order are not obvious: `dh_list_index_at` is z-order blind, and
`dh_place_at_point` FLIPS a drop-down above its anchor when there is no room below, so a drop's rows
can lie **on the bar row** and a context menu always lies on a pane — asked in the wrong order, a
press on `Delete` also reads as a bar cell or a pane row. Popup first, then bar, then strip (which
sits inside a header `crab_hit` also records), then sidebar, then panes. **Every arm consumes**:
there is no answer that means "not mine, try the panes". Pinned by 31 assertions and seven
mutations (bar before popup; fall-through instead of dismiss; right on a pane as a click; pane
before strip; the separator shift forgotten; trusting the popup widget rather than the open flag;
blocked ignored) — each producing a named failure.

⛔ **THE BUTTON NUMBERS ARE aethersafha's, MIRRORED, AND NOT X11's.** `CRAB_BTN_LEFT = 1`,
`CRAB_BTN_RIGHT = 2`, `CRAB_BTN_MIDDLE = 3` — `wire = kernel_bit + 1`, the decision recorded in
aethersafha's `src/input.cyr` and pinned by both suites. X11 is 1=left **2=middle 3=right**; a reader
who "corrects" these puts Delete on the middle button. ⚠ setu is the right eventual home for the
constants; until it names them, crab's copy sits next to the one function that reads them. ⚠ On a
compositor older than 0.16.24 every press still arrives as `1`, which is inert rather than wrong.

⛔ **THE SEPARATOR IS THE DANGEROUS HALF, AND IT HAS ITS INVERSE NOW.** `crab_menu_row` maps a menu
item to the list row it lands on once the separator is counted; the pointer needs the other
direction, and feeding a list row straight to `crab_menu_accel` would fire **Delete for a press on
New folder** — the off-by-one that once painted the wrong highlight, now acting instead of painting.
`crab_menu_item_at` inverts it; the suite pins the round trip for every item. ⚠ `dh_list_index_at`
already refuses inert rows — the separator and every greyed verb — so a pick never names a disabled
entry; that is dhancha's contract, pinned in dhancha's own suite, and the -1 here is defence.

⭐ **Phase 0's last piece: `crab_pointer_blocked` split out of `crab_pointer_modal`.** The delete
prompt and the edit sheet REFUSE the pointer (nothing on them to click); an open popup or a revealed
bar CAPTURE it (its own rows are what the press is for, everything else dismisses). The press arm
asks the first; `crab_pointer_modal` is the union and still gates the wheel. The 0.8.0 group's every
assertion holds unchanged.

⚠ Two things deliberately not done, and named: the middle button does nothing anywhere yet and
says so by returning `CRAB_PA_NONE` rather than by being absent from the table; and pointer MOTION
does not move a popup's highlight — only a press acts. ⚠ **Not run on QEMU or iron.** The whole
arm is inside `src/main.cyr`'s agnos-only `#ifdef`; the decisions were lifted into `src/ui.cyr`
for exactly that reason, and the wiring is what an on-target run is for. New oracle lines for it:
`crab: context menu opened by pointer` · `crab: menu pick by pointer` · `crab: verb by pointer`
(a synthesised verb, kept separate from `crab: key press` so the harness's received-vs-acted ratio
is not disturbed) · `crab: popup dismissed by pointer` · `crab: bar click` · `crab: switcher click`.

### Added — ⭐ the sidebar shows where the active pane IS (the *you are here* marker, M6)

Since 0.8.0 the PLACES sidebar was a list of destinations that never said which one the operator was
in. `crab_sb_here` lights the row whose path CONTAINS the active pane's — **containment, deepest
wins, never equality**: a pane is almost never *at* a place, it is *under* one. `/` contains
everything, a volume prefix contains its own subtree, Home contains Documents, and the row that says
the most is the longest containing path. Equality would light Root only at `/`, the one place nobody
stays. Painted under the muted line colour when the sidebar has no focus (*you are here*), and
**never** when it is focused with no cursor — `crab_sb_shown_row` has ruled that since 0.8.3, because
a location under `accent` reads as "Enter acts on this row" while Enter would send the pane to an
ancestor. The keyboard cursor and the location are two different facts and stay two.
⚠ **Ties go to the lowest row.** Root the place and `/` the volume both contain everything at
length 1; PLACES come first, so the place lights. On a desktop whose root is a mounted volume that is
the normal state, not a corner case. Pinned.
⭐ **One containment test, not two.** `crab_path_within` — the copy-into-itself guard — is the same
rule, and it lived in `src/app.cyr`, which the render path may not reach up into. Moved to
`src/path.cyr` with its ⛔ header intact; its only other caller resolves unchanged.
⚠ Derived per frame and never stored; one bounded compare per row over at most a dozen rows.

### Fixed — a place or volume stored with a trailing slash could never be *here*

`crab_path_within` matches the root as a prefix and then requires a separator, so a root that already
ends in one never contains the bare directory it names: `/home/macro/` does not contain
`/home/macro`. `$HOME` is spelled by whoever set it and the kernel's mount prefixes by the kernel —
both outside crab's control — so **both model builders now normalise what they store**
(`crab_path_trim_slash`, keeping `/` itself; the volume's `PLEN` follows the stored bytes so `statfs`
is asked about the string the sidebar shows). The roadmap had this as Phase 0's last open piece.

### Fixed — a 64-byte volume prefix lost its terminator when `statfs` landed

The ABI allows `prefixlen` 1..64 and crab terminates its own copy — so a 64-byte prefix needed a
65th byte, and the field was 64: the NUL sat at offset 80, `CRAB_VOL_BSIZE`'s first byte, and the
block size overwrote it. Every read of the prefix is bounded, so this was a wrong path rather than an
overrun — `crab_sb_path` handing `crab_goto` a mount point with the block size's bytes spliced on —
and a wrong path is what Enter acts on. The prefix field is 72 bytes now (BSIZE 88, BLOCKS 96,
BFREE 104, record 112); the mutation that restores the old offsets fails with **`got 80`**, the
string running sixteen bytes into the capacity fields. agnos spells nothing longer than `/mnt/exfat`
today; the field is sized to the ABI, not to today.

### Fixed — ⛔⛆ the wheel was not gated by a modal question, and it was the same hole 0.8.0 closed

`POINTER_SCROLL` moved `sel_l` / `sel_r` with no guard at all. A scroll between `d` and `y` moved the
selection, so the prompt named one entry and the delete took another — exactly the pane-click path
0.8.0 closed, one input kind over. Gated on `crab_pointer_modal` — the union, popup and bar included,
because a wheel has no rows of its own to take: while anything is up, it does nothing.

### Fixed — the sidebar arm never consumed the double-click pair

Click a row, click a place (the pane re-lists), click the same row index inside 400 ms — and
`crab_is_double` descended into whatever entry now sat at that index in a directory the first click
never saw. Named as latent in the roadmap; closed by one line above every arm rather than one per
arm, so a route added later cannot omit it.

### Fixed — a right or middle release could end a left drag

Before 0.16.24 every release was left; now a right release arriving mid-drag would have dropped or
disarmed it. The release arm acts on `CRAB_BTN_LEFT` only.

### Upstream — aethersafha 0.16.24 is RELEASED

The fix crab filed on 2026-09-09 is made and shipped: every kernel button bit is forwarded, window
management stays left-only structurally, the numbering is decided and pinned, and six assertions
that had never run in its input suite run now. Its own pre-cut gate (`scripts/check-dep-tags.sh`)
failed on five stale dep tags and they were moved after reading each diff. **27 / 27 suites, `input`
136 → 183.** ⭐ **Tagged `041ac85`, on the remote, CI and Release both green (2026-09-13)** — checked
with `git ls-remote --tags` and the Actions API, not read from a clone. crab's copy of the issue
records the resolution:
[`docs/development/issues/2026-09-09-aethersafha-forwards-only-the-left-button.md`](docs/development/issues/2026-09-09-aethersafha-forwards-only-the-left-button.md).

### Verified — the cut checks

`cyrius test` **1695 → 1790 / 0** (+53 pointer routes, +42 marker/trim/layout) · render_test
**53 / 0** · fuzz 100,000 rounds · `fmt --check` clean across `src/` and `tests/` · coverage **88 %**
(265/298) · `vet` + `deny` 0 · `deps --verify` 50 / 0 · host **1,049,480 B** `fa588e69…` · `--agnos`
**1,090,216 B** `3438489f…`. ⚠ Both sizes are UNCHANGED from the pointer-route build while every
hash moved — the marker, the trim and the layout fit inside the padding. `cmp`, never `ls -l`.
⭐ **Check four re-run at the cut**, all four `path` overrides disabled: 7 deps / 0 errors, lock
**3 → 7 commit-pinned**, both binaries **byte-identical** to the path-resolved build, 1790 / 0 in the
scratch copy. Every gate `ci.yml` runs, run here, at the 6.6.2 pin. Thirteen mutations across the
release (seven on the pointer routes, six on the marker) plus one on the layout, each caught.
⛔ **Not run on QEMU or iron** — the pointer arm and the sidebar's agnos `mountlist` path are both
invisible to the host suite by construction.

## [0.8.4] — unreleased — the aethersafha button blocker, measured and filed rather than guessed

> ⛔ **THIS SECTION EXISTS SO `[0.8.3]` BELOW IS NEVER TOUCHED.** 0.8.3 was committed and tagged
> (`a2fa067`) while this work was in flight, so it is a record now — **including where it is wrong**:
> its header still reads *"unreleased"* and its opening note still says *"nothing is committed, tagged
> or pushed"*. Both were true when written and both are false now. They are **left alone**, because
> editing a released section is the failure 0.7.2 exists to enforce against — the tag and the notes
> would disagree, and the notes are what a consumer reads.
> ⚠ **`VERSION` still reads `0.8.3`** and stays there until the operator cuts. This heading is where
> post-tag work accumulates, not a claim that a release happened.
> ⚠ Whether 0.8.3 is on the remote could not be checked from here — `git ls-remote` fails with
> `Permission denied (publickey)`. **An unverifiable tag is treated as released**, which is the safe
> direction: the cost of being wrong is a section that could have been edited and was not.

### Investigated — ⛔⛆ the context-menu pointer route is blocked upstream, and the block is bigger than the bug

**Asked for: fix the aethersafha button-code blocker. It could not be done, and shipping nothing was
the right answer.**

⭐ **The diagnosis is clean, and the loss is not crab's.** crab cannot tell a right-click from a left
one because the information never arrives:

- the **kernel** publishes a full bitmap — `bit0 left, bit1 right, bit2 middle`
  (`agnos/kernel/arch/x86_64/usb/hid.cyr:311`), with `buttons_seen` OR-folded beside it;
- **bhumi** passes both through intact (`bhumi_button_state` / `bhumi_button_seen`);
- **setu**'s wire carries `button` as a full i64 (`SETU_INPUT_PTR_BTN`);
- **dhancha** delivers it in `POINTER_BTN`'s `a`.

⛔ **aethersafha is the single point of loss.** `input_btn_transitions(ae_ptr_btn, cur, seen, 1)` —
mask `1`, commented *"left button only, for now"* — and then `ae_ptr_forward(comp, 1, 1, pressed)`
with the button number **hardcoded**. Everything else is discarded before the wire.

⭐ **And the fix there is small**: `input_btn_transitions` already takes a mask and `ae_ptr_forward`
already takes a button number. Only the belief state is a scalar and the two call sites pass
constants. ⛔⛔ **Window management must stay left-only** — the press arm does click-to-focus,
`deco_hit` close/maximize/minimize and drag-start, so a naive loop over all buttons would make a
**right-click close a window**.

### ⛔⛔ Why it was not done: aethersafha does not build on any available toolchain

| toolchain | result |
|---|---|
| `6.5.33` (its own pin) | **not installed, and not installable** — `cyrius install 6.5.33` answers *"Package registry not yet available."* The toolchain also flags the pin as carrying a **critical** defect and says to re-pin |
| `6.5.36` (the prescribed floor) | same — not installable |
| `6.6.0` | **57 errors**, none in aethersafha's own source |
| `6.6.1` | **57 errors**, none in aethersafha's own source |

The 57 reproduce against the **committed** `lib/`, so they are not an artifact of re-resolving deps.
They are dependency-versus-dependency conflicts: `sigil` and `agnostik` disagree about the `result_*`
arities (`'result_unwrap' expects 2 arguments, got 1`; `duplicate fn 'result_print_err' … last
definition wins, so calls to the other arity would silently mis-bind`), and `agnodrm` was never
updated for the `: stack` multi-return. ⚠ **`agnostik 1.5.1`, `agnodrm 1.5.3` and `sigil 3.12.16` are
each already at their repository's highest tag**, so no combination of existing releases resolves.

⇒ **Three upstream repos must reach the 6.6.x language, and agree with each other, before aethersafha
can be built, tested or changed at all.**

⛔ **crab shipped nothing rather than route around it.** The two available shortcuts were
hand-editing vendored `lib/` (forbidden) and pushing an unverified change to a repo that cannot be
compiled. Both are worse than the gap. ⚠ **aethersafha was left exactly as found**, at tag `0.16.22`
with its `6.5.33` pin — every file `cyrius deps` touched was reverted.

⚠ **And no button number was guessed.** No repository in the stack defines a button constant and
aethersafha is the only producer, so the numbering is an unmade decision. **X11's order is the wrong
default here**: setu already diverged from X11 deliberately on the neighbouring question, giving the
wheel its own kind rather than spending buttons 4/5 on detents. The filing recommends mirroring the
kernel's own bit order (`wire = bit + 1`, so `1 = left, 2 = right, 3 = middle`) — derivable rather
than remembered, and consistent with the one fact that exists. ⇒ Same discipline that declined a
mount probe before agnos minted `mountlist`#104: **a guess that happens to work is indistinguishable
from a contract until the day it changes.**

⭐ **Filed IN aethersafha** — `docs/development/issues/2026-09-09-forwards-only-the-left-button.md` —
because an issue about another repository that lives only in crab is one nobody who could act on it
will ever read. crab keeps a copy at
[`docs/development/issues/2026-09-09-aethersafha-forwards-only-the-left-button.md`](docs/development/issues/2026-09-09-aethersafha-forwards-only-the-left-button.md).

⭐⭐ **AND THE CAUSE IS A PIN SKEW, NOT A WALL.** cyrius changed `: stack` enums to return two values;
`sigil` (pin **6.6.0**) was migrated, `agnostik` and `agnodrm` (both **6.5.35**) were not. The work is
mechanical: **291 call sites** in agnostik, agnodrm's own set, `cyrius distlib` in each, then
aethersafha's pin. ⚠ `path` wins over `tag`, so the sibling's `dist/` is what compiles — a source fix
with no `cyrius distlib` behind it changes nothing.

### Verified — 0.8.3's cut checks, re-run after the tag

All nine gates green on the tagged tree: **1695 / 0** · host **1,045,288 B** · `--agnos`
**1,081,768 B** · render_test **53 / 0** · fuzz 100,000 rounds · `deps --verify` 49/0 · coverage
**88 %** · `vet` + `deny` 0 · `fmt --check` clean.
⭐ **Check four re-run with all four `path` overrides disabled**: 7 deps / 0 errors, lock **3 → 7
commit-pinned** (the tell the overrides were really off), and both binaries **byte-identical** to the
path-resolved build. ⚠ `build/` holds no fixture debris after a run.

## [0.8.3] — unreleased — M6 interaction gaps: the sidebar answers the keyboard, and `Open` was dead

> ⛔ **THIS SECTION EXISTS SO `[0.8.2]` BELOW IS NEVER TOUCHED.** 0.8.2 is tagged and on the remote.
> `git describe` answered `0.8.2` exactly before a word of this was written.
> ⚠ **`VERSION` reads `0.8.3`; nothing is committed, tagged or pushed.**

### Fixed — ⛔⛆ `Open` was DEAD on both menu surfaces, in every build that shipped either

The context menu and the menu bar both answer Enter by rewriting `u` to the key the chosen entry
names and falling through to the one implementation of that command. That is the right design, and
it is why the accelerator column cannot drift from what the entry does. **But both arms sat BELOW
the binding table**, so a rewrite only reached handlers defined further down. `CRAB_MI_OPEN` rewrites
to `0x28`, and `0x28` — Enter, descend-or-open — is handled **above** them. Choosing `Open` from
either menu consumed the keypress, closed the menu, and did nothing.

`r`/`n`/`d`/`c`/`m` worked, and nothing made that true but their line numbers.

⇒ **Both arms are hoisted above every binding**, so a rewrite reaches its handler whatever the
target is and a verb added to a menu later cannot be born dead. ⚠ Behaviour-neutral otherwise:
nothing between the new site and the old reads or writes `mb_sel`/`mb_drop`/`mb_item`/`menu_open`/
`menu_sel`, and `mcnt2`/`misdir2`/`mmark2` are established far above, on the event.

⭐ **And the map they both copied is now one function** — `crab_menu_accel(mi)` in `src/ui.cyr`,
where the suite can reach it. Both six-line chains lived inside `src/main.cyr`'s agnos-only `#ifdef`
with no `#else`, so neither was assertable. The test that matters is the **agreement**: the
accelerator the operator reads in the menu must be the key the entry sends. ⚠ `Open`'s exemption —
Enter is not a character, so `crab_key_char` answers 0 — is asserted rather than skipped.

### Added — ⛔ the PLACES sidebar answers the keyboard (M6 gap closed)

The sidebar shipped reachable **only by mouse**, in an application whose own source says it is
*"keyboard-first by construction: a menu only a mouse can reach is invisible to an operator who
never touches one"* — and on agnos the compositor may spawn crab **with no pointer at all**, which
made the sidebar purely decorative there.

`Tab` (previously unbound) moves focus between the panes and the sidebar; arrows move a cursor that
steps **over** the inert section headers; Enter sends the active pane to that row's path; Tab or Esc
returns. Six pure functions own every rule — `crab_sb_rows`, `crab_sb_first_row`, `crab_sb_step`,
`crab_sb_row_of`, `crab_sb_path`, `crab_sb_key` — because the dispatch that calls them cannot be
tested at all.

⛔⛔ **FOCUS IS A MODE, NEVER A THIRD PANE.** `active_pane` keeps its 0/1 domain: `if (active_pane ==
1) { … } else { … }` appears dozens of times in `src/main.cyr`, including the arms that pick the
**delete target** and the copy/move source, and a 2 would read as pane A in every one of them.
`active_pane` stays meaningful while the sidebar is focused — it is the pane Enter sends to, which
is the rule the click path already followed.

⛔⛆ **AND THE MUTATING VERBS ARE EATEN, NOT LEFT LIVE.** A focus model that consumed only its own
keys would leave `d` deleting, `c`/`m` transferring **with no confirmation at all**, `r` renaming,
`n` making a directory and Backspace ascending — every one against a pane the toolkit is painting
**muted** because the keyboard is somewhere else. They are consumed and refused out loud; silence is
not an option, because a key that does nothing and says nothing is indistinguishable from a dropped
keypress and the operator's next move is to press it again. ⚠ The assertion is stated as the
**property** — *"while the sidebar is focused, `0x10` never answers NONE"* — not as a list of codes a
future verb could quietly fall off.

⛔ **Esc is not unconditional**: while a transfer runs it still cancels the transfer. That has been
Esc's meaning since 0.7.3 and it outranks leaving a focus mode.

⛔⛆ **THE CURSOR IS NEVER SEEDED FROM WHERE THE PANE IS.** A location is usually an *ancestor* of the
pane's path, so seeding from it would make Tab-then-Enter navigate the pane **up** and clear its
marks — from a gesture that reads as a no-op. Tab seeds from the first selectable row.

⛔⛆ **AND FOCUS IS CLAIMED AFTER THE PANE BUILD.** The sidebar must be built first to be the leftmost
child, but `crab_pane` calls `dh_focus_set` on the active pane — so a sidebar claiming focus during
its own build would lose it three lines later, every frame, and paint its row under the muted line
colour instead of `accent`. The operator would be driving a list that does not look driven. ⚠ Pinned
by a render assertion, not by reasoning.

⚠ **`crab_sb_shown_row(focused, cursor, here)` takes three arguments, and the third state is the one
that lies.** `dh_draw_list_selection` paints under `accent` when the list has focus and under the
muted line otherwise, so the contract is *muted = you are here, accent = your keys drive this*. A
focused sidebar with no cursor therefore paints **nothing** — falling back to a location would put a
location under `accent`, reading as "Enter acts on this row" while Enter would send the pane to an
ancestor.

⚠ The click path writes the cursor too, or the two producers of one cursor drift; a **pane click
returns the keys to the panes**; and the held-key repeat arm — a second, mode-blind copy of Up/Down —
now asks the same `crab_sb_key`.

### Added — the menu bar has a fit rule, and its drop-downs have a second one

The bar was the only M5/M6 surface with **no fit rule**. Its intrinsic width is fixed and crab
accepts any window width, so a narrow window **clipped it silently** — "File Edit Go" and no View,
with nothing saying a menu was missing. `crab_menubar_fit` / `crab_menubar_sel` refuse it instead,
asked by **both** F10 and the renderer so the key cannot report a bar the frame will not draw, and a
window resized narrower drops the bar rather than clipping it.

⛔⛆ **THE DROP-DOWN IS A SECOND RULE AND THE FIRST DOES NOT IMPLY IT.** `dh_place_at_point` **clamps**
a popup that would overhang, sliding it left until it fits — so on a narrow window `Edit`'s menu,
Delete included, hangs under the word **File**. A menu pointing at the wrong label is worse than an
absent one: the operator reads the label to know what they are in. The bar needs its intrinsic width;
a drop needs `crab_mb_cell_x(m) + CRAB_MENU_W + CRAB_MENU_MARGIN`. ⇒ **There is a whole band of
widths where the bar fits and Edit's drop still opens misplaced**, so a single threshold would
certify as good the exact widths where the symptom survives. Asserted as that band.

⚠ `CRAB_MENU_MARGIN` is lifted out of a bare `4` at `dh_place_at_point`'s last argument — two
spellings of one margin would let the rule certify a width the placer then clamps.

### Fixed — a bounds read in a destructive guard, without changing its answer

`crab_path_within` — the guard that refuses copying a folder into itself — read `root[-1]` when the
root was empty. ⛔⛔ **The fix guards the READ and deliberately leaves the ANSWER alone.** Returning 0
for an empty root would read as tidier and would **flip a destructive guard's fail-safe direction**:
the only production caller is the copy-into-itself refusal, where 1 means *refuse*. A bounds fix must
never change a safety answer in the same edit. Both halves are asserted.

### Changed — `crab_goto`, and two stale contracts

⭐ **`crab_goto` makes adopt-relist-clear one indivisible move.** crab's marks are INDEX-based, so a
mark that outlives the listing it was made in points at whatever now occupies that index — the shape
the 2026-09-03 `/bin` incident was filed against. It was written out twice (once per pane) and the
keyboard route would have made four copies, each free to forget the clear. ⚠ Callers must guard
`n >= 0`: a -1 written into a pane's count reaches `crab_maxsel` and every verb arm as a live row
count.

⛔ **`crab_sidebar_hit`'s header said, three times, that it returns a PLACE index and "subtracts the
inert header row". It does neither** — it returns the raw LIST row, as its own ⚠ twelve lines below
already said. The two halves of one comment had contradicted each other since VOLUMES landed and made
the translation stop being a subtraction; a caller who believed the header would index the PLACES
array with a VOLUME's row. Corrected here and in the roadmap.
⛔ `src/main.cyr`'s `#ifdef` self-reference said "263 … 1484"; it is **293 … 1907**.

### Verified

**1695 passed / 0 failed** (from 1676) · host **1,045,288 B** · `--agnos` **1,081,768 B** ·
render_test **53 / 0** · fuzz 100,000 rounds · `deps --verify` 49/0 · coverage **88 %** ·
`vet` + `deny` 0 · `fmt --check` clean.

⚠ Mutation-proven, including the two that matter most: dropping `d` from the eaten set fails 2, and
claiming focus before the pane build instead of after fails the render assertion.

### Still open in this milestone

⛔ **Four of the six M6 interaction gaps remain**, with a designed and adversarially-verified plan
recorded: the menu bar / A/B switcher pointer routes, the context-menu pointer route (**crab cannot
currently tell a right-click from a left one — `POINTER_BTN` carries the button code in `a` and
crab's arm reads only `b`**; aethersafha forwards button 1 hardcoded, so this is gated upstream and
must not be guessed), the sidebar's *you are here* marker, and `View`'s items. ⚠ The pointer routes
**must land together**: a bar click that opens a drop-down the pointer can neither pick from nor
dismiss is a half-wired gesture. ⛔ `Go` is deliberately **not** being filled — see the roadmap.

## [0.8.2] — 2026-09-09 — the audit backlog's correctness bugs, and a recursive walk that never ran

> ⛔ **THIS SECTION EXISTS SO `[0.8.1]` BELOW IS NEVER TOUCHED.** 0.8.1 is tagged and on the remote,
> so it is a record now. `git describe` answered `0.8.1` exactly before a word of this was written —
> HEAD *was* the tag — which is the check 0.7.2 exists to enforce.
> ⚠ **`VERSION` still reads `0.8.1` and stays there until the operator cuts.** The heading is where
> post-tag work accumulates; it is not a claim that a release happened.

### Fixed — ⛔⛆ RECURSIVE COPY AND RECURSIVE DELETE HAD NEVER RUN, IN ANY SHIPPED BUILD

`d` on a folder asked *"delete this FOLDER and everything in it?"*, took the `y`, and **deleted
nothing** — reporting `failed part-way: the copy is incomplete`. A recursive copy created the
destination directory and then failed the same way, leaving an empty folder wearing the source's
name. This has been true since M4 shipped the walk.

**The mechanism is one call.** `crab_op_step` is the dispatcher — single-file kinds to the chunk
loop, `CTREE`/`DTREE` to `crab_walk_step` — and it carries the comment *"THE single entry point the
idle tick calls"*. The idle tick called `crab_copy_step` instead, which is only the chunk loop. It
reads `CRAB_OP_FIN` unconditionally, and a walk's `FIN` is `-1` until a file is actually open, so
`sys_read(-1, …)` returned EBADF, the loop took its `got < 0` arm, and the walk died on its **first
idle tick**. `crab_op_step` had **zero callers**.

⛔⛆ **THE SUITE PASSED THROUGHOUT, AND THAT IS THE PART WORTH KEEPING.** Every walk test drove
`crab_op_step` — the *right* entry point — while the only caller that ships drove the wrong one. The
tests were green, thorough, and testing a function production did not call. ⇒ **A test that calls a
different function than the shipping caller is not testing the shipping path.** The new block names
the production call site rather than the tidy one.

⚠ Proven before it was fixed, not after: driving the new test through `crab_copy_step` returns
**-15** and leaves the tree on disk; through `crab_op_step` it returns OK and the tree is gone.
⚠ `crab_copy_step` now **refuses a walk kind** (returns 0, "nothing to do") instead of reading a
descriptor that is not open — so the wrong caller gets something a test can see rather than a
plausible I/O failure against a real filesystem.
⚠ The idle tick also maps the walk's `2` ("an item finished") onto "keep going", or a tree copy
would report *done* at its first completed file.

### Fixed — a cancelled tree copy takes its partial tree with it

`crab_copy_cancel` unlinked `CRAB_OP_DST` — the **one file in flight** — and left every directory
and every completed file the walk had already written sitting at the destination under the source
folder's own name. The operator cancelled and was shown a folder that looks copied.

The function's own rule for a single file — *"leaving a truncated file wearing the real file's name
is the worst outcome available"* — is **more** true of a tree, not less: a half-copied folder cannot
be told from a whole one without comparing it entry by entry. A cancelled `CTREE` now re-roots the
**same record** as a `DTREE` over what it wrote, stepped by the idle tick like any other tree
operation, with the status line saying `cancelled — removing the partial copy...`.

⛔⛔ **WHY DELETING THERE IS SAFE, AND IT RESTS ON ONE INVARIANT.** `crab_walk_begin` refuses with
`EEXIST` if the destination already exists, then creates `DROOT` itself. So `DROOT` is **always a
directory crab made during this very operation** — removing it undoes crab's own work and can never
reach anything the operator put there. ⇒ **If that guard is ever relaxed to allow merging into an
existing destination, the cleanup becomes a data-loss bug and must be deleted in the same change.**
Written into the function, because the 2026-09-03 `/bin` incident is what crab has already paid for
once. ⚠ Mutation-proven in the dangerous direction: aiming the cleanup at the SOURCE root fails
`...every level of it`, which is the assertion that separates a correct cleanup from data loss.

⚠ **And a cancel now stops the QUEUE explicitly.** The idle tick's rule is *"a refusal does not stop
the queue — only a cancel stops everything"*, and a cancel enforced it only by **accident**: it
released the record, so the tick's `crab_op_active()` guard went false and the rest were stranded
rather than stopped. The cleanup keeps the record alive, so that accident is gone — without the
reset the tick would reach `crab_queue_advance` when the cleanup ended and start the next marked
file, from a keypress the operator meant as *stop*.

### Fixed — drag between panes ignores the marked set

Dragging with ten files marked moved the **one under the pointer** and silently discarded the other
nine. Every other verb honoured marks; drag predates M4 and was never revisited, so crab held two
descriptions of "move" that disagreed.

A drop is a move, so it now answers to **`crab_transfer_plan`** — the same pure function `c`/`m`
call. The rule is not restated at the drop site; that is the only way the two stay in step, and it
is reachable by the suite, which the drop branch is not.

⛔ **A BUG INTRODUCED AND CAUGHT WHILE WIRING THIS, recorded because the shape recurs**: the folder
arm read `namescr` before writing it. `namescr` is a long-lived scratch shared with every other
verb, so that arm would have acted on **whatever name the previous operation left there**. The name
is now captured once, above every arm.

### Fixed — `m` across filesystems blocked the event loop

`src/main.cyr` read *"a move still tries `rename` first … so the common case never reads or writes a
byte"* and then called `crab_fs_move`, which on a rename failure runs `crab_fs_copy` — a **blocking**
whole-file copy — to completion inside the keypress branch. No frames were drawn while it ran, so
**the tray never appeared and Esc could not cancel it**, and the stepped `CRAB_OP_MOVE` path was
unreachable for the one case it exists for. A large file moved across a mount point froze crab.

**`crab_fs_move_rename`** is the cheap half alone, answering the new **`CRAB_FS_EXDEV`** when rename
refuses — an *instruction to the caller*, not an answer for the operator. Both call sites (the `m`
key and the drag) now step the copy instead of blocking. ⚠ `crab_fs_move` still blocks, deliberately,
for callers that want the move finished on return; that is asserted so the split is a new entry
point rather than a silent behaviour change. ⚠ The real refusals stay real — `ESAME` and `EEXIST`
must not be swallowed by `EXDEV`, or the operator would be told a transfer started when crab refused.

### Fixed — a folder could not be moved at all, even within one filesystem

crab answered *"folders cannot be moved yet — copy it, then delete the original"* on the reasoning
that no `CRAB_OP_MTREE` walk exists. That reasoning is **sound across filesystems**, where the only
implementation would be copy-then-delete wearing a move's name. It was **never true within one**,
where the kernel does the whole job in a single `rename` and there is nothing to walk. One correct
sentence had been applied to two different cases.

`CRAB_PLAN_NOMOVEDIR` becomes **`CRAB_PLAN_MOVEDIR`**: plan the *attempt*, and let rename — the only
thing that knows whether two directories share a filesystem — decide. `EXDEV` is then refused out
loud, which is the original reasoning applied where it is actually true.

### Fixed — a spent thumbnail budget destroyed working thumbnails

`crab_thumb_step` claimed a cache slot and **only then** asked whether it was allowed to decode.
Slots are evicted round-robin, so once the session ceiling was crossed every image scrolled past
destroyed a good thumbnail to store a refusal where it had been. **The cache emptied itself fastest
at the exact moment its contents had become irreplaceable** — past the ceiling no further decode can
run to rebuild them. ⇒ A refusal that costs a good entry is more expensive than the decode it
declined to do.

The order is inverted, and the decision is lifted into **`crab_thumb_may_claim`** — the same answer
this codebase gives whenever a situation needs 32 MB of real spend to reach but the *decision* does
not. ⚠ The gallery's idle walk now asks the budget **directly** rather than relying on the SPENT
refusal being cached: that refusal is no longer cached, so the cache-miss gate would have stayed
open forever and turned the idle loop into a busy loop repainting an unchanged frame.

### Fixed — with a gallery open, the preview drew the gallery's picture

`crab_thumb_pixels()` reads `crab_th_slot`, which records what the last **step** was about — and
with a gallery open the idle tick steps gallery cells. So the preview column drew whichever cell the
walk had just decoded, under the selected entry's name. **And it persisted**: the preview's own
redraw fires only when its state *changes*, and OK -> OK is not a change, so nothing ever corrected
the frame.

**`crab_thumb_slot_for`** answers about a *named* entry, decoding nothing and touching nothing, and
the gallery's redraw now uses it for the selection. ⚠ The property the fix rests on — that a lookup
does not disturb `crab_th_slot` — is asserted directly rather than reasoned about.

### Fixed — the gallery's arrow keys were never wired

The arrow dispatch tested `view_mode == CRAB_VIEW_GRID` in **three** places, under a comment
describing the behaviour for cell views generally. The gallery is the same `dh_grid` with a taller
cell, so in it Left/Right still switched panes and Up/Down stepped by **one entry** rather than by a
row: a 2-D arrangement the operator could see, answering arrows as if it were a list. One predicate,
**`crab_view_is_grid`**, is now asked in all three places, so a fourth view cannot be added and
half-wired the way this one was.

### Fixed — gallery cells never said WHY a thumbnail was missing

The *"four differently-named nothings"* rule was honoured in the preview and nowhere else. A cell
whose decode had been refused drew an empty band and said nothing, so "too large", "budget spent",
"cannot decode" and "not decoded yet" were one identical blank — **in the view that shows dozens at
once**, which is exactly where telling them apart matters most. A screen of blanks reads as a broken
gallery.

`crab_thumb_note_cell` gives the same four states cell-width words: `too large` · `budget` ·
`no decode`. ⛔ **`NONE` and `OK` stay silent, and that is the load-bearing half** — an entry the
idle tick has not reached yet is not a refusal, and labelling it would report a failure that never
happened, on every cell of a gallery the instant it opens. ⚠ The notes are chosen to fit a 10-character
cell **whole**, asserted against `crab_col_chars(CRAB_GAL_CELL_W)`: a clipped "too large to prev"
reads as a rendering fault.

⚠ The gallery's reserved band is now a `LABEL` rather than a `BOX`. Three existing assertions pinned
the widget *kind*; the invariants they were protecting are that **no CANVAS is drawn over a slot
whose pixels were never written** and that the band keeps its height — both now asserted directly,
which is stronger than the kind check they replace.

### Verified

**1571 passed / 0 failed** (from 1462; +109 assertions) · host **1,037,024 B** · `--agnos`
**1,073,288 B** · render_test **53 / 0** · fuzz 100,000 rounds · `deps --verify` 49/0 · coverage
87 % · `vet` + `deny` 0 · `fmt --check` clean on all 8 files.

⚠ **EVERY FIX ABOVE IS MUTATION-PROVEN**, including the two that matter most for data safety: aiming
the cancel cleanup at the SOURCE root fails 3 assertions, and reverting the idle tick to
`crab_copy_step` fails the walk outright.
⛔ **WHAT NO HOST TEST REACHES, STATED PLAINLY**: the idle tick, the arrow dispatch and the drop all
live inside `src/main.cyr`'s single `#ifdef CYRIUS_TARGET_AGNOS` with no `#else`. That is *why* this
batch's defects survived, and it is why each fix moved its RULE into a pure function the suite can
interrogate — `crab_transfer_plan`, `crab_view_is_grid`, `crab_thumb_may_claim`,
`crab_thumb_slot_for`, `crab_thumb_note_cell`. What remains untestable is that `main.cyr` still
calls them: the irreducible gap, unchanged in shape and smaller in surface.

### Investigated — what this batch means for the `/bin` incident

⛔ **Still 🔴 OPEN, and the mechanism is still unproven** — but the search space is smaller.
**`DTREE` could not delete anything on the burned build**, so every hypothesis routing through a
recursive tree delete is now impossible for 0.8.0, including the `crab_pointer_modal` path's step 5
(*"if entry 0 of the newly-listed directory was a folder, that is a recursive tree delete of a
directory the operator never chose"*). That path could still delete a wrong single FILE, which is
what the five `/bin` entries were. ⚠ All 16 `crab_relist` call sites were re-audited at HEAD and
each clears marks within four lines, so the stale-marks hypotheses stay refuted.

### Changed — toolchain pin `6.6.0` -> **6.6.1**

⛔⛆ **THE BUMP LANDS ON crab's SHIPPING TARGET, AND THE OLD BEHAVIOUR FAILED SILENTLY.** 6.6.1
rebinds `chrono`'s AGNOS monotonic clock from `sys_uptime_ms` (**#40**, `timer_ticks`) to
`sys_uptime_us` (**#95**, `rdtsc`). A foreground `run` program on AGNOS executes with **IF cleared**
— only `/bin/agnsh` gets IF=1 — so the 100 Hz timer ISR never fires, `timer_ticks` never advances,
and #40 is **frozen for that program's entire run**: anything timing itself with it read exactly
zero, forever, **with no error**. ⭐ **crab is spawned by the compositor, so it is precisely that
shape of program.** Resolution improves as a side effect — µs rather than the 10 ms tick.

⚠ **The trap was documented two files away the whole time**, above the wrapper `chrono` did not call
(`lib/syscalls_x86_64_agnos.cyr`, `sys_uptime_us`: *"#95 is the only correct clock there"*), and
agnos's own ABI note records that it cost two iron burns before it was understood. A general-purpose
clock still walked into it for four minors. ⇒ **A correct note in the right file is not a guard.**

### Changed — vendored `lib/`: two leaves moved, and only two

- **`lib/chrono.cyr`** — the clock rebind above.
- **`lib/sankoch.cyr`** — 2.7.11 → **2.7.14**: an `out_max` absolute output ceiling threaded through
  `_deflate_decode_block` / `_deflate_decompress_inner` and the dict variants, replacing the fixed
  `DECOMPRESS_MAX_OUTPUT` compare. crab reaches it **transitively through chitra**, which is the
  ~399 KB inflate leaf PNG requires; crab calls none of it directly.

⭐ **The other five stdlib files 6.6.1 touched are NOT in crab's graph and did not land** — `ganita`,
`math`, `niyama`, `patra`, `sakshi`. Establishing that is the point: *"the pin moved"* and *"crab's
vendored tree moved"* are different claims, and only the second one is testable here.

⛔ **`cyrius deps` refreshed both leaves correctly this time, and that was VERIFIED rather than
assumed** — every file in `lib/` hashed against `~/.cyrius/versions/6.6.1/lib/`, not inferred from
the command's exit code. The 6.5.41 bump is why: `cyrius lib sync` walks only the **declared** stdlib
set, so three transitive thread leaves stayed at 6.5.36 content and had to be copied by hand, and
`lib/hashseed.cyr` arrived untracked. **Diff the whole vendored tree after any bump.**

### Verified — all nine CI gates, at the new pin

`deps --verify` **49/0** · host build **1,036,944 B** (from 1,032,848; +4,096) · `--agnos`
**1,068,976 B** · `cyrius test` **1462 / 0** · `render_test` **53 checks / 0** · fuzz **100,000
rounds** · `fmt --check` clean on all 8 files, one invocation each · coverage **87 %** (floor 85) ·
`vet` + `deny` **0 violations**.

⭐⭐ **AND CHECK FOUR WAS RE-RUN, WITH THE STRONGEST RESULT IT HAS EVER RETURNED.** In a scratch copy
with **all four `path` overrides disabled**, so `cyrius deps` genuinely clones the tags: 7 deps / 0
errors, lock **3 → 7 commit-pinned** (the tell the overrides were off), **1462 / 0**, and both
binaries **byte-identical to the path-resolved ones** — host `1,036,944`, agnos `1,068,976`.
⇒ That equality says more than *"the declared graph resolves"*: it says **the declared graph is what
the local build has been compiling all along**. ⚠ It holds only because every sibling tree sits
exactly on its tag, clean — `path` is what makes drift possible, so re-derive it rather than
assuming it.

### Changed — dependencies: already current, and confirmed against the remotes

All **seven** declared tags equal that repository's highest tag **on its remote**, fetched rather
than read from a stale clone: sadish `0.5.3` · rupa `0.1.6` · rekha `0.3.6` · kashi `1.0.6` ·
dhancha `0.9.28` · setu `0.8.8` · chitra `1.0.1`. **Nothing moved**, which is the finding.
⚠ rekha's clone is one commit past `0.3.6` (`6560734`, *"update ci gates"*) — crab resolves rekha
**by tag only**, deliberately, so that commit is outside crab's graph and no floor changes.

### Fixed — documentation currency: state.md rotted a THIRD time, and so had handoff.md and the README

⛔⛆ **THE THIRD ROT SURVIVED THE FIRST TWO BEING WRITTEN UP DIRECTLY ABOVE IT.** From 2026-09-02 to
2026-09-08, `docs/development/state.md` asserted *"0.8.0 in preparation"* and *"0.7.7 is the last
RELEASED version"* across **two tagged releases**, a pin that had moved twice (6.5.41 → 6.6.0 →
6.6.1), a dependency table naming **rekha 0.3.5** and **dhancha 0.9.26** against a manifest declaring
0.3.6 and 0.9.28, a **"6 deps"** count against seven, *"VOLUMES — enumeration still open"* after
0.8.1 shipped it, and *"there is no `rekha_advance_width`"* after rekha 0.3.6 added it.

- **`docs/development/handoff.md`** carried the same numbers **in its *Where things stand* table** —
  the one its own header points at with *"read this and nothing above it for numbers."*
- **`README.md` § Status was wrong for the THIRD time, and under-claiming again**: *"No columns
  (miller) view, and no sidebar"*, naming the `mount`/`umount` stubs as *"the one genuine block
  left"*, when `b` opens PLACES and 0.8.1 ships VOLUMES on `mountlist`#104. ⇒ The section's own ⛔
  says under-claiming misleads exactly as much as over-claiming; it has now done both twice.
- **`docs/development/roadmap.md`**'s M6 bullet and gate-table row still described VOLUMES as filed
  and waiting.

⇒ **The rule was already written and is still the only thing that would have worked**: refresh
`state.md` in the *same commit* as the CHANGELOG entry. Nothing enforces it — `cyrius audit` does not
gate doc currency — so it lives in `CLAUDE.md` Process step 5 and nowhere else.

### Investigated — the `/bin` incident, re-checked at HEAD rather than at the burned tag

⛔ **Still 🔴 OPEN; the mechanism is still unproven.** All **16** `crab_relist` call sites in
`src/main.cyr` were re-audited **at HEAD** (0.8.1 + this section's changes) and every one clears
marks within four lines — including the sidebar-click path, which rewrites `lpath` and relists a
*different* directory and is the exact shape hypothesis #1 describes. ⇒ The stale-index-marks
hypotheses stay **refuted** at HEAD, not merely at the tagged 0.8.0 the issue audited.
⚠ **What remains is unchanged**: the batch sheet's relist, and ordinary operator error. ⛔ **The two
preventions shipped in 0.8.1 are not a diagnosis, and there is still no undo.**

## [0.8.1] — 2026-09-07 — VOLUMES: the gate crab filed is closed, and the fix was the one we asked for

> ⛔ **THIS SECTION EXISTS SO `[0.8.0]` BELOW IS NEVER TOUCHED.** 0.8.0 is tagged and on the remote,
> so it is a record now — including where it is wrong. `git describe` answered `0.8.0-1-g254b9e2`
> before a word of this was written; checking that first is the habit 0.7.2 exists to enforce.

### Fixed — ⛔⛆ the delete prompt now NAMES what dies, and refuses to be quiet about `/bin`

Five system binaries left an AGNOS iron box on one confirm and nothing surfaced it until the machine
would not boot —
`docs/development/issues/2026-09-03-five-contiguous-bin-entries-deleted-on-agnos.md`. **The mechanism
is still unproven.** These are the two guards that would have prevented the OUTCOME whatever the
trigger was, which is why they are worth having before the cause is known.

**The prompt said `delete the MARKED entries?`** — true, and it never said the set was **five files
and one of them was `agnsh`**. It now reads `SYSTEM DIR! delete 5 marked: agnsh, boot, cp +2 more?`.
⚠ Three names then a remainder, because the status line is one row and nine names wrap into nothing.
⛔ The `SYSTEM DIR!` warning leads, where the eye lands.
⛔ **Exact match on the directory, not a prefix test** — `/bin` is guarded, `/bin/sub` is not, because
a guard that swallowed the subtree would make it undeletable and teach the operator to route around
the warning. The rows that died were direct children of `/bin`.

⛔⛆ **AND THE TEST CAUGHT AN INVERSION BEFORE IT SHIPPED.** `crab_streq_n` returns **1 for equal** —
it is not `strcmp` — and the first draft tested `== 0`, so the guard was backwards: it warned on
`/home/macro` and stayed silent on `/bin`. Caught by the assertion pair that names `/bin` and
`/binary` in the same breath, which is exactly why that pair is in the suite. Mutation-proven:
re-inverting it fails 8, dropping the names fails 4, dropping the warning fails 2.

### Investigated — what the incident is NOT

Checked against the **tagged 0.8.0**, which is the code that was on the box, not against HEAD:

- **Stale index-based marks surviving a re-list** — the issue's leading hypothesis. **Refuted.**
  Every `crab_relist` call site in the burned tree clears marks within three lines, and each clears
  **its own** pane's marks (checked for the cross-pane shape too, which would have been subtler).
- **Marks surviving a re-SORT**, where a permutation would leave indices pointing at different
  files. **Refuted** — the `s` handler cleared both panes' marks in 0.8.0 already.
- **`crab_mark_clear` bounded by the live count**, which would leave high-index marks alive across a
  small directory. **Refuted** — all 24 call sites pass `CRAB_MAX_ENTRIES`, not the count.

⚠ **One thing found that IS real, and does not explain the shape.** The tagged 0.8.0 has **no**
`crab_pointer_modal`: a click during a delete confirmation could re-list the pane and change what
`y` deleted. That was live on the burned box. But it yields **one** wrong entry (or one wrong tree),
not five contiguous siblings — so it is an unguarded path to a wrong delete, not this incident's
mechanism. Fixed after the 0.8.0 tag; see the moved section below.

### Added — sidebar VOLUMES (M6), on agnos `mountlist`#104

⭐⭐ **THE BLOCKER crab FILED ON 2026-09-02 IS REPAIRED, AND agnos TOOK BOTH PIECES OF ADVICE IN THE
FILING.** Their ABI credits it by name and adopts the reasoning verbatim — *"it is an enumeration
because a probe cannot answer it"* — plus the recommendation to **mint a new number rather than
widen `mount`#11**, whose unused argument registers carry stale values rather than 0. crab now
enumerates mounts instead of guessing at three hardcoded prefixes, which is what the filing asked
for and why it declined to ship the probe.

The sidebar grows a second section: one row per mounted volume, its filesystem name over a capacity
bar. Clicking one navigates to its mount prefix, exactly as a place does — a volume *is* a path.

⛔ **BOTH RECORD LAYOUTS ARE FROZEN ABI AND NEITHER IS VENDORED**, so crab spells them out:
`mountlist` gives 80-byte records (backend @0, prefixlen @8, prefix @16 — **64 bytes NUL-PADDED, and
the ABI says the tail is not meaningful**, so crab copies `prefixlen` bytes and terminates it
itself); `statfs` gives 32 bytes (`f_bsize` @0, `f_blocks` @8, `f_bfree` @16). ⚠ The kernel reports
**blocks**; userland does the multiply.

⛔⛆ **THE ALIAS DEDUP IS WHY AN ENUMERATION WAS NEEDED.** agnos's `vfs_mount_init` gives an
ext2-less boot the **same backend under both `/` and its `/mnt/…` prefix** — its own comment calls
them "harmless redundant aliases". Harmless to routing; to a sidebar they are **one volume listed
twice**, and `statfs` on each returns identical numbers, so a probe cannot tell them apart. The
backend id travelling with the prefix is what distinguishes them. crab keeps the **first** occurrence
per backend, which is the shortest prefix, because `vfs_mount_init` adds `/` before any `/mnt/…`.

⛔ **AGNOS-ONLY, AND THE `#ifdef` IS THE WHOLE FUNCTION.** Neither syscall has a host arm, so
`crab_volumes_build` returns 0 on the host and the sidebar shows PLACES alone — a platform
difference, not a failure, and the VOLUMES heading is **absent rather than empty**. ⇒ **No host test
can exercise it**, which is why everything decidable was lifted into pure helpers the suite *can*
reach: the record accessors, the capacity arithmetic, the alias dedup and the row map.

⛔ **THE ROW MAP IS A FUNCTION, NOT ARITHMETIC AT THE CALL SITE.** Two sections and two inert headers
mean a row is no longer `index + 1`. **This exact off-by-one has shipped twice here** — the context
menu highlighting the wrong verb when a separator was hand-counted, and the sidebar's own `row - 1`
which was correct only while PLACES was the only section. `crab_sb_row_kind` / `crab_sb_row_index`
own it now.

⚠ **The volume row is a `BOX_V`, deliberately.** `dh_progress_new` leaves its width 0, and in a
`BOX_H` the bar's thickness is the cross axis, which `ALIGN_STRETCH` ignores — so it would render
0 px wide. It also does **not** copy `crab_tray`, whose own bar was 0 px *tall* for the mirror-image
reason until 0.7.7.

### Testing

44 assertions over the reachable half. Mutation-proven: removing the alias dedup fails 3, showing
FREE instead of USED fails 6, and a row map that forgets the VOLUMES header fails 4 — including
`row 5 -> VOLUME 0` returning place 1.

### Moved here from `[0.8.0]` — these shipped AFTER that tag

⛔⛆ **THE 0.7.2 FAILURE, AGAIN, AND THIS TIME TO OUR OWN NOTES.** The four sections below were
written into the `[0.8.0]` body and then committed **after `0.8.0` was tagged** (`7db2422`) — so the
released section described fixes the released artifact does not contain. Verified by measurement:
`git show 0.8.0:src/ui.cyr` has **no** `crab_pointer_modal`, **no** `crab_mb_item_first` and **no**
`crab_sidebar_shown`. ⚠ The tag's own copy of the CHANGELOG never claimed them — the drift was
introduced by the commit that landed past the tag, which is precisely the shape 0.7.2 exists to warn
about. ⇒ Moved to the release that actually carries them.
⛔ **This has now happened twice in five days.** `git describe --tags` before writing into the top
section is not a habit yet; it needs to be.

### Fixed — ⛔⛆ A MODAL QUESTION DID NOT OWN THE POINTER, AND THE WORST CASE WAS A WRONG DELETE

Found by a completeness audit over M1–M6, in code this same release added.

`d` latches a delete confirmation and the status line asks *"delete this FOLDER and everything in
it? y = yes"*. ⛔ **That latch is on the STATUS LINE, not an overlay** — so nothing pruned pointer
input the way the overlay layer prunes it for the menu and the sheet. The chain:

1. A sidebar click, unguarded, rewrote `lpath`, ran `crab_relist`, called `crab_mark_clear` and set
   `sel = 0`.
2. `y` then found `dmarked == 0` — the marks had just been cleared — and took the **single-entry**
   branch against `dsel = 0`.
3. If entry 0 of the newly-listed directory was a folder, that is
   `crab_walk_begin(CRAB_OP_DTREE, …)` — **a recursive tree delete of a directory the operator never
   chose, from a "y" they typed about a different file.**

⛔ **THE SAME HOLE EXISTED FOR PANE CLICKS AND PREDATES THE SIDEBAR**: clicking another row between
`d` and `y` moved the selection, so the prompt named one entry and the delete took another. It was
invisible while the overlay layer was 2 px tall and pruned nothing — every surface was equally
unguarded. Fixing the overlay root made `crab_hit` modal and left the sidebar as the one live hit
path, which is *worse* than uniformly absent, because the comment then described a half-truth.

⇒ `crab_pointer_modal` refuses pointer input while a confirmation, a sheet, the context menu or the
menu bar is up. The keyboard side was already right — every modal state is checked before the
bindings and consumes the key — and this is the half that was missing. ⚠ It is a **predicate**, not
an inline test, because the key dispatch it guards is inside the agnos-only `#ifdef` that no test on
any target can execute. Mutation-proven: dropping the confirmation arm fails 3, and testing `mb_sel`
with `!= 0` — which would read menu `File` (index 0) as closed — fails 4.

### Fixed — a bar drop-down could land on a greyed row, and Enter fired it anyway

A disabled entry is made INERT, and `dh_list_select` **refuses** an inert row — storing nothing and
returning -1 — so an open drop-down painted **no highlight at all** while `mb_item` pointed at the
greyed verb and Enter rewrote its key and ran it. Opening a menu landed on slot 0 regardless, and
arrowing stepped over disabled rows without noticing them. The context menu has skipped disabled
entries since it shipped; the bar simply never grew the equivalent. New `crab_mb_item_first` /
`crab_mb_item_move`. Mutation-proven: 3 and 4 failures.

### Fixed — `b` reported success and drew nothing whenever the preview was open

The key handler asked `crab_sidebar_fit(w, 1)` — against the whole **window** — while `crab_render`
asks against what the preview left. On a window wide enough for one of them but not both, the key
said *"sidebar on"* and nothing appeared. **A key that says it worked and does not is
indistinguishable from a broken one**, which is the exact discipline `p`'s own comment claims. One
`crab_sidebar_shown`, asked by both, so the answer cannot differ. Mutation-proven: 2 failures.

### Docs — 38 unfinished items from M1–M6 are now on the roadmap

⛔⛆ **"SHIPPED" AND "FINISHED" HAD DRIFTED APART.** A five-probe audit over M1–M6, the v1.0 criteria
and every ⚠ marker in `src/` found 38 items done enough to ship and never converted into work —
most recorded nowhere, several as single sentences inside released CHANGELOG sections, which are
records rather than backlogs. Three were correctness bugs and are fixed above; the rest are now a
named section in `docs/development/roadmap.md`, grouped by whether they are wrong today, missing an
interaction story, an absent affordance, or a fact that was never made an item.
⇒ **The rule that section enforces: a limitation noticed while shipping is an ITEM, not a comment.**

### Changed — toolchain pin `6.5.41` -> **6.6.0**

`sys_mountlist` arrives with it. ⛔ **The `#50` hazard bit again**: `cyrius lib sync` walks only the
declared `[deps].stdlib` set, and left **seven** transitive leaves stale — `sankoch`, `thread` and
its four backends, `sync_macos`. Copied by hand, then the whole vendored tree verified byte-identical
to the 6.6.0 snapshot (0 drifting). ⚠ No leaf was added this time, so `#51` did not fire. The
drift warning is gone; the pin is honest again.

## [0.8.0] — 2026-09-02 — M6: the sidebar, the menu bar, the switcher, and Bueller

> ⛔ **THIS SECTION EXISTS SO THE `[0.7.7]` SECTION BELOW IS NEVER TOUCHED.** 0.7.7 is tagged at
> `6c9dd18` and on the remote, so it is a **record** now, not a scratchpad — including where it is
> wrong. Everything here landed AFTER that tag.
>
> ⭐ **M6 SHIPS EVERYTHING THAT IS BUILDABLE.** Three items remain and all three are genuinely
> gated, not deferred: **sidebar VOLUMES** (agnos cannot enumerate mounts — filed 2026-09-02, and
> the ask turned out to be a getter over a table the kernel already keeps), **the 🦀 chrome button**
> (CP437 has no crab glyph; it needs an icon path or proportional text — the M5 gate), and **the
> held-key repeat number** (agnos-runtime behaviour no host test can see). ⚠ *A milestone closing
> with gated items is the normal shape here — M2 shipped 5 of 7, M3 shipped 4 of 7.*
>
> ⚠ **This is a MINOR, not a patch**, and unusually for this project that matches semver: M6 is new
> user-facing surface. The last four feature cuts rode patch numbers by operator ruling; this one
> was directed as 0.8.0.
> ⛔⛔ **AND IT WAS ALMOST WRITTEN INTO IT.** The dep-bump note below was drafted straight into the
> released `[0.7.7]` body before anyone checked `git describe`, which answered `0.7.7-1-g8ebe9d8` —
> one commit past the tag. **That is exactly how 0.7.2 came to exist**: three commits landed past
> 0.7.1's pushed tag while its section was still being edited, and the notes a consumer reads
> stopped matching the artifact they describe. ⇒ **Check `git describe --tags` before writing into
> the top section, every time.**
> ⚠ `VERSION` still reads **0.7.7** and nothing is committed or tagged — the operator drives all of
> that, including which number this becomes.

### Added — the A/B view switcher (M6), and dhancha 0.9.26's horizontal LIST gets its first consumer

⭐ At the shipped **380x220** default crab is **always solo** — `crab_two_panes_fit` needs 600 px —
so the only signal that a second pane exists at all was the `A `/`B ` text prefix on the header. It
is now a real strip: two cells, the current one highlighted **by dhancha**, with the pane's path
beside it in the same 28 px band.

⛔⛔ **THE STRIP IS A TOOLKIT KIND BECAUSE OF ADR 0001, NOT FOR CONVENIENCE.** A row of items with
the current one highlighted, composed from a `BOX_H` of labels, makes the APP paint that highlight —
which means crab naming `accent`, which ADR 0001 forbids. `dh_list_new_h` + `dh_list_select` put it
back in the toolkit where the theme lives. That is the entire reason dhancha 0.9.26 added the kind,
and **this is its first consumer anywhere** — it shipped consumed by nobody.

⭐ **`active_pane` ALREADY IS THE SELECTED INDEX** (0 = left = A, 1 = right = B), so the widget
carries no model state of its own and cannot drift out of step with the pane it labels: there is
only one number. No new key binding either — Left/Right and `h`/`l` already move it.

⛔ **SOLO ONLY.** Two panes side by side distinguish themselves by POSITION; a strip in each of two
headers would be redundant and would beg which one is authoritative. The switcher exists for the
case with no other signal.

⚠ **IT IS A DISPLAY, NOT A CONTROL, AND THAT IS DELIBERATE.** `crab_hit` walks parents against
exactly `crab_llst` / `crab_rlst` and returns a 0/1 pane index **that reaches the write layer** —
`crab_drag_targets` rejects only negatives and same-pane, so a drop resolved against a widened index
would execute a real `crab_fs_move`. Making the strip clickable means giving it its own hit
function; that is a separate change.

⚠ **The fit rule is DERIVED, not picked** — the same discipline behind `crab_two_panes_fit`'s 600 and
the preview's 303. `crab_switcher_fit` needs the strip's 40 px plus `CRAB_COL_NAME_MIN` (90 px, ten
characters — crab's existing number for "enough to tell things apart"), so below **130 px** the strip
is refused and the header falls back to the text prefix, which costs 3 characters instead of 40 px.
⚠ It stays **inside the header's existing 28 px band** rather than becoming a new root child: at
220 px every row is contended, and a strip above the panes would take one from the listing to say
what the header can already say.

### Testing

⛔ **A NEW RENDER-PATH BRANCH WITHOUT A ZERO-ALLOCATION ARM IS A NEW BLIND SPOT, NOT A COVERED
FEATURE** — the rule 0.7.6 learned when `crab_overlay` leaked 32 B per frame for three cuts because
the gate's fixture never opened an overlay. The strip gets its own arm: twenty frames with it on
screen, plus a non-vacuity assertion that the branch was really entered.
⚠ **Mutation-proven, both ways**: selecting a hardcoded cell instead of `which` fails 1 assertion,
and a single `alloc(32)` inside the strip fails the arm at **640 bytes** — 32 × 20 frames, the exact
shape of the leak that shipped in 0.7.5.
Also covered: the fit rule at its floor and one pixel under, both panes selecting their own cell,
**no** strip in two-pane mode, and the text-prefix fallback still working when the window is narrow.

### Added — the menu bar (M6, canvas turn 2), on `F10`

Four menus — **File · Edit · Go · View** — in a row at the top of the window, the current one
highlighted by dhancha. **Collapsed by default**, which is what makes it affordable: at the shipped
380x220 every row is contended, and a bar that were always present would cost a listing row forever
to show four words. Collapsed it costs **zero**.

⛔⛆ **THE CANVAS ASKS FOR A 🦀 BUTTON AND crab CANNOT DRAW ONE.** Turn 2 reads *"Clicking 🦀 toggles
the File / Edit / Go menu row… the crab earns its place in the chrome by being the door to it."*
crab draws with `font = 0` — kashi's **CP437 8x16 bitmap** — and `dh_draw_text` walks the string one
BYTE per glyph. CP437 has 256 glyphs and no emoji; there is no crab, and no UTF-8 path to reach one.
This is the limit already recorded at `crab_name_trunc`: *"'~' (126), not '…' — the kashi system font
is CP437 8x16 and has no ellipsis glyph."* ⇒ **`F10` is the door and the crab button is DEFERRED
with its reason stated**, rather than substituted with an ASCII stand-in that would read as a
placeholder nobody removed. It needs an icon path or proportional text — the M5 gate.

⛔ **TAGS AND INDEX ARE NOT HERE, DELIBERATELY.** The canvas draws six menus; crab ships four. Both
belong to M7, are gated on daimon (which `cyrius.cyml` declares nowhere), and have **no implemented
items at all**. A menu that opens on nothing, or one greyed out forever, is a promise crab is not
keeping; an absent menu is honest and arrives with its contents. ⚠ `Go` and `View` ARE on the bar —
the canvas draws them and crab will fill them — and pressing Enter on an empty one says so rather
than opening a blank popup.

⭐⭐ **THE DROP-DOWNS ARE A SELECTION OVER `CRAB_MI_*`, NOT A SECOND SET OF VERBS.** The context menu
already names every verb crab has, with its real accelerator and its own enabled rule. A bar that
defined its own would be a second place for `Delete` to be described, and the two would drift. So
`crab_menu_label`, `crab_menu_key` and `crab_menu_enabled` answer for both surfaces unchanged, and —
exactly as the context menu does — Enter **rewrites the key the entry names and falls through**, so
there is one implementation of each command and the accelerator column cannot lie.

⭐ **A drop-down is the same overlay, opened somewhere else** — under its own bar cell, at the bar's
own height. Not a second popup mechanism: the overlay layer, the placement clamp, the arena
discipline and the zero-allocation arm all still apply. ⚠ *This only works because the overlay-root
bug above is fixed; before it, every drop-down would have been off-screen too.*

⚠ **`dh_list_new_h(0)` — dhancha 0.9.28's per-item-width mode**, and the bar is its first consumer.
`File` and `Go` are not the same width; a uniform cell wide enough for the longest label would waste
a third of a 380 px window. ⚠ crab **measures its own text** because dhancha cannot (`dh_measure`
sizes a childless widget from its PREF, never its label) — exact at `font = 0` where every glyph is
`CRAB_COL_CHARW`, and one more thing that stops being exact the day proportional text lands.

⚠ **Movement is CLAMPED, not wrapped** — the opposite of the context menu's rule and for the reason
that rule gives: a menu is a short list you see all of, so wrapping is faster than stopping; a bar is
a row you travel *along*, and wrapping from `View` back to `File` reads as the selection jumping.

### Testing

37 assertions. Mutation-proven: uniform cells fail 3, a bar that does not cost the panes a row fails
2, and a drop-down opened at the pointer instead of under its cell fails 2.
⛔ **One guard is unfalsifiable today and is labelled as such rather than deleted.** `crab_menu_row`
maps a model index past the context menu's separator; a bar menu has none, so the mapping must not
be applied — but it only shifts indices above `CRAB_MI_RENAME` (3) and the largest bar menu holds
three items, so applying it anyway returns the same number and **mutating the guard out fails
nothing**. It becomes load-bearing when a bar menu reaches a fourth item, which `Go` and `View` will
when M7 fills them. Kept and labelled, the way `crab_thumb_draw`'s clip test is.

### Changed — `crab_sidebar_hit` is the toolkit's walk, not crab's

It hand-rolled the whole thing: hit-test the root, climb to the sidebar remembering the child, scan
the child chain for its index, special-case index 0 as the inert header. **`dh_list_index_at` does
all of it**, and better — it checks the point is inside the LIST *and* inside the ROW (a row
scrolled half out of the viewport is still at its real coordinates, so testing the row alone leaves
its hidden half clickable), and it refuses an inert row outright rather than returning the
neighbour. ⇒ crab's PLACES header is handled for free, without the hand-counted index-0 case that is
the same off-by-one which made the context menu highlight the wrong verb. **26 lines out, 15 in**,
behaviour identical — every sidebar assertion still passes unchanged.

### Fixed — ⛔⛔ THE CONTEXT MENU AND THE RENAME SHEET HAVE NEVER BEEN VISIBLE

Both shipped in **0.7.5**. Neither has ever been drawn on screen. Measured at 760x260 with the menu
open at a requested `(40, 60)`:

| root child | y | h |
|---|---|---|
| panes | 0 | 236 |
| status line | 236 | 22 |
| **overlay layer** | **258** | 260 |

Two pixels of a 260 px window. The menu placed inside it landed at **y=318, bottom 475** — entirely
below the window. The sheet with it.

⛔ **THE CAUSE IS ONE WORD IN dhancha's CONTRACT THAT crab DID NOT HONOUR.** `overlay.cyr`'s header
says it outright: an offset is measured from the layer's padded content origin, so *"in a NONE root
at (0,0) with pad 0 the layer's content origin IS (0,0) and an offset is a window coordinate. **Any
other root, and the caller subtracts.**"* crab's root was a `BOX_V` and crab passed window
coordinates without subtracting — so the layer was **stacked after the status line** instead of
layered over everything.
⇒ Fixed by giving dhancha the root its contract describes: a `NONE` root holding a full-window
`content` box (which carries the panes, tray and status flow) and the overlay layer, both at (0,0),
layer last so sibling order is still z-order. ⚠ *Not* by subtracting — that would mean re-deriving
the layer's flow position from its siblings' heights, which is the same duplicated-rule shape that
left the tray's progress bar 0 px tall.

⛔⛔ **WHY NOTHING CAUGHT IT, AND THIS IS THE PART WORTH KEEPING.** Every assertion crab had about
the menu was a **state** check — `crab_menu_list() != 0`, `dh_list_selected(...) == n`,
`dh_list_count(...) == 7`. **All of them pass for a widget positioned entirely off-screen**, and all
of them did. `render_test`'s 53 pixel checks never open an overlay. The 0.7.7 menu-highlight fix was
written against this same blind spot: it corrected *which row* was selected in a menu nobody could
see. ⇒ **A state check cannot see a geometry bug.** `t_overlay_geometry` now asserts the root's
shape and that both overlays are fully inside the window; the mutation restoring the `BOX_V` root
fails **8** assertions, including `got 318, expected 60`.

### Added — 🦀 Bueller has a voice (M6)

`docs/development/mascot.md` carried an explicit *"Easter egg — implementation TODO"* since before
0.5.0. After a full minute of nothing, the status bar deadpans the absentee roll-call:
`Bueller...` · pause · `Bueller...` · pause · `Bueller...?` — and then **goes quiet for good** until
the operator touches something.

⛔⛔ **THE DISCIPLINE IS THE FEATURE, NOT A CONSTRAINT ON IT.** The mascot doc says *"subtle and
infrequent… the whole thing dies if it's trying too hard"*, and that is a rule about the code. He
speaks four times in the eight seconds after a minute of silence and never repeats on his own. The
pauses are part of it: a line that simply sat there would be a status, not a joke.

⛔⛔ **HE NEVER DISPLACES A REAL MESSAGE.** The status line is a **single slot with many writers**,
and the notice channel carries `delete this FOLDER and everything in it? y = yes`. A mascot that
overwrote a destructive confirmation would be the worst bug in the app, so `notice_present` is the
first thing `crab_mascot_stage` checks and the event loop only ever clears a line it wrote itself —
compared by pointer identity, which is why the text is a **literal**: `crab_set_notice` stores the
pointer without copying, so an arena-allocated line would be freed under the status bar by the next
`dh_frame_begin`.

⛔ **IT NEEDED ITS OWN IDLE BRANCH, because there is no unconditional per-tick redraw to ride.**
Every other idle-path `crab_render` fires only when it did work — the tray when a step moved bytes,
the gallery when a picture landed — deliberately, so the idle loop does not become a busy loop
drawing the same frame. The mascot renders **on a stage transition only**: at most four frames, then
nothing. ⚠ *Any* event resets the clock, not just a keypress — pointer motion, a resize and a scroll
are all the operator being present.

⚠ **`crab_mascot_stage` is a pure function of elapsed time**, so the whole sequence is interrogable
without a clock and the event loop's only job is to notice when the answer changes. Twenty-one
assertions cover every beat. Mutation-proven: letting him speak over a notice fails 3, letting the
closer stick fails 4, and removing the pauses fails 3.

### Added — the PLACES sidebar (M6), on `b`

⛔⛔ **NO dhancha GATE, AND THERE NEVER WAS ONE.** The roadmap carried *"Gate: dhancha TREE widget"*
for this; it was the **fifth false gate**. `LIST` gives scroll, selection and a toolkit-painted
highlight, `DH_FLAG_INERT` gives a section header the keyboard steps over, and padding gives indent.
Nothing was missing — the sidebar was buildable the whole time.

⭐ **A PLACE IS ONLY LISTED IF IT EXISTS.** Home, then the well-known subdirectories in the order a
desktop lists them, then Root last — every candidate stat'd, non-directories dropped. A sidebar
listing `Downloads` on a machine that never had one is a row that does nothing, and the operator
cannot tell it apart from a row that is broken.
⚠ Built **once at startup**, not per frame: it stats up to six paths, and doing that on the render
path would put six syscalls on every keystroke — the exact cost the deferred-statting work exists to
keep off it. The trade is that a `Downloads` created while crab runs is not noticed; that belongs on
a refresh key, not on the frame.

⛔⛔ **THE SIDEBAR HAS ITS OWN HIT FUNCTION, AND THAT IS THE MOST LOAD-BEARING DECISION HERE.**
`crab_hit` returns a **0/1 pane index that reaches the write layer** — `crab_drag_targets` rejects
only negatives and same-pane, so putting a third surface behind that number is one drop away from
moving a real file. `crab_sidebar_hit` returns a **place** index and shares no vocabulary with it,
and the click path asks it **first**, so a sidebar coordinate never reaches `crab_hit` at all. Both
directions are asserted: a click on a place reports **no pane**, and a click in a pane is **not** a
place.

⛔ **THE MODEL IS BUILT IN `app.cyr` AND PASSED DOWN, AND THE SPLIT WAS VERY NEARLY A SIXTH
LAYERING VIOLATION.** `src/ui.cyr` is included BY `app.cyr`, and `render_test` includes `ui.cyr`
alone — so the render path may not call `getenv` or `stat`. The first cut put the record accessors
upstairs with the builder; it compiled through `main.cyr` and would have left `render_test` with
undefined symbols the moment the sidebar became reachable. ⇒ **The record layout and its accessors
are shared vocabulary and live at the bottom; only `crab_places_build`, which needs the filesystem,
lives in `app.cyr`.**

⚠ **The width rule is the preview's, with a different constant.** The sidebar is subtracted from the
window *before* the two-pane decision, so `crab_two_panes_fit` and `crab_cols_for_width` are asked
about the width the panes actually get and **no rule needs an "unless the sidebar is open" clause**.
It composes with the preview for free — with both open the panes get what is left of both — and
opening it can legitimately collapse two panes into one, which is the ratified small-window rule
answering a narrower pane area rather than a second behaviour.
⚠ `CRAB_SB_W` is `CRAB_COL_NAME_MIN` — ten characters, crab's own floor for a legible label, and
every place name it can produce is shorter than that. ⚠ Off by default, and `b` refuses out loud on
a window too narrow, storing the WANT so a later resize shows it without a second keypress.

### Testing

Twenty-four assertions, including a **zero-allocation arm** — a new render-path branch without one
is a new blind spot, not a covered feature. Mutation-proven three ways: a non-inert section header
fails 2, a `crab_sidebar_hit` that forgets the header offset fails 4, and a places model that stops
stat-checking fails 2. Also asserted: the widget is cleared when a frame builds none, and
hit-testing then refuses rather than walking a dead tree.

### Fixed — clicking a pane's HEADER focuses it (M6; the last M1–M4 residue)

`crab_hit` walked a click up the tree looking for `crab_llst` / `crab_rlst`, so a click on a pane's
header bar met neither, resolved to no pane, and returned 0 — **clicking pane B's header did nothing
at all**. The click path had always intended otherwise; its own comment reads *"clicking a pane
focuses it, even off-row"*. The header simply was not reachable. Headers are now recorded per frame
and matched in that walk.

⛔ **THE ROW STAYING `-1` IS WHAT MAKES IT SAFE**, not merely possible. The click path does
`active_pane = hp` unconditionally and only touches the selection `if (hr >= 0)`, so a header click
focuses and moves nothing. A drop resolves to the pane, which is already the documented drop target
(*"the pane is the unambiguous target"*) and a header is part of that pane. Scroll guards the index.
All three consumers of that index — and it **reaches the write layer** — were checked before widening.

⛔⛔ **THE TEST THAT SHOULD HAVE CAUGHT THIS WAS DEFENDING IT, AND A SECOND ERROR HID THE FIRST.**
The suite asserted `crab_hit(lx, r0y - 26) == 0` under the heading *"a click on the NAME/SIZE header
is not in a pane"*. Measured: `r0y` is **46**, so `r0y - 26` is **25** — inside the **pane header
bar** (0–28), not the column header (28–46). So it pinned the residue as if it were intent, *and*
the column header it named was never covered by anything. ⇒ **A test whose coordinate and whose
description disagree defends the bug and leaves the real case untested.** Both are now asserted, at
coordinates verified against the laid-out tree.

⛔ **AND A SECOND GAP THE FIRST MUTATION MISSED.** Removing the header arms failed 4 assertions, but
removing the *per-frame clear* passed everything — so the slots got an accessor and their own test.
In solo mode only one pane is built, and a header pointer left from a previous two-pane frame names
arena memory `dh_frame_begin` has rewound; `crab_hit` compares by identity and the arena reuses
addresses, so it can match an unrelated widget and report a pane that is not on screen. With the
clear removed the assertion now reports a live stale pointer (`140580738836688`) instead of 0.
`crab_pane_header` mirrors `crab_pane_widget`, whose own docs already say the zero is what
`crab_hit` relies on.

### Changed — the declared graph moves to rekha 0.3.6 and dhancha 0.9.27

⛔⛔ **THE UPSTREAM HALF OF THE PROPORTIONAL-TEXT GATE IS CLOSED, AND THE GATE WAS MIS-AIMED.**
`roadmap.md` named it *"rekha + dhancha font plumbing"*. Both existed. What did not was **advance
widths**: rekha declared `REKHA_TAG_HHEA` and `REKHA_TAG_HMTX` in its first SFNT commit and never
referenced either again, so dhancha hard-coded `advf = (h * 6) / 10` — *"fixed advance ~0.6 em"* —
and rendered every proportional face at monospace pitch. Correct glyph shapes at wrong positions,
which reads as a rasterizer bug. ⇒ Fixed upstream: **rekha 0.3.6** adds `rekha_advance_width` /
`rekha_char_advance_px`, **dhancha 0.9.27** consumes them in `dh_text_advance`.

⚠ **crab consumes neither directly** — it still passes `font = 0` and draws kashi's CP437 bitmap.
It declares both because dhancha's fold calls into rekha.
⛔ **FLOOR >= rekha 0.3.6, HARD, and measured rather than predicted**: with the tag at 0.3.5, `path`
wins for dhancha, so crab compiles the local 0.9.27 against 0.3.5 and fails to link with
**`undefined function 'rekha_char_advance_px'`**. Same shape as 2026-08-27's `POINTER_SCROLL`.

⚠ **Size, measured**: host **byte-identical** at 1,023,968 B — the scalable path is unreachable from
`font = 0` and dead-code elimination removes it entirely. `--agnos` 1,047,832 → **1,051,928**
(+4,096). crab pays on the shipping target for a feature it does not yet use, which is what being
downstream of a toolkit fold costs.

⛔ **crab's OWN 9 px assumption is untouched and still open.** `CRAB_COL_CHARW = 9`, and the five
constants deriving from it, are crab-side work that only matters the day crab stops passing
`font = 0`.

⛔ **Release order is rekha → dhancha → crab.** ✅ rekha **0.3.6 is pushed** (`cf41b41`, verified on
the remote), so crab resolves and is green: `deps --verify` 49/0, both targets, **1230 / 0**,
`render_test` **53 / 0**. ⏳ **dhancha 0.9.27 is not** — the remote is still at 0.9.26 — so a clean
checkout cannot resolve crab's declared graph yet; `path = "../dhancha"` is what makes the local
build work. **A green local build is not evidence the declared graph resolves.**
⚠ The chain was verified end to end with a temporary `path` override on rekha before it was pushed,
and the override was then **removed** — rekha is one of only two deps crab resolves by tag alone, and
that property is what catches a phantom tag.
⚠ **A populated local dep cache masks an unpushed tag**: `~/.cyrius/deps/<dep>/<tag>/` persists, so
after a temporary override has resolved a version, a later `cyrius deps` finds it there and reports
success while a fresh runner fails. Only `git ls-remote --tags` answers whether a tag is published.

## [0.7.7] — 2026-09-02

> ⛔ **THIS SECTION EXISTS SO THE 0.7.6 SECTION BELOW IS NEVER TOUCHED.** 0.7.2 exists because
> 0.7.1's section was still being edited after its tag was pushed; `0.7.6` is tagged at `26f38ed`
> and on the remote, so it is now a record, not a scratchpad — **including where it is wrong.** Two
> corrections below apply to text inside released sections and are therefore made *here*.
> ⚠ **Nothing is committed, tagged or pushed** — the operator drives all git operations. `VERSION`
> and this heading were set on the operator's explicit direction to cut 0.7.7.

> ⭐ **A REPAIR CUT. No roadmap item advanced.** Five defects that had ALREADY SHIPPED were found by
> reading code, each closed with a mutation-proven test; the toolchain pin moved **6.5.36 → 6.5.41**;
> and CI went from a single `cyrius test` to nine gates, closing the three long-open CI gaps.

### Fixed — five shipped defects, and the shape they share

⛔⛔ **EVERY ONE WAS A DECISION STATED CORRECTLY IN A COMMENT AND IMPLEMENTED WRONGLY NEARBY, OR A
RULE WRITTEN TWICE WHERE ONLY ONE COPY WAS UPDATED.** None was a typo and none was subtle at the
point of failure — they survived because nothing could reach them. ⇒ **A duplicated rule does not
drift symmetrically; it drifts on whichever side someone remembered.**

**1. `crab_copy_begin` had no directory guard, and the damage was on disk.** Handed a folder, it ran
`open`+`open` and **created a 0-byte file at the destination**, then failed the first step and
refused every retry with `EEXIST` — because the stray it left was now what the overwrite guard
found. ⚠ On agnos it is worse than empty: reading a directory yields raw dirent bytes, so the stray
is **non-empty and corrupt**. Reproduced on disk before the fix. New `crab_fs_isdir` (a stat-based
question, because the transfer queue holds NAMES and has no record to consult) and a new
`CRAB_FS_EISDIR`. ⛔ The guard is in `crab_copy_begin`, not at its four call sites — two of them
arrive from the queue with no `CRAB_REC_TYPE` in hand, which is exactly how a folder reached `open`.
⚠ The marked-DELETE path already re-derived the type per entry; copy and move were the outlier.

**2. `crab_queue_advance` abandoned the queue on the first refusal, while two comments promised it
would not.** It started ONE entry and returned the refusal, so the idle tick saw `more != 1`,
reported, and stopped — and with nothing in flight that arm is never entered again. **Marking ten
files with the first already present at the destination transferred none of the other nine.** Both
`src/app.cyr` and `src/main.cyr` carried the words *"A REFUSAL DOES NOT STOP THE QUEUE"* above the
code that did. It now loops, skipping refusals and returning the last one only when nothing could
start. ⛔ `EBUSY` is not skipped past — it means a transfer is already running, so consuming the rest
of the queue against it would discard every remaining entry.

**3. `m` on a folder performed a COPY, and said so in the wrong direction.** The directory arm
dispatched `crab_walk_begin(CRAB_OP_CTREE, …)` **without consulting the key**, then set the notice
`copying folder...` — so the operator was told the truth about what happened and the wrong thing
about what they asked for. There is no `CRAB_OP_MTREE`: the walk kinds are IDLE/COPY/MOVE/CTREE/DTREE
and no source-removing tree walk exists. Now refused out loud, in the shape the folder-drag refusal
already used.

**4. A cursor resting on a folder silently discarded every mark.** The marked set was read **only in
the not-a-folder arm** — three lines below a ⭐ comment reading *MARKS OUTRANK THE CURSOR*. Marking
five files and leaving the cursor on a sixth entry that was a folder acted on the folder and dropped
the five. The marks are now read **before** the cursor's type, which is what makes that comment true.

**5. The context menu highlighted the wrong verb, or none.** `crab_overlay` inserts `dh_menu_sep`
after RENAME and then passed the **model** index to `dh_list_select`, while the separator makes the
LIST one row longer from that point on. Measured: `CRAB_MI_DELETE` (4) selected list row 4 — the
inert separator — which `dh_list_select` refuses, so **the open menu painted no highlight at all**;
`CRAB_MI_NEWDIR` (5) selected row 5, which is **Delete** — the destructive verb highlighted while
Enter would create a folder. New `crab_menu_row` maps model index to list row. ⚠ One-way on purpose:
crab navigates the menu in model space and never calls `dh_list_move_sel`.

**6. The transfer tray's progress bar was 0 px tall whenever a rate was known.** `crab_render` grew
the reserved band by `CRAB_TRAY_DETAIL_H` when a rate existed; `crab_tray` set its own `pref_h` to a
bare `CRAB_TRAY_H`. So the tray claimed 31 px of a 45 px band, and inside those 31: 31 − 8 padding
− 18 title − 14 detail = **−9**, clamped to 0 — and the bar is the only child with `flex 1`.
⇒ **The bar was missing exactly when there was something to report**, and present only in the first
half-second before a rate existed. One `crab_tray_h`, asked twice.

⛔⛔ **AND THE REASON NONE OF THEM WAS CAUGHT.** **1,221 of `src/main.cyr`'s 1,494 lines sit inside a
single `#ifdef CYRIUS_TARGET_AGNOS` with no `#else`**, and nothing includes `main.cyr` — so the whole
key-dispatch table is uncompiled on the host and unreachable by every test crab has. `render_test`
stayed **53 / 0 green** through the menu mutation as well. ⇒ Four of the six decisions were lifted
out into pure functions the suite can interrogate — `crab_transfer_plan`, `crab_menu_row`,
`crab_tray_h`, `crab_fs_isdir` — the same move `crab_drag_targets` and `crab_two_panes_fit` already
made. **The event loop keeps the wiring; the RULE goes where it can be asked a question.**

⚠ **Every fix is mutation-proven** — the guard removed and the suite watched to FAIL (9, 4, 5, 3 and
5 failures) — because this project has shipped three tests that could not fail in their first draft.
⛔ **The mutation run also exposed a defect in one of the new tests**: its pre-clean deleted the
stray `adir` only as a directory, so the 0-byte FILE the bug creates survived, and the next run
failed in five places unrelated to the bug. **A cleanup that assumes the code under test worked
cannot clean up after it failing.** Fixed, and the mutation now yields a stable 9 across consecutive
runs.

### Changed — the toolchain pin moves 6.5.36 → 6.5.41, and it retires two gates

⭐ **The pin was a false declaration before this.** `cyrius.cyml` said 6.5.36 while the installed
`cycc` was already 6.5.41, so every build printed `warning: cyrius.cyml pins 6.5.36 but cycc is
6.5.41 — toolchain drift`. It no longer warns. 6.5.41 is tagged on the remote (verified), which is
what CI installs from.

⭐⭐ **AND THE BUMP CLOSED TWO ROADMAP GATES WITHOUT A LINE OF crab WORK:**
- **cyrius 6.5.37 shipped `sys_statfs`** — the peer the Sidebar VOLUMES *capacity* gate was waiting
  on. crab vendors it now (`SYS_STATFS = 103`), and cyrius's issue is archived. Both `roadmap.md`
  and its own corrections list still read *"filed 🟡 OPEN against cyrius"*. ⚠ agnos-only (no host
  arm), and **no `STATFS_*` offsets are vendored** — the frozen 32-byte layout must come from
  agnos's docs. *Enumeration* stays open.
- **6.5.37 also shipped `sys_lstat`**, on both targets. `crab_fs_exists`' comment had said *"there is
  no `sys_lstat` to call"* as the stated reason crab cannot detect a symlink. ⇒ **That is now a
  decision, not a limit.**
⇒ **Both had been written as OPEN for four cyrius releases** — the same failure as the idle-poll buffer, carried
OPEN for nine while the fix sat in the declared graph. **Re-derive a gate before believing it.**

⛔ **`cyrius lib sync` walks only the DECLARED `[deps].stdlib` set, and it left three leaves behind**
— `thread_agnos`, `thread_local`, `thread_macos`, all transitive, none named by any declaration.
Copied by hand, then the WHOLE vendored tree verified byte-identical to the 6.5.41 snapshot (0 files
drifting). ⚠ `lib/atomic.cyr` — the leaf this hazard has always named — **escaped only by luck**: it
is byte-identical between the two versions.* ⚠ 6.5.41 also **added** a leaf,
`lib/hashseed.cyr` (6.5.39's hash-flooding defence, pulled by `hashmap`), which arrived untracked and
took the lock from 48 entries to 49.*

⭐ **Check four re-run in full at the new pin**, all four `path` lines disabled in a scratch copy so
`cyrius deps` really clones the tags: **7 deps / 0 errors**, lock **7 commit-pinned** (3 with the
overrides on — that jump is the tell), **1230 / 0**, and both binaries **BYTE-IDENTICAL** to the
path-resolved ones — host `5170a452…` **1,023,968 B**, `--agnos` `446b7f6a…` **1,047,832 B**.
*That equality is the evidence; the rest is merely consistent with it.*

⚠ **A byte delta of +16 host / +48 agnos is the shape `state.md` documents as "the signature of a
poisoned toolchain".** This one was legitimate — the vendored `lib/` genuinely changed. ⇒ The delta
tells you to look; it does not tell you which. **Diagnose by content, never by the version string.**

### Fixed — CI is a gate now, not a claim, #15, #36*)

⛔⛔ **THE WHOLE AUTOMATED GATE WAS ONE STEP: `cyrius test`.** `.github/workflows/ci.yml` now runs
`deps` + `deps --verify` · host build · **`--agnos` build** · `cyrius test` · **`render_test`** ·
`fuzz` · a per-file `fmt --check` loop · `coverage --min 85` · `vet` + `deny`. Every one was executed
locally, in order, before being written down. `release.yml` already gates on this workflow via
`uses:`, so a tag inherits all of it — **#15 is closed for releases too**.

- ⛔ **`CYRIUS_TARGET=agnos` IS SILENTLY IGNORED.** Measured: it builds a byte-identical HOST binary
  and exits 0, with no warning, even for a garbage value. A `--agnos` step written that way would
  look like the gate and be the host build twice. The flag is `--agnos`.
- ⚠ **`cyrius fmt --check` and `cyrius lint` take ONE FILE PER INVOCATION** — ordinary CLI design,
  not a defect. It matters only because the *obvious* CI line `cyrius fmt --check src/*.cyr` would
  then check `src/app.cyr` alone and pass green while eight files rotted. The step is a loop, one
  invocation per file, and says so. ⛔ *This was briefly filed as and withdrawn the same
  day: the ledger is for crab's problems, and a tool behaving as designed is not one.*
- ⚠ **`coverage --min` is a real gate**, verified able to fail (`--min 95` exits 1). The floor is 85
  against a measured **87 %** (199/227), so it gates with headroom.
- ⚠ **Three tools are deliberately left OUT**, recorded in `ci.yml` so the omissions are decisions:
  `bench` (a timing on a shared runner is noise, and a step that cannot fail is not a gate), plain
  `lint` (exits 0 whatever it finds) and `audit` (bundles lint). ⛔ **`lint --strict` DOES gate**
  (exit 2) —'s headline said no gate existed, and that was half wrong. The price is
  reformatting **83** over-long lines, most in `main.cyr`'s event loop, which is its own change.
- ⛔ **`release.yml` publishes an x86_64-linux binary as the release asset for an AGNOS application**,
  folded into `SHA256SUMS`. Recorded at the step, **not changed** — which artifacts a release
  publishes is a distribution decision.*

### Docs — the stale-comment purge (*and three new deferrals, #49–#51*)

⛔⛔ **A COMMENT THAT WAS TRUE WHEN WRITTEN AND IS FALSE NOW IS WORSE THAN NO COMMENT**, because it is
read as current fact and acted on. Audited every ⛔/⚠/⭐ marker in `src/` and every volatile claim in
the docs against measurement. Corrected, among others:

- ⛔ **`state.md`'s poisoned-snapshot remedy had INVERTED and become destructive.** It instructed
  `grep -c SYS_READDIR_AT lib/syscalls_x86_64_agnos.cyr` **must be 0** and `git checkout -- lib/` to
  undo. At the 6.5.41 pin that grep must be **non-zero** — 6.5.36 onward vendors `sys_readdir_at` and
  crab calls it in three places. **Following it would revert correct vendoring.** The durable lesson
  (a version directory can hold a stdlib that is not its version's — diagnose by content) is kept;
  the recipe is replaced by the check that actually works.
- ⛔ **`handoff.md`'s *"One deliberate interim, with an expiry"* section was entirely dead** and has
  been DELETED, not annotated. It described a hardcoded `CRAB_SYS_READDIR_AT = 101` removed on
  2026-08-30, asserted *"6.5.36 IS UNRELEASED — no tag"* (tagged, and five releases superseded), and
  claimed the wrapper *"has never been CALLED"* while `src/app.cyr` calls it three times. **The
  interim closed at its written expiry, which is the outcome it existed to force.**
- ⛔ **`state.md`'s per-file line counts are DELETED, not corrected** — `main.cyr (1,287)` against a
  real 1,494 and `app.cyr (2,484)` against a real 3,007, understated by 523 lines in the largest file
  in the project, three lines below the section's own warning never to trust them. **A number nothing
  gates does not survive being corrected; it survives being removed.**
- **`mount` is agnos syscall #11, not #23** — #23 is `timerfd_settime`, and crab's own vendored
  header says so. Wrong in three places, including the row that tells a reader which syscall to
  cite when filing the enumeration half upstream.
- `CLAUDE.md`, `README.md` and `docs/guides/getting-started.md` all told the reader `cyrius test`
  runs `[build].test`. It does not — **the file is never even compiled** — and `getting-started.md`
  went further and invited contributors to add cases there, four lines under the false command.
  ⚠ remains **OPEN**: `cyrius.cyml` still reads `test = "src/test.cyr"`.
- `CLAUDE.md` told the reader to build with `cc5`, a compiler renamed to `cycc` in cyrius 6.0.0 and
  absent from the pinned toolchain — and to sync a version into `cyrius.cyml`, which carries
  `version = "${file:VERSION}"` and has no number to sync.
- `cyrius.cyml`'s package description stated the daimon AI arc in the present tense as what crab IS,
  while no daimon dependency is declared anywhere () and `README.md` says outright that none of
  it exists. Marked **PLANNED, NOT SHIPPED**.
- `docs/architecture/README.md`'s index said *"every frame allocates ~750 KB"* — flatly contradicting
  the note it indexes, which records that bound **closed** at 0.6.0.
- `cyrius.cyml`'s four-way dhancha check was quoting **0.9.17's** figures under a `tag = "0.9.26"`
  line — evidence for the wrong thing, which is the exact failure that block exists to prevent.
  Re-taken for 0.9.26.
- `src/app.cyr`'s *"crab copies a single file per keypress and has no multi-select, so a queue would
  be scaffolding for a caller that does not exist"* — the queue it calls hypothetical sits **250
  lines above it in the same file**.
- `src/ui.cyr`'s *"The menu's LIST, recorded for hit-testing"* — the menu is never hit-tested;
  `crab_hit` walks only the two pane lists, and `crab_menu_list` has no caller in `src/` at all.
- `src/main.cyr`'s *"Opened from the KEYBOARD as well as the pointer"* — **there is no pointer
  route.** `menu_open` is set in exactly one place, under the Menu key, and the `POINTER_BTN` arm
  reads only press/release, never a button code, so a right-click runs the ordinary left-click path.

### Changed — the declared graph moves to dhancha 0.9.26, and check four gets its evidence back

⭐ `cyrius.cyml` declares **dhancha 0.9.26** (was 0.9.25). `dh_list_new_h` — the horizontal
selectable strip — is now resolvable from the *declared* graph, which **retires M6's menu-bar gate**
in fact rather than in principle. ⚠ crab consumes none of it yet.

⛔⛔ **AND IT CLOSES A DIVERGENCE THAT SHIPPED IN 0.7.6.** `path` wins over `tag`, so the released
tag committed a `lib/dhancha.cyr` **byte-identical to 0.9.26's `dist/`** while the manifest declared
**0.9.25**. Not the 0.4.13 phantom-tag failure — 0.9.25 was real and the declared graph built green
— but the committed record and the declaration disagreed *in a released artifact*, and **no gate
could see it**: four of seven deps carry `path`, and a `path` dep gets no `cyrius.lock` commit line.
⇒ Filed as; the class needs's automation, not another manual pass.

**Check four re-run in full** (all four `path` lines disabled so `cyrius deps` must clone the tags):
7 deps / 0 errors · lock **3 → 7 commit-pinned** (the tell the overrides were really off) · host
**1,019,784 B** · `--agnos` **1,047,624 B** · **1138 / 0** · `render_test` **53 / 0**.
⭐ **And the clause that had gone dead is alive again: the declared-graph binaries are BYTE-IDENTICAL
to the path-resolved ones** (`d7bd8125…` / `93f5c139…`). Before the bump they were not — host
differed by 16 B and both differed in ~650,000 bytes. *That equality is the evidence; the rest is
consistent with it.*

⚠ 0.9.26 also fixes a nineteen-release-old latent bug crab was **not** hitting, and why it was not is
worth keeping: `dh_list_new(row_h)` stored the row height in `DH_W_PREF_H` — the list's own preferred
height — so any `dh_widget_set_pref` on a LIST silently rewrote it and every scroll figure derived
from it. **The list did not break; it scrolled wrong.** crab's single list (`src/ui.cyr`
`dh_list_new(26)`) takes `dh_widget_set_flex(lst, 1)` and never a pref, so it was clear *by
construction* rather than by care.

### Fixed — the fuzz harness reports how much work it did

⛔⛔ **SIX DOCUMENTS SAID 60,000 ROUNDS. IT DRIVES 100,000, AND NOTHING IT PRINTED COULD SAY SO.**
`cyrius fuzz` emitted `fuzz: ok` and no magnitude, so when two loops were added mid-cut (the EXIF
round, then the batch-rename round) the old figure went stale in every document at once with a green
run agreeing with all of them. It now prints `fuzz: rounds 100000`.
⚠ **Counted, not computed** — `5 * FZ_ROUNDS` would be the same unverifiable claim, moved into code.
Rounds advance through `fz_round()`, so a sixth loop written in the same idiom is counted, and one
that is not looks unlike its five neighbours at the point it is written.
⇒ **This is exactly the failure `render_test` was fixed for in 0.7.6**, when it began printing its 53
checks after years of exiting 0 whether it ran 26 or none. Same shape, different harness.

### Fixed — CI's Test step no longer claims a gate that does not exist, #40*)

⛔⛔ **`cyrius test` DOES NOT RUN `[build].test`, AND THE COMMENT SAID IT DID.** Proven by mutation,
not inferred: `src/test.cyr` rewritten to `return 1` in a scratch tree left the step at **1138
passed, 0 failed, exit 0**. Only `tests/crab.tcyr` is compiled and run. That makes worse than
filed — the file is not merely redundant, **CI told the next reader it was a live gate**. The comment
now states what the step actually gates, and what it does not: no `--agnos` build (), no
`render_test` (), no fuzz/bench/lint/fmt/coverage ().

### Fixed — three live claims about `chitra 1.0.0`, one of them load-bearing

crab pins **chitra 1.0.1**. Corrected where the text was a claim about the *current* dependency:
`docs/development/roadmap.md`, `src/ui.cyr` (the supported-format set), and — the one that matters —
⛔ **`src/app.cyr`, which asserted an upstream defect the declared version no longer has.** 1.0.0
spent **26,617,512 bytes** to return a bare `CHITRA_ERR_INFLATE`; 1.0.1 refuses from the header in
under 64 KiB with `CHITRA_ERR_INFLATE_LIMIT`. crab's behaviour is unchanged — the per-image budget
already kept it clear of that cliff — but **a comment asserting a fixed upstream bug is how a budget
gets called redundant and deleted**, so the history is kept and labelled as history.
⚠ The other eight `chitra 1.0.0` mentions are historical narrative about the false gate and are
correct as written. They are deliberately untouched.

### Docs — the verification sweep, and eight new deferrals (*#40–#47*)

Every state claim in the repo was re-measured against `0.7.6` with the pinned toolchain. `roadmap.md`
gains a **Filed 2026-09-01** section carrying *#40–#47*, each **pinned to a repair release**, plus
in-place corrections to entries that had gone stale:

- ⛔⛔ ** was carried as OPEN in two places, one stamped `verified: 2026-08-31`, while
  the fix had been in the declared graph for nine releases.*** `dh_setu_poll_event` was hoisted onto
  a per-process scratch buffer in **dhancha 0.9.16** and consumed at crab **0.6.1**. ⇒ **A stale OPEN
  is not harmless**: #09 is the stated precondition for the mascot line () and any
  self-repainting element, so the entry was blocking work that was already free.
- The M5 gate line *"Gate: dhancha GRID / COLUMNS — verified absent"* was stale in **both** halves,
  and contradicted its own section four bullets later. **No dhancha gate survives on it.**
- Sidebar VOLUMES is **two** gates and has re-aimed: agnos ships `statfs`#103 (all three backends
  since 1.56.57, naming crab as the consumer) and its issue is archived, so *capacity* now waits on
  **cyrius**'s missing `sys_statfs` peer — while *enumeration* is untouched, because `mount`#23 /
  `umount`#24 remain documented no-op stubs ().
- ⚠ **Two corrections that belong to released sections and are therefore recorded HERE, not there:**
  `[0.5.0]`'s *"**39 deferrals** were harvested"* is wrong — exactly **34** numbers are cited in any
  blob of any commit reachable from `--all`; **#08, #22, #27, #30 and #31 have no anchor anywhere in
  history** and no subject can be recovered (). And `[0.7.6]`'s *"60,000 rounds"* is the same
  error the fuzz fix above corrects. ⛔ **New deferrals start at #40, never in those five gaps** —
  reusing a harvested number would make the ledger lie twice.

## [0.7.6] - 2026-08-31 — M5: preview, thumbnails, EXIF, GRID and GALLERY — and every audit finding closed

### Fixed — F1: the gallery parses what the operator can SEE, not what they opened

⛔⛔ **THE AUDIT'S HEADLINE, AND A TRUST-MODEL CHANGE RATHER THAN A BUG.** The gallery's idle-tick
fill walked **every entry in the pane**, so opening a folder ran ~22,500 lines of `chitra` +
`sankoch` over every image in it — in-process, unsandboxed, on bytes someone else chose, over an
allocator with **no guard pages**. Measured: **8 decodes and 1,033,768 permanent bytes for opening a
folder, against 1 and 165,256 for selecting an entry.** The per-image and session budgets bound
**memory**; nothing bounded the code paths a crafted file could reach.

⇒ `crab_grid_visible` reports the index range the pane is actually showing, and the fill walks only
that. **Scrolling is consent; opening a folder is not.**
⚠ **One row of overscan each way, deliberately** — a thumbnail that only starts decoding once its
cell is fully on screen arrives after the operator has already looked at it. It widens the exposure
by a row, which is a real cost and is why it is one row and not five.
⚠ Read from the laid-out widget (`dh_grid_cols`), never recomputed from the pane width: a second
answer would be free to disagree by one at exactly the widths that matter.
⚠ A pane that was not drawn reports **no** range and decodes nothing — correct, because none of it
is on screen.

### Fixed — F2: no check-then-act before a spawn. The ORDER is the fix.

⛔⛔ `crab_fs_launch` read the ELF magic, **closed the file**, then called `spawn_path` — which opens
it again. Between the two opens the file could be replaced, so the bytes crab vetted were not
necessarily the bytes the kernel ran.
⛔ **It cannot be closed by spawning the checked descriptor**: agnos has no `fexecve`, no `execveat`
and no fd-taking spawn — `spawn_path`(43) takes a path and nothing else (ABI verified 2026-08-31).
⇒ **So crab stops checking first.** The kernel's `elf_load_from_file` already rejects a non-ELF,
which means the pre-check was never a security boundary — it duplicated a decision the kernel makes
anyway, and duplicating it *earlier* is precisely what created the window. The magic read moves to
the **failure** path, where it is a diagnostic rather than a gate. **There is no window left because
there is no longer a check to act on.**
⚠ The cost is one wasted syscall when the operator hits Enter on a text file. That is a mistake's
price, not a hazard. `CRAB_FS_ENOEXEC` still reads differently from `CRAB_FS_ESYS`.

### Fixed — F4: an untyped directory is re-dispatched, not reported as a failure

The record's type byte comes from `DT_DIR` on the host and `ftype == 2` on agnos; a filesystem
answering `DT_UNKNOWN` leaves it 0, so a real directory arrived as a leaf, `unlink` refused it, and a
recursive delete reported a failure instead of descending. On a failed unlink the walk now marks the
record and rewinds the batch index by one, so the next step takes the descend branch.
⛔ **It cannot loop**: the mark is written once (the retry takes the type-1 branch, which never
returns there), and an entry that is genuinely not a directory fails its readdir one step later with
the honest error. `CRAB_WALK_STEPS_MAX` is the backstop under all of it. It costs nothing when the
backend types correctly — the retry only ever runs after a syscall that already failed.
⚠ The decision is lifted into `crab_walk_retry_as_dir` so the suite can assert it; **the trigger
cannot be produced from a host test** (the host's `getdents64` types directories correctly), so the
wiring is held by review. Stated rather than implied.

### F3 — the coverage gap closed, and a **false finding** corrected

⛔⛔ **The audit's first draft reported an unbounded read in `crab_batch_name`. It was wrong**, and
the correction is kept in the document rather than deleted. `w` and `j` increment together, so the
destination cap trips before the read index reaches `CRAB_NAME_MAX` — verified empirically with a
poison-tailed unterminated record. What is real: the safety is a coincidence of two constants that
can move independently, now **asserted** (`CRAB_EDIT_CAP <= CRAB_NAME_MAX`), and the expander was the
last parser over non-authored bytes with no fuzz coverage — now closed.

### Added — the audit itself

[`docs/audit/2026-08-31-audit.md`](docs/audit/2026-08-31-audit.md), crab's first, closing a v1.0
criterion. It records what was checked and found **sound** as well as the findings — notably that a
recursive delete **cannot descend a symlink**, safe by construction with no `lstat`.

⚠ **Every finding is now closed**: F1 and F2 by a design change, F4 by a re-dispatch, F3 by coverage
plus the correction of its own claim.

---

⚠ **M5 is substantially in**: the preview column, thumbnails, EXIF, and the GRID and GALLERY views.
Thumbnails were adopted by operator ruling on a measured +115 % binary cost (below). What remains of
M5 is proportional text (**rekha**); Columns is not gated on dhancha at all — see the gate table.

### Added — the first security audit (`docs/audit/2026-08-31-audit.md`)

A v1.0 criterion, and the one `state.md` called *"the criterion that moved furthest while nobody was
looking at it"*. Four findings, ranked by what an attacker gets.

⛔ **F1 is a change in the TRUST MODEL rather than a bug.** Gallery view decodes **every image in a
folder the operator merely opened**, so crab now runs ~22,500 lines of third-party parser (`chitra`
6,143 + `sankoch` 16,373) in-process, unsandboxed, on attacker-chosen bytes — on a bump allocator
with **no guard pages**, where an overread lands in live heap instead of faulting.
⭐ **Measured**: 8 decodes and 1,033,768 permanent bytes for *opening a folder*, against 1 decode and
165,256 bytes for *selecting an entry*. ⛔ The two budgets bound **memory**; **none bounds the code
paths reached** — a crafted 64×64 PNG passes every budget and runs the full inflate and unfilter.

⚠ F2: a TOCTOU between the ELF check and `spawn_path` — low, and the check is a UX gate rather than a
boundary. F4: a `DT_UNKNOWN` readdir type would make a recursive delete report a failure — fails safe.

⭐ **What was checked and found sound** is listed too, because an audit reporting only findings gives
no evidence of coverage — notably that **a recursive delete cannot descend a symlink**, safe by
construction (both backends set the type byte from "is a directory" alone, so a link arrives as a
leaf and the link is unlinked, never its target) with **no `lstat`**, which does not exist in the
pinned stdlib.

### Fixed — F3's coverage gap, and F3's own **false finding**

⛔⛔ **THE AUDIT'S FIRST DRAFT REPORTED AN UNBOUNDED READ IN `crab_batch_name`. IT WAS WRONG.** The
`*` splice has no explicit bound on its read index and `orig` is a 64-byte record the kernel is
trusted to terminate — the exact shape of the `crab_name_cell` defect from 0.7.1, which is why it
looked like one. But `w` and `j` increment together and `w` starts at or above `j`, so `w >= lim`
trips at 63 before `j` can reach 64: the read is already confined to the record, **by the destination
cap**.
⭐ **Caught by planting the mutation the finding implied and watching the suite stay green**, then
disproved empirically — the guard removed, an unterminated record with a poison tail pushed through,
**no poison in the output**.
⇒ The guard is **kept as defence in depth** with a comment saying it is unreachable today, and the
suite now asserts **`CRAB_EDIT_CAP <= CRAB_NAME_MAX`** — because that safety is a coincidence of two
constants that can move independently, and raising the edit cap alone would make the overread real.
⚠ **The coverage gap was real and is closed**: `crab_batch_name` was the last parser over
non-authored bytes with no fuzz coverage, and `tests/crab.fcyr` now drives it.
⛔ **The lesson is the finding.** An audit reporting a bug which is not there spends the reader's
trust and sends them to "fix" working code. Every other finding was re-derived afterwards, and F1's
headline was re-measured rather than left on reading alone.

### Re-derived — both M6 gates were wrong, in different ways

- ⛔ **The sidebar's "Gate: dhancha TREE" is FALSE — the fifth false gate.** Every piece exists:
  `LIST` (scroll, selection, toolkit-painted highlight), `DH_FLAG_INERT` (section headers the
  keyboard steps over), `PROGRESS` (capacity bars), padding (indent). Expansion is app state either
  way; the small-window drawer is `dh_place_pinned` plus the overlay layer, both shipped in 0.9.23.
- ⛔ **VOLUMES is gated on DATA, not on a widget** — re-aimed. There is **no `statfs`/`statvfs`
  anywhere**: not in cyrius's syscall tables, not in agnos's ABI, where `mount`/`umount` are
  documented **no-op stubs**. crab cannot learn a filesystem's size, its free space, or what is
  mounted. Filed against agnos. ⇒ **crab is not building a half-populated sidebar**: its canvas draws
  four sections and only PLACES is reachable, and a panel with one working section and three empty
  headers is the painted-but-inert failure this stack keeps naming.
- ⛔ **The menu bar's gate was REAL but MIS-NAMED** — what was missing was a horizontal selectable
  strip, not a `MENU BAR` kind. dhancha 0.9.26 adds `dh_list_new_h`.

### Added — the GALLERY view, and the thumbnail cache behind it

`g` now cycles **list → grid → gallery**. A gallery cell is the grid cell with a thumbnail above the
name — everything else (wrap, selection, arrows, keep-visible, hit-test) is dhancha's `GRID` and is
shared, which is what made the kind worth building.

⛔⛔ **THE VIEW NEVER TRIGGERS WORK; THE IDLE TICK DOES.** The render path cannot decode — decoding
needs syscalls and lives in `app.cyr`, above `ui.cyr` — and a decode costs ~2.5× an image's RGBA size
*permanently*. So a cell shows a picture when the tick has already produced one, and a name until
then. **Opening a gallery of a thousand files costs one frame**; the pictures arrive at one per tick
and the operator watches them land. Same shape as the deferred stat sweep and the stepped copy.
⛔ **And it stops by itself, three ways** — the walk ends at the entry count, refusals are cached so a
declined file is never retried, and the session budget refuses every decode once spent. None of
those needs a flag. ⚠ The active pane only: filling both would halve the rate at which the pane the
operator is looking at fills in.

⛔ **THE BAND IS RESERVED WHETHER OR NOT THERE IS A PICTURE** — otherwise a cell changes height when
its thumbnail lands and the whole grid reflows under the operator's eye mid-scroll.

### Added — `crab_tc_*`: a 64-slot thumbnail cache

⛔ **IT CACHES THE RESULT, WHICH IS THE ONLY THING WORTH CACHING.** The 16 KB downsampled thumbnail is
cheap; the decode that produced it is not, and its memory never comes back. Caching results means a
gallery scrolled back over costs nothing. ⛔ **It caches REFUSALS too** — "too large" and "cannot
decode" are permanent facts about a file, so re-deciding one on the next tick would be a spin that
also re-reads it.
⚠ **Fixed slots, allocated once, replaced round-robin**: 64 × (path + state + 64×64 BGRA) ≈ 1.07 MB
taken once. A growable cache on an allocator with no `free()` is not a cache, it is a leak with a
lookup function. Round-robin rather than LRU because a gallery is browsed in index order, so the
oldest slot *is* the one furthest from the eye.

⛔ **THE CACHE LIVES IN `ui.cyr`, NOT `app.cyr`, AND THAT IS THE INCLUDE-ORDER RULE AGAIN** — the
gallery looks up a thumbnail for every visible cell while building the widget tree, which is the
render path, and the render path may never call up. Storage and lookup came down (they are pure);
only the decode stayed. **Fifth time this rule has decided where something goes**, and it has cost
this project four wrong guesses.

⭐ **The preview and the gallery now share one mechanism.** The decoder used to own a single buffer —
a gallery on that would have shown every cell the last thing decoded. `crab_thumb_pixels` returns the
slot the last step was *about*, so the preview shows that entry rather than whatever the tick decoded
while the operator was looking elsewhere.

⚠ **Measured**: 8 real 128×96 PNGs decoded and displayed for **1,075,160 bytes** of permanent spend
— comfortably inside the 32 MB session ceiling, which is what makes a gallery of an ordinary photo
directory affordable at all.

### Re-derived — the "dhancha COLUMNS" gate is FALSE

⛔ **The roadmap listed a miller-columns view as gated on a dhancha COLUMNS widget. It is not.**
Columns is a `BOX_H` of `LIST`s: each already carries its own selection and scroll, and each already
has its highlight painted by the toolkit — so the app never names a colour. It clears **neither** bar
of dhancha's own rule, exactly like MENU and SHEET, which 0.9.23 refused a kind and composed instead.
⇒ **The fourth false gate this project has recorded.** What Columns is actually gated on is crab's
own two-pane model — the source/destination pairing the entire M4 write layer rests on — which is a
design question, not a dependency. Recorded as such.

### Added — the GRID view (`g`), on dhancha 0.9.25

crab's first alternate view. `g` toggles both panes between the list and a wrapping grid of names.

⛔ **THE CELL SIZE IS DERIVED, NOT CHOSEN** — the same discipline behind `crab_two_panes_fit`'s 600
and the preview's 303. A cell is exactly `CRAB_COL_NAME_MIN` wide (the NAME column's own floor, below
which a name cannot tell real files apart) and exactly one list row tall. **So a grid cell shows
precisely what a list row's NAME column shows, and the only thing the view changes is how many fit** —
which is the honest description of what it is for. A pane 374 px wide gets 3 columns; at 187 (two
panes at the shipped 380) it gets 2.

⛔ **NO COLUMN HEADER IN GRID MODE**, and it is *skipped* rather than built-and-removed: the header
names NAME/SIZE/MODIFIED and a grid shows only names, so it would describe columns that are not
there. (dhancha has no `remove_child` — the right shape for an immediate-mode tree, and it forced the
honest structure instead of a build-then-undo.)

⛔⛔ **THE GRID VIEW CHANGES WHAT AN ARROW MEANS, AND ONLY IN GRID MODE.** A 2-D arrangement the
operator can see has to answer the arrow that was pressed — but Left/Right also switch panes, and
that binding predates this view. ⇒ In grid mode the **arrows navigate** and **`h`/`l` still switch
panes**; in list mode nothing changes at all. crab has had the vim aliases since M2, so the pane
switch never becomes unreachable.
⚠ **The vertical step is the column count the pane was LAID OUT with**, read back through
`dh_grid_cols` rather than recomputed from `panew` — a second answer would be free to disagree by one
at exactly the widths that matter, and the operator would see the cursor skip a row with nothing to
explain it.
⛔ **Toggling resets both scroll offsets.** A list offset is a pixel count in rows of 26; a grid's is
in rows of 32 holding three entries each. Carrying one across lands somewhere unrelated to what was
on screen — and `scroll_to` would clamp it, so it would not even look like a bug, just a jump nobody
could explain.

⚠ **NOT a thumbnail gallery, and that is a budget decision rather than a layout one.** A gallery of
40 images at 256×256 is ~28 MB of *permanent* decode spend against a 32 MB session ceiling — see the
thumbnail entry below. The preview column still shows the selected entry's thumbnail, which costs one
decode instead of forty.

⚠ **`crab_hit` walks the pane's children directly** instead of calling `dh_list_row_at` per index.
⛔ **Not a bug fix** — that accessor is a plain nth-child walk with no kind check, so it answered
correctly for a GRID too. It is O(n) instead of O(n²) (half a million pointer hops per click at the
1024-entry cap), and it stops depending on an accessor *named for one kind* happening to work for
another.

### Changed — dhancha 0.9.25

`GRID` is a real widget kind now: wrapping layout, cell selection, arrow keys that move by a whole
row, keep-visible at minimum move, and hit-testing from the laid-out cells. ⛔ **It earns a kind by
dhancha's own rule** — the one 0.9.23 applied when it *refused* one to MENU and SHEET: composing a
grid from boxes would make the app paint its own selection highlight, which means crab naming
`accent`, which ADR 0001 forbids. The highlight is dhancha's, and `render_test` asserts it appears.
⚠ **Check 4 re-run at this bump**: every `path` override disabled, 7 deps / 0 errors, the lock going
3 → **7 commit-pinned** (the tell that the overrides were really off), both targets built, 1,035
tests green, and **both binaries byte-identical** to the path-resolved ones. Only that last equality
is evidence; the other three checks would each have passed 0.4.13.

### Added — CAMERA and SHOT: EXIF, and the bounds that make reading it safe

The other half of the roadmap's preview line (*"real metadata — `42.8 MB · 8192 × 5464`, camera,
shot"*). `p` now shows the camera and the shutter time for a JPEG that carries them.

⛔⛔ **THIS IS THE MOST ATTACKER-CONTROLLED PARSER crab HAS.** `crab_img_dims` reads fields at fixed
offsets. EXIF is a TIFF directory with a **byte order chosen by the file**, an **entry count chosen
by the file**, and values reached through **offsets chosen by the file** — three independent ways to
make a parser read where the file points instead of where the data is. And `crab_jpeg_dims` already
proved that a correct bounds check on an index says nothing about the base it is applied to.
⇒ Every read is bounded at its **absolute** address, the entry count is capped, the Exif sub-IFD is
followed **exactly once and never recursively** (a crafted file can point a directory at itself), and
values are filtered to printable ASCII before they can reach a widget.

⭐ **Verified against real files in both byte orders**, cross-checked against an independent parser:
`Canon`/`EOS R5` little-endian, `NIKON CORPORATION`/`NIKON Z 8` big-endian, and a JPEG with no EXIF
and a PNG both correctly reporting nothing.

⚠ **The timestamp is reshaped into crab's own date format.** EXIF stores `2026:07:04 18:22:31`;
`crab_mtime_str` renders MODIFIED as `2026-08-31 00:26`. Left alone, the SHOT row drew as
`2026:07:04 18:22:` — different separators from the row directly above it, and three characters too
long for a 153 px column, so the seconds sheared off mid-field. ⛔ A value that is **not** EXIF-shaped
is passed through unchanged rather than reformatted into nonsense: it came out of a file.
⚠ **`DateTimeOriginal` first, `DateTime` as a fallback** — when the shutter fired versus when the
file was written are different questions, and an edited photo has a `DateTime` long after its
`DateTimeOriginal`.
⚠ **ASCII tags only.** Exposure and aperture are rationals, and would need a formatter and three more
type cases for two more lines in a 153 px column — the "small language nobody asked for" the
batch-rename sheet refused.

### Changed — one JPEG segment walker, shared by both parsers

`crab_jpeg_seg` is now the single marker walk behind `crab_img_dims` and `crab_exif_tiff`. ⛔ Written
once because **this walk is where the bugs have been**: a cursor dereferenced as an absolute address,
and a fill-byte skip that restarted at the SOI. A second copy would have to get both right again.
⛔ **A segment running past the buffer is CLAMPED, not rejected** — crab reads at most 64 KiB of a
file, so the last segment in the buffer is routinely truncated by the *read* rather than by the file.
A first draft rejected it and two dimension tests failed immediately.

### Fixed — the EXIF fuzzing was vacuous, and only planted bugs said so

⛔⛔ **AN OUT-OF-BOUNDS READ DOES NOT CRASH HERE.** cyrius's allocator is a bump allocator over a
large mapped heap, so reading past a fixture lands in ordinary mapped memory and returns garbage
rather than faulting. A first version of the EXIF round checked only *"returns 0 or 1"* and *"has a
NUL"* — and **four planted bounds bugs all survived it**.
⇒ Everything past the fixture is now filled with a **printable poison byte** that appears in no
fixture, and no accepted value may contain it. ⚠ The poison must be *printable*: the parser filters
to printable ASCII, so a NUL or control byte would be filtered out and the overread would go unseen.
⚠ **And the mutator must not be able to write it.** A uniform 0..255 draw eventually stores the
poison byte *into* the file, and a correct parser then returns it — that false positive fired on
unmutated code at round 3723 and cost a real debugging pass.

⛔ **Two guards are still not caught by anything, and the harness says so** rather than leaving it to
be assumed: `crab_ex_ifd`'s per-entry bound (deleting it reads past the buffer, but the bytes only
reach *control flow* — a detector watching the output cannot see them), and the entry-count cap
(bounded by that same line, so it limits work rather than reach). Both are kept; neither is
load-bearing for the harness's green.

⚠ **The fixture had no inline value at all until this cut**, so a mutation deleting the inline branch
entirely survived both gates. A TIFF entry stores a value of four bytes or fewer *inside* the entry;
every string in the fixture was longer. **A fixture where every value takes the same path tests one
path** — `Make` is now `HP`.

### Added — thumbnails, and the two budgets that make them affordable

⭐ **chitra 1.0.0 adopted by operator ruling.** `p` now shows a 64×64 thumbnail for PNG, JPEG, GIF
and BMP, decoded off the idle tick and box-filtered down. Host **466,056 → 1,003,168 B (+115.2 %)**,
agnos **491,896 → 1,026,872 (+108.8 %)** — the measured price, ruled and accepted.

⛔⛔⛔ **THE BINARY SIZE WAS NEVER THE HARD CONSTRAINT. THE ALLOCATOR IS.** Measured 2026-08-31:
chitra makes **31 `alloc()` calls**, `chitra_image_free` is a **no-op** (`return 0;`), and cyrius's
`alloc` is a bump allocator whose only reclaim is `alloc_reset()` — which rewinds the *whole* heap
and would invalidate crab's pane paths, readdir buffers, surface and client struct. **Every decode is
permanent, and a second decode of the same file costs it again:**

| source | RGBA bytes | permanent alloc | ratio |
|---|---:|---:|---:|
| 64×64 | 16,384 | 83,976 | 5.1× |
| 256×256 | 262,144 | 700,280 | 2.7× |
| 512×512 | 1,048,576 | 2,670,608 | 2.5× |
| 1024×1024 | 4,194,304 | 10,549,168 | 2.5× |
| 2048×2048 | 16,777,216 | 42,057,928 | 2.5× |

⇒ **~2.5× the RGBA size, never returned.** A file manager that decoded on every arrow key would
exhaust a machine browsing one photo directory. Hence two budgets, and neither is optional.

- ⛔ **Per image, as a PRE-CHECK.** `chitra_image_decode_budget` reads the declared dimensions from
  the header and refuses before allocating: measured, a refusal costs **16 bytes**, against
  **26,617,512** for the same file through the unbudgeted entry point. That asymmetry is why crab
  never calls bare `chitra_image_decode`. `CRAB_THUMB_MAX_RGBA` is 4 MB (≈1024×1024).
- ⛔ **Per session, because the per-image budget does not compose.** A cap on one decode says nothing
  about a hundred. crab measures `alloc_used()` across each decode and stops at
  `CRAB_THUMB_TOTAL_MAX` (32 MB) — and **says so** rather than silently showing nothing.

⭐⭐ **And the per-image budget keeps crab clear of an upstream defect.** chitra 1.0.0 fails with
`CHITRA_ERR_INFLATE` on any PNG whose inflated scanline data exceeds **16 MiB** — bracketed exactly:
2200×2200 (14,522,200 B) decodes, 2370×2370 (16,853,070) does not. That is ~5.6 megapixels of RGB, so
an ordinary phone photo saved as PNG fails — **and it fails expensively**, spending 26.6 MB to return
0. A 1024×1024 RGB PNG inflates to ~3.1 MB, comfortably under the cliff, so crab cannot reach it.
⚠ Fixed in **chitra 1.0.1** (prepared, not yet pinned here — the tag must exist on the remote first).
The ceiling itself is sankoch's and only sankoch can raise it; filed there.

⛔ **ON THE IDLE TICK, NEVER THE KEYSTROKE PATH**, after the transfer step and before the stat drain:
a transfer the operator started outranks a picture they have not seen, and a picture they are looking
at outranks size columns they have not asked about. ⚠ **At most one decode per tick** — chitra's
entry point is one call that returns an image or does not, so a decode cannot be resumed the way
`crab_copy_step` and `crab_walk_step` can. That is the finest granularity available.
⚠ **Memoised on the full path, including a remembered REFUSAL** — a file too big is still too big on
the next tick, and re-deciding it would be a spin. A closed preview decodes nothing at all.

⛔ **FOUR DIFFERENT NOTHINGS, NAMED SEPARATELY**, because they are not the same to an operator:
*too large to preview* is a permanent property of the file, *preview budget spent* is a property of
the session and explains why the next one will also be blank, and *cannot decode* is a property of
this build. One blank rectangle for all three is how a working feature gets reported as broken.

⚠ **RGBA in, BGRA out, alpha forced opaque.** chitra emits R,G,B,A and a sadish surface is B,G,R,A —
getting that wrong is a red/blue-swapped thumbnail that still looks like a picture. And crab declares
`SETU_SURF_PREMULTIPLIED` with a suite that gates `a == 255` across every pixel, so a thumbnail
carrying a source alpha would break that contract for the whole window. Both are pinned.
⚠ **Box average, not nearest-neighbour** — point-sampling 1024×1024 down to 64 reads one pixel in 256
and throws the rest away. The expensive part is already behind us.

### Changed — dhancha 0.9.24

⚠ crab uses none of it yet. The keys, the drag-under-arena fix and `dh_text_attach` are the
general answer to crab's three hand-rolled workarounds (`dh_dispatch`, drag, `TEXTINPUT`); adopting
them is its own change, not this one.

### Changed — `crab_render` takes ONE record instead of 32 positional parameters

⛔ **THE FAILURE THIS PREVENTS IS A MISCOUNTED COMMA.** At 32 positional `i64` arguments — nine of
them `0`, four of them `0 - 1`, six of them bare pointers — dropping or adding one shifted every
argument after it and still compiled. There were **twenty-three such call sites** at the time of the change — thirteen in
`tests/crab.tcyr`, five in `src/main.cyr`, five in `src/render_test.cyr` — and thirty-one now that
M5's own tests are in.
⭐ `crab_render(surf, font, rs)`; the record is filled by `crab_rs_pane` (11 params, indexed by pane
so there is no left copy and right copy to drift), `crab_rs_op`, `crab_rs_chrome`, `crab_rs_preview`
and `crab_rs_dims`. Max arity 11, and the compiler's arity check catches what commas hid.

⛔ **THE DEFAULTS NOW LIVE IN ONE PLACE, AND THAT IS THE OTHER HALF OF THE POINT.** `optotal`,
`oprate` and `opeta` are **-1 = cannot be said yet**, never 0 — 0 means "measured, and it is zero",
which the tray would render as a real `0 B/s`. A zeroed record is therefore *wrong*, and
`crab_rs_reset` is the single place that knows it. Every one of those call sites used to spell
`0 - 1` by hand, three times each.
⚠ **`main.cyr` keeps its locals and refills the record before each render** rather than mutating the
record from the key handlers. The loop's variables stay the source of truth, so there is no second
copy of the app's state to keep in step — and the event loop is `#ifdef CYRIUS_TARGET_AGNOS`, where
no host test could see a copy going stale.

### Added — the preview column (M5, ungated)

`p` toggles a right-hand inspector showing the ACTIVE pane's selected entry: **NAME · KIND · SIZE ·
MODIFIED · DIMENSIONS**.

⛔ **THE WIDTH RULE IS DERIVED FROM crab's OWN COLUMN RULE, not copied from the canvas.** The rule is
*the preview may not cost a pane its SIZE column* — so the threshold falls out of
`crab_cols_for_width` and is **303 px**, re-derivable when the font or the column set moves. This is
the same discipline `crab_two_panes_fit` records, and the same reason its 600 px was derived rather
than lifted from the mockup's 420.
⚠ **OPENING THE PREVIEW CAN COLLAPSE TWO PANES INTO ONE**, and that is the ratified small-window rule
answering a narrower pane area rather than a second behaviour: the column takes its width first and
the panes then divide what is left, so no layout rule needs an "unless the preview is open" clause.
⛔ **A WINDOW TOO NARROW REFUSES OUT LOUD** — `crab_set_notice`, the same discipline as
`crab_resize_wanted`. The operator's *want* is stored separately from what fits, so a window later
made wider shows the preview without a second keypress.
⚠ **THE SIZE AND DATE COME FROM THE SAME ARRAYS AND THE SAME FORMATTERS THE PANE USES.** A preview
that formatted its own size would be a second answer to a question the row two inches away already
answers — which is exactly the disagreement 0.5.0 fixed in `crab_status_str`.
⚠ **A pending stat stays pending here too**, so opening the preview does not put a synchronous stat
back on the keystroke path that M3 *#03* restructured the listing to clear.

### Added — image dimensions from the header alone, with no decoder

`crab_img_dims` reads PNG, JPEG, GIF and BMP dimensions out of header bytes. **Verified against real
files**: 137×42, 1×1 and 4096×2160 PNGs, a 91×33 GIF, a 65×17 BMP and its top-down twin, and a real
384×288 JPEG whose frame header sits past an APP0 block — every one matching `identify`.

⛔⛔ **THIS IS DELIBERATELY NOT chitra, AND THE REASON IS A MEASURED 528 KB.** Declaring
`chitra 1.0.0` takes the host binary from **453,304 B to 981,992 B (+116.6 %)** and the agnos binary
from **474,944 to 1,001,520 (+110.9 %)**, and `CYRIUS_DCE=1` reclaims **none** of it — it NOPs the
unreachable functions and the byte count does not move. Decomposed:

| added | host bytes | delta |
|---|---:|---:|
| (baseline, 0.7.5) | 453,304 | — |
| + stdlib `thread` | 461,608 | +8,304 |
| + stdlib `flags` | 461,624 | +8,320 |
| + stdlib `thread` + `sankoch` | 860,784 | +407,480 |
| + all three + chitra's own fold | 981,992 | **+528,688** |

⇒ **~399 KB of it is `sankoch`**, the RFC 1950/1951 inflate leaf PNG's IDAT needs, pulled in
transitively; chitra's own decoder fold is only ~113 KB. crab does not choose that — PNG does.
⚠ And crab need not *declare* the extra leaves: `cyrius deps` re-creates them from chitra's
`dist/chitra.deps` sidecar, so the cost arrives whether or not the manifest names them.
⇒ **Dimensions need none of it.** PNG, GIF and BMP put width and height at fixed offsets inside the
first 26 bytes, and JPEG's sit in an SOF segment a bounded marker walk reaches. The metadata half of
the preview is free; the **pixel** half — thumbnails — is now a decision with a price tag on it, and
it is the operator's to make. See *Still open* in `docs/development/state.md`.

⛔ **THE READ IS MEMOISED AND CAPPED.** `crab_preview_dims` caches on (directory, name) — one entry,
because the operator looks at one file at a time — and reads at most **64 KiB**. Recomputing per
frame would put an open/read/close on the arrow-key path, which is what *#03* existed to remove.
A closed preview reads no files at all, and a non-image extension is never opened.
⛔ **THE CACHE IS INVALIDATED IN `crab_relist`**, which runs after every write crab makes. One call
covers rename, delete, copy, move, mkdir and the batch sheet; invalidating at each call site would
be six chances to forget with no gate that would notice. A cache keyed on a name is stale the moment
the name means a different file — and crab is a program that makes names mean different files.

### Fixed — a per-frame allocation leak in `crab_overlay`, shipped in 0.7.5

⛔ `crab_overlay` allocated its placement rect with **`alloc(32)` rather than `dh_falloc(32)`**, on
both the menu branch and the sheet branch. That is 32 B per frame on an allocator with no `free()`,
for as long as either was open — and crab re-renders after every handled event, so arrowing down a
six-item menu leaked 32 B **per keystroke, permanently**.
⛔⛔ **THE GATE COULD NOT SEE IT, AND THAT IS THE LARGER FINDING.** crab's zero-allocation assertion
rendered twenty frames with **no overlay open**, so neither branch ever ran under it. A gate that
covers one state proves one state. The suite now runs the twenty-frame loop with the menu open, with
the sheet open, and with the preview open, plus a non-vacuity arm proving those branches were really
entered. Reverting either `dh_falloc` fails at exactly 640 B = 32 × 20.

### Fixed — `tests/crab.fcyr` is a real fuzz harness (roadmap *deferral #12*)

⛔ **It read none of its input.** `fuzz_main(data, len)` returned 0 without touching a byte, so
`cyrius fuzz` reported **PASS against any input whatsoever** — a green gate that could not go red. It
stood while crab grew a readdir parser, a write layer that deletes trees, and now a header parser.
⭐ It now drives **60,000 rounds** over mutated format headers, wholly random bytes, degenerate and
negative lengths, and arbitrary byte sequences through `crab_name_ok`, `crab_is_image` and
`crab_cstr_len` — deterministically, from a fixed seed, because a fuzz failure nobody can reproduce
is a rumour. It asserts an invariant, not just the absence of a crash: when the parser answers
*yes*, the dimensions must be inside the bounds it promises.

⛔⛔ **AND ITS FIRST DRAFT WAS ITSELF VACUOUS, WHICH IS THE POINT OF THE ENTRY.** An LCG modulo a
power of two has period 2^k in its low k bits, so `v % 4` cycles with period four; combined with the
stride between draws, the format selector returned **only 1 and 2 across all 20,000 rounds**. PNG and
JPEG were never seeded, `crab_jpeg_dims` was entered **zero times**, and the harness printed
`fuzz: ok`. It was caught by planting the known JPEG bug and watching the fuzzer pass. Sampling the
high bits fixes it. ⇒ **A fuzzer must be shown to catch a bug you plant on purpose** — the 0.6.0
"three tests could not fail in their first draft" lesson in a new costume.

### Fixed — `src/render_test.cyr` reports how many checks it ran

⛔ It returned `g_fails` and printed only its dump line, so it exited **0** whether it ran 26 checks
or none — and the handoff has quoted a check count for three cuts that nothing in the program
printed. It now prints `N checks, M failed`. **35 checks, 0 failed** at this cut (26 before the
preview's nine).

### Two defects found in this work, recorded because the reasoning outlives them

1. ⛔ **`crab_jpeg_dims` dereferenced its cursor as an ABSOLUTE ADDRESS** — `load8(p)` for
   `load8(buf + p)` — and segfaulted on the first marker. **Not one bounds check would have caught
   it**: `p` was compared against `len` correctly throughout, so the arithmetic was right and only
   the base was missing. A bounds check proves an index is in range; it says nothing about which
   buffer the index is used against. This is the bug the fuzzer now catches.
2. ⛔ **The fill-byte skip signalled its exit by assigning `p = len + len` and subtracting it back**,
   which restored `p` to **0** and restarted the walk at the SOI. It compiled, and on a JPEG without
   fill bytes it never ran. The loop carries an explicit flag now.

### Also

- ⚠ **`crab_is_image`'s first draft lowercased an extension into an `alloc(8)` scratch buffer** — on
  a path `crab_preview` reaches every frame. It is allocation-free now, and the fixture that would
  have caught it (a selection sitting on an image name) is in the zero-allocation loop.
- ⛔ **The test suite's `main` has a ceiling and it is reached silently.** At 2,517 lines and **279
  locals**, adding one group pushed its frame past what the process could touch and the suite
  **segfaulted** part-way through, after every prior assertion had passed. New groups get their own
  functions.
- ⛔ **`crab_fs_open_w` behaves differently on the two targets, and this is a flag rather than a
  fix.** The host arm is `O_WRONLY|O_CREAT|O_EXCL` — the M4 overwrite guard, which refuses an
  existing file — while the agnos arm is `AO_WRONLY|AO_CREAT|AO_TRUNC` with **no `AO_EXCL`**, so on
  the target that ships, the same call truncates instead of refusing. Pinned by an assertion on the
  host side; changing write semantics is not this slot's business.
- Tests **757 → 925**, `render_test` **26 → 35** checks, reference coverage **86 % → 87 %**
  (161/183). Host **466,056 B**, `--agnos` **491,856 B**.

## [0.7.5] - 2026-08-31 — the context menu, inline rename, and the batch sheet

**M4's last three items**, unblocked by dhancha 0.9.23's MENU, overlay layer and SHEET.

### Added — the context menu

⛔ **THE MENU IS NOT A SECOND SET OF VERBS.** Every entry maps to a key binding that already exists,
and activating one **rewrites `u` to that key and falls through** — so there is exactly one
implementation of each command and the accelerator column cannot drift from what the entry does. It
is a **discovery** surface, not a parallel command path.

⛔ **Entries are GREYED, never hidden.** A menu whose entries move around depending on the selection
forces the operator to re-read it every time; one whose entries stay put teaches their positions. A
greyed entry is `DH_FLAG_INERT` (dhancha 0.9.23), so the keyboard steps over it and the mouse misses
it — "greyed" is not merely cosmetic.
⚠ **Navigation WRAPS**, unlike a file list: six items the operator can see at once, so running off
the end and continuing is faster than stopping. A 1024-row pane is where wrapping would lose your
place.
⛔ **Rename is greyed the moment more than one entry is marked** — with a set marked the operator
means the *set*, and renaming a set is the batch sheet, a different command with a different surface.
⚠ **Opened from the keyboard as well as the pointer.** A menu only a mouse can reach is invisible to
an operator who never touches one, and crab is keyboard-first by construction.
⚠ A separator sits before Delete, so the destructive verb is never the entry you land on by
overshooting Rename.

### Added — inline rename and New folder

`r` renames the selected entry, `n` creates a folder. `crab_fs_rename` and `crab_fs_mkdir` have
existed and been asserted **since 0.7.1**; this is the field that finally reaches them.

⛔⛔ **crab DOES NOT USE dhancha's `TEXTINPUT`, AND THIS IS THE THIRD FEATURE WITH THE SAME
MISMATCH.** `dh_text_new` allocates its buffer with the *global* allocator so it survives frames —
but the **widget** holding it is `dh_falloc`'d and dies at every `dh_frame_begin`. An immediate-mode
app that rebuilds its tree each frame must therefore call `dh_text_new` each frame, and each call
leaks a fresh buffer into an allocator with no `free()`. TEXTINPUT is built for a **retained** tree.
⚠ Same structural shape as `dh_dispatch` (a press held as a widget pointer, operator ruling
2026-08-27) and as drag (`_dh_drag_src`, which dhancha 0.9.21 fixed by refusing to start). **Three
features, one mismatch — worth telling dhancha.**
⇒ crab owns the buffer, the length and the caret, exactly as it owns pane index, row index and the
operation record.

⛔ **The field takes every key while it is open** and returns before any binding below sees one — a
field that let `d` through would delete the file being renamed.
⚠ **The caret starts at the END.** Renaming is almost always appending or trimming a suffix;
starting at 0 makes every rename begin with a trip across the name.
⚠ **A full field refuses rather than truncating** — silently dropping the key just typed beats
silently dropping a character typed earlier. The cap **is** `CRAB_NAME_MAX`: a name that cannot be
typed could not be stored anyway.
⛔ **crab receives HID usages, not characters**, and nothing else in the stack translates them — so
`crab_key_char` exists and is US QWERTY, matching the tables agnos's HID layer installs at boot
rather than inventing a second answer. ⚠ **Shift is not on the wire yet**: `mods` carries
press/release, so names type lower-case until the compositor forwards modifiers. The map already
takes the flag.

### Added — the batch-rename sheet

`r` on a marked set opens a pattern field. **Two substitutions and nothing else**: `#` is the
sequence number, `*` is the original name. That covers prefix, suffix and numbering — the renames
people actually do — and every addition past it is a small language nobody asked for.
⛔ **Queued by name, not by index**, for the same reason multi-select copy is: each rename leaves the
pane buffer describing the *pre*-rename listing.
⛔ **The expansion still faces `crab_name_ok`.** `*` splices in a readdir name, so a pattern really
can produce `..` or a name carrying a separator — the guard refuses it, not the expander.
⚠ **Refuses rather than truncates**: a truncated name is a *different* name, and a batch that
silently renamed forty files to forty truncated names is worse than one that stopped.

### Added — the overlay layer

⛔ **The layer is the LAST child of the root, and that is load-bearing in both passes.** `dh_hit_test`
prunes any subtree whose root does not contain the point and the painter culls identically — so a
popup parented to the row that spawned it would be invisible and unclickable **together**. A
full-window layer added last wins the click and the pixel by construction.
⭐ **It is also what makes the popup modal**: it swallows every hit the popup does not take, so the
panes beneath are unreachable for exactly as long as it is in the tree. **dhancha holds no modal
state — the tree IS the state.**
⚠ The rename sheet is **pinned**, not drawn over its row. Inline-over-the-row is what the canvas
draws, but it needs the row's laid-out rect and the row is rebuilt after the overlay runs. A pinned
sheet is unambiguous, identical in solo and dual pane, and a form the canvas itself draws at its
small size.

### Verified

**694 → 757 passing**; `render_test` 26/26; reference coverage **86 %** (137/159); both targets
build; `fmt --check` clean.
⭐ **Five mutations, each of which fails the suite**: Rename staying live on a marked set; menu
navigation clamping instead of wrapping; navigation ignoring the enabled predicate and landing on a
greyed entry; the edit field truncating instead of refusing; and the batch expander truncating.

⛔ **A brace error inside `#ifdef CYRIUS_TARGET_AGNOS` compiled clean on the host and failed only on
`--agnos`.** The whole event loop lives in that region, so **a host build proves nothing about the
key handling** — build both, every time. Caught here; recorded so it is not re-learned.

### ⚠ Known debt, stated rather than left to be discovered

**`crab_render` now takes 33 parameters.** Each one arrived for a good reason and follows the
established rule that state flows *down* rather than being reached *up* for — but 33 is past the
point where a struct would read better, and every new panel adds two or three more. It is not
changed here because doing so touches every call site and every test in the same commit as three new
features; it is the first cleanup of the next slot.

## [0.7.4] - 2026-08-31 — M4 complete: recursion, multi-select, rate and ETA

### Added — recursive copy and delete, as a stepped walk

⛔⛔ **AN EXPLICIT STACK, NOT LANGUAGE RECURSION.** A recursive function would blow the ring-3 stack
on a deep tree *and* could not be paused mid-way — which is the whole requirement, because a tree
operation must yield to the idle tick like everything else.
⚠ **A stack slot holds no path.** `sys_readdir_at` is path-based, not fd-based, so there is no
directory handle to retain: a level is a resume cursor plus the two path lengths a pop truncates back
to. **32 bytes.** The naive design — one entry buffer per level — is 64 KB *per level* against a bump
allocator that never frees.

⛔⛔ **A DELETE RE-READS FROM CURSOR 0 EVERY STEP, AND THIS IS THE MOST IMPORTANT LINE IN THE
RELEASE.** `ext2_dir_remove` coalesces a removed entry into its **predecessor** by extending that
record's `rec_len`; `readdir_at` parks its cursor on the record it did not take and resumes by
reading a header at that offset — which, after a coalesce, is **interior to the predecessor's
extended record**. The kernel then parses a record header **out of the middle of a filename**.
⛔ **The two guards do not save it**: the 4-byte alignment check passes because a valid record start
stays 4-aligned, and `ext2_dirent_valid` is satisfied by ordinary filename bytes. The failure is not
a crash and not an empty listing — it is a **fabricated entry name handed to `unlink`**, and a
fabricated name that happens to match a real sibling **deletes the wrong file**.
⇒ `crab_walk_cursor_for` is a predicate rather than an inline branch, for the same reason
`crab_readdir_stalled` is one: the rule is what a host test can assert.

⛔ **The descend gate is the record's type byte and NOTHING ELSE — never a stat.** A symlink,
including one pointing at a directory, arrives with type 0 because both backends set the byte from
`ftype == 2` alone. So the walk treats it as a leaf and `unlink`s the **link, never the target** —
correct `rm -r` behaviour, achieved with no `lstat` (which does not exist in the pinned stdlib) and
safe *by construction* rather than by a check that could be forgotten.
⚠ The 2026-08-30 burn's `/lp` — a self-referential ELOOP link — is type 0: unlinked as a leaf, never
entered. **No visited set is needed**: `ext2_link` refuses directory hard links, symlinks are never
descended, and depth is bounded absolutely at 127 by `CRAB_PATH_MAX`.

⛔ **Copying a folder into itself is refused** (`CRAB_FS_ELOOP`) before anything is created — `/a`
into `/a/b` produces `/a/b/a/b/…` until the path bound stops it. `crab_path_within` is
**boundary-aware**: `/ab` is *not* inside `/a`, and refusing that copy would be a false refusal the
operator cannot argue with.
⛔ **The destination is joined FIRST**, before the source and before any `mkdir` or `open`: a tree
legal at the source can be illegal at the destination, because the bound is on the absolute path.
That ordering is what makes a partial tree always a **prefix of a correct copy, never a corrupt one**.

### Added — `crab_walk_readdir`, and the walk is host-testable

⭐⭐ **This is why the recursion has real tests rather than a predicate and a hope.**
`crab_readdir_into` is agnos-only — its body is inside `#ifdef CYRIUS_TARGET_AGNOS` and yields 0 on
the host — so a tree walk built on it would have been another `#101`-shaped blind spot, **in the most
destructive code in the app**. Linux's `getdents64` carries `d_off`, a seek cookie with *exactly*
`#101`'s semantics, so the same state machine runs against real directories in the suite.

### Added — multi-select

Space marks and advances (the Norton Commander gesture). Marked rows tint via `DH_W_FG`, and **yield
to the selection's on-accent guarantee** — accent-on-accent is the one invisible combination, and
dhancha settled that at 0.9.20.
⛔ **Marks are indices, so 18 sites clear them** — every re-list, descend, ascend, **and the sort
cycle**. The sort is the subtle one: the listing does not change, only its *order*, so nothing else
would notice that every mark now points at a different file.
⛔ **The transfer queue holds names, not indices.** A multi-file copy re-lists after each file, so any
index captured when `c` was pressed is wrong by the second file. ⚠ A refusal does **not** stop the
queue — one file already at the destination must not abandon the other nine.
⛔ The delete prompt says whether it is the **marked set**, the cursor row, or **a FOLDER and
everything in it** — the operator answers the question they were asked.

### Added — rate and ETA

⛔ **Both refuse to answer before they can.** Under 500 ms the elapsed time is dominated by the cost
of starting, so a rate computed from it is noise dressed as a measurement — and a **wrong rate is
worse than none**, because the operator plans around it. The detail line shows each half only if it
is true; `0 B/s · 0s left` reads as a stall.
⚠ Averaged over the whole run, not the last step: a per-step rate swings with every disk hiccup and
never settles. Durations are two units, never three, zero-padded (`3m 04s`).

### Changed — the idle tick dispatches through `crab_op_step`

A new return code **`2`** means *an item finished, the walk continues*: redraw, but do **not** relist.
⚠ It is not cosmetic — the completion arm relists **both** panes, and a 500-file tree copy reporting
per-file completion would do that 500 times on the idle path.
⛔ A tick is **either** 64 chunks of one file **or** 32 directory entries, never both, so the tick
cost is bounded by the larger rather than their sum.

### Changed — dependencies: sadish **0.5.3**, rupa **0.1.6**, dhancha **0.9.23**

All three released first, then declared, then check 4 re-run — the order the 2026-08-28 phantom-tag
failure exists to enforce. ⭐ **6 deps / 0 errors with every `path` override disabled, both targets,
and byte-identical binaries.**
⚠ **No `path` override was added for sadish.** sadish and rekha are the only two deps crab resolves by
tag alone, which makes them the only two whose remote resolution a local build actually exercises —
the one thing standing between this manifest and a repeat of that failure. The chain was verified
with a *temporary* override, which was then removed.

### Verified

**520 → 653 passing**; `render_test` 26/26; reference coverage **75 % → 87 %** (122/140) after 31
functions were added; both targets build; `fmt --check` clean.

⭐ **Mutations that fail the suite**: a delete carrying its cursor (the coalesce corruption); the
destination-inside-source guard removed; `crab_path_within` losing its boundary check; a move
unlinking at the start instead of at EOF; a cancel leaving the partial destination; the source never
stat'd; the percentage dividing before multiplying.
⚠ **One equivalent mutant, stated rather than papered over**: the descend gate's `== 1` versus
`!= 0`. Both backends emit only 0 or 1 today, so no test distinguishes them. `== 1` is kept because a
future backend passing a raw ext2 `ftype` would make `!= 0` descend a **symlink**, whose ftype is 7.

⛔ **Not covered**: the idle-tick wiring itself. The walk and the step machine are driven by hand in
the suite; that they are *called* from the tick is agnos-only event-loop code no host test reaches —
the same irreducible gap `main.cyr` has always had, and what a QEMU run would confirm.
⚠ **crab cannot recreate a symlink.** A recursive copy copies whatever `open` + `read` yields through
one — the target's bytes, or a failed open for a dangling link. `sys_symlink` exists but crab cannot
*learn* that a source is a link without `readlink`'s ambiguous negative. Honest limit of the current
kernel ABI, and it moves when `lstat`#102 gets its cyrius peer.

## [0.7.3] - 2026-08-31 — the copy steps, and the transfer tray exists

### Added — M4: the stepped copy. What makes a progress bar mean anything

⛔⛔ **A PROGRESS BAR OVER A BLOCKING CALL IS A DECORATION.** `crab_fs_copy` ran its whole read/write
loop inside the `c`/`m` keypress branch, so the event loop drew **no frames** while it executed — a
bar driven from that renders once at 0 %, never repaints, and vanishes when the copy returns.
**dhancha 0.9.22 shipping `PROGRESS` did not fix that; this does.** The roadmap recorded the tray as
"genuinely gated on a dhancha PROGRESS widget", which sent the work to the wrong repo — corrected.

The copy is now a state machine that does at most `CRAB_COPY_STEP_CHUNKS` (64) chunks per call and
returns, driven by the **existing** idle tick that already drains `crab_stat_batch` a bounded batch
at a time and already re-renders when it did work. No new loop, no new timing model.

⛔ **The operation record is `alloc`, not `dh_falloc`, and that is not a style choice.** It holds two
open file descriptors across many frames; `dh_falloc` draws from the frame arena, which
`dh_frame_begin` rewinds every render — so the fds would be handed out as widget memory mid-transfer
and the copy would write into a widget tree. Everything that outlives a frame is global; everything
that does not is arena'd.

⭐ **A move still tries `rename` first** and only steps when it refuses, so the common case (both
panes on one filesystem) never reads a byte and never shows a bar.
⛔ **A move unlinks its source only at EOF.** Unlinking early would lose the file outright if the
machine died between the unlink and the last chunk.
⛔ **One transfer at a time** (`CRAB_FS_EBUSY`). Silently replacing a running one would leak two
descriptors and abandon a half-written file.

### Added — Esc cancels, and a cancel is not an I/O failure

⛔ **A stepped operation the operator cannot stop is a worse control than a blocking one** — the
blocking version at least ended by itself. Shipped in the same change as the stepping, not after it.

⛔⛔ **A CANCEL DELETES THE PARTIAL DESTINATION; AN I/O FAILURE DOES NOT.** That asymmetry is the
whole difference between the two paths. On an I/O error the filesystem is already unhappy and issuing
another write-path syscall into it is how a bad situation becomes worse — so the evidence is left in
place. On a cancel nothing is wrong: the operator changed their mind, and leaving a truncated file
wearing the real file's name is the worst outcome available, because the next reader cannot tell it
from the whole thing. ⚠ The source is never touched: a cancelled move is a no-op, not a half-move.

### Added — `crab_fs_stat_size`, so the bar has a denominator

⭐ **Nothing asked this before, which is why the tray had none.** `crab_fs_exists` stats the
*destination* for the overwrite guard and throws away the statbuf it just filled; the source was
opened without ever being measured. ⚠ **-1 is a first-class answer**, not a failure to handle: it
flows into the operation's total and the bar renders **indeterminate**, which is exactly the state
dhancha 0.9.22 added `den <= 0` for. On a host build that is the normal answer and the copy still runs.

### Added — M4: the transfer tray

A strip above the status line: a title line (name + percentage) over a `dh_progress_new(4)` bar.

⛔ **A strip, not a right-hand inspector column.** The canvas draws the tray in an inspector — but
that column is **M5** and does not exist, so lifting its *position* now would mean building half of
M5 to hold one bar. The canvas's own 420 px variant already drops the tray's detail line, so a
compact form is a drawn variant rather than a compromise.
⚠ **The tray is not a permanent fixture** — when nothing is running the band is not reserved, no
widgets are built, and the panes get the 31 px back.
⛔ **crab names no colour on the bar.** That is the whole reason `PROGRESS` is a dhancha widget
rather than a BOX crab tints itself.
⛔ **The percentage is a sibling label, not text on the bar** — dhancha's own header says why: one
centred "68 %" straddling the fill edge cannot be legible in a single ink.
⛔ **An unknown total reads `"--"`, never `"0%"`.** They are different facts, and printing 0 % beside
a bar that is deliberately *not* showing 0 % would make the two disagree on screen. An **empty file**
is a real case and reads 100 % — complete the moment it starts, not a divide by zero.

### Fixed — a layering inversion I introduced in this change

⛔ The tray first read the operation record by calling `crab_op_active()` from `src/ui.cyr`. That
compiled through `main.cyr` and left `src/render_test.cyr` — which includes `ui.cyr` **alone** — with
four undefined symbols. `ui.cyr` sits *below* `app.cyr` in the include chain; reaching up into it is
the exact inversion this codebase carved `path.cyr` and `app.cyr` out to prevent. The tray now takes
`opname` / `opdone` / `optotal` as parameters, the same rule `lstate` / `rstate` already follow.

### Changed — dependency: dhancha 0.9.21 → **0.9.22**

Released and pushed first, then declared, then check 4 re-run — the order the 2026-08-28 phantom-tag
failure exists to enforce. Remote SHA verified against the sibling before the bump.

### Verified

**476 → 520 passing**; `render_test` 26/26; reference coverage **80 %** (88/109) held while 13
functions were added; both targets build; `fmt --check` clean; `lint` at baseline.

⭐ **Seven mutations, each of which fails the suite** — a move that unlinks at the start instead of at
EOF; a cancel that leaves the partial destination; a second transfer silently replacing the first; the
source never stat'd; a step that never yields (one giant blocking step again); the percentage
dividing before multiplying; and an unknown total printing `0%`.

⚠ **Check 4 re-run after the dep bump** with all four `path` overrides disabled: 6 deps / 0 errors,
both targets, and byte-identical binaries.
⚠ **Not covered:** the idle-tick wiring itself. The step function is driven by hand in the suite;
that it is called from the tick, and that the tray repaints per step, is agnos-only event-loop code
no host test can reach — the same irreducible gap `main.cyr` has always had.

## [0.7.2] - 2026-08-31 — Enter opens, the second pane leaves, and files move by drag

⚠ **Everything here landed AFTER the `0.7.1` tag** (`4ac21eb`, pushed). It was written into 0.7.1's
section while that work was in flight; 0.7.1 is released, so its section is restored below to exactly
what shipped and this one carries the rest. ⛔ **A released section is not a scratchpad** — editing
one after its tag makes the tag and the notes disagree, and the notes are what a consumer reads.

### Added — M4: `open`. Enter on a file is no longer silent

⭐⭐ The roadmap's M4 section opens by naming this: *"crab is a read-only browser. Enter on a file
does nothing, silently."* Enter reached `crab_descend`, which refuses a non-directory and returns
-1, and the call site dropped that -1 with no else arm — indistinguishable from a keypress crab
never received.

⛔ **"Open" means "run", and only that, because nothing else exists on this system yet** — no handler
registry, no MIME association, no daimon binding anywhere in the stack, so a text file has nothing to
be opened *with*. crab says so plainly instead of inventing an association. When an association
mechanism exists (M7's daimon arc), `crab_fs_launch` is where it hooks in and the ELF path becomes
one arm of it rather than the whole thing.

⛔ **Enter is gated on a real ELF-magic read, not on mode bits.** Enter can now start a process, so
what it refuses matters more than what it runs: `crab_is_elf` reads four bytes off disk and refuses
anything else. On a single-user always-root kernel the permit bits say almost nothing about what a
file *is*. ⚠ A short read is **not** an ELF — the magic buffer is reused for the process, so treating
a short read as success lets a 3-byte file inherit the previous read's 4th byte. The test asserts
this in the order that makes the hazard real (a genuine ELF read immediately before).

⚠ `spawn_path`#43 caps its path at **127 bytes**, half of `CRAB_PATH_MAX` — a path crab can list and
stat is not necessarily one it can launch, and the difference is reported rather than truncated,
because a truncated path is a different file and this one gets executed. Non-blocking by
construction: crab does not reap the child.

### Fixed — navigation ran BEFORE the delete confirmation

⛔ The arrow keys and Enter sat above the confirmation gate, so a key answering *"delete this?"* also
moved the selection or descended — and then the gate cancelled, having already acted. The gate's own
comment claimed keys were resolved there first; they were not. Navigation now lives inside it.
⚠ Introduced by 0.7.1's own confirmation work, and fixed here rather than left for a burn to find.

### Added — M4: the second pane leaves at small widths, and MODIFIED finally appears

⭐⭐ **The canvas's open question is ratified: below 600 px crab shows ONE pane plus an A/B switcher.**
⛔ **The threshold is derived, not the canvas's 420 px copied in.** Two panes are worth it only when
**each** can honestly show the full column set — NAME + SIZE + MODIFIED — so `panew >= 297`, i.e. a
600 px window. A pixel lifted from a mockup is a number nobody can re-derive when the font or the
column set changes; this rule agrees with the drawing without quoting it.

⭐ **The switcher is the keys crab already has.** Left/Right (h/l) set `active_pane`, which in solo
mode changes which pane is *drawn*. The header gains an `A` / `B` label, because side-by-side is no
longer what tells the two apart — and that label is asserted, not left visual-only.

⭐⭐ **This fixes the MODIFIED column at the shipped default.** At 380x220 the two-pane split gave each
pane 187 px, so `crab_cols_for_width` returned **2** and MODIFIED never appeared — while the README
and this file's own headline read "NAME · SIZE · MODIFIED". One pane at 380 is 374 px and shows all
three.

⛔ **The pane that is not shown is not built** — its list pointer stays 0, which is what keeps
`crab_hit` from routing clicks into a pane nobody can see. Rendering it off-screen instead would
still hit-test.
⚠ **`render_test` and the click test now render at 640, not 380** — every assertion in them is about
two panes side by side. `render_test` gained solo-layout checks at 380 in exchange.

### Added — M4: drag between panes

⭐⭐ Press a row, drag past a 4 px Manhattan threshold, release over the other pane — the file
**moves**, matching the `m` key. Both panes re-list and the status line reports the result. crab had
no pointer-release arm at all before this; a press did everything and the button coming back up was
ignored.

⛔ **crab does not use dhancha's `DRAG_*` events, and this is not a workaround — it is the
architecture crab already had.** dhancha synthesizes drag inside `dh_dispatch`, which tracks a press
as a **widget pointer**, and crab rebuilds its whole tree every frame with the arena rewinding
underneath it (operator ruling 2026-08-27). Double-click hit that wall first and was solved with
**pane index + row index**; the wheel likewise. Drag is the third gesture and takes the same shape:
the toolkit supplies geometry through `crab_hit`, crab supplies identity that survives a frame.

⛔ **A drop inside the source pane is not a transfer** — dragging within a pane is how an operator
changes their mind. The destination is the other pane's *directory*, not the row under the pointer:
a row-targeted drop is a different gesture, and guessing wrong moves a file somewhere nobody pointed
at. Folders are refused, for the same reason `c` / `m` refuse them.
⚠ **A drag consumes the double-click pair**, or the release ending a drag would pair with the next
press and descend.

### Fixed upstream — dhancha 0.9.21, drag stopped half-working

⛔⛔ dhancha's drag API and its frame-arena API were mutually exclusive: `dh_frame_begin` calls
`dh_reset_input()`, which zeroes `_dh_drag_src` **every frame**, so an app using the frame arena saw
`DRAG_START` and then never `MOVE`, `DROP` or `END`. The clear is **correct** — after an arena rewind
that pointer addresses memory about to be reused — so the conflict is structural rather than a typo.
dhancha 0.9.21 makes `dh_drag_progress` refuse to begin a drag it cannot finish, and adds
`dh_drag_available()`.
⚠ **crab does not consume it** (see above), and crab's declared `tag` stays at **0.9.20** until
0.9.21 is pushed — declaring an unreleased tag is the 2026-08-28 failure that left no consumer able
to resolve dhancha at all.

### Fixed — a manufactured burn gate, removed

⛔ 0.7.1's notes claimed *"a re-burn is a gate, not a formality"* because agnos 1.56.55 rewrote
`is_user_range`. **That is agnos's validator on every ring-3 buffer in the system** — if it
regressed it regresses for everything, and proving it is agnos's job, not a reason to hold a crab
release. crab passes ordinary BSS/stack buffers and has no special exposure. Removed from
`handoff.md` and three source comments.
⭐ **QEMU runs a real agnos kernel and confirms these paths** — it is how every M3 item was
confirmed. Iron is for what QEMU cannot show: the GPU shader path, real HID timing, real disk
latency. A syscall wrapper call is not one of those.

### Added — coverage, raised by writing the assertions that were missing

**408 → 476 passing**, `render_test` **19 → 26** pixel checks, reference coverage **73 % → 81 %**
(78/96 fns) against a v1.0 criterion of 80 %.

⛔ **Not chased.** This is *reference* coverage — a function counts as covered the moment any test
names it, so `assert(crab_say("x") > 0)` would raise the number and prove nothing. The roadmap now
splits the untested set into three groups and says to write only the first.
⭐ **The biggest gap was `crab_entry_cmp`**, exercised only *through* `crab_sort_entries` — so every
ordering rule (dirs-first outranking the key, the -1/-2 sentinels, NAME as universal tiebreak) was
asserted in aggregate and none of them separately.
⚠ `docs/development/roadmap.md` now carries coverage as a **per-release gate**: a criterion checked
only at v1.0 gets further away at every cut that adds code, which is exactly what happened twice.

### Verified at the cut

- Both targets build; **476 / 0** tests; `render_test` **26 / 26**; `fmt --check` clean on all eight
  sources under the pinned 6.5.36; `lint` at or below the 0.7.0 baseline.
- ⭐ **Every new behaviour is mutation-proven** — the drag threshold's direction-independence, the
  same-pane drop refusal, the `-1` no-pane guard, the ELF short-read guard, the A/B pane label, the
  solo layout's unbuilt second pane, and the two-panes-fit threshold each fail the suite when broken.
- ⚠ **Check 4 has NOT been re-run since 0.7.1** — the dep graph is unchanged (crab still declares
  dhancha 0.9.20), so the 0.7.1 evidence stands, but re-run it if the tag moves.

## [0.7.1] - 2026-08-30 — the hang no test could reach, and crab starts writing

### Verified at the cut

- **The declared dependency graph resolves** — check 4 re-run with all four `path = "../X"` overrides
  disabled, which is the only one of the four checks that is evidence: **6 deps / 0 errors**, both
  targets build, **408 / 0** tests, and both binaries **byte-identical** to the path-resolved ones.
  (`cyrius.lock` going 2 → 6 commit-pinned is the tell that the overrides were genuinely off.)
- **`fmt --check` clean** on all eight sources under the pinned toolchain, `lint` at or below the
  0.7.0 baseline, and `lib/` verified **byte-identical to the released 6.5.36 tarball**.
- ⚠ **Reference coverage FELL, 81 % → 73 % (65/89 fns)** — arithmetic, not rot: 29 functions were
  added faster than references to them. The v1.0 criterion is 80 %, so this moved away from target.

### Changed — toolchain pin 6.5.35 -> 6.5.36, and the hardcoded syscall number is retired

⭐ **`CRAB_SYS_READDIR_AT = 101` is gone.** It was an interim with a written expiry — *"switch to
`sys_readdir_at` and delete this constant the moment crab's pin moves to >= 6.5.36"* — and that
condition is now met: **cyrius 6.5.36 is released** (tag + assets on the remote; the docs said
"UNRELEASED", which was true when written on 2026-08-28). The pin moves, `cyrius lib sync` vendors
`sys_readdir_at`, and both `#101` call sites go through the wrapper.

⚠ **crab is the FIRST consumer to actually CALL that wrapper.** agnos's own `rdat.cyr` proves the
kernel contract but uses the raw number, so until now the cyrius peer was asserted to compile and
had never been executed. The next burn is what proves it.
⚠ `cyrius lib sync` copies the **declared** `[deps].stdlib` set — **29** files — while crab vendors
**30**. The odd one out is `lib/atomic.cyr`, a transitive leaf no declaration names; checked by hand
and identical to the 6.5.36 snapshot.
⚠ 6.5.35 and 6.5.36 format crab's files identically, so the bump caused no formatting churn.

### Fixed — the two P0 defects the M3 review found and left open (2026-08-30)

Both were in code that built and passed **253 / 0**, which is why the suite never found them.

- ⛔ **Neither `#101` readdir walk terminated on a stalled cursor — a hang, not a crash.**
  `agnos/kernel/core/ext2.cyr:2401` and `:2405` `store64(cursor_uva, pos)` with `pos` **unchanged**
  and `return count`, which may be `0` and is **not negative** — so a persistent block-read failure
  satisfied `k >= 0` and `cur != -1` forever, spinning crab in a syscall loop with no output.
  Both loops now break on zero-records-AND-an-unmoved-cursor, and both carry a
  `CRAB_READDIR_STEPS_MAX` backstop. ⚠ The listing loop's `n >= CRAB_MAX_ENTRIES` test was **not**
  a second escape: `n` advances only by `n = n + k`, and on a stall `k` is the zero.
  The decision is lifted into `crab_readdir_stalled` so the suite can assert it — the loops
  themselves are inside `#ifdef CYRIUS_TARGET_AGNOS` and cannot be reached from a host test.
- ⛔ **`crab_name_cell` scanned kernel readdir data with an unbounded `strlen`** (`src/ui.cyr`),
  the same data whose unbounded copies caused the 0.5.0 P-1. It was safe only because `chars` was
  small at the shipped 380x220 window — and `chars` is width-derived, so above ~1,860 px it exceeds
  the cell and the kernel's own NUL becomes the sole guard. crab already accepts a 2560x1440 resize,
  so one F5 reached it. Now bounded by `CRAB_REC_TYPE`, and the destination is derived from that
  constant rather than a bare `80`.

### Fixed — a full buffer is not a truncation (P1)

A directory of **exactly** `CRAB_MAX_ENTRIES` entries printed *"has more entries than are shown"*.
⛔ **The obvious fix is wrong**: `ext2.cyr:2412` *parks* the cursor on the record it declined to take
whenever the batch budget is reached, so `cur != -1` at exactly the cap and a cursor test still
reports the false truncation. The oracle is the **count** — `crab_truncation_note` now returns
"say nothing" / "showing n of total" / "has more, count unknown", and the unknown arm no longer
depends on a digit buffer to terminate its line.

### Changed — the sort is O(n log n), off the keystroke path (P2)

`crab_sort_entries` was insertion sort with a 64-byte record swap done **one byte at a time**, and it
runs once per listing — every descend, every ascend, both panes on `s`. M3 *#02* raised the cap
256 -> 1024 without re-deriving the comment that justified it ("`CRAB_MAX_ENTRIES` is 256"). Now a
bottom-up merge over an **index array**, then one cycle-following permutation pass, so the 64-byte
payload is touched once per entry instead of O(n^2) times. Measured on native x86_64:

| n | order | insertion (was) | merge (now) | |
|---|---|---:|---:|---|
| 1024 | scrambled | 89.1 ms | **447 us** | 199x |
| 1024 | reverse-sorted | 182.9 ms | **414 us** | 442x |
| 256 | scrambled | 5.48 ms | 84 us | 65x |
| 122 (the iron `/`) | scrambled | 1.24 ms | 36.5 us | 34x |

⚠ **Stability is preserved and is load-bearing** — it is what keeps a re-sort from reshuffling equal
rows under the selection. `crab_sort_insertion` is **retained** as the fallback when the index
scratch cannot be allocated, and doubles as the differential oracle the merge sort is tested against.
⚠ The old per-call `alloc(CRAB_REC_SZ)` swap slot leaked 64 B **per keypress** into an allocator with
no `free()`; the scratch is now allocated once for the process.

### Fixed — stale cost comments the cap bump invalidated (P2)

Seven sites, not the three the review listed: `~280 ms at the 256 cap` (now ~1.1 s) in two files,
"8 ticks" (now 32), "114 entries on iron" (now **122**, measured 2026-08-30), and two more.
`src/main.cyr` also described listing as *"via the readdir syscall (#81)"* in two places when both
live call sites are `#101`.

⭐ **And the arena comment was stale for a second, undocumented reason.** It claimed 256 KiB
"absorbs a full frame at the `CRAB_MAX_ENTRIES` ceiling without ever chaining". Re-measured with
`arena_capacity_total` (**not** `arena_used`, which reports the current chunk only and shows 13,104 B
for a 2.6 MB frame):

| window | entries | cols | chain total | chunks |
|---|---:|---:|---:|---:|
| 380x220 | 122 | 2 | 262,144 | 1 |
| 380x220 | 1024 | 2 | 1,835,008 | **7** |
| 2560x1440 | 1024 | 3 | 2,621,440 | **10** |

Two things moved under it: the cap 256 -> 1024, and *#32* taking a row from one widget to
1 + `ncols`. ⭐ Still **not** a leak — `arena_reset` keeps the chain, so the global heap still sees
zero bytes per steady-state frame. 256 KiB is kept deliberately: it fits the case the burn actually
ran.

### Added — M4: pane states, and the write layer

- ⭐ **An empty pane now says why it is empty.** Blank meant four unrelated things — empty, gone, not
  a directory, unreadable — because `crab_readdir_into` clamps its return to `>= 0`. The kernel's
  code now survives the call (`crab_listing_err`) and `crab_pane_state` classifies it.
  ⛔ **There is no "permission denied" state, and inventing one would be a lie**: agnos is
  single-user always-root, `ext2_readdir_at_sys` has no `EACCES` arm, and `getuid`#15 is a literal
  `return 0`. The burn's *"operation not permitted"* lines came from `stat` failing on a broken
  symlink, not from any denial crab can observe.
- ⭐⭐ **The write layer — copy, move, delete — and M4's stated gate does not exist.** The roadmap
  read *"Gate: agnos write syscalls"*. Every arm has been real and mount-routed since **1.41.3**
  (`open`#7, `mkdir`#9, `rmdir`#10, `unlink`#30, `rename`#31), and crab's **already-pinned cyrius
  6.5.35** vendors a wrapper for every one. No pin move was required.
  ⛔ **The trap**: agnos's own userland-ABI *table* still calls mkdir/rmdir *"stub -> 0"*, contradicted
  by the dispatcher in the same repo. Verify against the dispatcher, never the table.
- ⛔ **The two targets disagree on every signature** — agnos takes an explicit `pathlen`, Linux takes
  a NUL-terminated path and a **mode** in the same position. Same arity, different meaning, so a call
  written for one compiles clean against the other. One shim per operation; nothing else calls `sys_*`.
- ⛔ **agnos has no `AO_EXCL`**, so the kernel cannot refuse an overwrite. Every destination is
  checked with `crab_fs_exists` first — a copy that relied on the open failing would refuse correctly
  on the host and **silently truncate on agnos**. The residual TOCTOU window is disclosed, not hidden.
- Keys: `c` copy, `m` move to the other pane (the dual-pane idiom — no text entry needed), `d` delete
  behind a `y`/anything-else confirmation shown in the status line. ⚠ Not F5/F6: aethersafha takes
  F5 for maximize, so a client binding it would never see the key.
- ⛔ **No recursion, deliberately.** `rmdir` refuses a non-empty directory and crab reports it;
  folders cannot be copied. A recursive delete behind one keypress cannot be undone or interrupted,
  and crab has neither a progress surface nor a trash. That is M4's genuinely gated part.

### Added — tests, and a benchmark that measures something

**253 -> 408 passing**, plus `render_test` **14 -> 19** pixel checks.
⭐ The write layer's tests perform **real syscalls against a real filesystem** — unlike `#101`
readdir, `mkdir`/`unlink`/`rename`/`open`/`read`/`write` all exist on the host, so the refusals, the
bounded join, the overwrite guard and multi-chunk copying are exercised for real.
⛔ **The data-loss path needed forcing to reach**: `crab_fs_move` tries `rename` first and falls back
to copy-then-delete only when it refuses — and within one filesystem rename never refuses, so a
mutation that deleted the source regardless of the copy's result **passed the entire suite**. It is
now reached by naming a destination directory that does not exist.
⚠ `tests/crab.bcyr` timed `bench_noop` — an empty function — until now, which is why the latency
regression above shipped unnoticed. It now measures the sort at the cap, at 256, and at the burn's
real 122.


### Fixed — the declared dependency graph did not resolve (release-plumbing repair, 2026-08-28)

The M3 gate work shipped green locally while naming two dependency tags that **existed on no remote**.
`path = "../X"` wins over `tag`, so every local build resolved against sibling working trees and said
nothing. With the `path` lines disabled — the only check that is evidence:

```
fatal: Remote branch 0.1.5 not found in upstream origin
fatal: Remote branch 0.9.20 not found in upstream origin
4 deps resolved, 2 errors
```

- **rupa `0.1.5` and dhancha `0.9.20` are now genuinely released** (`27e8385`, `61a1e39`). dhancha
  0.9.20 had been unusable by *any* consumer, because it pinned the same phantom `rupa 0.1.5`.
- **Re-verified with every `path` override disabled**: `cyrius deps` → **6 deps / 0 errors**, host and
  `--agnos` both build, **253 / 0** tests.
- **`dhancha 0.9.19` never existed** — no tag, no CHANGELOG entry, no commit. `dh_theme_on_accent`
  landed in **0.9.20**. Corrected in `cyrius.cyml`, `CHANGELOG.md`, `docs/development/roadmap.md` and
  `src/render_test.cyr`.
- **cyrius `6.5.36` is unreleased** — the docs claimed it "ships" `sys_readdir_at`. It has no tag; the
  latest release is `6.5.35`, and CI installs releases. `CRAB_SYS_READDIR_AT = 101` therefore stays
  hardcoded, and the expiry note now says why the pin *cannot* move yet.
- **`lib/` decontaminated.** The local `~/.cyrius` 6.5.35 stdlib snapshot had been overwritten with
  6.5.36 content, and `cyrius deps` vendored it into crab's tracked `lib/` — `SYS_READDIR_AT` and
  `sys_readdir_at` appeared in a 6.5.35-pinned project. Restored from the released tarball;
  `cyrius.lock` now differs from 0.7.0 by exactly the two dep-bundle hashes.
- ⚠ **rupa and dhancha also moved their toolchain pin to `6.5.35`.** The rest of the desktop stack
  stays on `6.5.27` deliberately.

### Documented — five open defects, none fixed

A full review of the M3 gate work found five defects in code that builds and passes 253/0. They are
recorded with line numbers in [`docs/development/handoff.md`](docs/development/handoff.md) and
summarised in [`docs/development/state.md`](docs/development/state.md): a non-terminating `#101`
readdir loop, an O(n²) sort regression put back on the keystroke path by the 256 → 1024 cap bump
(**measured 6 ms → 100 ms native**), a false truncation warning at exactly the cap, three stale cost
comments, and an unbounded `strlen` over kernel-supplied data.

### Fixed — the selected row's text was unreadable (the `on-accent` gate)

The selected row is filled with `accent` and its label was drawn in the theme's primary `ink` — on
MUDRA dark, `0xE7E9EF` on `0x00E5FF` is a contrast ratio of **1.27:1**. The one row the operator is
looking at was the one row that could not be read.

- **rupa 0.1.4 → 0.1.5** — publishes the `on-accent` token, plus `rupa_luminance` / `rupa_contrast` /
  `rupa_ink_on`. All four grounds clear the WCAG AA floor (12.72 / 4.64 / 11.27 / 5.49 : 1).
- **dhancha 0.9.18 → 0.9.20** — binds `dh_theme_on_accent()` and paints the focused selection's text
  with it. Also fixes a second, older defect: the scalable-font path blitted hardcoded white and
  ignored the theme entirely, so it was unreadable on both light grounds.
- ⛔ **rupa >= 0.1.5 is a hard requirement**, not a freshness preference: dhancha 0.9.20 references
  `rupa_theme_on_accent`, and crab pulls rupa's dist directly.
- `src/render_test.cyr` gains pixel checks: the selected row carries on-accent ink and **not** the
  primary ink, with the unselected pane as the control. Mutation-proven by removing the swap from
  dhancha's **dist bundle** — ⚠ mutating dhancha's `src/` proves nothing, because crab compiles
  `dist/dhancha.cyr`.

### Added — real columns (*deferral #32*)

- **NAME · SIZE · MODIFIED with headers**, on dhancha 0.9.20's shared column-width spec. The
  **13-character name column is gone**: NAME is the remainder column and grows with the pane.
- ⛔ **Columns that do not fit are dropped, not squeezed.** The default 380x220 window gives a ~187 px
  pane — about 20 characters. A MODIFIED column is 153 px on its own, which would leave four
  characters for the filename. The pane picks its column set from its own width.
- ⛔ **The header sits outside the LIST**, so it neither scrolls away nor can ever be selected.
- ⚠ Truncation still says it was truncated (`~`), now at the column's width rather than a
  hardcoded 13.
- New: `crab_cols_for_width`, `crab_col_chars`, `crab_name_cell`, `CRAB_COL_*`.

### Added — directories larger than the cap (*deferral #02*)

- **`CRAB_MAX_ENTRIES` 256 → 1024** (the design canvas asks for `812 items` in a pane), and listings
  are read in 64-record batches through agnos 1.56.50's new **`#101 readdir_at`** cursor.
- **The truncation warning now carries numbers**: `showing 1024 of 1200`, not `has more entries than
  are shown`. Once the pane buffer fills crab keeps walking **without storing**, purely to count.
- ⛔ **Falls back to `#81` when `#101` is absent.** On a kernel older than 1.56.50 the dispatcher
  returns `-1` for the unknown syscall; without the fallback crab would show an empty pane.
- New: `crab_listing_total()`, `CRAB_READDIR_BATCH`, `CRAB_SYS_READDIR_AT`.
- ⚠ **No host test is possible** — the paged path is entirely inside `#ifdef CYRIUS_TARGET_AGNOS`.
  Proven in QEMU against a seeded 1200-entry directory, and mutation-proven by deleting the counting
  walk.

## [0.7.0] - 2026-08-28 — M3: a browser you would actually use

**M3 — "a browser you would actually use"**. crab listed a directory in whatever order the filesystem
handed it back, forgot where you had been the moment you pressed Backspace, always started in the same
two hardcoded directories, and blocked the keystroke that descended on one synchronous `stat` per
entry. All four are fixed.

⚠ **Four of M3's seven items shipped. The other three are gated upstream, not unfinished**: real
columns (*#32*) needs a dhancha TABLE widget, directories past the 256-entry cap (*#02*) needs a
resumable `readdir` from agnos, and `on-accent` needs rupa.

### Added — sorting (*deferral #33*)

- **`s` cycles four sort modes** across BOTH panes and re-sorts them: name (case-insensitive) · size
  (largest first) · modified (newest first) · kind (extension, then name). Applied to the initial
  listings and to every descend and ascend.
- **Directories sort first under every mode**, unconditionally.
- Dotfiles are **not** hidden. `.` is 0x2E, so they sort ahead of every letter.

### Added — selection memory (*deferral #34*)

- **Backspace lands on the directory you just left**, not at the top of the parent.
- **`s` follows the selected entry** through the re-sort instead of resetting the selection.
- A name that is no longer present reports `-1` and the caller falls back to row 0.

### Added — starting paths from argv (*deferral #11*)

- **`crab [LEFT] [RIGHT]`**. `/bin` and `/` were hardcoded; `args` was declared in `[deps].stdlib` and
  never called. Both defaults remain, and the agnos desktop always uses them — the compositor spawns
  crab through the launcher with a path and no arguments.
- **A path that is not absolute, or that does not fit `CRAB_PATH_MAX`, is refused and announced on the
  console**, then the default is used. It is not silently substituted.
- argv on agnos comes from `lib/args_agnos.cyr` (the init rsp parked in **r15** at entry, argc capped
  at 8 by the kernel); on the host it comes from `/proc/self/cmdline`.

### Changed — deferred statting (*deferral #03*)

- **A listing no longer stats anything.** It marks every entry pending and returns; the event loop
  drains **32 entries per idle tick**. Measured cost of the old behaviour on agnos under QEMU: **1.1
  ms per entry** (50 ms for 45 entries in `/bin`, 10 ms for 7 in `/`) — **~280 ms** at the
  `CRAB_MAX_ENTRIES` cap of 256, paid on the keystroke that descends or ascends.
- **A sort to size or modified forces the full sweep**, because those orders cannot be computed from
  partial data. Name and kind order need no stat data at all — dir-ness comes from the readdir record
  (`CRAB_REC_TYPE`), not from stat.
- **`-2` (pending) is distinct from `-1` (stat failed)** and renders differently: `?` versus `-`, in
  both the size column and the date.
- New: `crab_sz_pending`, `crab_stat_reset`, `crab_stat_next_pending`, `crab_sort_needs_stats`,
  `crab_stat_one`, `crab_stat_batch`, `crab_stat_for_listing`. `crab_stat_all` is now the full sweep
  only. `CRAB_STAT_BATCH = 32`.
- New console lines, one per sweep and one per drain: `crab: stat-cost <ms> ms for <n> entries in
  <path>` and `crab: stat-drain complete`. ⚠ Named `stat-cost`, not `stat`, so they are not counted as
  per-entry trace lines by `crab-listing-cap-test.py`.

### Testing

- **The dependency graph was verified with every `path` override disabled.** crab's manifest sets
  `path = "../X"` for rupa, kashi, dhancha and setu, and **`path` wins over `tag`**, so a local build
  proves nothing about the declared tags. With the paths disabled `cyrius deps` cloned the tags, both
  targets built, all 228 tests passed, and the binaries came out **byte-identical** to the
  path-resolved ones. Tag SHAs were confirmed against the GitHub API with `curl`.
- **Reference coverage 44/54 fns (81 %)**, 6/6 files — up from 27/36 (75 %) at 0.6.1, and the first
  time crab is above the v1.0 criterion of 80 %. ⚠ Reference coverage counts a function as covered
  when something references it: a floor, not a correctness proof.
- 228 host assertions pass. `crab /bin /bin` and `crab relative` are QEMU-proven against agnos in
  `agnos/scripts/harness/crab-listing-cap-test.py`, which also now records **zero** `stat-cost` lines
  where it previously saw six. `agnos/scripts/harness/crab-resize-test.py` reports `deferred stat
  drain completed: True` — the drain needs a compositor, so the two halves are proven separately.

---

## [0.6.1] - 2026-08-27 — M2: the window answers the pointer, the wheel, and a held key

**M2 — "the window is real"**. crab was a fixed 380x220 rectangle that understood one keypress at a
time. It now takes pointer input, a mouse wheel, key releases and held-key repeat, and it acts on a
compositor resize request instead of ignoring it.

### Added — pointer input (*deferral #05*)

Click to select, click to focus a pane, double-click to descend (400 ms, monotonic `clock_now_ms`).
⭐ **QEMU-proven**: `crab: click` on a real kernel, a click resolved to a pane, keys still answered
afterwards. crab is the **first client in the stack to decode `SETU_INPUT_PTR_MOVE`**.

⛔ **crab OWNS ITS INTERACTION STATE — `dh_dispatch` is deliberately NOT used.** It tracks a press by
storing a **widget pointer**, and `crab_render` opens with `dh_frame_begin()`, which rewinds the frame
arena and clears exactly those pointers — so a press and its release are separated by a rebuild and
the target no longer exists. dhancha 0.9.15 states the rule: *cross-frame widget identity and a
per-frame arena are mutually exclusive by construction*, and press/release tracking **is** cross-frame
widget identity. crab tracks **pane index + row index**; the toolkit supplies geometry via
`dh_hit_test` only.
⚠ `SETU_INPUT_PTR_BTN` carries **no coordinates**, so position comes from `PTR_MOVE`.

### Added — the mouse wheel, across six repos

`agnos 1.56.49` reads HID report byte [3] → `bhumi 1.4.3` carries `BHUMI_EV_SCROLL` →
`setu 0.8.8` defines `SETU_INPUT_PTR_SCROLL` → `dhancha 0.9.18` maps `POINTER_SCROLL` →
`aethersafha 0.16.21` forwards it → crab scrolls the pane under the cursor.

⛔ **The chain was broken at the BOTTOM, not at setu.** The gate read "setu has no wheel message
kind"; the real defect was that **agnos discarded the wheel byte** — `hid_process_mouse_report` read
bytes [1] and [2] and left byte [3], documented in its own layout comment as `wheel (s8, optional)`,
on the floor. `#98 ptrscan`'s record had no field for it and bhumi had no scroll concept. A setu patch
alone would have been a fourth dead wire after `SETU_CLOSE` and `SETU_CONFIGURE`.
⭐ The byte was **QEMU-measured before any layer above it was written**: `hid: wheel byte seen, b3=1`.

⛔ **crab's wheel moves the SELECTION, not the view.** `crab_render` restores each pane's scroll
offset and then calls `dh_list_scroll_to_sel`, so a free-scrolled view is snapped back on the next
frame by the machinery keyboard navigation depends on. A detached view-scroll is a separate change.

### Added — key releases and held-key repeat (*deferral #06*)

crab requests `SETU_SURF_FULL_KEYS`; the compositor honours it per surface (`mods` = 1 press /
0 release). ⭐ **QEMU-proven exactly: 6 keystrokes → 12 `key received` / 6 `key press`.**

⛔ **Asking for the flag without gating on it makes every key act twice**, and the compositor carries
that burn: *"three F3 presses produced SIX `theme switched` lines, and the launcher moved its
selection twice per keypress"* (2026-08-18). For crab it is two rows per Down and Enter descending
twice. ⚠ The flag and the gate are coupled — on a press-only surface `mods` is 0 for a **press** — so
they live together in `src/app.cyr`.

Held-key repeat: Up/Down/j/k only, 400 ms delay then 60 ms interval, both gates required.
⛔ **Enter and Backspace do NOT repeat** — they would walk the operator through the filesystem on one
held key, re-readdir'ing and re-stat'ing every step.
⭐ QEMU-proven with a QMP-held key (HMP `sendkey` cannot hold — it sends both edges): repeat fires and
**stops on release**.

### Added — resize (*deferrals #01, #04*)

`WINDOW_CONFIGURE` is handled: `dh_surface_resize` (dhancha 0.9.17), the `#86` shm slot **created
before the old one is closed**, `w`/`h`/`stride` as state, re-ATTACH + COMMIT after the next render.
Layout reflows for free — `crab_render` reads `w`/`h` off the surface.

⛔ **A new harness found a real bug on its first run.** The draft closed its only shm buffer before
knowing the replacement existed and exited. setu's own `setu_client_present` closes first, but it has
an inline-pixel fallback to land on; crab's LIVE-buffer path has none.
⛔ **And the byte cap was invented, not derived** — 16 MB from "the framebuffer's size", when agnos
caps a `#71` pmm slot at **2 MB** and only a `#86` GPU carveout reaches 32 MB, chosen at runtime.
⚠ **The refusal path is QEMU-proven; the ADOPT path is not** — QEMU has no carveout, so a
2048x2018 ask cannot be backed. crab refuses, keeps its extent, and stays alive.

### Fixed — the render/input loop allocates nothing (*deferral #09*)

`dh_setu_poll_event` allocated an 80 B message **before** it knew whether anything was pending
(dhancha 0.9.16). With 0.6.0's frame work, **crab's whole loop is allocation-free in steady state**,
which is what makes a self-repainting element affordable.

### Known — two M2 items are gated upstream

- ⛔ **`dh_dispatch` routing (*#07*) is blocked on a type confusion spanning three repos.**
  aethersafha sends an **HID usage**; setu's protocol calls the field a **`keysym`**; dhancha maps it
  into `DhEvent.a`, whose `DH_KEY_*` constants are **ASCII codepoints**. `DH_KEY_TAB = 9`, and HID
  usage 9 is **`f`** — so routing crab's keys through `dh_dispatch` would Tab-traverse whenever the
  operator typed `f`. **Gate: dhancha.** puka already pays this toll explicitly
  (`setuwin__hid_to_evdev`); dhancha never added the equivalent.
- ⚠ **The wheel's last hop is unproven.** QEMU's `usb-mouse` is RELATIVE, so no harness can place the
  cursor on crab's window; the compositor reports `got a scroll with NO client window under the
  cursor`, so crab's silence there is correct behaviour, not a defect.
- ⚠ **Repeat's observed RATE is far below its configured interval** (~1 per 1.6 s hold vs ~20
  expected) and is **not diagnosed**. Each repeat re-renders and rewrites 334 KB of shm, so frame cost
  is the likely bound — a hypothesis, recorded as one.

### Testing

**75 → 134 assertions**, reference coverage **70 % → 75 %** (27/36 fns, 6/6 files). New: the resize
policy, the pointer policy and hit-test geometry against a real rendered tree, the wheel, the
FULL_KEYS gate, and the repeat policy. Every group mutation-proven.

⭐ **New harness `agnos/scripts/harness/crab-resize-test.py`** — the only one that both starts crab
(F2 → **DOWN** → Enter picks `/bin/crab` specifically) and leaves it running. It found the resize bug
on its first run, and it **settles the loop-lifetime question open since 0.5.0**: keystrokes answered
long after launch, on a live desktop.
⚠ It is flaky by nature and says so — QEMU drains HID once per frame, and the key-delivery probe
measured 3/8 on one run and **0/8** on the next against the same image. It retries and returns
**INCONCLUSIVE** rather than a verdict when nothing was delivered.

## [0.6.0] - 2026-08-27 — a rendered frame costs nothing, and `main.cyr` is testable

Two structural things, no user-visible features. crab looks and behaves exactly as 0.5.0 did.

⚠ **This release took the version number the roadmap had reserved for M2** ("the window is real" —
resize, pointer input, key release, `dh_dispatch`). **None of M2 shipped here**; what shipped is the
gate that was blocking every milestone after it, plus the test floor that gate exposed. **M2 moves to
v0.6.1** in [`docs/development/roadmap.md`](docs/development/roadmap.md) — a patch, absorbed inside
the 0.6 line, so the ladder from M3 onward (v0.7.0 … v1.0.0) is unchanged.

### Fixed — every frame allocated ~750 KB and nothing was ever freed

`crab_render` cost **746,440 B per call** at 380x220 with 114 entries per pane, into a bump allocator
with **no `free()`** — so every frame crab ever drew was retained for the life of the process. It had
not bitten because crab repaints only on input and used to exit after two seconds; 0.5.0 removed both
accidents.

| | per steady-state frame |
|---|---:|
| 0.5.0 (dhancha 0.9.12) | 746,440 B |
| + dhancha 0.9.13 — `dh_surface_new`'s dead pixel buffer, deferred | 412,040 B |
| + dhancha 0.9.14 + crab — the sadish render target, reused | 77,568 B |
| + dhancha 0.9.15 + crab — the widget tree, arena'd | **0 B** |

Identical at 114 entries per pane (the real iron count for `/`) and at 256, the `CRAB_MAX_ENTRIES`
ceiling. ⚠ **Zero is per-frame, not total** — a one-time ~597 KB (334,432 B render target + 262,144 B
arena chunk) is allocated on the first frame and reused for the process's life. A fixed cost instead
of a per-keypress one is the whole point.

crab's side: `crab_render` takes a caller-owned `DhSurface` instead of minting one per call (18 → 17
parameters — `w`/`h` are read off the surface, so they can no longer disagree with it), every
per-frame allocation in `src/ui.cyr` goes through `dh_falloc`, and `crab_render` owns, installs and
rewinds the frame arena itself.

⛔ **`dh_widget_set_text` stores the pointer and does not copy**, so moving the row and status
strings onto the arena was a *lifetime* requirement, not an optimisation — a global-alloc string on
an arena widget outlives the widget forever.

⚠ Two upstream claims this project had recorded were wrong and are corrected in
[`docs/architecture/001`](docs/architecture/001-every-frame-allocates-and-nothing-is-freed.md): the
first step was **not** "one line" (the naive form breaks dhancha's `event_test` by downgrading
`dh_surface_present`'s refusal code), and the second was **not** purely upstream (a per-`DhSurface`
cache saves nothing while the caller mints a `DhSurface` per frame).

### Changed — dhancha 0.9.12 → 0.9.15, and two of the three are contract changes

- **0.9.14**: `dh_surface_render` may return the **same** surface twice. A caller wanting two frames
  at once needs two `DhSurface`s — `src/render_test.cyr` is exactly that caller and now creates two,
  guarded by `check(sds2 == sds, 0)`.
- **0.9.15**: `dh_frame_begin` rewinds the arena **and** clears dhancha's retained widget pointers
  (`_dh_focus`, `_dh_hover`, `_dh_press`, `_dh_drag_src`). The halves cannot be separated — never
  call `arena_reset` on a frame arena directly — and an app on a frame arena **must re-establish
  focus every frame**, which `crab_pane` does.

### Fixed — nothing in `src/main.cyr` was reachable from any test

`main.cyr` ends in `_entry();`, so including it from a suite runs the app. The readdir parser, the
stat layer, `crab_descend`, `crab_ascend` and the premultiplied surface flag therefore had **zero
reachable coverage** — in a program whose two shipped defects were both found on iron. `src/path.cyr`
was carved out of the same file at 0.5.0 for the same reason; **`src/app.cyr` finishes that
extraction.** `main.cyr` is now `main()` and `_entry()` and nothing else.

⭐ **And the arena setup was moved out of `main()` rather than tested around it.** It used to be
created and installed there, where deleting `dh_frame_arena_set` broke no test while restoring a
77 KB-per-frame leak. `crab_render` now owns it. The residual gap — `main()` itself is not callable
from a suite — is irreducible, and is now down to the event loop alone.

### Testing

**37 → 75 assertions**, reference coverage **53 % → 70 %** (19/27 fns, 6/6 files). New groups: the
reused render target, the zero-cost frame, and the application layer.

⭐ **Mutation-verified throughout** — 7 mutations against the app layer and arena ownership, 3 against
crab's surface reuse, and 5 against dhancha's arena, each producing named failures.

⛔ **Three tests could not fail in their first draft, and only mutation testing said so.**
- The surface-reuse residue check rendered trees that repainted every pixel, so deleting dhancha's
  `sd_clear` left it green.
- The convergence check used a 256 KiB arena against a three-entry fixture, so twenty frames fitted
  with room to spare — deleting `dh_frame_begin()` **entirely** left the whole suite green. An arena
  that is merely big enough never touches the global heap whether it is rewound or not.
- dhancha's own grow test claimed to "force the chain to extend" against an arena four times larger
  than the frame it rendered.

⚠ **The zero-cost gate has two independent guarantors and no single mutation fails it** — deleting
dhancha's `sd_clear` leaves it green (crab's opaque root still covers) and making crab's root
transparent leaves it green (the clear still covers); only removing **both** fails, at 7,744 surviving
bytes. Correct for a property test, but it means a green crab suite is **not** evidence that the
toolkit still clears — that lives in dhancha's `programs/draw_test.cyr`.

### Known — the repaint rule moved rather than lifted

The frame is free, so the idle mascot line, M4's transfer tray and M7's index progress are no longer
blocked by it. ⛔ But `dh_setu_poll_event` still calls `setu_msg_new()` **before** it knows whether
anything is pending — ~80 B per poll, never reclaimed, ~4.8 KB/s at 60 Hz. Continuous repaint implies
continuous polling, so closing that (roadmap M2, *deferral #09*, **gate: dhancha**) is the
precondition for anything that repaints without input.

⚠ **`lib/` is not what compiles.** Measured: appending garbage to `lib/dhancha.cyr` leaves the build
green while appending it to `../dhancha/dist/dhancha.cyr` fails — the `path` override compiles the
sibling's `dist/` directly. `lib/alloc.cyr` is inert too; the stdlib comes from the installed
toolchain. ⇒ The stdlib's arena internals **cannot be mutation-tested from this repo**, and the
`path`-wins hazard is worse than previously documented.

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

⛔ **And the idle wait is load-bearing, not politeness.** `dh_setu_poll_event` calls `setu_msg_new()`
*before* it knows whether anything is pending, so every idle poll leaks ~80 B into an allocator with
no `free`. Removing the frame cap without slowing the poll would have turned a bounded 152 MB leak
into an **unbounded** one — roughly 80 MB/s of idle growth. The loop now waits on an interrupt when
the poll comes back empty, and drains at full speed when it does not. The upstream fix is a dhancha
gate (roadmap M2).

⚠ Not `dh_client_next_event`, which blocks: crab must be able to redraw without input for the idle
mascot line, transfer progress and index progress. A blocking read forecloses all three.

### Fixed — the idle wait froze the whole desktop, and only QEMU caught it

⛔⛔ **A draft of the fix above used `sys_sleep_ms(16)`, and it was a REGRESSION THAT PASSED THE
ENTIRE HOST SUITE.** `sleep_ms` (#41) is the **DOOM frame-pacing** primitive: it calls
`preempt_disable()` and then halts until its tick target. The kernel's own comment says the quiet
part out loud — *"we can't be preempted off mid-sleep"*. That is correct for a game that owns the
machine and catastrophic for a desktop client: **while crab slept, nothing else could be scheduled.**

Measured A/B against `scripts/harness/puka-terminal-test.py` on a real agnos kernel in QEMU
(`-smp 4`), same image, same mode, only the binary changed:

| build | clients placed | clients presented | `--clients` verdict |
|---|---:|---:|---|
| 0.4.15 baseline | 2 | **2** | exit 95 — pass |
| 0.5.0 draft (`sleep_ms`) | 2 | **0** | never returned — fail |
| 0.5.0 shipped (`sys_pause`) | 2 | **2** | exit 95 — pass |

⚠ **crab did not merely fail to yield — it stopped the compositor from running at all.** Both
clients went dark, not just crab, and `aethersafha --clients` never finished.

⭐ The shipped primitive is **`sys_pause` (#14)**, whose syscall handler **yields to a ready proc
first** and only falls through to a safe `IF=1 hlt` when nothing else is runnable. Other processes
get the CPU; on a genuinely idle machine crab waits on an interrupt instead of spinning, which is
also what bounds the per-poll leak above.
⚠ **Not `sys_sched_yield` either** — yield hands off and comes straight back, so an idle desktop
still spins a core at poll speed. `pause` is yield-*then-wait*.

⛔ **The host suite was green for all three builds.** 37/37 passed against the version that froze the
desktop, because the loop lives inside `#ifdef CYRIUS_TARGET_AGNOS` and no host test executes it.
This is the exact class of defect the QEMU harnesses exist for, and the reason the cause is now
written as a ⛔ block at the call site rather than left as a one-word choice.

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

`cyrius build` OK on x86_64 (381,536 B) and `--agnos` (381,592 B) · `cyrius tests` **37 / 0** ·
`fuzz` PASS · `bench` PASS · `vet` 1 dep, 0 untrusted, 0 missing · `deny` 0 violations ·
`fmt --check` clean · coverage **53 %** (was 23 %).

⚠ Binary grew 377,288 → 381,536 B (**+4,248**), which is the bounds checks. That is the price and it
is worth stating rather than burying.

⭐ **Run on a real agnos kernel in QEMU** (`-smp 4`), which is what caught the `sleep_ms` regression
above:

- `scripts/harness/crab-listing-cap-test.py` — **PASS**, exit 0. The `/bin` pane listed **45 of 45**
  entries with no truncation warning and no fault, exercising the repaired path layer on real ext2:
  `crab_join_n` once per entry into the 256-byte scratch, the readdir clamp, and the
  `STAT_SIZE` / `STAT_MTIME` / `STAT_BUFSZ` named offsets.
- `scripts/harness/puka-terminal-test.py` — **PASS**, background exit **95**, "both clients connected
  and presented", 2 compositor presentations, 0 faults. crab connected over the **current channel-band
  transport**, presented, and left its loop with `crab: compositor closed the window -- exiting` —
  which is the 0.5.0 `WINDOW_CLOSE` path, observed on a real compositor rather than argued for.

⚠ **Still not run on iron**, and QEMU is not a control for timing- or pressure-dependent behaviour —
the harness README is explicit that a lossy-queue failure which killed a client on iron reproduced
not at all under QEMU. The per-frame allocation ceiling in particular is an iron question.

⚠ **`crab_descend` / `crab_ascend` were not exercised on agnos.** Both harnesses run crab without
driving navigation keys, so the bounded-join *refusal* path has host assertions but no agnos run.

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
