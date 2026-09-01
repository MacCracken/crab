# Handoff — **M5: preview, thumbnails, EXIF and the GRID view are all in.**

> ⭐ **Updated 2026-08-31**, after M5's first slice and the chitra adoption. Everything below the
> *What landed* table is kept from the 0.7.5 handoff because the reasoning is still the reasoning —
> but **every number in it is 0.7.5's or older**. Re-derive before quoting.
>
> ⛔⛔ **THE HEADLINE: EVERY THUMBNAIL DECODE IS PERMANENT, AND THAT — NOT THE +115 % BINARY
> — IS THE LIVE CONSTRAINT.** chitra makes **31 `alloc()` calls**, `chitra_image_free` is a **no-op**
> (`return 0;`), and cyrius's `alloc` is a bump allocator whose only reclaim rewinds the whole heap.
> **Measured: ~2.5× the RGBA size per decode, never returned**, and a second decode of the same file
> costs it again. crab bounds it two ways and **both are load-bearing**: a per-image pre-check
> (`CRAB_THUMB_MAX_RGBA`, 4 MB — a refusal costs **16 bytes** against up to 26.6 MB unbudgeted) and a
> **session ceiling** (`CRAB_THUMB_TOTAL_MAX`, 32 MB), because a cap on one decode says nothing about
> a hundred.
> ⇒ **Those two constants are the first thing to delete** the day the allocator gains a `free()` or
> chitra takes an arena. They are not taste; they are the allocator's shape.
>
> ⭐ **The gate on the thumbnail line was false TWICE, in opposite directions.** First *"no image
> decoder exists"* (chitra 1.0.0 shipped one). Then the correction implied that made it free
> (+115 %). And the real constraint was in neither. ⇒ **A gate is a claim about another repository:
> it can be wrong about existence AND about price, and neither is visible from the line.**

> ### What is verified, and what is not
> Stated as fact, not as a recommendation. The last on-target run was **2026-08-30 against the 0.7.0
> tree**; everything since has run on the host and under QEMU only, and every
> `#ifdef CYRIUS_TARGET_AGNOS` region is invisible to the host suite by construction. agnos has moved
> 1.56.53 → 1.56.55. ⚠ **Sequencing verification is the operator's call and is not this file's to
> rank.**

## Cross-repo work, and where each repo stands

| repo | state |
|---|---|
| **dhancha 0.9.25** | ⭐ **RELEASED** (`2698ae1`, verified) and crab's tag moved to it. **0.9.25 adds the `GRID` kind** — wrapping layout, cell selection, row-wise arrows, keep-visible, hit-test; 66 checks, six mutations. 0.9.24 before it: Stable widget keys: dhancha identified widgets by *pointer*, which a per-frame arena invalidates — so focus, hover, press and drag were all unreachable from an immediate-mode app. Fixed at the cause. **Drag now works under a frame arena** (0.9.21 could only refuse it); `dh_text_attach` lets an app own the edit buffer. 16/16 suites, new `key_test` at 50 checks, six mutations. ⚠ **crab uses none of it yet** — adopting it deletes three hand-rolled workarounds and is its own change. |
| **chitra 1.0.1** | ⭐ **RELEASED** (`b777d34`, verified on the remote) and crab's tag moved to it. A valid PNG past sankoch's 16 MiB inflate ceiling used to spend **26,617,512 bytes** to return a bare `CHITRA_ERR_INFLATE`; it now refuses from the header in **under 64 KiB** with its own `CHITRA_ERR_INFLATE_LIMIT`. Its sidecar drops three unused stdlib leaves — **-4,176 B** in crab. 3,020 tests green, two mutations caught. ⚠ Changes no crab behaviour: the per-image budget already kept crab clear of that cliff. |
| **sankoch** | Issue filed: `DECOMPRESS_MAX_OUTPUT` is an absolute 16 MiB with **no caller override**, and the streaming API enforces it identically — so no entry point can inflate a ~5.6 MP RGB PNG. Only sankoch can change that. |
| **agnos** | Issue filed: `open`#7 has no `AO_EXCL`, so crab's overwrite guard exists on the host only. agnos's own `syscall.cyr:1138` already says so; crab is a second consumer. |
| **cyrius** | ⚠ **An issue I filed earlier this session was WITHDRAWN** — I claimed `cyrius distlib` copies `[deps].stdlib` into the sidecar. It does not; it derives it from `src/lib.cyr`'s includes, which is sound. The symptom was real, the cause I named was not. Corrected in chitra's own issue. |

## What landed (unreleased, on top of 0.7.5)

⚠ **`VERSION` is untouched, nothing is committed or tagged** — the operator drives all of that.
Full accounting in [`../../CHANGELOG.md`](../../CHANGELOG.md) under `[Unreleased]`.

| | |
|---|---|
| `crab_render` | **32 positional parameters → one record.** Filled by `crab_rs_pane` (11 params, indexed by pane), `_op`, `_chrome`, `_preview`, `_dims`. ⛔ The point is not brevity: at 32 `i64` arguments across 23 call sites, a miscounted comma shifted everything after it and still compiled. And `crab_rs_reset` now owns the three **`-1` = cannot be said yet** defaults that every one of those sites used to spell by hand — `0` there would make the tray render a real `0 B/s`. |
| Preview column | `p` toggles it. NAME · KIND · SIZE · MODIFIED · DIMENSIONS. ⛔ Width rule **derived** from crab's own column rule (*it may not cost a pane its SIZE column* → 303 px), not lifted from the canvas. Refuses out loud below that, via `crab_set_notice`. |
| Grid view | `g` toggles both panes onto dhancha 0.9.25's `GRID`. ⛔ Cell size DERIVED from the NAME column's floor, so the view changes only how many entries fit. No column header (a grid shows only names). ⛔ **Arrows navigate in grid mode; `h`/`l` still switch panes** — list mode unchanged. ⚠ **Not a gallery**: 40 thumbnail cells is ~28 MB of permanent decode against a 32 MB ceiling. |
| EXIF | CAMERA and SHOT, both byte orders, verified against an independent parser. ⛔⛔ **The most attacker-controlled parser crab has** — byte order, entry count and value offsets are ALL chosen by the file. Sub-IFD followed exactly once, never recursively. ⚠ Its fuzz round was **vacuous at first**: an out-of-bounds read does not crash on a bump allocator over a large mapped heap, so four planted bounds bugs survived. A printable poison tail fixed it — and the mutator had to be stopped from writing the poison byte itself. |
| Thumbnails | 64x64, PNG/JPEG/GIF/BMP, **decoded off the idle tick** — at most one per tick, because chitra's entry point is a single call that cannot be resumed the way `crab_copy_step` can. Memoised on the full path **including a remembered refusal**; a closed preview decodes nothing. ⛔ Four differently-named nothings, because "too large" is a property of the FILE, "budget spent" of the SESSION, and "cannot decode" of this BUILD. |
| Image dimensions | `crab_img_dims` — PNG/JPEG/GIF/BMP from header bytes, **no decoder, no dependency**. Verified against real files against `identify`: 137×42, 1×1, 4096×2160 PNGs, a 91×33 GIF, a 65×17 BMP and its top-down twin, and a real 384×288 JPEG whose SOF sits past an APP0 block. |
| Fuzz harness | ✅ **Deferral #12 CLOSED.** 60,000 deterministic rounds. It caught a real segfault in `crab_img_dims` the day it was written. |
| Leak fixed | ⛔ `crab_overlay` used `alloc(32)` not `dh_falloc(32)` on both the menu and sheet branches — **32 B per frame, shipped in 0.7.5**, invisible because the zero-allocation gate never opened an overlay. |
| Numbers | tests **757 → 1,057/0** · `render_test` **26 → 53** checks · coverage **86 %** (183/212) · source **5,368 → 7,513** lines · host **1,015,576 B** · `--agnos` **1,039,312 B** · fmt clean · both targets build |

### ⛔⛔ THE FOUR LESSONS FROM THIS SLICE, EACH CHEAP AND EACH EXPENSIVE TO RE-LEARN

**1. A gate that covers one state proves one state.** crab's zero-allocation assertion rendered
twenty frames with **no overlay open**, so `crab_overlay`'s two `alloc(32)` calls leaked from 0.7.5
until now with a green suite. The loop now runs with the menu open, the sheet open and the preview
open, plus a **non-vacuity arm** proving those branches were really entered. ⇒ **A new render-path
branch without an arm here is a new blind spot, not a covered feature.**

**2. A fuzzer must be shown to catch a bug you plant on purpose.** The new harness's own first draft
was vacuous: an LCG's low bits have period 2^k, so its format selector returned **only two of four
values across 20,000 rounds** — PNG and JPEG were never seeded and the JPEG walk ran **zero** times
while it printed `fuzz: ok`. Nothing but a planted bug would have said so. Sample the high bits.

**3. A bounds check proves an index is in range. It says nothing about which buffer.**
`crab_jpeg_dims` dereferenced its cursor as an **absolute address** (`load8(p)` for `load8(buf + p)`)
and segfaulted on the first marker — while every comparison of `p` against `len` was correct. That
class is invisible to a bounds review and instant to a fuzzer.

**4a. AN OUT-OF-BOUNDS READ DOES NOT CRASH IN THIS STACK, so "the fuzzer is green" is a weak
claim.** cyrius's allocator is a bump allocator over a large mapped heap: reading past a buffer
returns garbage rather than faulting. Detecting it needs a **poison tail** — and the poison must be
printable (the parser filters non-printables, so a NUL would be invisible) and must be excluded from
the mutator's alphabet (or a correct parser returns one and the detector fires on clean code).
⇒ **Guards whose bytes only reach control flow are undetectable this way, and two EXIF guards are
in exactly that position.** They are kept and labelled.

**4. `render_test` is where four of seven mutations were caught, and CI runs none of it.**
*Deferral #14* stopped being a tidiness item: a swapped pair of pane blocks, and a preview column
painted over panes that never gave up their width, both ship through CI green today. It at least
prints its check count now — it used to exit **0** whether it ran 26 checks or none.

### ⚠ Two things found and deliberately NOT changed

- ⛔ **`crab_fs_open_w` diverges between targets, and the shipping target is the permissive one.**
  Host: `O_WRONLY|O_CREAT|O_EXCL` — M4's overwrite guard, returns `EEXIST`. agnos:
  `AO_WRONLY|AO_CREAT|AO_TRUNC`, **no `AO_EXCL`** — it truncates. So every host assertion about
  "crab will not overwrite" is a claim about the host only. Pinned by a host assertion; changing
  write semantics is an operator call and a recursive copy is what would notice.
- ⚠ **The preview's dimension read is on the selection path, not the idle tick.** Memoised on
  (directory, name) and capped at 64 KiB, so it costs one open/read/close per newly-selected image
  and nothing otherwise — but arrowing fast through a directory of large JPEGs still pays per entry.
  The idle-tick stepping thumbnails would need is the same machinery that would move it.

---

> **Written 2026-08-26 at 0.5.0; rewritten 2026-08-27 at 0.6.0, then updated across M2 and M3; cut at 0.7.0 on 2026-08-28.** Read this, then [`CLAUDE.md`](../../CLAUDE.md), then
> [`state.md`](state.md), then [`roadmap.md`](roadmap.md).
>
> ⚠ **Refresh or delete this file when the next release ships. A stale handoff is worse than none.**
> crab's `state.md` has already rotted twice — once across five releases, once across eleven — and
> both times it was found by a human, never by a gate. **There is no gate for handoff staleness
> either.** Treat every number below as a claim to re-derive, not as evidence.

---

## Where things stand

| | |
|---|---|
| Version | **0.7.5** cut 2026-08-31 — **EVERY M4 ITEM IS IN**: the context menu, inline rename and the batch sheet, on top of 0.7.4's recursion, multi-select, rate and ETA. 0.7.3 carried the stepped copy, Esc cancel and the tray. 0.7.2 carried `open`, the small-window ratification and drag. **0.7.1** (tagged and pushed at `4ac21eb`) carried the five M3 defects, M4's write layer and the 6.5.36 pin. ⛔ **0.7.2 exists because 0.7.1's CHANGELOG section was still being edited after its tag was pushed** — three commits landed past it. A released section is not a scratchpad; it is now byte-identical to the tag. ⛔ **The operator handles all git operations: never commit, tag or push.** `VERSION` and the `## [x.y.z]` heading are the operator's to DIRECT — 0.7.1 was cut on their explicit instruction, not on this file's initiative. **Nothing is committed or tagged.** |
| Toolchain | cyrius pin **6.5.36** (moved at 0.7.1; the latest *release*). ⭐ This retired the hardcoded `CRAB_SYS_READDIR_AT = 101` at its written expiry — crab now calls `sys_readdir_at`, and is the **first consumer ever to call that wrapper**. ⛔ **THE LOCAL `~/.cyrius` SNAPSHOT IS NOT TRUSTWORTHY** — build through the released tarball; see *Verifying anything in this stack* below. |
| Build | x86_64 **453,304 B** · `--agnos` **474,944 B** at 0.7.5 · `--win` fails (pre-existing). ⭐ At the 0.7.0 tree a clean-tarball build reproduces **398,504 / 406,992** exactly, and the agnos binary is **byte-identical to the artifact that burned on iron** — so that artifact IS reproducible, contrary to an earlier claim here. |
| Tests | `cyrius test` **757 / 0** · `render_test` **26** checks, 0 failed (it was 14, never 15) · ⭐ **bench now measures the sort**, not `bench_noop` · fuzz still a scaffold that reads none of its input |
| Coverage | **137/159 fns (86 %)**, 6/6 files — **the v1.0 criterion, met.** It dipped to 73 % mid-cut and was recovered by writing the assertions genuinely missing (`crab_entry_cmp` directly, the result messages, the notice channel, the listing accessors), not by naming functions to move the number. ⛔ Still reference coverage — a floor, not a correctness proof. The roadmap now gates it per release. |
| Source | **5,368** lines across **six** files. ⚠ Re-derive with `wc -l src/*.cyr`; the per-file numbers that used to sit here went stale at two of the last three cuts, so they are deliberately not repeated. ⚠ Re-derive with `wc -l src/*.cyr`; this row has been stale at two of the last three cuts. |
| Deps | ⛔ **SIX, AND ONLY SIX**: rekha 0.3.5 · kashi 1.0.6 · sadish **0.5.3** · rupa **0.1.6** · dhancha **0.9.23** · setu 0.8.8. This row used to also list **agnos, bhumi, sigil and aethersafha** — none of them is a declared dependency (`grep '^\[deps' cyrius.cyml`); agnos is the *kernel* crab runs on and the others are peers. Misreading that row as the dep graph is how a tag check checks the wrong things. ⭐ **CHECK 4 RE-RUN 2026-08-31, AFTER the dhancha tag moved 0.9.20 → 0.9.21** (and at the 0.7.1 cut before it) — the manifest copied with all four `path` lines disabled, so `cyrius deps` actually cloned the declared tags: **6 deps / 0 errors** (lock goes 2 → 6 commit-pinned, which is the tell that the overrides were really off), host **and** `--agnos` build, **476/0** tests, and both binaries came out **byte-identical** to the path-resolved ones. That last equality is the evidence; the rest is consistent with it. |
| Mid-arc work | M3 is complete and its five review defects are **closed in 0.7.1**. M4 is **started, not done**: copy/move/delete and the pane states shipped; rename/mkdir are written and tested but not key-bound, and the **transfer tray SHIPPED in 0.7.3** — the copy is stepped off the idle tick, Esc cancels it, and the bar is dhancha 0.9.22's `PROGRESS`. ⛔ **The tray was never really gated on dhancha**: the blocker was crab's own synchronous copy, and the widget alone would have rendered a bar once at 0 % and vanished. ⚠ **Still open in M4**: recursive copy/delete (needs a walk, and the tray is now the surface that makes it stoppable), multi-select, rate/ETA, and the context menu / inline rename / batch-rename sheet. ⚠ agnos HEAD is **1.56.55**; the 2026-08-30 burn ran **1.56.53**. ⛔ **THAT IS NOT A crab GATE AND MUST NOT BE WRITTEN AS ONE.** 1.56.55 rewrote `is_user_range`, which is agnos's own validator on every ring-3 buffer in the system — if it regressed, it regresses for every app, and proving it is **agnos's** job on agnos's schedule, not a reason to hold a crab release. crab passes ordinary BSS/stack buffers and has no special exposure. |

### ⛔⛔ TWO HAZARDS THIS MILESTONE TAUGHT, BOTH CHEAP AND BOTH EXPENSIVE TO RE-LEARN

**1. A host build proves NOTHING about the event loop.** The whole key-handling region lives inside
`#ifdef CYRIUS_TARGET_AGNOS`, so a brace error there compiles **clean** on the host and fails only
on `--agnos`. It happened at 0.7.5. ⇒ **Build both targets, every time.**

**2. `ui.cyr` is BELOW `app.cyr`, and the render path must never reach up.** `render_test.cyr` and
the suite include `ui.cyr` **alone**, so a render-path call into `app.cyr` compiles through
`main.cyr` and leaves `render_test` with undefined symbols. This happened **three times in one
milestone** — the pane state, the mark predicates, and the edit buffer — each fixed by moving the
code down rather than by threading a parameter. ⇒ **Anything the render path touches lives at or
below `ui.cyr`.**

⚠ **And a third, upstream:** dhancha has now had **three** features an immediate-mode app cannot use
— `dh_dispatch` (press as a widget pointer), drag (`_dh_drag_src`), and `TEXTINPUT` (a per-widget
buffer on an arena'd widget). Each assumes a **retained** tree. crab works around all three by owning
its own state, which is correct for crab; **it is worth telling dhancha that the pattern is a
pattern.**

### ⛔⛔ RELEASE ORDER, LIVE RIGHT NOW — FOUR REPOS, AND crab IS LAST

**crab does not build from a clean checkout until sadish and rupa are pushed.** That is expected and
correct, not a break: crab declares `sadish 0.5.3` and `rupa 0.1.6`, dhancha's fold calls
`sd_fill_rect_blend` and `rupa_theme_scrim`, and neither tag exists on a remote yet.

**Push in this order, each verified before the next:**

| # | repo | version | why it moved |
|---|---|---|---|
| 1 | **sadish** | 0.5.3 | `sd_fill_rect_blend` — the first fill that reads its destination |
| 2 | **rupa** | 0.1.6 | the `scrim` colour + alpha token, per palette |
| 3 | **dhancha** | 0.9.23 | MENU + SHEET; the scrim becomes a real veil instead of a dither |
| 4 | **crab** | 0.7.4 | consumes all three |

⭐ **The chain was verified end to end** with a TEMPORARY `path` override on crab's sadish dep:
6 deps / 0 errors, both targets build, **565 / 0**. The override was then **removed** —
⛔ **do not add one permanently.** sadish and rekha are the only two deps crab resolves by tag alone,
which makes them the only two whose remote resolution a local build actually exercises. A path
override there would delete that property, and it is the one thing standing between this manifest and
the 2026-08-28 phantom-tag failure.

### ✅ RELEASE ORDER HONOURED — dhancha 0.9.21 and crab 0.7.2 are both pushed (2026-08-31)

⭐ **Sequenced correctly, and check 4 re-run after the bump.** dhancha `0.9.21` and crab `0.7.2` are
tagged on their remotes; crab's declared `tag` moved 0.9.20 → **0.9.21** only once that tag existed,
and the remote SHA (`d75be97`) matches the sibling's. Check 4 with all four `path` lines disabled:
**6 deps / 0 errors**, both targets build, **476 / 0** tests, and both binaries **byte-identical** to
the path-resolved ones.
⚠ The section below is kept because the hazard is permanent, not because it is currently live.

### ⛔⛔ THE RULE THAT PRODUCED THAT ORDER — RE-READ IT BEFORE THE NEXT CROSS-REPO CHANGE

dhancha was fixed to **0.9.21** on 2026-08-30 (drag no longer half-fires under a frame arena).
crab's manifest still declares **`tag = "0.9.20"`**, and that is **deliberate, not an oversight**:

- ⛔ **Declaring a tag that exists on no remote is the exact failure of 2026-08-28** — crab named
  `rupa 0.1.5` and `dhancha 0.9.20` before either was pushed, `path` masked it locally, and check 4
  failed with *"Remote branch not found"*, `4 deps resolved, 2 errors`. **No consumer could resolve
  dhancha at all.** Do not repeat it: **release first, then bump, then re-run check 4.**
- ⚠ **`lib/dhancha.cyr` in this tree is 0.9.21 content while the manifest says 0.9.20**, because
  `path = "../dhancha"` wins over `tag` and every `cyrius deps` re-vendors the sibling's `dist/`.
  Reverting it is futile — the next build brings it back. It is harmless today because **crab uses
  nothing from 0.9.21**: crab does not call `dh_drag_available`, and does not use `dh_dispatch` at
  all (ruling 2026-08-27). CI resolves 0.9.20 and builds green.
- ⇒ **Order: push dhancha 0.9.21 → bump crab's `tag` to 0.9.21 → re-run check 4** (the manifest
  copied with every `path` line disabled). Until then crab is correct as it stands.

⛔ **AND AFTER ANY `cyrius distlib` IN dhancha, RUN `sh scripts/sync-deps-sidecar.sh`.** Raw distlib
writes `kashi_font_data` into `dist/dhancha.deps` as if it were a stdlib leaf; it is vendored, so
`cyrius deps` then fails in **every** consumer with *"dep dhancha requires 'kashi_font_data'"*.
crab's build broke exactly this way during the 0.9.21 work. The sidecar's own header says so.

### ✅ THE FIVE DEFECTS ARE CLOSED (2026-08-30) — kept below for the reasoning, not as open work

Found by a full review of the M3 gate work. All five were in code that **built and passed 253/0**,
which is exactly why the suite did not find them. **All five are now fixed** — see the CHANGELOG's
`[Unreleased]`. The descriptions are kept because the reasoning is still the reasoning; the list is
no longer a work queue.

⛔ **TWO CORRECTIONS TO THIS LIST, BOTH FOUND WHILE FIXING IT. Read them before trusting a review.**

1. **Defect 3's prescribed fix — "Guard on `cur`" — DOES NOT WORK.** `agnos/kernel/core/ext2.cyr:2412`
   *parks* the cursor on the record it declined to take (`store64(cursor_uva, blkbase + off)`)
   whenever the batch budget is reached; only `:2440` (walked off the end) and `:2392`
   (`start >= dir_size`) ever store `-1`. So at exactly the cap the final batch fills, the cursor is
   parked rather than exhausted, and a cursor test **still reports the false truncation**. The
   COUNT is the oracle: the counting walk resumes from that parked cursor, takes `:2392` at once,
   and adds nothing — leaving `crab_dir_total == n`.
2. **Defect 1's ranking rationale was wrong.** The review called the counting loop "the worse of the
   two" because the listing loop has an `n >= CRAB_MAX_ENTRIES` escape. **That escape can never fire
   on a stalled cursor** — `n` advances only by `n = n + k`, and on a stall `k` is the zero. Both
   loops were equally unbounded. There was no safer half.

⚠ **And the review under-counted the stale comments**: defect 4 says three sites; there are **seven**,
plus two of a different class (`src/main.cyr` describing listing as `#81` when both live call sites
are `#101`), plus the arena comment in `src/ui.cyr`, which was stale for a **second, undocumented
reason** — *#32* took a row from one widget to 1 + `ncols`, so the frame chains **7 chunks** at the
shipped window, not the one the comment promised. Measure that with `arena_capacity_total`;
`arena_used` reports the current chunk only and shows 13,104 B for a 2.6 MB frame.

1. **Neither `#101` loop terminates on a stalled cursor — a hang, not a crash.**
   `src/app.cyr:566` (the listing walk) and `src/app.cyr:587` (the counting walk) both loop
   `while (load64(&cur) != -1)` and break only on `k < 0`. But the kernel's I/O-error paths —
   `agnos/kernel/core/ext2.cyr:2329` and `:2333` — `store64(cursor_uva, pos)` with **`pos`
   unchanged** and `return count`, which may be `0` and is **not negative**. A persistent
   block-read failure therefore spins crab forever in a syscall loop, silently. Neither loop bounds
   its iterations. ⇒ Break when a call returns `k == 0` with the cursor unmoved, and cap the
   iteration count.

2. **The cap went 256 → 1024 and re-broke the keystroke path that M3 *#03* existed to fix.**
   `crab_sort_entries` (`src/app.cyr:427`) is insertion sort — O(n²) with a 64-byte record swap done
   byte-at-a-time. **Measured on native x86_64** (agnos under QEMU is far slower):

   | n | random order | reverse-sorted |
   |---:|---:|---:|
   | 256 | 6 ms | 14 ms |
   | 1024 | **100 ms** | **200 ms** |

   It runs once per listing — i.e. on every descend/ascend keypress. *#03* treated ~280 ms there as
   unacceptable and restructured statting to remove it; this quietly put it back. ⛔ The comment at
   `src/app.cyr:424` still reads *"`CRAB_MAX_ENTRIES` is 256"* — the justification was never
   re-derived at the new size.

3. **A directory of exactly 1024 entries reports a false truncation.** `src/app.cyr:601` guards the
   warning on `n >= CRAB_MAX_ENTRIES`, but the comment directly above it says the oracle is `cur`,
   not `n` — *"a directory of exactly the cap is complete, not truncated"*. With exactly 1024
   entries the walk ends with `cur == -1` and `crab_dir_total == n`, so the `crab_dir_total > n`
   branch is false and it prints `has more entries than are shown`. Guard on `cur`.

4. **Three stale cost comments the cap bump invalidated.** `src/app.cyr:424` ("is 256", above);
   `src/app.cyr:647` — *"at the cap of 256 that is ~280 ms of blocking syscalls"*, now ~1.1 s at
   1024; `src/ui.cyr:385` — *"256 entries per pane"* for the 256 KiB arena sizing. The arena is
   GROWable so it degrades safely, but the measurement is no longer the one stated.

5. **`crab_name_cell` scans kernel data with an unbounded `strlen`.** `src/ui.cyr:205`:
   `while (load8(name + n) != 0) { n = n + 1; }`, over a readdir record. It is safe *today* only
   because the kernel NUL-terminates at ≤ 62 bytes — an invariant asserted nowhere at that call
   site. `src/path.cyr`'s own header exists because unbounded copies over exactly this data caused
   the 0.5.0 P-1. Bound it by `CRAB_NAME_MAX` like `crab_strcpy_n` does.

⚠ **Not a defect, but know it:** at the shipped default 380x220 a pane is ~187 px, so
`crab_cols_for_width` returns **2** — NAME + SIZE. The MODIFIED column never appears at the default
window size, though the README/CHANGELOG headline is "NAME · SIZE · MODIFIED". It is disclosed in
the ⚠ lines; just do not expect to see a date on first run.

### ⛔⛔ Verifying anything in this stack — the traps that cost a whole session (2026-08-28)

1. **`cyrfmt` is NOT `cyrius fmt`.** The raw `~/.cyrius/versions/<v>/bin/cyrfmt` **silently ignores
   `--check`**: it prints the file and exits **0**, so every file "passes" vacuously. CI runs the
   `cyrius` *wrapper*. Always `cyrius fmt <f> --check`, never the raw binary. And never run bare
   `cyrius fmt <f>` while diagnosing — it rewrites in place.
2. **A versioned wrapper still delegates to whatever is first on `PATH`.** Calling
   `~/.cyrius/versions/6.5.27/bin/cyrius` does **not** give you 6.5.27's formatter; it gives you the
   PATH one. Formatter rules genuinely differ — 6.5.27 wants continuation lines at the **statement's
   own indent**, 6.5.35 wants **+2 per open paren**. Getting this backwards produced a "fix" that
   failed CI.
3. **The only faithful reproduction is the released tarball.**
   `curl -sSfL https://github.com/MacCracken/cyrius/releases/download/<v>/cyrius-<v>-x86_64-linux.tar.gz`,
   extract, then `CYRIUS_HOME=$D PATH=$D/bin:$PATH $D/bin/cyrius …` (create `$D/versions/<v>/{lib,bin}`
   symlinks or `cyrius deps` refuses).
4. ⛔ **`~/.cyrius`'s 6.5.35 stdlib snapshot was overwritten with 6.5.36 content** (2026-08-28
   08:25). The released 6.5.35 tarball has **0** hits for `v6.5.36` and **0** for `SYS_READDIR_AT`;
   the local copy has both. **Consequence: every `cyrius build` / `cyrius test` here re-vendors
   6.5.36 stdlib into crab's tracked `lib/` and rewrites `cyrius.lock`.** After any build, check
   `git status lib/` and `grep -c SYS_READDIR_AT lib/syscalls_x86_64_agnos.cyr` (must be **0**);
   `git checkout -- lib/ cyrius.lock` to undo. Fix the root cause with
   `curl -sSf https://raw.githubusercontent.com/MacCracken/cyrius/main/scripts/install.sh | CYRIUS_VERSION=6.5.35 sh`
   — **ask the operator first**, they may be developing 6.5.36 deliberately.
5. **Ground truth for "is this state CI-canonical" is the GitHub Actions API**, not a local run:
   `curl -sS https://api.github.com/repos/MacCracken/<repo>/actions/runs?per_page=5` gives
   sha/conclusion, and `…/runs/<id>/jobs` gives per-step outcomes. Find the last **green** run and
   diff against its SHA. ⚠ Job *logs* need admin auth (403); run and step conclusions are public.
   ⚠ Do **not** use the `gh` CLI — `curl` only.
6. **`git ls-remote` works fine here.** An older note in this file called it "inconclusive due to SSH
   auth in the sandbox" — that was not true on 2026-08-28; it resolved every tag. Use
   `sed 's|.*refs/tags/||' | sort -V` — **`sort -V`, not lexical**, or `1.56.9` outranks `1.56.50`.

### ⛔ One deliberate interim, with an expiry

**crab hard-codes syscall `101`** (`CRAB_SYS_READDIR_AT` in `src/app.cyr`). That is the bug class
agnos's roadmap tracks, and it is temporary on purpose: cyrius **6.5.36** ADDS `SYS_READDIR_AT` +
`sys_readdir_at`, but crab pins **6.5.35**, whose vendored `lib/` has no wrapper — and `lib/` must
not be hand-edited. **Switch to the wrapper and delete the constant when crab's pin moves to
>= 6.5.36.**
⛔ **6.5.36 IS UNRELEASED — no tag, a local commit only** (verified 2026-08-28; the latest cyrius
release is **6.5.35**). CI installs *releases*, so the pin CANNOT move there and the wrapper does not
exist for any consumer. ⚠ Do not be misled by a local `~/.cyrius` that already carries it: on
2026-08-28 the installed 6.5.35 stdlib snapshot had been overwritten with 6.5.36 content, so
`lib/syscalls_x86_64_agnos.cyr` briefly showed `sys_readdir_at` here while a clean checkout had
nothing. Re-check against the released tarball, not the local snapshot.
Filed on both sides:
`agnos/docs/development/issues/2026-08-28-syscall-101-readdir-at.md` and
`cyrius/docs/development/issues/2026-08-28-cyrius-syscall-101-readdir-at-wrapper.md`.

⚠ **The cyrius wrapper is asserted to exist and compile; it has never been CALLED.** agnos's
`rdat.cyr` proves the kernel contract but uses the raw number. crab is the first consumer that will
exercise the path through the wrapper, and it cannot until the pin moves.

### ⛔ Read this before touching the column or listing code

- **A column spec built per frame is a leak.** `alloc` is a bump allocator with no `free()`, so
  `dh_cols_new` inside a render path retains 40 B per pane per frame. This was caught by the existing
  zero-allocation assertion at **1600 B over twenty frames** — reopening the gate dhancha
  0.9.13-0.9.16 and crab 0.6.0 spent four releases closing. Use `dh_cols_reset` on a spec allocated
  once.
- **Hardcoded pixel coordinates in tests encode the layout.** Adding the column header moved every
  row down and silently invalidated the literals in both the click test and `render_test`; they now
  read the row's position from the laid-out widget.
- **`rt_row_has` samples ONE scan line.** Fine for a filled rectangle, unreliable for text — whether a
  glyph lights a given row depends on the letterform. Text assertions use `rt_band_has`.
- **A cross-repo mutation must target `dist/`, not `src/`.** crab compiles `dist/dhancha.cyr`;
  mutating dhancha's `src/` without `cyrius distlib` changes nothing and the "mutation" silently
  passes.
- **`on-accent` is deliberately the same value as `bg` on MUDRA dark** (`0x0B0C0E`). "This pane has no
  on-accent pixels" therefore cannot be written as a pixel check — it finds background.

### ⭐ The dep graph was verified four ways at this cut — only the fourth is evidence

crab's manifest sets `path = "../X"` for **rupa, kashi, dhancha and setu**, and **`path` wins over
`tag`**. So a green local build says nothing about whether the declared tags resolve — this has been
the dominant recurring hazard across this stack.

1. Sibling `VERSION` == declared tag — all six ✅
2. `git rev-parse <tag>` == that sibling's HEAD — all six ✅
3. The tag exists on the remote with the same SHA — checked with `git ls-remote --tags` (**which
   works**; an older revision of this file claimed it fails on SSH auth in the sandbox — it did not
   on 2026-08-28) or `curl` against the GitHub API. ⚠ Sort with **`sort -V`**, never lexically.
4. ⭐ **The manifest copied with every `path` line disabled, so `cyrius deps` actually cloned the
   tags.** Both targets built, all 228 tests passed, and the binaries came out **byte-identical** to
   the path-resolved ones.

⛔ **Checks 1–3 can all pass while the declared graph is broken.** Only 4 exercises it.

⭐ **RE-RUN 2026-08-28, and this time check 4 was the one that mattered.** Between the 0.7.0 cut and
that date the manifest named **`rupa 0.1.5` and `dhancha 0.9.20`, neither of which existed on any
remote** — both were local-only. Checks 1–3 looked fine locally because `path` wins; check 4 failed
outright: `fatal: Remote branch 0.1.5 not found in upstream origin`, `4 deps resolved, 2 errors`.
dhancha 0.9.20 was broken the same way (it pinned the same phantom `rupa 0.1.5`), so **no consumer
could resolve it either**. After the operator released rupa `0.1.5` (`27e8385`) and dhancha `0.9.20`
(`61a1e39`), check 4 was re-run with every `path` line disabled: **6 deps / 0 errors, host and
`--agnos` both build, 253/0 tests.** That is the current evidence.

⚠ **A `## [x.y.z]` CHANGELOG heading is not a release, and neither is a local tag.** Four repos
carried headings for versions that had never been pushed. The only proof is
`git ls-remote --tags <url> | sed 's|.*refs/tags/||' | sort -V | tail`.

---

## ✅ The allocation gate is CLOSED — a rendered frame costs the global heap zero bytes

| | per steady-state frame |
|---|---:|
| baseline (dhancha 0.9.12) | **746,440 B** |
| + step 1 (0.9.13, published `c273159`) — `dh_surface_new`'s dead pixel buffer, deferred | 412,040 B |
| + step 2 (0.9.14, published `b228a8b`) — the sadish render target, reused | 77,568 B |
| + step 3 (0.9.15, published `935a84c`) — the widget tree, arena'd | **0 B** |

Measured with a host probe taking `alloc_used()` deltas around back-to-back `crab_render` calls at
380×220, into one surface and one arena — the lifetimes `src/main.cyr` uses. **Identical at 114
entries per pane (the real iron count for `/`) and at 256, the `CRAB_MAX_ENTRIES` ceiling.**

⚠ **Zero is per-frame, not total.** crab pays a one-time **~597 KB** — the 334,432 B render target
plus the 262,144 B arena chunk — allocated on the first frame and reused for the process's life. A
fixed cost instead of a per-keypress one is the whole point.

**Step 3** adds `dh_frame_arena_set` / `dh_falloc` / `dh_frame_begin` to dhancha. Only the two
genuinely per-frame allocations move onto the arena — `dh_widget_new` and layout's measure scratch.
Surfaces, the setu shared buffer, event records and queues, textinput buffers and canvas surfaces stay
on the global allocator because a caller holds them across rewinds. crab installs a **growable** arena
in `src/main.cyr`, and `crab_render` calls `dh_frame_begin()` at the top so every caller is correct
without remembering.

⛔ **`dh_frame_begin` DOES TWO THINGS AND THEY CANNOT BE SEPARATED.** `_dh_focus`, `_dh_hover`,
`_dh_press` and `_dh_drag_src` are raw widget pointers dhancha holds across calls; after a rewind they
address memory the arena is about to hand out again, so `dh_focus_within` would walk recycled parent
links. **Never call `arena_reset` on a frame arena directly.** ⇒ It follows that **crab must
re-establish focus every frame**, which `crab_pane` does. Cross-frame widget identity and a per-frame
arena are mutually exclusive by construction.

⛔ **Anything added to the render path must allocate through `dh_falloc`, not `alloc`.** One plain
`alloc()` in `crab_render` reintroduces a per-frame leak. This is a *lifetime* requirement too, not
just a budget one: `dh_widget_set_text` stores the pointer and does not copy, so a row's display
string must die with its widget.

**Verified**: dhancha 11/11 suites, lint + fmt clean, `vet` 0 untrusted, dist in sync; crab
`cyrius test` **55/0**, `render_test` 0 failed checks, host 381,608 B, `--agnos` 381,672 B, re-staged.
All three steps mutation-verified on both sides.

## M2 — started. *Deferral #09* is closed: the loop allocates nothing

**dhancha 0.9.16** (`68c60f8`, tagged and consumed). `dh_setu_poll_event` opened with `setu_msg_new()` — an
80-byte `alloc` — and only *then* asked whether a frame was pending. With no `free()` under it, an
idle desktop grew the heap once per wakeup forever, and a client repainting without input grew it at
the repaint rate: ~4.8 KB/s at 60 Hz.

The message is pure scratch — `setu_client_poll_input` fills it and `dh_setu_map_input` reads it to
build a **separate** `DhEvent` — so it is now one hoisted buffer per process, handed out zeroed.

⭐ **This was the last unbounded per-cycle allocation.** 0.9.13–0.9.15 took a rendered frame to zero;
this is the other half. **crab's whole render/input loop now allocates nothing in steady state**, and
the "no continuously-repainting element" rule is lifted **outright** rather than moved — the idle
mascot line (*deferral #29*), M4's transfer tray and M7's index progress are unblocked.

⚠ An **event** still costs 56 B (`dh_event_new`). That is per input, not per cycle, and bounded by how
fast a human types.

⚠ **NOT on the frame arena, and the trap is worth naming**: routing the scratch through `dh_falloc`
would be wrong, because polling happens in the event loop while `dh_frame_begin` rewinds the arena
inside the caller's render — the scratch would be freed under a loop still using it. Per-frame and
per-poll are different lifetimes.

**Verified on the host**: dhancha 11/11 suites; `poll_test` gains 8 checks including **200 idle polls
moving the global heap by exactly 0 bytes**, with a non-vacuity arm proving a client that *does* have
frames still costs something. Mutation-verified: reverting to per-call allocation fails. crab
`cyrius test` 75/0, `render_test` 0 failed.

⭐ **Verified on a real agnos kernel in QEMU (2026-08-27)**, because the poll is inside the event loop
and the ⛔ below says no loop change may be claimed without one. `puka-terminal-test.py` — **PASS**,
background exit **95**, both clients connected, **2 presentations**, and the serial log carries
`crab: dual-pane file-manager UI presented over setu` followed by
`crab: compositor closed the window -- exiting`. So crab still connects, presents and leaves cleanly
*through the changed poll*.
⚠ **What that run does NOT show**: this harness has the compositor close crab's window quickly, so —
exactly as the open-items section below has said since 0.5.0 — **it cannot distinguish loop-lifetime
behaviour**. It is evidence the poll change did not regress connect/present/close, and nothing more.
⛔ **One assertion in that group is honestly labelled as NOT pinning what it looks like.** The
"reused scratch is handed out clean" checks cannot fail — `dh_setu_map_input` maps `SETU_CLOSE` with a
literal `a = 0`, and every kind that reads an arg has it guaranteed by `setu_decode`'s argc check. The
zeroing is **unexercised defence**, measured not assumed, and both the test and the source say so.

---

## ⚠ Resize — built, and a new harness found a real bug in it on the first run

`WINDOW_CONFIGURE` has reached apps since dhancha 0.9.12 and crab dropped it for five releases, so a
maximise grew the window and not the file manager. It is now handled — **dhancha 0.9.17** adds
`dh_surface_resize` (which also closes a latent overflow 0.9.13 created, and makes 0.9.14's dormant
dimension check live), and crab acts on the event.

### ⛔ What the harness caught, and it would have shipped otherwise

`agnos/scripts/harness/crab-resize-test.py` — **new**, and the only harness that both starts crab and
leaves it running. It drives boot → `aethersafha` → **F2 → DOWN → Enter** → F5. The DOWN is
load-bearing: the launcher registry is `/bin/puka` at 0 and `/bin/crab` at 1, and `lnch_openp` resets
the selection to 0, so a harness without it launches puka and scores whatever puka did.

**First run: crab died.** `crab: buf_create failed on resize -- exiting`. The draft closed its only
shm buffer before knowing the replacement existed — it destroyed a working surface to attempt an
upgrade. setu's own `setu_client_present` closes first, but it has an **inline-pixel fallback** to
land on; crab's hand-rolled LIVE-buffer path has none, so copying that order was fatal.
⇒ **Create before close**, and a failed create now leaves the old buffer and the old extent in place.

**And the byte cap was invented, not derived.** It was 16 MB, taken from "the framebuffer's own
size". agnos actually caps a `#71` pmm slot at **2 MB** and only a real GPU carveout (`#86`) reaches
**32 MB** — and `setu_buf_create` picks between them at runtime, so a client cannot know which
applies. ⇒ The cap is now the absurdity bound and **the kernel is the arbiter**.

### What is proven, and what is not

⭐ **PARTIAL PASS, and it is recorded as partial.** crab launches from the launcher, presents,
receives a real CONFIGURE for **2048x2018** (~16.5 MB), refuses it because QEMU has **no GPU
carveout** so only the 2 MB pmm slot is available, stays at its old extent, and **answers 6
keystrokes afterwards**.

- ✅ **The refusal path is proven** — which is exactly the bug the first run found.
- ⛔ **The adopt path is NOT proven and cannot be here.** It needs a machine whose `#86` carveout can
  back the ask. **Do not read the PARTIAL as a pass for resize working.**
- ⭐ **This also settles the loop-lifetime question open since 0.5.0.** Six answered keystrokes, well
  after launch, on a live desktop — no existing harness both started crab and left it running, and
  "no exit line" was never evidence. An ANSWER is.

⭐ **AND IT HAS BEEN PROVEN TO GO RED.** That matters more than usual here, because under QEMU the
honest outcome is a *refusal*, so "did not resize" is ambiguous unless the log separates the two.
Measured against the same image with only the binary changed:

| build | serial | verdict |
|---|---|---|
| real crab | `cannot back a surface of 2048x2018`, 6 keys answered | PARTIAL (rc 0) |
| `WINDOW_CONFIGURE` branch removed | no CONFIGURE line at all | **FAIL** (rc 1) |

⇒ The discriminator is that crab **saw the ask and said so**. A build that ignores the event is
silent, and silence is what the harness scores red.

⚠ **The harness is flaky by nature and says so.** QEMU drains HID once per frame, so a burst that
lands between drains is gone: the key-delivery probe measured 3/8 on one run and **0/8** on the next
against the same image. It now retries the probe four times and the launch six, and returns
**INCONCLUSIVE** rather than a verdict when nothing was delivered — a harness that scores a pass for a
test it never performed is worse than none.

⚠ It lives in `agnos/scripts/harness/` beside its siblings; *deferral #16* (move crab's harnesses into
crab's own repo) is still open.

---

## ✅ Pointer input — and the ruling that shaped it

⛔ **crab OWNS ITS INTERACTION STATE. `dh_dispatch` is deliberately NOT used** (operator ruling
2026-08-27). `dh_dispatch` tracks a press by storing a **widget pointer** (`_dh_press`,
`_dh_drag_src`, `_dh_hover`) and matching it on release. crab rebuilds its whole tree every frame and
renders after **every handled event**; `crab_render` opens with `dh_frame_begin()`, which rewinds the
arena and clears exactly those pointers. A press and its release are separated by a rebuild, so the
target no longer exists.
⚠ Not a dhancha bug — it is 0.9.15's own rule (*cross-frame widget identity and a per-frame arena are
mutually exclusive by construction*) meeting a feature that **is** cross-frame widget identity.
⇒ crab tracks **pane index + row index**, which survive a rebuild because they are its own model.
dhancha supplies geometry only, through `dh_hit_test` over the tree the last render built — valid
because those pointers are refreshed by the same render and read only between renders.

**Shipped**: click to select, click to focus a pane, double-click to descend (400 ms, monotonic
`clock_now_ms`). ⭐ **QEMU-proven**: `crab: click` on a real kernel, a click resolved to a pane, and
5 keystrokes answered afterwards. crab is the **first client to decode `SETU_INPUT_PTR_MOVE`** —
aethersafha's own note says no shipped client did, so this wire had never run end to end.

⚠ `SETU_INPUT_PTR_BTN` carries **no coordinates**, only button and state, so position comes from
`PTR_MOVE`. A client that ignores motion has nowhere to put a click.

### ⚠ Two things NOT done, and not claimed

- **Scroll wheel.** setu has no wheel message kind; `PTR_BTN` carries a button code and X11's 4/5
  convention is not in the protocol. Claiming it would be inventing a wire. **Gate: setu.**
- **Clicking a pane header to focus it.** The header is a sibling of the list, not inside it, so the
  hit walk never reaches a LIST. Reasonable to want; simply not built.

⚠ **M4's drag-between-panes is governed by the same ruling** — `DRAG_START`/`MOVE`/`DROP`/`END` all
route through `_dh_drag_src`, so it must be built on crab's model too, not on dhancha's.
⚠ **#07's KEY half remains safe** (`dh_dispatch` routes KEY to `dh_focus_get()`, re-established every
frame by `crab_pane`); adopting it wholesale is not, because the LIST would keep a selection on a
widget destroyed each frame — which is why `sel_l`/`sel_r` are app state.

---

## ✅ And `src/main.cyr` is testable — the gap that hid two of the above

`main.cyr` ends in `_entry();`, so a suite that included it would run the app. Everything in it was
therefore **unreachable from any test**: the readdir parser, the stat layer, `crab_descend`,
`crab_ascend`, the premultiplied surface flag. In a program whose two shipped defects were both found
on iron. `src/path.cyr` was carved out of the same file at 0.5.0 after a P1 memory-safety repair
landed where the suite could not see it; **`src/app.cyr` finishes that extraction** and `main.cyr` is
now `main()` and `_entry()` and nothing else.

⭐ **The frame-arena setup was MOVED rather than tested around.** It used to be created and installed
in `main()`, where deleting `dh_frame_arena_set` broke no test while quietly restoring a
77 KB-per-frame leak. `crab_render` owns it now — created on first use, installed if nothing else is,
rewound every frame. There is no setup step left for `main()` to forget, which is a better answer than
an assertion would have been.

⚠ **The residual gap is irreducible and is not pretended away**: `main()` itself still cannot be
called from a suite that would then run the app. It is now down to the **event loop alone** — and that
loop is already the thing `state.md` says needs a QEMU run before any claim about it.

**Result**: 37 → **75** assertions, reference coverage **53 % → 70 %** (19/27 fns, 6/6 files), against
a v1.0 criterion of 80 %. Seven mutations against the newly reachable layer, each producing a named
failure — including "`crab_render` stops installing the arena", which is the original gap.

---

### ✅ The dependency is published and the manifest was moved honestly

`[deps.dhancha] tag` is **0.9.15**, verified four ways — sibling `VERSION`; `git rev-parse 0.9.15` ==
HEAD (`935a84c`), tree clean; `git ls-remote --tags` at that commit; **and `path` disabled so
`cyrius deps` actually cloned the tag**, giving `lib/dhancha.cyr` = `7b99ec62…`, identical to the
`path` build and to `git show 0.9.15:dist/dhancha.cyr`.

⛔ **Only the fourth is evidence**, and finding #1 below is why it matters even more than "path wins"
suggested: `lib/` is not what compiles at all. The first three would each have passed 0.4.13.
Re-run all four at every cut; automating it is deferral **#19**, still open.

### ⭐ The repaint rule is LIFTED — and immediately replaced

"Do not add a continuously-repainting element" stood for three releases and was right at 45 MB/s. The
frame is free now, so the idle mascot line (*deferral #29*), M4's transfer tray and M7's index
progress are no longer blocked by it.

⛔ **But `dh_setu_poll_event` still calls `setu_msg_new()` BEFORE it knows whether anything is
pending** — ~80 B per poll, on the global heap, never reclaimed. Continuous repaint implies continuous
polling, so at 60 Hz that is ~4.8 KB/s of permanent growth. Four orders of magnitude better than what
it replaces, and still unbounded. **Closing *deferral #09* (M2, gate: dhancha) is now the precondition
for anything that repaints without input.**

### ⚠ Three findings worth carrying forward

1. ⛔ **`lib/` IS NOT WHAT COMPILES.** Measured by appending garbage: crab's `lib/dhancha.cyr` → build
   still green; `../dhancha/dist/dhancha.cyr` → build **fails**. The `path` override compiles the
   **sibling's `dist/`** directly, and crab's committed `lib/dhancha.cyr` is a record that
   `cyrius deps` refreshes and `cyrius.lock` hashes. `lib/alloc.cyr` is also inert — the stdlib comes
   from the **installed toolchain**. ⇒ `state.md`'s Toolchain note about `cyrius lib sync` describes
   the snapshot, not the compiler input, and **the stdlib's arena internals cannot be mutation-tested
   from this repo**.
2. ⛔ **Two convergence tests were worthless in their first draft, and only mutation said so.** crab's
   used `main.cyr`'s 256 KiB arena against a 3-entry fixture, so twenty frames fitted with room to
   spare and deleting `dh_frame_begin()` outright left the suite green. dhancha's group E claimed to
   "force the chain to extend" against a 256 KiB arena with ~60 KiB of widgets — it never grew at all.
   ⇒ **An arena test needs an arena sized to about one unit of work**, or it proves only that the
   arena is big.
3. ⚠ **`cyrius distlib` MUST be followed by `sh scripts/sync-deps-sidecar.sh`** in dhancha — distlib
   re-adds `kashi_font_data` to the sidecar and every consumer then hard-fails `cyrius deps`. I hit it
   again this session, and separately found `dist/dhancha.cyr` had gone stale against `src/` — caught
   only by running dhancha's own CI gate by hand. Run that gate before tagging.

⚠ **Owed at the 0.6.0 cut**: crab's CHANGELOG has no entry for any of the three steps. `VERSION` is
still 0.5.0 and this is mid-arc, so per CLAUDE.md's Process the entry and version sync land with the
release. Do not let that be the reason it goes unrecorded.

⚠ **The probe is still a scratch harness**, so the 0-bytes-per-frame figure is not re-derivable by
anyone else — though `tests/crab.tcyr` now *gates* it, which is the more important half. A real bench
harness is deferral **#13**; `tests/crab.bcyr` still times `bench_noop`.

---

## What 0.6.0 did

Two structural things, **no user-visible features** — crab looks and behaves exactly as 0.5.0 did.
The allocation gate closed (746,440 → 0 B per frame, across dhancha 0.9.13/0.9.14/0.9.15 and the
matching crab halves), and `src/app.cyr` was extracted so `main.cyr`'s contents could be tested at
all. Full accounting in the [CHANGELOG](../../CHANGELOG.md).

⚠ **0.6.0 took the version number the roadmap had reserved for M2.** None of M2 shipped. M2 is now
**v0.6.1** — a patch, absorbed inside the 0.6 line, so the ladder from M3 onward (v0.7.0 … v1.0.0) is
**unchanged**. Operator ruling 2026-08-27. ⛔ Re-derive the number at each cut rather than trusting a
roadmap heading; the milestone→version mapping has now been wrong once and nothing gates it.

---

## What 0.5.0 did

A P-1 audit of the whole codebase plus its repairs, the deferral tail bubbled into a real roadmap,
the design canvas sequenced to 1.0, and two ADRs. **No new user-visible features** — it is the
release that stops building on a floor with holes in it.

Two P1s repaired: the path helpers had no bounds (ordinary Enter presses overflowed `pathscr` and
each pane's path into the *other* pane's buffers), and the event loop was a spin count that ended
sessions after ~2 s. Plus a size ladder that printed `1024K` and rendered `i64` max as a bare `K`,
date formatters that could emit an embedded NUL from kernel mtime data, and two defects in the test
suite itself. Tests 11 → 37, all mutation-proven. Full accounting in the
[CHANGELOG](../../CHANGELOG.md).

---

## ⛔ Read this before touching the agnos event loop

**A 0.5.0 draft froze the entire desktop, and the host suite was 37/37 green for it.**

The draft idled with `sys_sleep_ms(16)`. `sleep_ms` (#41) is the **DOOM frame-pacing** primitive — it
calls `preempt_disable()` and then halts, and the kernel's own comment says *"we can't be preempted
off mid-sleep"*. While crab slept, **nothing else on the machine could be scheduled**.

Measured against `agnos/scripts/harness/puka-terminal-test.py`, same image, only the binary changed:

| build | clients placed | clients presented | `--clients` |
|---|---:|---:|---|
| 0.4.15 baseline | 2 | **2** | exit 95 — pass |
| 0.5.0 draft (`sleep_ms`) | 2 | **0** | never returned — fail |
| 0.5.0 shipped (`sys_pause`) | 2 | **2** | exit 95 — pass |

crab did not merely fail to yield — it stopped the compositor from running at all. Both clients went
dark. The shipped primitive is **`sys_pause` (#14)**, whose handler yields to a ready proc first and
only halts when nothing else is runnable. **Not `sys_sched_yield`** either — yield hands off and
comes straight back, so an idle desktop still spins a core.

⇒ **Any change to the loop in `src/main.cyr` needs a QEMU run before it is claimed.** The `#ifdef
CYRIUS_TARGET_AGNOS` region is invisible to every host test. The full reasoning is a ⛔ block at the
call site; don't delete it.

---

## What is verified, and what is not

⭐ **Verified on a real agnos kernel in QEMU, crab 0.6.0 (2026-08-27)** — the first on-target run of
the whole 0.6.0 tree (the `app.cyr` extraction, the reused render target, the per-frame arena):

- `agnos/scripts/harness/crab-listing-cap-test.py` — **PASS**, exit 0. `/bin` listed **45 of 45**, no
  truncation, no fault, no per-entry stat noise. So the extraction and the arena did not disturb the
  readdir/stat path on real ext2. ⚠ This harness never reaches the compositor — crab does both pane
  readdirs before it touches setu — so it says nothing about the event loop.

**Verified on a real agnos kernel in QEMU (`-smp 4`), crab 0.5.0:**

- `agnos/scripts/harness/crab-listing-cap-test.py` — **PASS**, exit 0. `/bin` listed **45 of 45**,
  no truncation, no fault. Drives the repaired path layer on real ext2: `crab_join_n` once per entry,
  the readdir clamp, the `STAT_SIZE`/`STAT_MTIME`/`STAT_BUFSZ` named offsets.
- `agnos/scripts/harness/puka-terminal-test.py` — **PASS**, background exit **95**, 2 presentations,
  0 faults. crab connects on the current channel-band transport, presents, and leaves via
  `crab: compositor closed the window -- exiting` — the 0.5.0 `WINDOW_CLOSE` path, observed.

**⚠ NOT verified — pick these up:**

1. **The loop-lifetime fix itself is still unproven on agnos.** Both harnesses have the compositor
   close crab's window quickly, and the **0.4.15 baseline exits the same way** without ever reaching
   its 2 s cap — so neither harness distinguishes the fix from the bug.
   - ⛔ **A shell-driven probe cannot work, and one was already tried and wasted.** Typing
     `aethersafha &` then `crab &` at the prompt comes back INCONCLUSIVE ("crab never presented"):
     once the compositor runs it **owns the console**, so the typed `crab &` never reaches agnsh —
     and crab cannot be launched from a shell under the current transport anyway, because it needs
     `AGNOS_CHAN`, which only the compositor sets when it mints and endows a channel. Do not repeat
     this approach.
   - ⛔ **`AE_CLIENTS_MODE=desktop` does not work either, and that was also tried.** It runs
     `aethersafha` in the foreground with **no `--clients`**, i.e. launcher mode — measured
     `launched: False, placed: 0, presented: 0`. The desktop stays up but **spawns nothing**.
   - ⇒ **What is actually needed is a new harness**, because no existing one both starts crab *and*
     leaves it running. The only path that starts a client on a persistent desktop is the **F2
     launcher**: boot → `aethersafha` foreground → `sendkey f2` → select crab → Enter.
     `scripts/harness/launcher-panel-test.py` already does the F2-and-Enter half (it proves the panel
     appears, and its own header says it does **not** prove which app launches) — so it is the
     skeleton to copy, not the test to run.
     Oracle once crab is up: crab prints `crab: key received` for every key, so a keystroke answered
     ≥30 s after `presented over setu` proves the loop is still turning. ⛔ **Silence is not
     liveness** — "no exit line" cannot tell a running process from one its parent killed, which is
     why the count-and-wait approach is not sufficient on its own.
   - ⚠ This is roadmap deferral **#16** ("bring crab's agnos/iron harness into crab's own repo")
     arriving with a concrete first job.
2. **`crab_descend` / `crab_ascend` were never exercised on agnos.** No harness drives navigation
   keys, so the bounded-join *refusal* path has host assertions only.
3. **Nothing has run on iron.** QEMU is explicitly *not* a control for timing- or pressure-dependent
   behaviour — the harness README records a lossy-queue failure that killed a client on iron and
   reproduced not at all under QEMU. The per-frame allocation ceiling is an iron question.

⚠ **Staging note**: `agnos/build/rootfs/bin/crab` currently holds the agnos build linked against
**dhancha 0.9.13** (381,592 B), re-staged 2026-08-26. It is a gitignored build artifact. Re-stage
after any rebuild — the harnesses read it, and a stale binary produces a confidently wrong result.

⛔ **AND THE SIZE DID NOT CHANGE WHEN THE LIBRARY DID.** The 0.9.12- and 0.9.13-linked agnos binaries
are **both 381,592 B** — the fix removes runtime allocation, not code — so the staged artifact was
byte-different while looking untouched, and `ls -l` could not tell them apart. This is the same shape
of evidence `state.md` flags for the 6.5.28 → 6.5.35 toolchain bump (identical size, 63.7 % of bytes
different). ⇒ **Compare with `cmp`, never with the size.**

---

## ⭐ M3 (v0.7.0) — SHIPPED. Four items in, and the rest are gated upstream

**Done and QEMU-proven**: sorting (*#33*), selection memory (*#34*), argv start paths (*#11*), and
deferred statting (*#03*). Details and the mutation evidence are in [`roadmap.md`](roadmap.md).

⭐ **The one finding worth carrying**: *#03* was **measured before it was built**, and that mattered.
The 2026-08-19 iron slowness had already been misattributed once — it looked like the listing and was
the per-entry narration. So the sweep was made to report its own cost first: **~1.1 ms per entry**
(50 ms for 45 in `/bin`), i.e. ~280 ms at the 256 cap on the keystroke that descends. ⚠ QEMU numbers;
the per-entry **linearity** is the durable finding, not the absolute figure.

⛔ **And the saving and the drain had to be proven in DIFFERENT harnesses.**
`crab-listing-cap-test.py` runs crab with no compositor, so crab exits before the event loop — it can
show the listing no longer sweeps (**zero** `stat-cost` lines, was six) but **cannot** show the sizes
ever arrive. `crab-resize-test.py` runs the real desktop and reports `deferred stat drain completed`.
Reporting only the first would have shown that crab stopped statting, not that it moved the work —
and those are indistinguishable right up until the sizes never come.

⚠ **`main.cyr`'s event loop is `#ifdef CYRIUS_TARGET_AGNOS`, so the drain has no host test at all.**
The policy helpers around it do (28 assertions, five mutations); the loop wiring is QEMU-only.

---

## Superseded — M2 (v0.6.1), shipped

[`roadmap.md`](roadmap.md) sequences eight milestones to 1.0. Next is **M2 — the window is real**:
resize (`WINDOW_CONFIGURE` is decoded by dhancha and dropped on the floor), pointer input (dhancha
synthesizes eight kinds; crab consumes none), key release, and routing through `dh_dispatch`.

> ✅ **The dhancha per-frame-allocation gate is CLOSED and no longer blocks anything** — see the
> section above: **746,440 → 0 B per steady-state frame**, at 114 and at 256 entries per pane. Full
> accounting in
> [`../architecture/001-every-frame-allocates-and-nothing-is-freed.md`](../architecture/001-every-frame-allocates-and-nothing-is-freed.md).
>
> ⛔ **The repaint constraint has MOVED, not vanished. M2's own *deferral #09* now carries it.**
> `dh_setu_poll_event` allocates ~80 B on every poll, pending or not, and continuous repaint means
> continuous polling — ~4.8 KB/s at 60 Hz, on a heap with no `free()`. Close #09 before adding the
> idle mascot line, a transfer tray, or index progress.

Other named upstream gates, per milestone, are in the roadmap: **rupa** (`on-accent`, without which a
selected row cannot carry legible text), **setu** (`SETU_SURF_FULL_KEYS`), **agnos** (resumable
readdir — a pane cannot exceed 256 entries and the canvas draws 812; plus M4's write syscalls),
**rekha** (proportional text), **daimon** (the vector store the whole AI arc rests on).

⚠ **No upstream issues have been filed**, and that is still true — but "nothing outside crab was
touched" is **no longer** true: the dhancha gate was closed by editing `../dhancha` directly rather
than by filing. The remaining named gates (rupa `on-accent`, setu `SETU_SURF_FULL_KEYS`, agnos
resumable readdir + write syscalls, rekha proportional text, daimon) are still enumerated only in
crab's roadmap. Filing them is an open, un-started task.

---

## Decisions that are settled — do not relitigate

- **[ADR 0001](../adr/0001-compositor-owns-theming.md)** — the compositor owns theming; crab ships no
  palette and no theme UI. The canvas's light and dark shells are two **compositor states**. ⚠ "Add a
  dark mode toggle" is excluded *architecturally*, not deferred — it will be asked for.
- **[ADR 0002](../adr/0002-semantic-find-is-a-mode.md)** — semantic find is a **mode over any view**.
  ⚠ Its cost lands early: the entry record must carry optional match metadata from **M3**, not M7,
  because the readdir record is the syscall's fixed 64 bytes and cannot hold it.
- All three canvas directions are absorbed on the way to 1.0, with **1b's wireframe scoped to the
  assisted-search surface** specifically — not the app shell.

---

## Known-stale, and owned by nobody yet

- ✅ **CLOSED 2026-08-27 — nothing in `src/main.cyr` was reachable from a test.** See the section
  above. *Deferral: none — it was never filed, which is part of why it survived.*
- ✅ **CLOSED 2026-08-26 — `CLAUDE.md`'s two `cyrius init` placeholders.** The operator supplied the
  mission statement; the identity line and `## Goal` are real. *Deferral #23.*
- ✅ **CLOSED 2026-08-26 — `README.md` § Status**, which had opened **"Scaffold."** and listed the
  dual-pane GUI under *Planned scope* since 0.2.0 (2026-07-10). It now says what works, what does not,
  and defers every volatile number to `state.md`. The retired `anu` codename and the rekha-TrueType
  claim went in the same pass. *Deferrals #24, #25, #26.*
  ⚠ **The replacement text over-claimed once before it landed, and was caught by reading `crab_row`
  rather than by a gate**: it said the panes had "size and modified columns". They do not — a row is
  one LABEL (13-char name, `~` on truncation, then `/` or a size) and the mtime lives only in the
  status line. ⇒ **Rewriting a stale claim is exactly when a new one gets introduced**; check the
  replacement against the code, not against the old text.
- CI runs only build + `cyrius test`. It **never builds `--agnos`** (the real target), never runs
  `render_test.cyr`'s ten pixel assertions, and runs no fuzz/bench/lint/fmt/vet/deny/coverage.
- The fuzz harness reads **none** of its input; the bench harness times an empty function. Both are
  scaffolds, so both are green against anything.
