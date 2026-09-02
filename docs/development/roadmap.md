# crab — Roadmap

> Milestone plan through v1.0. State lives in [`state.md`](state.md); this file is the
> **sequencing** — what ships, in what order, against what dependency gates.
>
> ⭐ Starting a slot? [`handoff.md`](handoff.md) has the current state and the open items.
>
> ⛔ **This file was the `cyrius init` scaffold template until 0.5.0** — `### M1 — _Title_ (v0.2.0)`
> with the body `_Replace this with the first real milestone._` — while crab shipped fifteen
> releases past it. Every deferral the codebase accumulated had nowhere to be sequenced, so they
> lived as ⛔/⚠ prose scattered across `src/`, the CHANGELOG and `state.md`, and the only way to
> find them was to read all of it. That is what this file now exists to prevent.

## The north star

crab is designed against [`Crab File Manager Mockups.dc.html`](../../Crab%20File%20Manager%20Mockups.dc.html)
at the repo root — a design canvas with three directions, each drawn full-screen (1280×768) and small
(420×560). **All three are absorbed on the way to 1.0**; they are not alternatives.

| direction | what it contributes | lands in |
|-----------|--------------------|----------|
| **1a** hairline dual-pane | the shell: list/details, view switcher, sidebar, hairline seams, 28px rows | M2–M4 |
| **1c** dense operator view | density: columns browser, gallery preview, transfer tray, volumes, menu bar | M5–M6 |
| **1b** meaning-first | the **assisted-search surface**: NL query, ranked results, WHY IT MATCHED, dupes | M7–M8 |

### Three decisions that are settled, and shape everything below

1. ⛔ **The compositor owns theming. crab ships no theme switcher and no palette of its own.**
   It reads surface/text/accent from aethersafha via `dh_theme_*` → rupa, and declares only which
   parts of the UI accent is permitted to tint. The light and dark shells in the canvas are two
   **compositor states**, not two crab settings. Any "add a dark mode toggle" request is out of
   scope by construction — see [ADR 0001](../adr/0001-compositor-owns-theming.md).

2. ⭐ **Semantic find is a MODE over any view, not a view of its own.** 1b is drawn as a screen, and
   a screen is cheaper — but a screen means two result paths that drift, and the ranked/dupe/why-it-
   matched affordances would exist only in one of them. One result model that list, grid, columns and
   gallery all render. See [ADR 0002](../adr/0002-semantic-find-is-a-mode.md).

3. ⚠ **1b's wireframe is the assisted-search surface specifically** — the query bar, REFINE facets,
   ranked list, WHY IT MATCHED and APPEARS IN panels, the dupes-in-set grouping, and the on-device
   guarantee. It is not the app shell; the shell is 1a's.

### Still open (from the canvas's own "Open questions")

- ✅ **Does the second pane ever leave? — ANSWERED and RATIFIED (v0.7.2).** Yes, below **600 px**,
  where two panes could no longer each show the full column set. ⛔ The threshold is **derived from
  crab's own column rule**, not copied from the canvas's 420 px: a pixel lifted from a mockup is a
  number nobody can re-derive when the font or the column set changes. The rule agrees with the
  drawing without quoting it.
- ✅ **Which accent roles does the compositor hand over? — ANSWERED (rupa 0.1.5, dhancha 0.9.20).**
  `on-accent`, and all four grounds clear the WCAG AA floor. ⚠ **The trap that outlives the answer**:
  on MUDRA dark `on-accent` is the *same value as `bg`*, so "this pane has no on-accent pixels"
  cannot be written as a pixel check — it finds background. `render_test` says so at the call site.
- ⛔ **Open: what does the app do about a layout the canvas never drew?** Every artboard is
  1280×768 or 420×560. crab now runs at both, and at 380×220, and accepts anything up to 4096. The
  small-window ratification answered one case by deriving a rule; the rest of the canvas is still
  two fixed sizes, and the next view (M5's grid) will ask the same question again.

## v1.0 criteria

- [ ] All three canvas directions absorbed — **1a shell is in** (M1–M4); 1c density is M5–M6;
      1b assisted search is M8
- [x] Reference coverage ≥ 80 % — **87 % at 0.7.7** (199/227). ⚠ It has fallen below the line twice
      mid-milestone and been brought back both times; the roadmap gates it **per release** rather
      than at v1.0, because a criterion checked once gets further away at every cut.
      ⭐ **And it is no longer gated by hand**: `ci.yml` runs `cyrius coverage --min 85`, so a cut
      that drops below the floor fails before anyone has to remember to look. (Verified the gate can
      fail: `--min 95` exits 1.)
- [ ] `docs/benchmarks.md` captured from a real bench harness. ⚠ **Half done**: `tests/crab.bcyr`
      no longer times `bench_noop` — it measures the sort at the cap, at 256 and at the real iron
      122 — but nothing writes `docs/benchmarks.md` from it.
- [ ] Green on iron, not only QEMU. ⚠ **The last burn was 2026-08-30 against 0.7.0's tree.**
      Everything from 0.7.1 onward — the write layer, the tray, recursion, the menu, the M5 views,
      thumbnails, EXIF and 0.7.7's five defect fixes — has run only on the host and under QEMU.
      Both defects crab has ever shipped were iron-only. ⛔ **Sequencing an iron run is the
      operator's call, not this file's.**
- [ ] CHANGELOG complete from v0.1.0; ADRs written for every ⛔ invariant now living in comments.
      ⚠ Two ADRs exist; the *Rules that outlive their milestone* section below is the shortlist of
      what still deserves one.
- [x] Security audit pass — ✅ **[`docs/audit/2026-08-31-audit.md`](../audit/2026-08-31-audit.md)**,
      the first. Four findings, ranked by what an attacker gets.
      ⛔ **F1 is a change in the TRUST MODEL, not a bug**: gallery view decodes every image in a
      folder merely opened, so crab now runs **~22,500 lines of third-party parser** (`chitra` +
      `sankoch`) in-process on attacker-chosen bytes. Measured: 8 decodes for opening a folder
      against 1 for selecting an entry. The two budgets bound MEMORY; none bounds code paths reached.
      ⛔⛔ **One finding in the first draft was WRONG and is kept with its correction** — it claimed
      an unbounded read `crab_batch_name` cannot perform. Caught by planting the mutation the finding
      implied and watching the suite stay green, then disproved empirically with a poison tail.
      *An audit that reports a bug which is not there spends the reader's trust.*
      ⚠ Re-run it when the trust model moves again — a new parser, a new dependency, or a view that
      widens what gets read.
- [ ] `docs/examples/` populated — still empty.

---

## Milestones

### Shipped — M1 through M4 (v0.5.0 … v0.7.5)

> ⚠ **These were 516 of this file's 763 lines** — design rationale for code that already exists,
> while M5–M7 (the actual future) had 29 between them. The roadmap had become a record instead of a
> plan. The rationale lives where it is useful: in the source comments beside the code it explains,
> and in [`../../CHANGELOG.md`](../../CHANGELOG.md) release by release. **This file is the
> sequencing.** Anything below that is still live guidance has moved to *Cross-cutting*.

| milestone | version | what shipped |
|---|---|---|
| **M1** Hardening | 0.5.0 | the P-1 sweep: bounded path helpers, an event loop that does not end itself, a size ladder that cannot overflow; `src/path.cyr` extracted so the suite can reach it |
| **M1.5** The allocation gate | 0.6.0 | **a rendered frame costs the global heap ZERO bytes** (746,440 → 0, across dhancha 0.9.13–0.9.15); `src/app.cyr` extracted from `main.cyr`, which ends in `_entry()` and so could never be tested |
| **M2** The window is real | 0.6.1 | resize, pointer (click, click-to-focus, double-click), the wheel, held-key repeat, `SETU_SURF_FULL_KEYS` |
| **M3** A browser you would use | 0.7.0 | sorting, selection memory, argv start paths, deferred statting, real columns, directories past the cap (`#101 readdir_at`), the readable selection (`on-accent`) |
| **M4** File operations | 0.7.1 – 0.7.5 | copy · move · delete · open · **mkdir · rename**; empty-pane states; the small-window ratification; drag between panes; the transfer tray with rate and ETA; multi-select; **recursive copy and delete**; the context menu; the batch-rename sheet |

⚠ **M4 rode four PATCH numbers by operator ruling** (0.7.1 – 0.7.5) rather than the v0.8.0 the ladder
below reserves. The milestone→version mapping has now been wrong twice. **Re-derive the number at
each cut**; do not trust the headings.

#### Still open from the shipped milestones

These are the only parts of M1–M4 that are not done. They are here so collapsing those sections did
not quietly lose them.

- **Focusing a pane by its HEADER** (*deferral #05* residue). The header is a sibling of the list,
  not inside it, so `crab_hit` resolves a header click to no pane. Clicking a *row* focuses correctly.
- **The keycode confusion spans three repos and one number** (*deferral #07*). aethersafha forwards
  `bhumi_key_usage(ev)` — an **HID usage** — while dhancha's `DhKey` constants are evdev. puka
  already pays the toll explicitly (`setuwin__hid_to_evdev` exists for exactly this). crab reads raw
  usages and is correct today *because* it never uses `dh_dispatch`; anything that starts to would
  inherit the mismatch. ⇒ **Fix it in one place or document it in three.**
- ✅ **CLOSED — the idle poll's per-cycle buffer** (*deferral #09*). ⛔ **This was carried as OPEN
  here and in the gate table, one of them stamped `verified: 2026-08-31`, while the fix had been in
  the declared graph for nine releases.** `dh_setu_poll_event` was hoisted onto a per-process scratch
  buffer in **dhancha 0.9.16** and consumed at crab **0.6.1** (`lib/dhancha.cyr:4314–4368`; the idle
  path returns before any allocation). ⇒ **A stale OPEN is not harmless — #09 is the stated
  precondition for the mascot line (*#29*) and any self-repainting element, so this entry was
  blocking work that was already free.** Found by measurement, not by a gate; that absence is *#37*.
- **The idle mascot line** (*deferral #29*) — unblocked since the allocation gate closed.

### M5 — Views (v0.9.0) — **STARTED; the two ungated items are in**

> ⭐ **Landed (unreleased, on top of 0.7.5)**: the **preview pane** with real metadata, **image
> dimensions read from headers with no decoder at all**, and the `crab_render` parameter cleanup
> that preceded them. Detail in [`../../CHANGELOG.md`](../../CHANGELOG.md).
>
> ⛔⛔ **AND THE THUMBNAIL GATE WAS FALSE IN A NEW WAY — chitra EXISTS, AND IT COSTS 528 KB.** The
> line below was corrected once already (from *"no image decoder"* to *"chitra 1.0.0 ships one"*).
> **Measured 2026-08-31**: declaring `chitra 1.0.0` takes crab from **453,304 → 981,992 B on the
> host (+116.6 %)** and **474,944 → 1,001,520 on agnos (+110.9 %)**, and `CYRIUS_DCE=1` reclaims
> **none** — it NOPs and the byte count does not move. **~399 KB of it is the `sankoch` inflate leaf
> PNG's IDAT requires**, pulled in transitively; chitra's own fold is ~113 KB.
> ⇒ **A gate can be false in both directions.** "No decoder exists" was wrong; "therefore it is
> ungated" is also wrong. The dependency is available and the *price* is the obstacle — which is an
> operator ruling, not a technical block. Precedent: kashi's library face was refused at **+50 %**.
> ⭐ **The metadata half shipped without paying it**, and the preview pane, the format sniffing, the
> budget's shape and the cache policy are the common prefix of both futures — so the ruling can be
> made late and cheaply.

- **Grid**, **Columns** (miller), **Gallery** — the canvas's four-way view switcher.
  ⛔ **NO dhancha GATE SURVIVES ON THIS LINE — it was stale in BOTH halves** (corrected 2026-09-01).
  GRID shipped in 0.9.25 and crab calls `dh_grid_new` at [`src/ui.cyr:943`](../../src/ui.cyr); the
  old text *"verified absent (`grep '^fn dh_grid_new' → 0`)"* contradicted this section's own
  fourth bullet. COLUMNS was **never** a dhancha gate — see the table below.
  ⚠ `CANVAS` (0.9.9) exists and could carry app-drawn grids, but the toolkit is the right home:
  *"the rule lived in three apps, now it lives once."*
- ✅ **Preview pane** with real metadata — **SHIPPED (unreleased)**. `p` toggles a right-hand
  inspector: NAME · KIND · SIZE · MODIFIED · DIMENSIONS.
  ⛔ **Its width rule is DERIVED from crab's own column rule** (*the preview may not cost a pane its
  SIZE column* → **303 px**), not lifted from the canvas — the same discipline that produced
  `crab_two_panes_fit`'s 600. ⚠ Opening it can collapse two panes into one, which is the ratified
  small-window rule answering a narrower pane area rather than a second behaviour.
  ✅ **Camera and shot metadata — SHIPPED (unreleased).** EXIF `Make`/`Model` and
  `DateTimeOriginal`, in both byte orders, verified against an independent parser.
  ⛔⛔ **It got its own slot with the fuzz harness pointed at it, and that was the right call.** EXIF
  is a TIFF directory whose **byte order, entry count and value offsets are all chosen by the file**
  — three independent ways to read where the file points rather than where the data is. The first
  version of its fuzz round was **vacuous**: an out-of-bounds read does not crash on a bump allocator
  over a large mapped heap, so four planted bounds bugs all survived a harness that checked only
  "returns 0 or 1". A printable poison tail past the fixture now catches the ones whose bytes reach
  the output; ⛔ two guards are caught by nothing and the harness says which.
- ✅ **Gallery view — SHIPPED.** `g` cycles list → grid → gallery. A cell is the grid cell with a
  thumbnail above the name; everything else is dhancha's GRID, shared.
  ⛔ **The view never triggers a decode — the idle tick does, one per tick**, so opening a gallery of
  a thousand files costs one frame. It stops by itself three ways: the walk ends, refusals are
  cached, and the session budget refuses once spent.
  ⚠ Backed by a 64-slot, ~1.07 MB, allocate-once thumbnail cache that stores results **and
  refusals** — the 16 KB result is cheap and the decode that made it is not.
- ✅ **Grid view — SHIPPED, both halves.** dhancha 0.9.25 added the `GRID` kind (wrap arithmetic,
  cell selection, row-wise arrows, keep-visible, hit-test); crab's `g` toggles both panes onto it.
  ⛔ **It earned a kind by dhancha's own rule** — the one 0.9.23 applied when it refused one to MENU
  and SHEET: composing a grid from boxes would make the app paint its own selection highlight, which
  means crab naming `accent`, which ADR 0001 forbids.
  ⚠ **Cells are names, not thumbnails**, and that is a budget decision: 40 cells at 256x256 is
  ~28 MB of permanent decode against a 32 MB session ceiling. The preview column carries the one
  thumbnail. A true gallery is a separate view and a separate ruling.
- ✅ **Thumbnails — SHIPPED (unreleased).** 64x64, PNG/JPEG/GIF/BMP, decoded off the idle tick and
  box-filtered down. **`chitra 1.0.1` is declared** (⚠ not 1.0.0 — corrected 2026-09-01; the
  manifest, `cyrius.lock` (`b777d34e8ef2`) and `sha256(lib/chitra.cyr)` all agree on 1.0.1. The
  1.0.0 mentions elsewhere in this file are historical narrative about the false gate and stand).
  ⛔⛔ **THE GATE ON THIS LINE WAS FALSE TWICE, IN OPPOSITE DIRECTIONS, AND THAT IS THE LESSON.**
  First it read *"Gate: an image decoder; none exists in the stack today"* — chitra 1.0.0 was
  released and shipping. Then the correction implied that made it free. **Measured**: declaring it
  costs **+115 %** of the binary, ~399 KB of which is the `sankoch` inflate leaf PNG requires.
  ⇒ A gate is a claim about another repository. It can be wrong about existence *and* about price,
  and neither is visible from the line itself.
  ⛔⛔ **AND THE REAL CONSTRAINT WAS IN NEITHER**: chitra makes 31 `alloc()` calls,
  `chitra_image_free` is a no-op, and cyrius's `alloc` has no `free()` — so **every decode is
  permanent**, ~2.5x the RGBA size, and a re-decode costs it again. That is bounded by a per-image
  pre-check (a refusal costs **16 bytes** against 26.6 MB unbudgeted) and a 32 MB session ceiling.
  ⚠ The budgets are the feature, not a rail bolted to it — and they are the first thing to delete
  when the allocator gains a `free()`.
- **Proportional text** (*deferral #26*). crab passes `font = 0` (kashi CP437 bitmap) and calls no
  `rekha_*` function (`grep 'rekha_' src/ → 0`), while the README claims rekha TrueType and the
  canvas assumes Barlow + JetBrains Mono. ⛔ **Gate: rekha** + dhancha font plumbing.
  ⚠ It also invalidates crab's caret arithmetic: `dh_draw_widget_ink` positions the TEXTINPUT caret
  with a fixed 9 px advance, correct only for a monospace bitmap.

### M6 — Sidebar, volumes, density (v0.9.x)

- **Sidebar** — PLACES / SMART FOLDERS / TAGS / VOLUMES with capacity bars.
  **Gate: dhancha** TREE widget; drawer overlay for the small window.
- **Menu bar** (1c) — File · Edit · Go · Tags · Index · Window. **Gate: dhancha** MENU.
- **The 🦀 menu** (canvas turn 2) — the mascot's chevron menu, light and dark, collapsed and revealed.

### M7 — The index (v0.10.0)

- **Local index** — `Local · 41,208 files`, `index fresh`, background indexing that
  `pauses on battery`. **Gate: daimon.**
- **Tags** — manual and suggested (`SUGGESTED TAGS · src → + toolchain + cyrius + wip`).
- **Smart folders** — Recent, Duplicates, Untagged, Large & old, Raw only, Unrated.
- **Duplicate detection** — byte-identical grouping, `Keep newest`.
- ⛔ **Declare the daimon dependency or stop promising the AI arc.** The package description, the
  `[deps]` comment and the README all commit to it; `cyrius.cyml` declares no daimon dep. daimon
  2.1.0 exists locally with vector/RAG stores. *Deferral #18.*

### M8 — Assisted search (v1.0.0)

1b, **as a mode over every view**.

- NL query bar (`invoices from last spring, the paid ones`), `⌘K` from anywhere
- Ranked results with a MATCH column; REFINE facets (Kind · Date · Size · Location)
- **WHY IT MATCHED** and **APPEARS IN** panels
- Dupes-within-result-set grouping
- ⭐ **`no external service` · `index stays on device`** — stated in the canvas UI, and it is a
  promise the implementation must actually keep. **Gate: daimon** local-only embedding.
- `SAVE AS → Smart folder…` closes the loop back to M7.

---

## ⛔ Everything gated on another repo, in one table

> **Read this before planning a slot.** A gate is a claim about a *different repository*, and claims
> about other repositories go stale without anything failing. This project has recorded **three false
> gates** — M4's write syscalls (real since agnos 1.41.3), drag ("gated on nothing" when it was
> gated), and thumbnails ("no image decoder" when chitra 1.0.0 ships one). ⇒ **Re-derive before
> believing.** The `verified` column is when the claim was last actually checked against the repo.

| item | milestone | gated on | verified |
|---|---|---|---|
| ~~Grid view~~ | M5 | ✅ **SHIPPED both halves.** dhancha 0.9.25 added the `GRID` kind; crab's `g` view is built on it. | 2026-08-31 ⭐ done |
| Columns (miller) view | M5 | ⛔ **FALSE GATE — the FOURTH.** Not dhancha: columns is a `BOX_H` of `LIST`s, each already carrying its own selection, scroll and toolkit-painted highlight, so it clears neither bar of dhancha's kind rule (as MENU and SHEET did not). **It is gated on crab's two-pane model** — the source/destination pairing the M4 write layer rests on. A design question, not a dependency. | 2026-08-31 ⭐ re-derived |
| ~~Thumbnail PIXELS~~ | M5 | ✅ **SHIPPED** — operator ruled 2026-08-31 and chitra **1.0.1** is declared (⚠ *not* 1.0.0, as this row and ~10 other places said). ⚠ The cost figures below are from the **pre-grid/gallery tree** and no longer describe the binary: host **+537,112 B (+115.2 %)**, agnos **+534,976 (+108.8 %)**. ⛔ The live constraint is not size but that **every decode is permanent** (~2.5x RGBA, no `free()`); two budgets bound it. | 2026-08-31 ⭐ done |
| Proportional text | M5 | **rekha** + dhancha font plumbing; crab calls no `rekha_*` | 2026-08-31 |
| Sidebar — PLACES | M6 | ⛔ **FALSE GATE (the fifth).** Not dhancha: `LIST` gives scroll/selection/highlight, `DH_FLAG_INERT` gives section headers, `PROGRESS` gives bars, padding gives indent. Buildable in crab today. | 2026-08-31 ⭐ re-derived |
| Sidebar — VOLUMES + capacity bars | M6 | ⛔ **TWO GATES, AND THE CAPACITY ONE IS NOW CLOSED.** *Capacity*: agnos shipped **`statfs`#103** in 1.56.56, all three backends answer it since **1.56.57**, and agnos's issue is archived. ⭐ **cyrius 6.5.37 shipped the stdlib peer, and crab VENDORS it as of the 0.7.7 pin bump to 6.5.41** — `lib/syscalls_x86_64_agnos.cyr` carries `SYS_STATFS = 103` and `fn sys_statfs(path, pathlen, buf)`; cyrius's issue is archived too. This row said *"filed 🟡 OPEN against cyrius"* and that is **no longer true**; there is nothing left to wait for and no raw-syscall interim needed. ⚠ Two crab-side facts remain: it is **agnos-only** (no host arm, so no host test can exercise it) and **no `STATFS_*` field offsets are vendored** — the frozen 32-byte record's layout has to come from agnos's docs. *Enumeration*: **still fully open** — `mount`#11 / `umount`#24 are documented no-op **stubs**, so crab cannot learn **what is mounted** (*deferral #44*). | 2026-09-02 ⭐ re-derived |
| Sidebar — SMART FOLDERS + TAGS | M6→M7 | **daimon**, like the rest of the AI arc. crab declares no daimon dep. | 2026-08-31 |
| Menu bar | M6 | ✅ **UNGATED as of dhancha 0.9.26.** The gate was real but MIS-NAMED: what was missing was not a MENU BAR kind but a **horizontal selectable strip**, since composing one from boxes makes the app paint the current item's highlight — i.e. name `accent`. `dh_list_new_h` is that strip, and it serves a menu bar, a tab strip, a toolbar and crab's own A/B switcher. ✅ **The wait is over**: 0.9.26 is pushed (`cb855c8`) and crab's manifest declares it as of 2026-09-01, verified by check four. ⚠ crab consumes none of it yet. | 2026-09-01 ⭐ resolvable |
| Local index · tags · smart folders | M7 | **daimon** — and crab declares no daimon dep at all | 2026-08-31 |
| Duplicate detection | M7 | **daimon**, or a content hash crab could do alone | 2026-08-31 |
| Assisted search | M8 | **daimon** local-only embedding | 2026-08-31 |
| ~~The idle poll's per-cycle buffer~~ | M2 residue | ✅ **CLOSED, AND IT HAD BEEN FOR NINE RELEASES.** Hoisted in dhancha **0.9.16**, consumed at crab **0.6.1**. ⛔ This row's `verified` stamp said 2026-08-31 while the fix was already in the declared graph — a re-derivation that re-derived nothing. *#09.* | 2026-09-01 ⭐ measured |

**Ungated and available now**, listed because a gated table invites the assumption that everything is:
~~preview pane (M5)~~ **shipped** · the 🦀 menu (M6) · smart-folder *plumbing* without the index (M7)
· every item under *Small, cheap, unblocked*.
⚠ **Thumbnails have moved OUT of this list** — not because a dependency is missing, but because the
one that exists costs **+117 %** of the binary. That is a decision, and a decision is not the same
kind of thing as a gate. It has its own row in the table above so it stops being read as free.

⚠ **daimon is the one to settle first.** Three milestones name it, `cyrius.cyml` declares it nowhere,
and **daimon 2.1.2 exists locally** with vector/RAG stores. *Deferral #18* has been open since the
roadmap was written: **declare the dependency or stop promising the AI arc.**

## Filed 2026-09-01 — the 0.7.6 verification sweep (*deferrals #40–#47*)

> ⛔ **Every row below was MEASURED against tag `0.7.6` (`26f38ed`) with the pinned toolchain**
> (`cyrius 6.5.36`, self-reporting `manifest-pin: 6.5.36`), not read off a document. They are here
> because the sweep that found them found them **outside the ledger** — which is the same absence
> *#37* names: nothing gates a claim, so a defect with no number has nowhere to be sequenced.
>
> ⚠ **The `pinned for` column is a PROPOSAL, not a schedule.** Version numbers are the operator's to
> direct — the milestone→version mapping in this file has already been wrong twice, and M4 rode four
> patch numbers by ruling. The column exists so no row is open-ended, not to commit a release date.

| # | subject | pinned for |
|---|---|---|
| **#40** | ✅ **CLOSED (in `[Unreleased]`, before 0.7.7).** ⛔⛔ **`cyrius test` DOES NOT RUN `[build].test`, AND CI's COMMENT SAYS IT DOES.** *Proven by mutation, not inferred*: `src/test.cyr` rewritten to `return 1` in a scratch tree left `cyrius test` at **1138/0, exit 0**. [`.github/workflows/ci.yml`](../../.github/workflows/ci.yml) asserts the opposite — *"bare `cyrius test` picks up the `[build].test` entry AND auto-discovers `tests/*.tcyr`"*. Only `tests/crab.tcyr` is compiled and run. ⇒ This makes *#17* **worse than filed**: the file is not merely redundant, CI tells the next reader it is a live gate. `src/test.cyr`'s own comment is the accurate one. | **0.7.7** — doc + comment fix, no code |
| **#41** | ✅ **CLOSED (in `[Unreleased]`, before 0.7.7)** — it prints `fuzz: rounds 100000`. ⛔ **The fuzz harness could not contradict a stale claim about itself.** `cyrius fuzz` prints `fuzz: ok` and **no round count**. Docs say **60,000** in six places; the harness drives **100,000** — five `while (round < FZ_ROUNDS)` loops at [`tests/crab.fcyr`](../../tests/crab.fcyr) 176/191/230/292/349 at `FZ_ROUNDS = 20000`, with no break. Two loops were added mid-cut (EXIF, batch-rename) and no output could say so. ⇒ **Exactly the failure `render_test` was fixed for at 0.7.6** — it now prints its 53 checks. Make the fuzzer print its rounds. | **0.7.7** — one print + doc sweep |
| **#42** | ⚠ **The deferral ledger is short by five.** [`CHANGELOG.md`](../../CHANGELOG.md) records *"**39 deferrals** were harvested"*; exactly **34** numbers are ever cited in any blob of any commit reachable from `--all` (#01–#07, #09–#21, #23–#26, #28, #29, #32–#39). **#08, #22, #27, #30, #31 have no anchor anywhere in history** — they were assigned at the harvest and never transcribed, so no subject can be recovered. ⇒ Correct the count to 34 and say the five were never written down. Nothing is lost by admitting it; a reader hunting five phantom items loses an afternoon. ⛔ **New numbers start at #40, not in the gaps** — reusing a harvested number would make the ledger lie twice. | **0.7.7** — CHANGELOG correction |
| **#43** | ⛔⛔ **A `path` DEP GETS NO `cyrius.lock` COMMIT PIN, SO A DECLARED-TAG/VENDORED-RECORD DIVERGENCE IS INVISIBLE TO EVERY GATE EXCEPT CHECK FOUR.** **This shipped**: tag `0.7.6` committed a `lib/dhancha.cyr` byte-identical to dhancha **0.9.26**'s `dist/` while `cyrius.cyml` declared **0.9.25**. Not the 0.4.13 phantom-tag failure — 0.9.25 was real and the declared graph built green — but the record and the declaration disagreed *in a released artifact*, and the lock could not catch it because four of seven deps carry `path` and get no `commit` line (lock reads **3 commit-pinned**; disabling the overrides takes it to **7**, which is the tell). ⇒ Closed for dhancha by the 0.9.26 bump; the **class** is what needs the gate, and that gate is *#19*. | **0.8.0** — with *#19*'s automation |
| **#44** | ⚠ **crab cannot learn what is MOUNTED, and that half of the VOLUMES gate is unfiled.** agnos `mount`**#11** / `umount`#24 are documented **no-op stubs** and the mount table is not reachable from ring 3. ⛔ **THIS ROW SAID `mount`#23 AND THAT IS WRONG — #23 IS `timerfd_settime`.** crab's own vendored header settles it: `lib/syscalls_x86_64_agnos.cyr` has `SYS_MOUNT = 11`, `SYS_TIMERFD_SETTIME = 23`, `SYS_UMOUNT = 24`. Filing the enumeration half against agnos while citing the wrong syscall would have cost the reader of that issue an afternoon. The *capacity* half is now **CLOSED** (see the gate table) — so the sidebar can draw a bar for a path it is *told* about, but still cannot enumerate volumes. ⇒ File the enumeration half against agnos, citing **#11**. | **M6** — gate-bound, not date-bound |
| **#45** | ⛔ **crab's OWN 9 px monospace assumption is load-bearing, and the roadmap names only dhancha's caret.** [`src/ui.cyr:175`](../../src/ui.cyr) `CRAB_COL_CHARW = 9` feeds `crab_col_chars` (ui.cyr:257–260, `px / CRAB_COL_CHARW`), which decides how many characters a NAME column holds — which drives the **`~` truncation rule** that exists so two different files never render as one identical row (the 0.5.0 fix). Under a proportional face `px / 9` over-reports for wide glyphs and **the truncation marker stops being honest**. That is correctness, not cosmetics, and the column-set ladder (`CRAB_COL_NAME_MIN = 90` = "10 chars", `crab_cols_for_width`) rests on the same constant. ⇒ Add it to the proportional-text gate; it is crab-side work no upstream release delivers. | with **proportional text** (gated) |
| **#46** | ⚠ **`cyrius lint` exits 0 no matter what it finds — but `cyrius lint --strict` exits 2, so a gate DOES exist.** The headline here was half wrong: the tool can gate, crab just is not using it. Re-measured 2026-09-02 at the 6.5.41 pin: **12 untracked deferrals** and **85 warnings** — of which **83 are `line exceeds 120 characters`** (`src/main.cyr` 14, `tests/crab.tcyr` 58, `tests/crab.fcyr` 11) and **2 are consecutive-blank-line** hits in `crab.tcyr`. ⚠ *The count matched 0.7.6's 85 exactly, but the label did not — correct the noun, not the number; rewriting this to "83" silently drops two real warnings.* ⇒ Still two items: adopt-or-delete the 12 (⚠ all 12 are keyword matches on already-tracked work — "deferred" ×10 describing the *#03* stat sweep, "not yet" ×1, "SCAFFOLD" ×1 — so **deleting them is the honest resolution**, not filing them), and decide whether `--strict` becomes a gate. That means reformatting 83 lines, most in `main.cyr`'s event loop, and is its own change. | **0.7.7** — re-measured; `--strict` deferred |
| **#47** | ⚠ **The held-key repeat rate was never given a number.** [`CHANGELOG.md`](../../CHANGELOG.md) `[0.6.1]` records an observed **~1 repeat per 1.6 s hold against ~20 expected**, filed as a *hypothesis* (frame cost) rather than a deferral, so it has sat outside the ledger for five releases. ⚠ It is agnos-runtime behaviour: no host test can see it, which ties it to *#16*. | **M6** — with *#16*'s harness |

## Filed 2026-09-02 — the 0.7.7 cut (*deferrals #48–#51*)

> ⛔ Measured at the **6.5.41** pin against the 0.7.7 working tree, not read off a document.
> ⚠ Numbers continue from **#48**; the gaps #08/#22/#27/#30/#31 stay empty forever (*#42*).

| # | subject | pinned for |
|---|---|---|
| **#48** | ⛔⛔ **`cyrius fmt --check` AND `cyrius lint` SILENTLY PROCESS ONLY THEIR FIRST FILE ARGUMENT.** Measured by planting a deliberately misformatted file: passed **first**, `fmt --check` exits 1 and names it; passed **last**, it exits **0 and prints nothing**. `lint` behaves identically. ⇒ The obvious CI step `cyrius fmt --check src/*.cyr` gates **one file of nine** and reports success — a gate that is worse than none, because it looks like coverage. crab's `ci.yml` works around it with a per-file loop and says why at the step. ⚠ **This is upstream behaviour, and no issue has been filed** — cyrius is not crab's to change. ⇒ Raise it with the operator before filing anything against cyrius. | **operator** — upstream, needs approval to file |
| **#49** | ⛔ **A crab RELEASE PUBLISHES AN x86_64-LINUX BINARY FOR AN AGNOS APPLICATION.** `release.yml` builds only the host target, folds `build/crab` into `SHA256SUMS`, and attaches it — so the artifact a consumer downloads and checksum-verifies cannot run on the platform crab is written for. ⚠ **Not the *#15* blind spot any more**: the `ci` job is a `needs:` dependency and now builds `--agnos`, so the shipping target must compile before a tag publishes. What is open is the **asset**, not the gate. ⇒ Options: attach both under explicit names, attach only the agnos one, or attach neither and let the source tarball be the release (which is what `cyrius deps` actually consumes). Recorded in `release.yml` at the step. | **operator** — a distribution decision |
| **#50** | ⚠ **`cyrius lib sync` WALKS ONLY THE DECLARED `[deps].stdlib` SET, SO TRANSITIVE LEAVES GO STALE ACROSS A TOOLCHAIN BUMP.** At the 6.5.36 → 6.5.41 bump the sync copied **29** files and left **three** behind — `lib/thread_agnos.cyr`, `lib/thread_local.cyr`, `lib/thread_macos.cyr` — all transitive, none named by any declaration. They were copied by hand and the whole vendored tree then verified byte-identical to the 6.5.41 snapshot. ⛔ **`lib/atomic.cyr` escaped only by luck**: it is byte-identical between 6.5.36 and 6.5.41, so *#20*'s standing warning did not bite this time. It will. ⇒ This is *#20* with evidence and a wider blast radius: **after every bump, diff the WHOLE vendored tree against the snapshot**, not just the declared subset. | **0.8.0** — with *#19*'s automation |
| **#51** | ⚠ **A toolchain bump can ADD a vendored leaf, and nothing tells you.** 6.5.41 pulled in **`lib/hashseed.cyr`** (a per-process hash seed + finalizer against hash-flooding, introduced in 6.5.39 and included by `hashmap` / `hashmap_fast`). It arrived **untracked** — `cyrius.lock` grew from 48 entries to 49 and the file appeared as `??` in `git status`. A bump that adds a leaf and is committed without it leaves a tree that resolves locally and fails a clean checkout. ⇒ After a pin bump, reconcile `git status lib/` **and** the lock's entry count, not just the modified files. | **0.8.0** — with *#19*'s automation |

### ⛔ Corrections to entries already in this file, made in the same sweep

- ✅ **APPLIED — the `dh_grid_new` gate line is gone from M5's first bullet.** It read *"Gate: dhancha GRID / COLUMNS widgets — verified absent"* while `dh_grid_new` was vendored and called live from `src/ui.cyr`, and while the same section said GRID had shipped four bullets later — the file contradicted itself in place. The bullet now states that no dhancha gate survives on the line. ⚠ Kept here as the record; there is nothing left to fix.
- ⛔⛔ **`path` no longer masks a stale pin — the manifest declares dhancha `0.9.26`** (bumped 2026-09-01). Check four re-run with all four `path` lines disabled: **7 deps / 0 errors**, lock **3 → 7 commit-pinned**, both targets build, **1138 / 0**, `render_test` **53 / 0**, and — the clause that had gone dead — the declared-graph binaries are **byte-identical** to the path-resolved ones (host `1,019,784` / `d7bd8125…`, agnos `1,047,624` / `93f5c139…`). ⇒ The menu bar's *"crab must wait for 0.9.26 to be pushed"* clause is **retired**; `dh_list_new_h` is resolvable from the declared graph today. crab still consumes none of it.
- **chitra is pinned `1.0.1`, not `1.0.0`** — asserted as 1.0.0 in ~11 live places across this file, the CHANGELOG and `src/`. The manifest, the lock (`b777d34e8ef2`) and `sha256(lib/chitra.cyr)` all agree on 1.0.1. ⚠ The thumbnail row's `+537,112 B / +115.2 %` figures are from the **pre-grid/gallery tree** and no longer describe the binary.
- ✅ **SUPERSEDED at 0.7.7 — the VOLUMES capacity gate is CLOSED.** This bullet said the missing piece was cyrius's `sys_statfs` peer, *"filed 🟡 OPEN against cyrius, affecting the `6.5.36` pin"*. **cyrius 6.5.37 shipped it and its issue is archived**; the 0.7.7 pin bump to 6.5.41 vendors it into crab. No raw-syscall interim is needed. Enumeration (*#44*) is the half that remains. ⇒ **A gate written as OPEN outlived the fix by four cyrius releases** — the same failure as *#09*, which was carried OPEN for nine.

## Cross-cutting (not a milestone — do these continuously)

### Rules that outlive their milestone

Pulled out of M1–M4 when those sections collapsed, because each still governs work not yet written.

- ⛔⛔ **crab OWNS ITS INTERACTION STATE — it does not use `dh_dispatch`.** Operator ruling
  2026-08-27. dhancha tracks a press as a **widget pointer**, and crab rebuilds its whole tree every
  frame with the arena rewinding underneath it. crab tracks **pane index + row index** instead.
  ⚠ **This has now cost three dhancha features**: `dh_dispatch` itself, drag (`_dh_drag_src`, which
  0.9.21 fixed by refusing to start under an arena), and `TEXTINPUT` (a per-widget buffer on an
  arena'd widget — 0.7.5 owns the edit buffer instead). **Three features, one assumption: a retained
  tree.** Expect the fourth, and tell dhancha the pattern is a pattern.
- ⛔⛔ **NEVER `sys_sleep_ms` IN THE AGNOS EVENT LOOP.** It `preempt_disable()`s, so a 0.5.0 draft
  froze the entire desktop — placed 2, presented 0 — while the host suite was 37/37 green. The
  shipped primitive is `sys_pause` (#14). ⚠ The loop is inside `#ifdef CYRIUS_TARGET_AGNOS`, so
  **no host test can see this**: any change to it needs an on-target run before it is claimed.
- ⛔ **A host build proves nothing about the event loop.** The whole key-handling region is inside
  that same `#ifdef`, so a brace error there compiles **clean** on the host and fails only on
  `--agnos`. It happened at 0.7.5. **Build both targets, every time.**
- ⛔ **`src/ui.cyr` sits BELOW `src/app.cyr`, and the render path must never reach up.**
  `render_test.cyr` and the suite include `ui.cyr` **alone**, so a render-path call into `app.cyr`
  compiles through `main.cyr` and leaves `render_test` with undefined symbols. This happened **three
  times in one milestone**. Anything the render path touches lives at or below `ui.cyr`.
- ⛔ **DOTFILES ARE NOT HIDDEN, DELIBERATELY.** Hiding them is only safe when there is a way to
  reveal them, and crab has no such affordance and no settings surface to put one on. A file manager
  that silently omits files is worse than one that shows too many.
- ⛔ **Directories-first is unconditional and outranks the sort key.** Interleaving them by name
  means hunting for the one folder among a hundred files.
- ⛔ **crab names NO colour.** It reads `dh_theme_*` and declares only which parts of the UI accent
  may tint. See [ADR 0001](../adr/0001-compositor-owns-theming.md). The pressure to break this
  arrives as "just this one shade" — in 0.7.5 it arrived as a progress bar and a menu, and both were
  answered by putting the colour in dhancha or rupa instead.
- ⚠ **A milestone closing with gated items is the normal shape here.** M2 shipped 5 of 7, M3 shipped
  4 of 7. Calling that "done" is a lie and calling it "not shipped" is another.

### Testing and CI — the gate is thinner than it looks

> ⭐⭐ **THE GATE STOPPED BEING THIN AT 0.7.7.** `.github/workflows/ci.yml` was rewritten and now
> runs, in order: `deps` + `deps --verify` · host build · **`--agnos` build** · `cyrius test` ·
> **`render_test`** · `fuzz` · a per-file `fmt --check` loop · `coverage --min 85` · `vet` + `deny`.
> `release.yml` gates on that workflow via `uses:`, so a tag inherits every one of them.
> ⇒ **#14, #15 and #36 are CLOSED.** The three bullets below are kept as the RECORD of what was
> wrong and what it cost — they are history now, not open items. Everything after them is still live.

- ✅ **CLOSED at 0.7.7 — `src/render_test.cyr` runs in CI**, as its own step, with its exit code
  (the failed-check count) as the assertion. *Deferral #14.* ⚠ **53** checks now, not the 35 this
  line claimed nor the 26 before that — re-derive it, do not quote it.
  ⛔⛔ **WHY IT MATTERED, KEPT:** of seven mutations planted against the render-state record and the
  preview column, **four were caught ONLY by `render_test`** — the suite CI ran was green for all
  four. A refactor swapping the two pane blocks, or a preview column painted over panes that never
  gave up their width, **shipped through CI green** for as long as this was open.
  ⚠ Still true, and why the step is explicit rather than discovered: `cyrius test` does not find it,
  and `[build].test` is inert (*#17*).
- ✅ **CLOSED at 0.7.7 — CI builds `--agnos`**, the target `state.md` calls "the real target".
  *Deferral #15.* ⛔ The flag is `--agnos`: `CYRIUS_TARGET=agnos` is **silently ignored** and yields a
  byte-identical host binary at exit 0, so a step written that way would be the host build twice.
  ⛔⛔ **WHY IT MATTERED, KEPT:** on 2026-08-27, with `path` disabled and a fresh resolve against the
  DECLARED graph (dhancha 0.9.17, which predates `POINTER_SCROLL`): **host PASS · `--agnos` FAIL —
  `undefined variable 'POINTER_SCROLL'`.** A commit in that state went green through CI and could not
  build the thing that ships.
- ✅ **CLOSED at 0.7.7 — `fuzz`, `fmt --check`, `coverage`, `vet` and `deny` all run in CI.**
  *Deferral #36.* ⛔ **`fmt --check` and `lint` process only their FIRST file argument** (measured:
  a misformatted file passed first exits 1, passed last exits 0 silently) — so the step is a LOOP,
  one invocation per file, and `cyrius fmt --check src/*.cyr` would have gated one file of nine.
  ⚠ **Three are deliberately still out**, and the reasons are recorded in `ci.yml` itself: `bench`
  (a timing on a shared runner is noise, and a step that cannot fail is not a gate), plain `lint`
  (exits 0 whatever it finds, so it *cannot* gate) and `audit` (bundles lint, so it exits 1 on the
  same warnings). `lint --strict` **would** gate, at the price of reformatting 83 over-long lines —
  see *#46*.
- ✅ **CLOSED — the fuzz harness reads its input.** *Deferral #12.* **100,000** deterministic
  rounds over mutated format headers, random bytes, degenerate and negative lengths, and arbitrary
  bytes through `crab_name_ok` / `crab_is_image` / `crab_cstr_len`. It asserts an invariant, not just
  the absence of a crash, and it caught a real segfault in `crab_img_dims` the day it was written.
  ⛔⛔ **ITS OWN FIRST DRAFT WAS VACUOUS, AND ONLY A PLANTED BUG SAID SO.** An LCG's low bits have
  period 2^k, so the format selector returned **only two of four values across 20,000 rounds** — PNG
  and JPEG were never seeded and the JPEG walk ran **zero** times while it printed `fuzz: ok`.
  ⇒ **Plant a known bug in a new fuzzer and watch it fail before believing its green.**
  ⚠ Still uncovered: `crab_readdir_into` (agnos-only), the write layer's joins, `crab_batch_name`.
- ⚠ **The bench harness no longer times an empty function** — this line was stale from 0.7.0.
  `tests/crab.bcyr` measures the sort: merge **88.6 µs** vs insertion **5.66 ms** at 256 scrambled,
  and **38.3 µs** vs **1.28 ms** at the real iron 122. ⛔ **What is still missing is the half the
  v1.0 criterion names**: nothing writes `docs/benchmarks.md` from it. *Deferral #13, half open.*
- **Bring the agnos/iron harness into this repo** — both real defects crab shipped were agnos-runtime
  behaviour no host test can see. *Deferral #16.*
- ✅ **CLOSED 0.6.0 — nothing in `src/main.cyr` was reachable from a test.** `src/app.cyr` carries the
  application layer now; `main.cyr` is `main()` and `_entry()` and nothing else. ⚠ The residual gap is
  irreducible and is down to the event loop alone — and the one setup step that used to live in
  `main()` (the frame arena) was **moved** into `crab_render` rather than tested around.
- ⛔ **Three tests could not fail in their first draft, and only mutation testing said so** (0.6.0):
  a residue check over trees that repainted every pixel; a convergence check whose arena was four
  times larger than the frame it measured; and dhancha's grow test, which never grew. ⇒ **Assert that
  the thing you are measuring can be observed to FAIL**, and size fixtures against the mechanism
  rather than for comfort.
- **Automate the `path`-wins-over-`tag` re-verification** — 0.4.13 shipped a manifest naming a
  library the build never compiled. *Deferral #19.*
- **Enforce `state.md` currency** — it has rotted twice, once across eleven releases, and the file
  itself diagnoses why: nothing gates it. *Deferral #37.*

### Documentation debt

- ✅ **CLOSED 2026-08-26 — `CLAUDE.md`'s two `cyrius init` placeholders are gone.** The identity line
  and the `## Goal` section now carry the operator's wording. *Deferral #23.*
- ✅ **CLOSED 2026-08-26 — `README.md` § Status.** It had said **"Scaffold."** since before the
  dual-pane GUI shipped in 0.2.0, and listed that GUI under *Planned scope*. It now states what works,
  what does not (read-only, fixed 380×220, keyboard-only, no AI arc), and defers every volatile number
  to `state.md`. *Deferral #24.* The retired `anu` codename (*#25*) and the rekha TrueType claim
  (*#26*) are corrected in the same pass — crab passes `font = 0` and draws with kashi's CP437 bitmap.
  ⚠ **A third over-claim was found while writing it and is also fixed**: the draft said the panes had
  "size and modified columns". They do not — a row is one LABEL holding a 13-char name plus `/` or a
  size, and the mtime appears only in the status line. Real columns are M3.
- ✅ **CLOSED (unreleased) — `README.md` § Status, for the second time and in the opposite
  direction.** *Deferral #24.* It had said **"Shipping, and read-only — no copy, move, rename,
  delete or mkdir; Enter on a file does nothing"** and quoted a retired **256-entry cap** and a
  **~280 ms** stat figure the cap bump had made ~1.1 s. Every one of those was true when written and
  all were false by M4. It now states the write layer, the preview column, and what is genuinely
  absent — including thumbnails, with the reason (**size, not capability**).
  ⛔⛔ **THE SECTION HAS NOW BEEN WRONG IN BOTH DIRECTIONS AND THE CAUSE IS IDENTICAL BOTH TIMES**:
  written once, correctly, and never re-read at a cut. The new text says so in its own body, so the
  next reader is warned by the file itself rather than by this one. ⇒ **Re-read § Status at every
  release.** Nothing gates it — the same absence that let `state.md` rot twice.

- `docs/architecture/` and `docs/adr/` are empty indexes. The ⛔ invariants live only in comments,
  which means they die with the line they annotate. *Deferral #28.*
- `docs/examples/`, `docs/benchmarks.md`, `docs/audit/` — declared, absent, and two are v1.0
  criteria. *Deferral #39.*

### Small, cheap, unblocked

- Drop the redundant `net` stdlib declaration — 0.4.15 measured the removal clean. *Deferral #21.*
- Cover `lib/atomic.cyr` in the toolchain sync, or make the sync walk transitive leaves. *Deferral #20.*
- Give the stat trace an arm that works where it is needed — on agnos the compositor spawns crab,
  so `CRAB_STAT_TRACE=1` in a shell environment never reaches it. *Deferral #35.*
- Resolve `[build].test` pointing at a file that must stay empty. *Deferral #17.*
- Close or formally park the `--win` failure (`sys_socket`/`sys_connect` absent from the Windows
  syscall table; nothing in crab causes it). *Deferral #38.*

### 🦀 Bueller

`docs/development/mascot.md` carries an explicit "Easter egg — implementation TODO", and the canvas
puts it exactly where the mascot doc asks: **one quiet home, the status bar**. *Deferral #29.*

- Long idle / empty pane → `Bueller…` · pause · `Bueller…` · pause · `Bueller…?` — needs M2's
  self-redraw (an event-driven-only loop cannot animate).
- `crab --about` closing on `…anyone? …anyone?` — M3's argv handling has landed (*#11*), so this is
  now unblocked; crab parses positional paths only and has no flag surface yet.
- ⚠ **The idle hook it needs now exists.** M4's transfer tray put a real per-tick redraw on the idle
  path (`crab_op_step` → re-render), so the self-redraw this was waiting on is shipped — the mascot
  line is a `crab_set_notice` on a long-idle counter, not new machinery.
- ⚠ **Discipline, from the mascot doc: subtle and infrequent.** "The whole thing dies if it's trying
  too hard." This is a constraint on the implementation, not colour commentary.

---

## Out of scope for v1.0

- **A theme switcher, a palette, or any crab-owned colour UI** — the compositor owns theming. This is
  the most likely thing to be asked for and the answer is architectural, not a preference.
- **Windows as a target** — `--win` fails on two absent syscall stubs and nothing in crab causes it.
  crab is an AGNOS desktop application.
- **Network / remote filesystems** — crab is local-first by design.
- **A plugin system.**
