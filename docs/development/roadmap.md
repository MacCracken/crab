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
- [x] Reference coverage ≥ 80 % — **86 % at 0.7.5** (137/159). ⚠ It has fallen below the line twice
      mid-milestone and been brought back both times; the roadmap now gates it **per release**
      rather than at v1.0, because a criterion checked once gets further away at every cut.
- [ ] `docs/benchmarks.md` captured from a real bench harness. ⚠ **Half done**: `tests/crab.bcyr`
      no longer times `bench_noop` — it measures the sort at the cap, at 256 and at the real iron
      122 — but nothing writes `docs/benchmarks.md` from it.
- [ ] Green on iron, not only QEMU. ⚠ **The last burn was 2026-08-30 against 0.7.0's tree.**
      Everything from 0.7.1 onward — the write layer, the tray, recursion, the menu — has run only
      on the host and under QEMU. Both defects crab has ever shipped were iron-only.
- [ ] CHANGELOG complete from v0.1.0; ADRs written for every ⛔ invariant now living in comments.
      ⚠ Two ADRs exist; the *Rules that outlive their milestone* section below is the shortlist of
      what still deserves one.
- [ ] Security audit pass (`docs/audit/YYYY-MM-DD-audit.md`) — ⛔ **nothing exists yet**, and crab
      now writes to the filesystem, spawns processes and deletes trees. This is the criterion that
      moved furthest while nobody was looking at it.
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
- **The idle poll's per-cycle buffer** (*deferral #09*). The steady-state frame allocates nothing,
  but an empty poll still costs a buffer. **Gate: dhancha** — hoist it, or accept a caller-owned one.
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
  ⛔ **Gate: dhancha** GRID / COLUMNS widgets — verified absent (`grep '^fn dh_grid_new' → 0`).
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
- ⭐ **Grid view — the dhancha half is BUILT (0.9.25).** `dh_grid_new` / `_add` / `_select` /
  `_move_sel` / `_scroll_to_sel` / `_index_at`, with the wrap arithmetic, the row-wise arrow keys and
  the selection highlight all inside the toolkit — so crab names no colour and re-derives no
  geometry. ⛔ **It earned a kind by dhancha's own rule**, the one 0.9.23 applied when it refused one
  to MENU and SHEET: composing a grid from boxes would make the app paint its own highlight.
  ⚠ **crab's half is not built.** A Grid view needs a view-mode switch, cells built from the readdir
  records, and a decision about thumbnails in cells — ⛔ **which the decode budget constrains**: a
  gallery of 40 images at 256x256 is ~28 MB of permanent spend against a 32 MB session ceiling. An
  icon grid costs nothing; a thumbnail gallery is a budget question before it is a layout one.
- ✅ **Thumbnails — SHIPPED (unreleased).** 64x64, PNG/JPEG/GIF/BMP, decoded off the idle tick and
  box-filtered down. `chitra 1.0.0` is declared.
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
| ~~Grid view~~ | M5 | ✅ **UNGATED as of dhancha 0.9.25** — `GRID` is a real kind now: wrapping layout, cell selection, row-wise arrow keys, keep-visible, hit-test. ⚠ crab must wait for the tag to be pushed before bumping. | 2026-08-31 ⭐ built |
| Columns (miller) view | M5 | **dhancha** COLUMNS — still absent (`dh_columns_new`: 0 hits in `dist/`) | 2026-08-31 ⭐ re-derived |
| ~~Thumbnail PIXELS~~ | M5 | ✅ **SHIPPED** — operator ruled 2026-08-31 and chitra 1.0.0 is declared. Cost paid: host **+537,112 B (+115.2 %)**, agnos **+534,976 (+108.8 %)**. ⛔ The live constraint is not size but that **every decode is permanent** (~2.5x RGBA, no `free()`); two budgets bound it. | 2026-08-31 ⭐ done |
| Proportional text | M5 | **rekha** + dhancha font plumbing; crab calls no `rekha_*` | 2026-08-31 |
| Sidebar | M6 | **dhancha** TREE widget (`dh_tree_new`: 0 hits) + a drawer overlay for the small window | 2026-08-31 ⭐ re-derived |
| Menu bar | M6 | **dhancha** MENU BAR — ⚠ *not* the popup MENU, which shipped in 0.9.23 | 2026-08-31 |
| Local index · tags · smart folders | M7 | **daimon** — and crab declares no daimon dep at all | 2026-08-31 |
| Duplicate detection | M7 | **daimon**, or a content hash crab could do alone | 2026-08-31 |
| Assisted search | M8 | **daimon** local-only embedding | 2026-08-31 |
| The idle poll's per-cycle buffer | M2 residue | **dhancha** — hoist it or take a caller-owned one | 2026-08-31 |

**Ungated and available now**, listed because a gated table invites the assumption that everything is:
~~preview pane (M5)~~ **shipped** · the 🦀 menu (M6) · smart-folder *plumbing* without the index (M7)
· every item under *Small, cheap, unblocked*.
⚠ **Thumbnails have moved OUT of this list** — not because a dependency is missing, but because the
one that exists costs **+117 %** of the binary. That is a decision, and a decision is not the same
kind of thing as a gate. It has its own row in the table above so it stops being read as free.

⚠ **daimon is the one to settle first.** Three milestones name it, `cyrius.cyml` declares it nowhere,
and **daimon 2.1.2 exists locally** with vector/RAG stores. *Deferral #18* has been open since the
roadmap was written: **declare the dependency or stop promising the AI arc.**

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

- ⛔ **`src/render_test.cyr`'s pixel assertions never run in CI** — **35** of them now, not ten.
  It is crab's strongest test and CI runs `cyrius test`, which does not discover it. *Deferral #14.*
  ⛔⛔ **THIS STOPPED BEING THEORETICAL IN THE UNRELEASED WORK.** Of the seven mutations planted
  against the render-state record and the preview column, **four were caught ONLY by `render_test`**
  — the suite CI runs was green for all four. A refactor that swapped the two pane blocks, or a
  preview column painted over panes that never gave up their width, both ship through CI today.
  ⭐ It at least **reports its own check count now**, so a run that silently skipped assertions is
  visible; it used to exit 0 whether it ran 26 checks or none.
- ⛔ **CI never builds `--agnos`** — the target `state.md` calls "the real target". Every
  `#ifdef CYRIUS_TARGET_AGNOS` region is uncompiled by the gate. *Deferral #15.*
  ⛔⛔ **THIS STOPPED BEING HYPOTHETICAL ON 2026-08-27.** With crab's `path` overrides disabled and a
  fresh resolve against the DECLARED graph (dhancha 0.9.17, which predates `POINTER_SCROLL`):
  **host build PASS · `--agnos` build FAIL — `undefined variable 'POINTER_SCROLL'`.** A commit in that
  state goes green through CI and cannot build the thing that ships. ⇒ #15 is not a tidiness item; it
  is the gate that would have caught a broken dependency graph, and it is the reason `path` currently
  masks one.
- CI runs neither `fuzz`, `bench`, `lint`, `fmt --check`, `vet`, `deny` nor `coverage`. *Deferral #36.*
- ✅ **CLOSED (unreleased) — the fuzz harness reads its input.** *Deferral #12.* 60,000 deterministic
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
