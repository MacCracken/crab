# crab — Roadmap

> Milestone plan through v1.0. State lives in [`state.md`](state.md); this file is the
> **sequencing** — what ships, in what order, against what dependency gates.
>
> ⭐ Starting a slot? Read **[Release batches](#release-batches)**. `handoff.md` has the current
> state; [`../../CHANGELOG.md`](../../CHANGELOG.md) has what already shipped and why.
>
> ⛔⛔ **A ROADMAP IS BATCHES OF WORK PINNED TO RELEASES. IT IS NOT A DEFECT LEDGER.** This file
> carried numbered `#NN` deferral tables — rows for things already fixed, rows for things that were
> never bugs, and a running "corrections to entries in this file" section. That is changelog
> material wearing a roadmap's clothes, and it buries the one question the file exists to answer:
> **what ships next.** Every open item now lives in the batch that will carry it, described by what
> it IS rather than by a number. Fixed defects live in the CHANGELOG, release by release.

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
not quietly lose them; each is carried in a batch above.

- **Focusing a pane by its HEADER.** The header is a sibling of the list, not inside it, so
  `crab_hit` resolves a header click to no pane. Clicking a *row* focuses correctly.
- **The keycode confusion spans three repos and one number.** aethersafha forwards
  `bhumi_key_usage(ev)` — an **HID usage** — while dhancha's `DhKey` constants are evdev. puka
  already pays the toll explicitly (`setuwin__hid_to_evdev` exists for exactly this). crab reads raw
  usages and is correct today *because* it never uses `dh_dispatch`; anything that starts to would
  inherit the mismatch. ⇒ **Fix it in one place or document it in three.**
- **The idle mascot line** — unblocked, and in the M6 batch.

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
- **Proportional text.** crab passes `font = 0` (kashi CP437 bitmap) and calls no
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
  2.1.0 exists locally with vector/RAG stores.

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
| Sidebar — VOLUMES + capacity bars | M6 | ⛔ **TWO GATES, AND THE CAPACITY ONE IS NOW CLOSED.** *Capacity*: agnos shipped **`statfs`#103** in 1.56.56, all three backends answer it since **1.56.57**, and agnos's issue is archived. ⭐ **cyrius 6.5.37 shipped the stdlib peer, and crab VENDORS it as of the 0.7.7 pin bump to 6.5.41** — `lib/syscalls_x86_64_agnos.cyr` carries `SYS_STATFS = 103` and `fn sys_statfs(path, pathlen, buf)`; cyrius's issue is archived too. This row said *"filed 🟡 OPEN against cyrius"* and that is **no longer true**; there is nothing left to wait for and no raw-syscall interim needed. ⚠ Two crab-side facts remain: it is **agnos-only** (no host arm, so no host test can exercise it) and **no `STATFS_*` field offsets are vendored** — the frozen 32-byte record's layout has to come from agnos's docs. *Enumeration*: **still fully open** — `mount`#11 / `umount`#24 are documented no-op **stubs**, so crab cannot learn **what is mounted**. | 2026-09-02 ⭐ re-derived |
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
and **daimon 2.1.2 exists locally** with vector/RAG stores. **Declare the dependency or stop promising the AI arc** — open since the roadmap was written.

## Release batches

> **This is the sequencing.** Each batch is work pinned to a release; a batch is done when
> everything in it ships. ⛔ **Version numbers are the operator's to DIRECT** — the milestone→version
> mapping in this file has already been wrong twice, and M4 rode four patch numbers by ruling. The
> headings say what a batch is FOR, not when it lands.
>
> ⛔ **Fixed defects are not roadmap items.** What broke, what it cost and how it was proven live in
> [`../../CHANGELOG.md`](../../CHANGELOG.md), release by release. What is left to DO lives here.

### 0.7.8 — the cheap unblocked ones

Nothing in this batch is gated on another repo, and none of it is large.

- **Drop the redundant `net` stdlib declaration.** setu removed TCP at 0.8.4 and crab has pinned past
  it since 0.4.5. Removal measured clean: `cyrius deps` re-creates the leaf from setu's sidecar and
  the build is byte-for-byte the same size. ⚠ The leaf lands at a different concatenation offset, so
  ~165 KB of the binary *differs* at the same size — record it as "same size, same tests, different
  layout" or the next reader thinks something broke.
- **Give the stat trace an arm that works where it is needed.** On agnos the compositor spawns crab,
  so `CRAB_STAT_TRACE=1` set in a shell never reaches it.
- **Write `docs/benchmarks.md` from the bench harness.** A v1.0 criterion, and the harness half is
  already done — it measures the sort at 1024, at 256 and at the real iron 122. Nothing writes the
  document. ⚠ Record the machine, `cycc --version` and the run-to-run spread; merge/256 was seen
  swinging 87.4 → 92.4 µs a minute apart, and a single number becomes the next stale claim.
- **Populate `docs/examples/`.** Declared, empty, and a v1.0 criterion.
- **Close or formally park the `--win` failure.** `sys_socket` / `sys_connect` are absent from the
  Windows syscall table and nothing in crab causes it; Windows is not a declared target. Park it in
  writing or stop listing it as a target.

### 0.8.0 — make the gates automatic

⛔ **The theme is that nothing currently gates a CLAIM.** Every failure in this batch was found by a
human reading carefully, which is not a process.

- **Automate the `path`-wins-over-`tag` re-verification.** 0.4.13 shipped a manifest naming a library
  the build never compiled, and only a full resolve with the overrides disabled would have caught it.
  ⛔ A `path` dep gets no `cyrius.lock` commit pin, so a declared-tag/vendored-record divergence is
  invisible to every other gate — that shipped in 0.7.6, with `lib/dhancha.cyr` byte-identical to
  0.9.26's `dist/` while the manifest said 0.9.25. The tell is the commit-pinned count: 3 with the
  overrides on, 7 with them off.
- **Make the toolchain sync walk the whole vendored tree.** `cyrius lib sync` walks only the declared
  `[deps].stdlib` set, so transitive leaves go stale across a bump and nothing says so. At the 6.5.41
  bump it left three `thread_*` leaves behind; `lib/atomic.cyr` escaped only because it happens to be
  byte-identical between the two versions. A bump can also ADD a leaf — 6.5.41 brought
  `lib/hashseed.cyr` in untracked and grew the lock from 48 entries to 49. ⇒ Diff the whole tree
  against the snapshot, and reconcile `git status lib/` **and** the lock count.
- **Enforce `state.md` currency.** It has rotted three times, once across eleven releases, and the
  file itself diagnoses why: nothing gates it. The same absence let `handoff.md` carry a table that
  was six rows wrong and `README.md` § Status be wrong in both directions.
- **Decide whether `cyrius lint --strict` becomes a gate.** It works (exit 2). The price is
  reformatting **83** over-long lines — 14 in `src/main.cyr`, 58 in `tests/crab.tcyr`, 11 in
  `tests/crab.fcyr` — most of them in the event loop, the widest mechanical edit in the most
  sensitive file. ⛔ Its own change, with the suite run before and after; never a rider on another.
- **Correct the CHANGELOG's deferral count.** `[0.5.0]` records *"39 deferrals were harvested"*;
  34 numbers were ever written down. Five were assigned and never transcribed, and no subject can be
  recovered — say so, rather than leaving a reader hunting them.

### M6 — sidebar, volumes, density (see the milestone above for scope)

- **Sidebar, PLACES section.** Buildable today — no dhancha gate. Four traps are recorded in
  `docs/development/handoff.md`; the load-bearing one is that `crab_hit` returns a 0/1 pane index
  that reaches the write layer, so a sidebar must get its own hit function or a drop onto it moves a
  real file.
- **The A/B view switcher, then the menu bar**, both on `dh_list_new_h`. The switcher is the smaller
  of the two and `active_pane` already *is* the selected index.
- **The 🦀 mascot line.** `docs/development/mascot.md` asks for one quiet home, the status bar:
  long idle → `Bueller…` · pause · `Bueller…` · pause · `Bueller…?`, plus `crab --about` closing on
  `…anyone? …anyone?`. ⚠ **There is no unconditional per-tick redraw to ride** — every idle-path
  `crab_render` fires only on change, deliberately, so this needs its own branch and an `idle_since`
  reset on every handled event. Gate it on there being no notice: the status line is single-slot,
  and a mascot must never displace `delete this FOLDER and everything in it? y = yes`.
  ⛔ Discipline, from the mascot doc: **subtle and infrequent.** "The whole thing dies if it's
  trying too hard."
- **Focusing a pane by its HEADER.** The header is a sibling of the list, not inside it, so
  `crab_hit` resolves a header click to no pane. Clicking a row is correct.
- **Give the held-key repeat rate a number.** `[0.6.1]` records an observed ~1 repeat per 1.6 s hold
  against ~20 expected, filed as a hypothesis and never measured. It is agnos-runtime behaviour, so
  it needs the on-target harness below.
- **Bring the agnos/iron harness into this repo.** Both real defects crab has ever shipped were
  agnos-runtime behaviour no host test can see.
- **Sidebar VOLUMES** — gate-bound, not date-bound. Capacity is unblocked; enumeration is not.

### Gate-bound — no release until the gate moves

- **Proportional text**, and with it crab's own 9 px monospace assumption. `CRAB_COL_CHARW = 9`
  feeds `crab_col_chars`, which decides how many characters a NAME column holds, which drives the
  `~` truncation rule that exists so two different files never render as one identical row. Under a
  proportional face `px / 9` over-reports and **the truncation marker stops being honest** — that is
  correctness, not cosmetics. Five more constants encode character counts computed at 9 px and feed
  the column-set ladder; those pick the wrong *layout* rather than failing honestly.
  ⛔ See the gate table for what is actually missing, which is **not** what this file said for months.
- **Columns / miller view** — gated on crab's own two-pane model, which is a design question and
  crab's to answer. The cheapest option that does not relitigate M4's source/destination pairing is
  columns as a fourth view mode *inside one pane*.
- **The keycode confusion spans three repos and one number.** aethersafha forwards an **HID usage**
  while dhancha's `DhKey` constants are **evdev**. crab is correct today *because* it never uses
  `dh_dispatch`; anything that starts to inherits the mismatch. ⇒ Fix it in one place or document it
  in three.
- **The AI arc** — M7 and M8 in full. ⛔ **Declare the daimon dependency or stop promising it.**
  Three milestones name it and `cyrius.cyml` declares it nowhere. daimon exists locally and has a
  `dist/`, so this is a crab-side decision about the transitive fold, not an upstream gate.

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

### Testing and CI — what the gate does, and what it still cannot see

⭐ **CI runs nine steps** — `deps` + `deps --verify`, host build, **`--agnos` build**, `cyrius test`,
**`render_test`**, `fuzz`, a per-file `fmt --check` loop, `coverage --min 85`, and `vet` + `deny`.
`release.yml` gates on that workflow, so a tag inherits all of it. What it deliberately omits, and
why, is recorded in `ci.yml` itself.

The parts a green CI still does not prove:

- ⛔⛔ **`src/main.cyr` IS COMPILED BY THE GATE AND EXECUTED BY NOTHING.** 1,221 of its 1,494 lines
  are inside one `#ifdef CYRIUS_TARGET_AGNOS` with **no `#else`**, and nothing includes the file —
  so the entire key-dispatch table is untested on *every* target, not merely unbuilt on one. Five of
  the six defects 0.7.7 fixed lived there. ⇒ **A decision that lands in that region must be lifted
  into a function the suite can reach**, the way `crab_transfer_plan`, `crab_menu_row`,
  `crab_tray_h`, `crab_drag_targets` and `crab_two_panes_fit` were. The event loop keeps the wiring.
- ⛔ **A gate that covers one state proves one state.** crab's zero-allocation assertion rendered
  twenty frames with no overlay open, so a 32 B-per-frame leak in `crab_overlay` shipped for three
  cuts with a green suite. Arms now exist for the menu, the sheet and the preview. **A new
  render-path branch without an arm here is a new blind spot, not a covered feature.**
- ⛔ **Prove a new test can FAIL before believing it.** Three tests could not fail in their first
  draft and only mutation said so; the fuzz harness's own first draft was vacuous, seeding two of
  four formats while printing `fuzz: ok`. Every 0.7.7 fix was mutation-proven for this reason — and
  that run also exposed a defect in one of the new tests, whose cleanup assumed the code under test
  had worked. **Size a fixture against the mechanism, not for comfort.**
- ⚠ **An out-of-bounds read does not crash in this stack.** The allocator is a bump allocator over a
  large mapped heap, so reading past a buffer returns garbage rather than faulting. A poison tail
  catches overreads that reach the OUTPUT; guards whose bytes only reach control flow are caught by
  nothing, and two EXIF guards sit in exactly that position. They are kept and labelled.
- ⚠ **Still uncovered by the fuzzer**: `crab_readdir_into` (agnos-only), the write layer's path
  joins, and `crab_batch_name`'s pattern expansion.
- ⚠ **No host test can see the agnos event loop**, and both real defects crab has ever shipped were
  agnos-runtime behaviour. The on-target harness is in the M6 batch.

### Documentation debt

- ⛔ **Re-read `README.md` § Status at every release.** It has been wrong in **both** directions —
  "Scaffold." long after the GUI shipped, then "read-only" long after the write layer did — and the
  cause was identical both times: written once, correctly, and never re-read at a cut. Nothing gates
  it, which is the same absence that let `state.md` rot three times.
- **Write ADRs for the ⛔ invariants that still live only in source comments.** A comment dies with
  the line it annotates. The shortlist is *Rules that outlive their milestone*, above; three ADRs
  exist so far.

---

## Out of scope for v1.0

- **A theme switcher, a palette, or any crab-owned colour UI** — the compositor owns theming. This is
  the most likely thing to be asked for and the answer is architectural, not a preference.
- **Windows as a target** — `--win` fails on two absent syscall stubs and nothing in crab causes it.
  crab is an AGNOS desktop application.
- **Network / remote filesystems** — crab is local-first by design.
- **A plugin system.**
