# crab — Roadmap

> Milestone plan through v1.0. State lives in [`state.md`](state.md); this file is the
> **sequencing** — what ships, in what order, against what dependency gates.
>
> ⭐ Starting a slot? Read **[The ladder to 1.0](#the-ladder-to-10--what-ships-next-in-order)** — one
> ordered list of what ships next, **each named by the version it will be cut as**, with what an
> operator can newly do, what blocks it and what closes it. Everything else in this file is the
> reasoning behind those releases.
> `handoff.md` has the current state; [`../../CHANGELOG.md`](../../CHANGELOG.md) has what already
> shipped and why.
>
> ⛔⛔ **A MILESTONE IS WHAT AN OPERATOR CAN NEWLY DO. IT IS NOT A DEFECT LEDGER AND IT IS NOT A
> CHORE LIST.** This file has been both. It carried numbered `#NN` deferral tables — rows for things
> already fixed, rows for things that were never bugs, plus a running "corrections to entries in
> this file" section: changelog material wearing a roadmap's clothes. ⛔ **And gates, automation and
> doc chores were then given VERSION NUMBERS of their own, which is worse** — a release says what
> crab can now do, and "the CI runs more steps" is not that.
> ⇒ **Milestones carry features. [Cross-cutting](#cross-cutting-not-a-milestone--do-these-continuously)
> carries everything else and gets no version.** Fixed defects live in the CHANGELOG, release by
> release, described by what they were rather than by a number.

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
   It reads surface/text/accent from the shared **rupa** tokens through `dh_theme_*` and declares
   only which parts of the UI accent is permitted to tint. The light and dark shells in the canvas
   are two **compositor states**, not two crab settings. Any "add a dark mode toggle" request is out
   of scope by construction — see [ADR 0001](../adr/0001-compositor-owns-theming.md).
   ⚠ **THERE IS NO WIRE, AND THE TITLE OVERSTATES WHAT SHIPS.** rupa's active theme is per-process
   and defaults to MUDRA dark, so crab and aethersafha agree by **sharing a default**, not because
   one hands the other a theme — no setu kind carries one. A compositor-driven theme change is a
   rupa/setu question nobody has opened. ⭐ The invariant itself holds: crab names no colour
   (`grep -nE '0x[0-9A-Fa-f]{6}' src/ui.cyr` → 0, against 24 `dh_theme_*` calls).

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

- [ ] All three canvas directions absorbed — **1a shell is in** (M1–M4) and **1c density is in**
      (M5–M6; ✅ **columns closed at 0.8.8**, the 🦀 button is **0.9.1**); 1b assisted search is M8.
      ⭐ The order the rest of this lands in is [the ladder to 1.0](#the-ladder-to-10--what-ships-next-in-order).
- [x] Reference coverage ≥ 80 % — **89 %** (2026-09-13; 87 % at 0.7.7). ⚠ It has fallen below the line twice
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
      ⚠ The CHANGELOG half is **done** — unbroken from `[0.1.0]`. **Three** ADRs exist; the *Rules
      that outlive their milestone* section below is the shortlist of what still deserves one.
- [x] Security audit pass — ✅ **[`docs/audit/2026-08-31-audit.md`](../audit/2026-08-31-audit.md)**,
      the first. Four findings, ranked by what an attacker gets.
      ⛔ **F1 was the trust-model finding, and it CLOSED in 0.7.6**: the gallery's idle walk covers
      only the visible range plus a row of overscan (`crab_grid_visible`) — *scrolling is consent;
      opening a folder is not.* ⚠ What does not close: crab still runs **~22,500 lines of
      third-party parser** (`chitra` + `sankoch`) in-process on attacker-chosen bytes, over a bump
      allocator with no guard pages. The budgets bound memory and the visible range bounds reach;
      neither bounds what one crafted visible file can do.
      ⛔⛔ **One finding in the first draft was WRONG and is kept with its correction** — it claimed
      an unbounded read `crab_batch_name` cannot perform. Caught by planting the mutation the finding
      implied and watching the suite stay green, then disproved empirically with a poison tail.
      *An audit that reports a bug which is not there spends the reader's trust.*
      ⚠ Re-run it when the trust model moves again — a new parser, a new dependency, or a view that
      widens what gets read.
- [ ] `docs/examples/` populated — still empty (`.gitkeep` only, checked 2026-09-13).

---

## Milestones

### Shipped — M1 through M6 (v0.5.0 … v0.8.8)

> ⚠ **COLLAPSED TWICE, FOR THE SAME REASON.** M1–M4 were once 516 of this file's 763 lines, and
> M5–M6 were 140 of its 631 — shipped-feature narrative, some of it still carrying `(unreleased)`
> labels that outlived their own tags. **A milestone that shipped belongs in the CHANGELOG, and its
> reasoning belongs in the source comment beside the code it explains.** What a collapse keeps is
> what is NOT done (below) and the lessons that still govern (*Cross-cutting*). **This file is the
> sequencing.**

| milestone | version | what shipped |
|---|---|---|
| **M1** Hardening | 0.5.0 | the P-1 sweep: bounded path helpers, an event loop that does not end itself, a size ladder that cannot overflow; `src/path.cyr` extracted so the suite can reach it |
| **M1.5** The allocation gate | 0.6.0 | **a rendered frame costs the global heap ZERO bytes** (746,440 → 0, across dhancha 0.9.13–0.9.15); `src/app.cyr` extracted from `main.cyr`, which ends in `_entry()` and so could never be tested |
| **M2** The window is real | 0.6.1 | resize, pointer (click, click-to-focus, double-click), the wheel, held-key repeat, `SETU_SURF_FULL_KEYS` |
| **M3** A browser you would use | 0.7.0 – 0.7.1 | sorting, selection memory, argv start paths, deferred statting, real columns, directories past the cap (`#101 readdir_at`), the readable selection (`on-accent`) — ⚠ four items at 0.7.0; real columns, the cap and `on-accent` closed their upstream gates at **0.7.1** |
| **M4** File operations | 0.7.1 – 0.7.5, **0.8.7** | copy · move · delete · open · **mkdir · rename**; empty-pane states; the small-window ratification; drag between panes; the transfer tray with rate and ETA; multi-select; **recursive copy and delete**; the context menu; the batch-rename sheet — and at **0.8.7** the **overwrite policy** (ask per collision, with an `all` arm) that the write layer had shipped without |
| **M5** Views | 0.7.6, **0.8.8** | the preview column with real metadata · header-only image dimensions (no decoder) · **thumbnails** (chitra) · EXIF camera + shot · the **GRID** and **GALLERY** views · the `crab_render` parameter cleanup — and every finding of the first security audit closed. At **0.8.8** the **COLUMNS** view (a view mode of one pane, not N miller panes — the write layer's 0/1 is why) and the **one reader for the 9 px advance** that proportional text will need |
| **M6** Sidebar, volumes, density | 0.8.0 – 0.8.6 | the PLACES sidebar (`b`) and its keyboard route · sidebar **VOLUMES** with capacity bars (agnos `mountlist`#104) · the **menu bar** (`F10`) with `File · Edit · Go · View` and its fit rules · the A/B view switcher · 🦀 Bueller's idle line · pane-header focus · the **pointer routes** (right-click menu, popup pick/dismiss, bar and switcher clicks) · the sidebar's *you are here* marker |

⚠ **M4 rode four PATCH numbers by operator ruling** (0.7.1 – 0.7.5), M5 landed inside `[0.7.6]`, and
M6 spread across `0.8.0 – 0.8.6` — none of them took the `v0.9.x` this file once reserved.
**The milestone→version mapping has now been wrong four times. Re-derive the number at each cut**;
do not trust a heading. ⛔ **And both closed LATE, out of milestone order** — M4's overwrite policy at
0.8.7 and M5's columns at 0.8.8, after all of M6 had shipped. ⇒ *A milestone is a grouping of
features, not a window in time*, which is exactly why the ladder below is ordered by **release** and
not by milestone.

#### What is left of the shipped milestones

Everything else in M1–M6 is done. These are the survivors, each with its reason:

- ✅ **Columns (miller) view — CLOSED (0.8.8).** ⛔ **Never a dhancha gate**; it was gated on
  **crab's own two-pane model**, and the answer was to stop trying to make it a pane arrangement:
  COLUMNS is a **view mode of ONE pane** — the active listing plus a narrow CONTEXT column showing
  the parent with the current directory marked. ⛔⛔ **That is a safety decision, not a layout one.**
  N independently navigable miller panes make `active_pane` something other than a 0/1, and
  `active_pane` is a 0/1 that the entire M4 **write layer** resolves every copy, move and delete
  against: a drag from column k into column k+1 would plan a `crab_fs_move` of a directory **into its
  own subtree**. K = 2 with one driven column answers that by construction. ⇒ If miller ever goes
  N-deep, **the gate is the write layer, not the renderer**.
  ⭐ Proven on QEMU (`crab-columns-test.py`) — the listing itself is inside the agnos `#ifdef` and no
  host test can reach it.
- ✅ **Proportional text — CLOSED (0.9.0).** crab draws in **Liberation Sans**, read from the kernel's
  own `/fonts/default.ttf`, and the `~` marker finally measures instead of counting.
  ⛔⛆ **The coincidence worth knowing before anyone "verifies" this**: Liberation Sans's `n` is
  1139/2048 em, which at 16 px rounds to **exactly kashi's 9** — so every derived column width comes
  out numerically identical to the bitmap face's. A check that asserted "the advance changed" would
  fail against a working face. The proof is `i=4 m=13`: *the columns did not move; what goes in them
  did.* What follows is the record of how it was carried before it closed.
- **Proportional text** — M5's other, and **half of the crab side closed in 0.8.8**. ⭐ The upstream
  half closed 2026-09-02 (rekha 0.3.6's `rekha_advance_width`, dhancha 0.9.27's `dh_text_advance`).
  ⭐ **0.8.8 gave the 9 ONE reader**: `crab_char_w()` answers `CRAB_COL_CHARW` for the bitmap font and
  the FONT's own advance for anything else, `crab_text_w()` **measures** a string rather than pricing
  it at `length × advance`, and every divide now goes through them. crab still passes `font = 0`, so
  it renders identically — which is exactly why that change was made on its own.
  ⛔ **What is left is the face itself**, and it is load-bearing correctness rather than cosmetics:
  `crab_col_chars` decides how many characters a NAME column holds, which drives the `~` truncation
  rule that exists so two different files never render as one identical row. Under a proportional
  face `px / advance` over-reports and the marker stops being honest.
  ⚠ `CRAB_COL_NAME_MIN`, `CRAB_COL_SIZE_W`, `CRAB_COL_MTIME_W`, `CRAB_PV_W`, `CRAB_GRID_CELL_W` and
  `CRAB_GAL_CELL_W` all encode character counts at 9 px, and they pick the **wrong layout** rather
  than failing honestly. ⚠ It also invalidates the caret arithmetic (`dh_draw_widget_ink` advances a
  fixed 9 px) and the scalable path is Latin-1 only (`load8(text + i)`).
- ✅ **The door — CLOSED (0.9.1); the 🦀 GLYPH is not.** A mark in the status line opens the menu
  row, so `F10` is no longer the only way in — and for seven releases it was no way at all, because
  aethersafha claimed that key until 0.16.25. ⛔ It costs **zero rows**: the status line became a
  `BOX_H` of [door][text], the shape `crab_pane` already uses for the A/B strip. It uses **no font**
  — three filled boxes — which also sidesteps something 0.9.0 made live: the same byte draws CP437
  through kashi and Latin-1 through rekha, so a mark made of characters renders differently on the
  host build and the target.
  ⛔⛆ **AND THE GLYPH IS STILL CLOSED — this bullet said proportional text would open it, and 0.9.0
  shipped proportional text.** That was wrong three times over: `rekha_char_to_glyph` returns 0 above
  U+FFFF, the face has no format-12 cmap, and the draw loop walks one byte per glyph. ⭐ **CANVAS is
  the open road** — crab already draws thumbnails through `dh_canvas_new`, and an icon is a glyph
  with no font.
- **Three key spaces, four repos, and crab reads raw wire numbers.** aethersafha forwards
  `bhumi_key_usage(ev)` — an **HID usage** — unchanged. dhancha's `DhKey` constants are puka's
  **ASCII/Unicode sym** space, not evdev (`dhancha/src/event.cyr:61-62`; the word *evdev* appears
  nowhere in dhancha). puka translates HID→**evdev** for its own terminal (`setuwin__hid_to_evdev`).
  crab reads the raw usage and is correct **because it never calls `dh_dispatch`**; anything that
  starts to would inherit the mismatch. ⇒ **Fix it in one place or document it in three.**
  ⚠ This bullet said "dhancha's constants are evdev" for four releases; they never were.

### The ladder to 1.0 — what ships next, in order

> ⛔⛆ **THIS SECTION EXISTS BECAUSE "WHAT IS NEXT" KEPT BEING A READING EXERCISE.** The remaining work
> was spread across *What is left of the shipped milestones*, *Absent affordances*, *Recorded as
> facts*, M7, M8 and the gate table — every item correct, none of them in an order, so each slot
> began by re-deriving the sequence from six places. ⇒ **One ordered list, and every entry is a
> VERSION** — named by what an operator can newly do when that version is cut.
>
> ⛔⛔ **THE ORDER IS THE COMMITMENT. THE NUMBER IS NOT.** This file has mapped milestones onto
> versions wrong **four times** — M4 rode four patch numbers, M5 landed inside one, M6 spread across
> seven, and the `v0.9.x` / `v0.10.0` these sections once reserved never happened. The numbers below
> are the *intended shape*, not a promise: **re-derive the number at the cut** from what the release
> actually contains. What does not move is the sequence and the reason each release sits where it does.
>
> ⚠ **Nothing in [Cross-cutting](#cross-cutting-not-a-milestone--do-these-continuously) appears
> here**, by the rule at the top of this file: gates, automation, lint policy and doc currency are
> how crab keeps working, not what crab ships. They ride along with whichever release is in flight.

> ⚠ **CALL THEM BY THEIR VERSION.** The left column is a reading aid for the dependency notes below
> the table and nothing else — `0.9.0` is the name of the next face, not "rung 2". A number an
> operator can put against a release is the only identifier worth using in a report.

| | version | what an operator can newly do | blocked by | cut it when |
|---|---|---|---|---|
| ✅ | **0.8.8 · Columns** | press `g` to a fourth view: the listing plus a context column naming where it sits | — | **shipped** |
| ✅ | **0.8.9 · Shift** | type a **capital letter** into a name — and `#` and `*`, the batch sheet's own two operators, into the field that advertises them | — | **shipped** |
| ✅ | **0.8.10 · Ready for a face** | *(nothing visible — it is the half of 0.9.0 that is not blocked)* every width crab computes is **derived from the font** instead of from kashi's 9 px, and the suite proves it against a synthetic **proportional** face | — | **shipped** |
| ✅ | **0.9.0 · A real face** | **read crab in a proportional font** — Liberation Sans, from the kernel's own `/fonts/default.ttf` | — | **shipped** |
| ✅ | **0.9.1 · The door** | **open the menu row with the pointer** — a mark in the status line, so `F10` is not the only way in | — | **shipped** ⚠ *not a crab GLYPH — that is closed at three independent levels; see below* |
| ✅ | **0.9.2 · `Go`** | **jump to a place from the menu bar** — every sidebar destination, picked through one navigator | — | **shipped** ⭐ *and it closed a key leak worse than the one recorded* |
| ✅ | **0.9.3 · Symlinks** | **see that a link is a link** — marked `@`, KIND says Link — and know what each verb does with one | — | **shipped** ([ADR 0004](../adr/0004-symlinks-are-shown-preserved-and-dereferenced-on-copy.md)) |
| ✅ | **0.9.4 · A preview that costs nothing** | arrow through a directory of large JPEGs without paying per entry — and see the picture of the file you are actually on | — | **shipped** ⛔ *and it found TWO live wrong-on-screen defects: the thumbnail lagged one file behind, and CAMERA persisted onto text files* |
| ✅ | **0.9.5 · Pointer polish** | **mark a row with the middle button**, and see a popup's highlight follow the pointer | — | **shipped** ⛔ *and the premise was wrong: middle had DISMISSED popups since 0.8.5, in three doc sites' teeth* |
| ✅ | **the daimon decision** | *(not a release — a ruling)* | — | **RULED 2026-09-14: DECLARED.** `cyrius.cyml` carries `[deps.daimon]` at 2.1.3. ⛔ Declared, NOT linked — daimon is a binary with no `dist/`; crab talks to its AF_UNIX socket. 0.10.0 and 0.11.0 are unblocked. |
| ◐ | **0.10.0 · The index** (M7) | ⭐ **find duplicates** (`Shift+D`, marks all but the newest) — tags and smart folders still to come | ✅ **daimon DECLARED 2.1.3** | the index is local, background, battery-aware, and the four smart folders are real |
| | **0.11.0 · Assisted search** (M8) | ask in words and get ranked results that say **why** they matched | **daimon** local-only embedding | the query bar, the MATCH column, WHY IT MATCHED / APPEARS IN, dupes-in-set, and `SAVE AS → Smart folder…` |
| ✅ | **0.9.6 · A drag cannot outlive its listing** | *(a fix release)* drag without a drop moving a file a prompt was asking about | — | **shipped** ⛔⛔ *two data-loss defects: no modal gate on the drop, and a row index that outlived its listing* |
| ✅ | **0.9.7 · H1** | *(a fix release)* cancel a merged copy without losing the folder it merged into | — | **shipped** ⛔⛔⛔ *the oldest confirmed data-loss defect; MEASURED on iron both ways (0/3 → 3/3)* |
| 🏁 | **1.0.0** | — | every box in [v1.0 criteria](#v10-criteria) | see below |

⭐⭐ **0.9.x IS COMPLETE AND ALL THREE KNOWN DATA-LOSS DEFECTS ARE CLOSED** — H1 (0.9.7) and the two
drag defects (0.9.6), each measured on iron both ways. What remains before 1.0 is **the daimon
ruling**, then 0.10.0 → 0.11.0. ⛔ The ruling is the operator's and nothing downstream can start
without it: `0.10.0` and `0.11.0` are both gated on it. The one chain left is **0.10.0 → 0.11.0, behind one ruling**. ⚠ 0.9.1 was
chained to 0.9.0 (the face, then the glyph that needs it); **0.9.0 shipped, so that chain is gone**.

### ✅ What blocked `0.9.0 · A real face`, and how both cleared in a day

Found by 0.8.10 asking where a face would come from **before** writing the code that loads one. Both
were real, both were filed in the repo that owned the fix, and **both closed within 24 hours**. Kept
as the record, because the pattern is the point: *state the need, decline to approximate it, and let
the owning repo choose the mechanism.*

1. ✅ **THERE WAS NO TRUETYPE FACE IN THE STACK — CLOSED by agnos 1.57.2 + rekha 0.3.8.** A search
   across nineteen first-party repos returned zero `*.ttf`; the `agnos` repo contained no occurrence
   of "ttf", "truetype" or "sfnt" anywhere. ⭐ **The operator's ruling** — *"rekha is that thing...
   but has yet to get Kernel support"* — named the answer, and agnos shipped it the same day as a
   **kernel-owned `/fonts` namespace**: the face embedded kashi-style, assembled at boot into a 2 MB
   direct-map region and **verified by FNV-1a-64 against the generator's hash** before it is exposed.
   **Liberation Sans Regular 2.1.5, unmodified, 410,820 bytes, SIL OFL 1.1** — ⚠ the licence text must
   travel with any redistribution.
   ⇒ **`/fonts/default.ttf`** is the stable contract; `/fonts/LiberationSans-Regular.ttf` is the same
   bytes under the provenance name. Read-only, a `VFS_MEMFILE` fd, ⛔ **`lseek` is -1 — read it
   front-to-back in one pass**. Contract: `agnos-userland-abi.md` §3.5.
   ⭐⭐ **THE FILING'S POSTURE IS WHY THIS IS THE RIGHT MECHANISM AND NOT A STAGED FILE.** It stated
   the need in one sentence and explicitly declined to design agnos's answer, citing VOLUMES. agnos
   then chose something the filing had not imagined and would not have asked for.
2. ✅ **dhancha's SCALABLE PATH ALLOCATED PER CALL — CLOSED by dhancha 0.10.0.** Three moves:
   **sadish 0.5.5** put every per-call `alloc(` behind an `sd_alloc` / `sd_alloc_set` hook and added
   `sd_canvas_blit_at`; **rekha 0.3.10** draws its outline scratch from the same seam; **dhancha
   0.10.0** installs `dh_falloc` as that hook for one `dh_draw_text_ink` and sizes the canvas to
   clip ∩ surface ∩ run rather than to the surface.
   ⭐ **crab's expiry assertion fired exactly as written and is now inverted**: a warm frame under a
   real face costs the global heap **exactly 0**. ⚠ And dhancha's hand-off **predicted crab's failure
   by name and to the byte** from measurements taken against crab 0.8.10 — not `scost > 0` but
   `arena_capacity_total(farena) == cap0`, *got 468,040, expected 16,384*. That is what a filing with
   a gate behind it buys.

⛔ **Three things crab must honour when it loads the face**, all of them from dhancha's hand-off and
all already reflected in the suite: pin **sadish >= 0.5.5** and **rekha >= 0.3.10** (without them the
build is REFUSED — `sd_alloc_set` and `sd_canvas_blit_at` are undefined); render **one warm-up face
frame** at the widest run before measuring, because a cold one chains ~452 KB of arena chunks (the
arena growing, not a leak); and open the face **outside** a draw, since `rekha_font_open` follows the
scoped hook and would die at the arena's first reset.

### M7 — The index

⚠ **0.10.0 on the ladder above, behind the daimon ruling.** The `v0.10.0` this section once reserved
is not a promise — M5 landed inside a patch and M6 across seven. Re-derive the number at the cut.

- **Local index** — `Local · 41,208 files`, `index fresh`, background indexing that
  `pauses on battery`. **Gate: daimon.**
- **Tags** — manual and suggested (`SUGGESTED TAGS · src → + toolchain + cyrius + wip`).
- **Smart folders** — Recent, Duplicates, Untagged, Large & old, Raw only, Unrated.
- ✅ **Duplicate detection — SHIPPED (0.10.0).** `Shift+D` groups by content and marks all but the
  newest. ⭐ Done WITHOUT daimon, as this row always allowed: size is a free pre-filter (every entry
  is already stat'ed), so only a size collision is opened and hashed. ⛔ It MARKS; it does not delete.
  ⚠ Bounded 64 KiB read and a 64-bit hash, so `crab_dup_same` also requires equal sizes and crab says
  "duplicate", not "identical". The byte-for-byte compare is the honest next increment.
- ✅ **DECLARED — 2026-09-14, the operator's ruling, and the oldest open item in this file is now
  closed.** `cyrius.cyml` carries `[deps.daimon]` pinned at **2.1.3**; `cyrius deps` resolves 7 deps
  and `--verify` reports 50/50.
  ⛔ **Declared, NOT linked, and that is the shape of the dependency.** daimon is a BINARY
  (`[build] output = "build/daimon"`) and ships no `dist/` — there is no module to fold in, and
  there must not be: linking an agent orchestrator would put its HTTP server, scheduler and
  federation code in crab's address space for the sake of a query. crab talks to the **AF_UNIX
  socket daimon binds per agent** (`agent_ipc_new(agent_id, socket_dir)`), and agnos carries the
  surface: `sock_connect` #47, `sock_listen` #56, `sock_accept` #57.
  ⚠ **No `modules` key**, deliberately — it would make `cyrius deps` fold a file that does not exist.
  The pin records which daimon crab's protocol is written against, which is what a pin is for.
  ⛔ **And crab still runs without it.** The index is an enrichment, not a precondition: a box with no
  daimon lists, copies, moves and deletes exactly as today, and the M7 surfaces must report that the
  index is unavailable rather than failing — the rule the preview already follows for a file it
  cannot decode: say which kind of nothing this is.

### M8 — Assisted search (the v1.0 surface)

⚠ **0.11.0 on the ladder above**, and the last feature release before 1.0.

1b, **as a mode over every view**.

- NL query bar (`invoices from last spring, the paid ones`), `⌘K` from anywhere
- Ranked results with a MATCH column; REFINE facets (Kind · Date · Size · Location)
- **WHY IT MATCHED** and **APPEARS IN** panels
- Dupes-within-result-set grouping
- ⭐ **`no external service` · `index stays on device`** — stated in the canvas UI, and it is a
  promise the implementation must actually keep. **Gate: daimon** local-only embedding.
- `SAVE AS → Smart folder…` closes the loop back to M7.

---

---

## ⛔ Everything gated on another repo, in one table

> **Read this before planning a slot.** A gate is a claim about a *different repository*, and claims
> about other repositories go stale without anything failing. ⇒ **Re-derive before believing.** The
> `verified` column is when the claim was last actually checked against the repo.
>
> ⛔⛔ **SEVEN FALSE GATES ARE ON RECORD, AND THEY WERE WRONG IN FOUR DIFFERENT WAYS.** About
> *existence*: M4's write syscalls (real since agnos 1.41.3), the sidebar's "dhancha TREE" (every
> piece existed), COLUMNS (never dhancha's at all). About *price*: thumbnails ("no image decoder"
> when chitra shipped one — and the real obstacle was a cost no gate line mentioned). About *which
> half was missing*: proportional text (the plumbing existed; advance widths did not). About *where*:
> drag ("gated on nothing" when it was gated, elsewhere). And one was **real but MIS-NAMED**: the
> menu bar needed a horizontal selectable strip, not a `MENU BAR` kind.
> ⇒ **A price is not a gate, and a table of gates invites reading one as the other.**

| item | milestone | gated on | verified |
|---|---|---|---|
| ✅ Proportional text | M5 → **0.9.0, SHIPPED** | ⭐ rekha 0.3.6 added the advance widths; dhancha 0.9.27 consumes them. ⭐ **0.8.8 gave the 9 one reader**, **0.8.10 derived every width from the font** and proved it against a synthetic proportional face. ⛔⛆ **The remaining gate is NOT crab's and this row said "gated on nothing" until 0.8.10 checked**: there is no TrueType face anywhere in the stack and nothing stages one onto the target (**agnos** + an operator licence ruling), and dhancha's scalable draw allocates a full-surface canvas per label per frame outside the arena (**dhancha**). See *What actually blocks 0.9.0*. | 2026-09-13 ⛔ **re-derived — the gate was real and mis-stated** |
| The 🦀 **glyph** | M6 → later | ⛔⛆ **STILL CLOSED, AND 0.9.0 DID NOT OPEN IT — this row said the face would.** Three independent walls: `rekha_char_to_glyph` returns 0 above U+FFFF in its own code (*"format 4 is BMP-only"*), the shipped face carries no format-12 subtable, and `dh_draw_text_ink` walks ONE BYTE per glyph in **both** branches. U+1F980 is 128,896. ⭐ **The open road is CANVAS**, which crab already ships for thumbnails (`dh_canvas_new(&crab_thumb_draw, pix)`) — an icon is a glyph with no font. **0.9.1 shipped the DOOR** (three filled boxes) and left the crab for whoever wants to bake a bitmap. | 2026-09-14 ⛔ **re-derived — the 0.9.0 claim was wrong** |
| Sidebar — SMART FOLDERS + TAGS | M6→M7 → **0.10.0** | ✅ **UNGATED — `[deps.daimon]` 2.1.3 is declared (0.10.0).** Declared, NOT linked: daimon is a binary with no `dist/`, so crab talks to the AF_UNIX socket it binds. | 2026-09-14 |
| Local index · tags · smart folders | M7 → **0.10.0** | ✅ **UNGATED — daimon declared (0.10.0).** ⚠ crab must still run WITHOUT it: the index is an enrichment, not a precondition. | 2026-09-14 |
| Duplicate detection | M7 → **0.10.0** | ✅ **UNGATED — daimon declared.** ⚠ And it was never fully gated: a content hash is something crab could do alone, which is the cheaper first half. | 2026-09-14 |
| Assisted search | M8 → **0.11.0** | ✅ **UNGATED — daimon declared (0.10.0).** The embedding is daimon's; the query bar and the WHY column are crab's. | 2026-09-14 |

⭐ **Closed gates are not listed.** GRID (dhancha 0.9.25), the menu bar's strip (0.9.26), thumbnails
(chitra, an operator ruling on price), sidebar PLACES and VOLUMES (agnos `mountlist`#104, minted
because crab filed rather than approximated), the idle poll's per-cycle buffer (dhancha 0.9.16) and
the compositor's claimed keys (aethersafha 0.16.25) all shipped; their reasoning is in the CHANGELOG
and in *Rules that outlive their milestone*.

⭐⭐ **THE ENTRY TO READ WHEN A GATE IS REAL AND THE TEMPTATION IS TO APPROXIMATE IT — sidebar
VOLUMES.** crab **could** have shipped a probe: the namespace was three fixed prefixes (`/`,
`/mnt/fat`, `/mnt/exfat`) and `statfs` validates a path. It **deliberately did not**, because a probe
hardcodes agnos's namespace into crab's binary, can only confirm strings crab already guessed, and
cannot see the aliasing that lists one volume twice when ext2 is absent. It filed the blocker
(2026-09-02) and shipped nothing meanwhile. agnos took **both** halves of the filing's advice and
credited it by name — that this is *"an enumeration because a probe cannot answer it"*, and that the
answer should **mint `mountlist`#104 rather than widen `mount`#11**, whose unused argument registers
carry stale values rather than 0. ⇒ **Declining to approximate is what got the right primitive
built.** ⛔ *We nearly filed the wrong syscall number: it is `mount`#11, not #23.*

**Ungated and available now**: see [the ladder to 1.0](#the-ladder-to-10--what-ships-next-in-order),
which is now the single ordered answer to "what is next". The overwrite policy (0.8.7) and columns
(0.8.8) closed out of that order, and Shift shipped as **0.8.9**. **Next is `0.9.0` — a real face.**
⚠ **daimon is deliberately NOT in the version order** — it is a ruling the operator owns, and M7/M8
stay gated behind it.

⚠ **daimon is the one to settle first.** Three milestones name it, `cyrius.cyml` declares it nowhere,
and **daimon 2.1.2 exists locally** with vector/RAG stores. **Declare the dependency or stop
promising the AI arc** — open since the roadmap was written.

## Unfinished from earlier stages — what the 2026-09-02 completeness audit left

> ⛔⛆ **THIS SECTION EXISTS BECAUSE "SHIPPED" AND "FINISHED" HAD DRIFTED APART.** A five-probe sweep
> over M1–M6, the v1.0 criteria and every ⚠ marker in `src/` found **38 items** that were done enough
> to ship and never converted into work. ⇒ **The rule it enforces: a limitation noticed while
> shipping is an ITEM, not a comment.**
>
> ✅ **Two of its three batches are CLOSED.** The **eight correctness bugs** went in `[0.8.2]`, each
> mutation-proven — plus a ninth nothing had recorded: **recursive copy and recursive delete had
> never run in any shipped build**, because the idle tick called `crab_copy_step` (the single-file
> chunk loop) instead of `crab_op_step` (the dispatcher), which had **zero callers** under a comment
> reading *"THE single entry point the idle tick calls"*. ⛔⛆ **The suite was green throughout,
> because every walk test drove `crab_op_step` while the only caller that ships drove the wrong
> one.** ⇒ **A test that calls a different function than the shipping caller is not testing the
> shipping path.** The **six M6 interaction gaps** closed across `0.8.3` (the sidebar's keyboard
> route, the menu bar's two fit rules — and `Open`, dead on both menu surfaces because both arms sat
> BELOW the binding table), `0.8.5` (the pointer routes, the *you are here* marker) and `0.8.6`
> (`View`'s items). What follows is the third batch, and it is what is left.

### Absent affordances

- ✅ **REFRESH — CLOSED (2026-09-13), on `u`.** Both panes relist (selection kept by NAME, marks
  cleared, refused out loud during a transfer), PLACES and VOLUMES rebuilt, the sidebar cursor
  re-seated by exact path and section. `View ▸ Refresh`. ⛔ Not F5 — the compositor claimed it, and
  still owns `Ctrl+F5`. ⭐ Proven on QEMU.
- ✅ **A FLAG SURFACE — CLOSED (2026-09-13).** `crab --help` (also `-h`) prints usage, the flags and
  **the key list** — which existed nowhere in writing, since the menus carry six verbs and every
  other binding was discoverable only by being told. `crab --about` closes on the Ben-Stein line, as
  `docs/development/mascot.md` asks. An unknown flag names itself, prints help and exits 2.
  ⛔ **A flag is never ambiguous with a path** — `crab_path_usable` requires an absolute path, so a
  `-x` could only ever have been refused as "left path unusable", which names the wrong problem.
  ⚠ **No `--version`, deliberately**: `VERSION` is the single source of truth and no build-time
  define reaches a source file, so the string would be a second copy that drifts. `--version` is
  honestly UNKNOWN rather than a lie. ⚠ On agnos the surface is unreachable today (the launcher
  spawns `/bin/crab` with no arguments and a shell cannot start it at all) — recorded, not pretended.
- ✅ **AN OVERWRITE POLICY — CLOSED (0.8.7): ask, per collision.** A collision **stops** the walk and
  names the file; `r` replace · `s` skip · `k` keep both · `Esc` stop, with `a` arming an **all** that
  applies the next answer to the rest. ⛔ Directories MERGE — only files ask, because that is where
  the answer destroys something. ⛔ Keep-both suffixes **before** the extension (`report (2).txt`) and
  tries until a free name is found. ⛔ Replace **unlinks first**, the one sequence that means the same
  on both targets (the host is `O_EXCL`, agnos has no `AO_EXCL` and truncates). ⚠ The policy resets
  every run: an armed `replace all` that survived would destroy without asking later.
- ✅ **`Go` — CLOSED (0.9.2), and all four recorded reasons were real defects rather than reasons to
  wait.** Its rows are every sidebar destination, picked through **one navigator** (`synth_goto`,
  mirroring `synth_u`) that the sidebar's own Enter now shares.
  · *"an 11-to-17 row drop-down is clamped and flipped over both the bar and the status line"* —
  `crab_mb_drop_fit_h` tests HEIGHT now, which no menu needed until one was as long as its model.
  **Six rows fit at 380×220**; longer is refused out loud rather than cut.
  · *"`d` is not consumed by the drop arm"* — **worse than recorded**: the arm handled six keys and
  let *every* other one through, so `c`, `m`, `r`, `n`, Backspace and Space all acted under the
  popup too. `crab_mb_drop_key` eats the mutating set, and an assertion ties that set to
  `crab_sb_key`'s so the two surfaces cannot drift.
  · *"two FAT volumes render as two identical rows"* — a volume is labelled by its **prefix**, which
  is unique by construction, rather than by a name two filesystems can share.
  · *"`Go ▸ Parent` would ship enabled-but-dead at `/`"* — **Parent is a VERB** (Backspace), not a
  destination, so it is absent everywhere rather than dead in one place.

### Recorded as facts, never as work

- ✅ **Names can hold capital letters — CLOSED (0.8.9).** This section carried it as a fact for five
  releases: *"`crab_key_char`'s shift flag is hard-coded to 0 at its only production call site."*
  aethersafha 0.16.25 forwards modifier edges on purpose — its own source says *"a client that wants
  Shift state has no other way to learn it"* — so what was missing was a crab-side latch, not a wire.
  `crab_shift_track` folds usages 0xE1/0xE5 into a **two-bit mask** (one bit per shift key: releasing
  one while the other is held must stay shifted, which a boolean loses) and the field reads
  `crab_shift_held()`. ⭐ Proven on QEMU — the ORDER of the latch against the modifier suppression is
  invisible to the suite and would have failed green.
- ✅ **Symlinks — CLOSED (0.9.3), and `sys_lstat` is called now.** ⛔ **readdir could never have told
  crab**: agnos's `ext2_readdir_at_sys` sets byte 63 with `if (ftype == 2) { t = 1; }` — one bit, DIR
  or not — so a link arrived indistinguishable from a file. ⭐ The stat sweep already visits every
  entry, so `lstat` costs **no extra syscall**; the kind goes into the type byte crab already had.
  ⇒ **Delete and move PRESERVE a link** (`unlink`/`rename` act on the link itself); **copy
  DEREFERENCES**, as `cp -r` does, now as a decision rather than an accident. Recreate is possible
  (`symlink`#63 + `readlink`#70, both with peers) and deferred: both are **ext2-only** and crab copies
  between volumes.
  ⛔⛆ **It found a real defect**: `crab_fs_delete` chose `rmdir` on `is_dir != 0`, so the moment the
  type byte had a third value every link would have been **undeletable**.
  ⭐ And the obvious fear is not real — a link cannot make the walk loop, because the walk descends
  only on `1` and a link is `2`.
- ✅ **The preview's synchronous 64 KiB read — CLOSED (0.9.4), and it was the smallest of three.**
  The read is on the idle tick (`crab_pv_step`, one file per tick, gated on `crab_preview_fit` so a
  too-narrow column reads nothing). MEASURED: **4 µs per keystroke, identical with an empty cache** —
  a miss is a lookup, never a read; and arrowing BACK is free, where the one-entry memo re-read
  everything. ⚠ The host figure understates agnos by ~100x: the cheaper *stat* sweep is recorded at
  ~1.1 ms/entry, which is why it was deferred too.
  ⛔⛆ **Moving it exposed two LIVE wrong-on-screen defects**, both measured before any change: the
  **thumbnail lagged one file behind** (the tick's `OK -> OK` gate drew no frame — 0.8.2's bug, obeyed
  by 1 of 8 render sites), and **`CAMERA: Canon EOS R5` stayed under a text file's name**. One root
  cause: the column read process-wide *"last touched"* state instead of the selection. ⇒ It is now a
  pure function of the selected path.
- ✅ **And the layering gate was half a gate — CLOSED (0.9.4), cross-cutting.** `render_test` includes
  `ui.cyr` alone so an up-call fails; an undefined **constant** is `error:` (it caught 0.9.3) but an
  undefined **function** is only `warning:` and the build exits 0. `ci.yml` fails on the warning now.
- ✅ **The batch-rename operators can be typed — CLOSED (0.8.9), and by the Shift work rather than by
  anything aimed at it.** The sheet advertised a language its own field could not produce: `#` is
  Shift+3 and `*` is Shift+8, and the whole shifted number row answered 0 under a comment reading
  *"the shifted row is symbols crab does not need"* — while crab needed two of them by name. The row
  is filled, all ten, and the suite pins `#` and `*` against `crab_batch_name` itself rather than
  against two literals, so the keyboard and the language cannot drift apart.
- ✅ **Pointer polish — CLOSED (0.9.5), and the premise was wrong.** The middle button never "did
  nothing": both `CRAB_PA_DISMISS` returns are button-BLIND, so it has closed popups since 0.8.5 —
  unasserted, while three doc sites said the opposite. It now also **marks the row under the pointer**,
  an ALIAS of `Space` rather than a verb of its own. ⛔ Chosen as the safest binding available: X11 is
  left/**middle**/right, so X11 muscle memory aims middle where crab's RIGHT lives and `Delete` is a
  row in that menu; a mark is self-inverse. **Button 3 observed on iron for the first time anywhere in
  this stack.**
  ⭐ The popup highlight follows the pointer, armed only after it MOVES (the popup is placed at the
  cursor and may FLIP above it), and OUTSIDE the popup restores the open-time choice rather than
  holding the last row swept.
- ✅ **And it fixed a live defect neither half was aimed at**: the context menu opened on an EMPTY pane
  with **no highlight at all** — `menu_sel = 0` is `CRAB_MI_OPEN`, which is greyed when the pane is
  empty, so `dh_list_select` refused it and `Enter` did nothing. `crab_menu_first` is the twin the
  menu bar's drop-down has had since 0.8.6.
- **Compositor-side focus is a left-button gesture**, so a right-click reaches an unfocused crab
  without focusing its window — aethersafha's policy, not crab's.

## Cross-cutting (not a milestone — do these continuously)

> ⛔⛔ **NOTHING IN THIS SECTION IS A RELEASE, AND IT MUST NEVER BE GIVEN A VERSION NUMBER.** Gates,
> automation, lint policy and doc currency are how crab keeps working; they are not what crab ships.
> A version says what an operator can now DO. ⇒ This work rides along with whatever milestone is in
> flight — it does not get a slot of its own.

### Rules that outlive their milestone

Pulled out of M1–M4 when those sections collapsed, because each still governs work not yet written.

- ⛔⛔ **crab OWNS ITS INTERACTION STATE — it does not use `dh_dispatch`.** Operator ruling
  2026-08-27. dhancha tracks a press as a **widget pointer**, and crab rebuilds its whole tree every
  frame with the arena rewinding underneath it. crab tracks **pane index + row index** instead.
  ⚠ **It cost three dhancha features**: `dh_dispatch` itself, drag (`_dh_drag_src`) and `TEXTINPUT`
  (a per-widget buffer on an arena'd widget — 0.7.5 owns the edit buffer instead). **Three features,
  one assumption: a retained tree.** ⭐ **dhancha 0.9.24 fixed the CAUSE** with stable widget keys
  (`dh_widget_set_key` + `dh_surface_set_root` re-binding), so all three are usable under an arena
  now — and crab uses none of them. Adopting `dh_dispatch` would relitigate the ruling, which is the
  operator's call and not a consequence of the fix. ⚠ crab would also inherit a key-space mismatch:
  `DhKey` is puka's ASCII sym space and crab reads HID usages.
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
  ⭐ **The remedy is to MOVE THE FUNCTION DOWN, not to thread a parameter** — the fourth instance
  (2026-09-13, the *you are here* marker needing `crab_path_within`, which lived in `app.cyr`) was
  fixed by moving it to `path.cyr`, where its only other caller — the copy-into-itself guard —
  reaches it unchanged. One containment rule, two callers, no copy.
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

**Still to automate — none of it a release:**

- **The `path`-wins-over-`tag` re-verification.** 0.4.13 shipped a manifest naming a library the
  build never compiled, and only a full resolve with the overrides disabled catches that class.
  ⛔ A `path` dep gets no `cyrius.lock` commit pin, so a declared-tag/vendored-record divergence is
  invisible to every other gate — that shipped in 0.7.6. The tell is the commit-pinned count: 3 with
  the overrides on, 7 with them off.
- **Make the toolchain sync walk the WHOLE vendored tree.** `cyrius lib sync` walks only the declared
  `[deps].stdlib` set, so transitive leaves go stale across a bump silently. At the 6.5.41 bump it
  left three `thread_*` leaves behind; `lib/atomic.cyr` escaped only because it happens to be
  byte-identical between the two versions. A bump can also ADD a leaf — 6.5.41 brought
  `lib/hashseed.cyr` in untracked and grew the lock from 48 to 49. Reconcile the whole tree,
  `git status lib/` **and** the lock count.
- **Enforce `state.md` currency.** It has rotted three times, once across eleven releases, and the
  file diagnoses why itself: nothing gates it. The same absence let `handoff.md` carry a table six
  rows wrong and `README.md` § Status be wrong in both directions.
- **Decide whether `cyrius lint --strict` becomes a gate.** It works (exit 2). The price is
  reformatting **147** over-long lines (>120 cols, measured 2026-09-13) — **17** in `src/main.cyr`,
  **119** in `tests/crab.tcyr`, **11** in `tests/crab.fcyr`. ⚠ It was **83** at 0.7.7 and has nearly
  doubled since, almost entirely in the SUITE rather than the event loop — so the "widest mechanical
  edit in the most sensitive file" framing no longer holds; the cost is now mostly in tests.
  **Re-count at the cut, never quote this number.** ⛔ Its own change, suite run before and after.
- **Bring the agnos/iron harness into this repo.** Both real defects crab has ever shipped were
  agnos-runtime behaviour no host test can see. ⭐ Three more joined the pair in
  `agnos/scripts/harness/` on 2026-09-13 — `crab-pointer-test.py` (the pointer routes, the refresh
  key, and the chrome-key contract as a **gate**: bare F10 must open crab's bar, bare Esc must not
  quit, Ctrl+Q must), `crab-columns-test.py` (0.8.8: `g` reaches the fourth view, the parent is
  listed and named, **and the memo holds** — redraws must not re-list, which is the arm that keeps a
  readdir off the render path) and `crab-shift-test.py` (0.8.9: `Shift+A Shift+B Shift+3 Shift+8`
  must commit exactly `AB#*`, which gates the latch, **the ORDER of the latch against the modifier
  suppression** — invisible to the suite and green either way — and both batch operators at once).
  Still there, still not here.
  ⚠ **The shift harness's first run measured the harness, not crab**: a blind `n` retry typed literal
  `n`s into the name it was asserting on. That is what earned crab's `crab: edit open <label>` line.
  ⇒ *A retry with no oracle is a mutation the harness performs on its own subject.* ⚠ Its header carries seven lessons about driving
  crab under QEMU — the Enter burst spawns a second compositor; DOWN bursts wrap; hold keys across
  the per-frame drain; the pin needs its own frame or the cursor folds to (0,0); derive where to
  press to MISS a popup; a pick can open a sheet whose scrim blocks the pointer; report which verb
  ran rather than assuming — and every one of them cost a QEMU run to learn.

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
- ⚠ **Still uncovered by the fuzzer**: `crab_readdir_into` (agnos-only, needs a syscall) and the
  write layer's path join (`crab_join_n`). ✅ `crab_batch_name`'s pattern expansion IS covered — a
  round drives arbitrary patterns with `*` and `#` biased in, and holds the NUL-inside-cap contract
  on the refusal path as well as the accepted one.
- ⚠ **No host test can see the agnos event loop**, and both real defects crab has ever shipped were
  agnos-runtime behaviour. ⭐ **QEMU does see it, and it is run now** — three harnesses drive crab on
  a real kernel (listing, resize, pointer). ⛔ **Iron still has not seen anything since 0.7.0**, and
  QEMU is explicitly not a control for timing- or pressure-dependent behaviour: the harness README
  records a lossy-queue failure that killed a client on iron and reproduced not at all under QEMU.

### Documentation debt

- ⛔ **Re-read `README.md` § Status at every release.** It has been wrong in **both** directions —
  "Scaffold." long after the GUI shipped, then "read-only" long after the write layer did — and the
  cause was identical both times: written once, correctly, and never re-read at a cut. Nothing gates
  it, which is the same absence that let `state.md` rot three times.
- **Write ADRs for the ⛔ invariants that still live only in source comments.** A comment dies with
  the line it annotates. The shortlist is *Rules that outlive their milestone*, above; three ADRs
  exist so far.
- **`docs/benchmarks.md`, written from the bench harness.** A v1.0 criterion, and the harness half is
  done — it measures the sort at 1024, at 256 and at the real iron 122. Nothing writes the document.
  ⚠ Record the machine, `cycc --version` and the run-to-run spread; merge/256 was seen swinging
  87.4 → 92.4 µs a minute apart, and a bare number becomes the next stale claim.
- **`docs/examples/`** — declared, holds only `.gitkeep`, and a v1.0 criterion.

### Small, cheap, unblocked

None of these is a milestone or a release; each is one change, ridden along with whatever is in
flight.

- **Drop the redundant `net` stdlib declaration** (still in `cyrius.cyml`, checked 2026-09-13).
  setu removed TCP at 0.8.4 and crab has pinned past it since 0.4.5. Removal is measured clean — `cyrius deps` re-creates the leaf from setu's sidecar
  and the binary is the same size. ⚠ The leaf lands at a different concatenation offset, so ~165 KB
  *differs* at that same size: record it as "same size, same tests, different layout" or the next
  reader thinks something broke.
- **Give the stat trace an arm that works where it is needed.** On agnos the compositor spawns crab,
  so `CRAB_STAT_TRACE=1` set in a shell never reaches it.
- **Close or formally park the `--win` failure.** `sys_socket` / `sys_connect` are absent from the
  Windows syscall table and nothing in crab causes it; Windows is not a declared target. Park it in
  writing or stop listing it.
- **Correct the CHANGELOG's harvested-deferral count.** `[0.5.0]` records *"39 deferrals were
  harvested"*; 34 numbers were ever written down. Five were assigned and never transcribed and no
  subject can be recovered — say so, rather than leaving a reader hunting them.

---

## Out of scope for v1.0

- **A theme switcher, a palette, or any crab-owned colour UI** — the compositor owns theming. This is
  the most likely thing to be asked for and the answer is architectural, not a preference.
- **Windows as a target** — `--win` fails on two absent syscall stubs and nothing in crab causes it.
  crab is an AGNOS desktop application.
- **Network / remote filesystems** — crab is local-first by design.
- **A plugin system.**
