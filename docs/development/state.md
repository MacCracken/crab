# crab — Current State

> Refreshed every release. CLAUDE.md is preferences/process/procedures
> (durable); this file is **state** (volatile).
>
> ⭐ **Picking the work up cold? Read [`handoff.md`](handoff.md) first** — it carries what is
> verified vs merely built, the open QEMU verification, and the one ⛔ that must be read before
> touching the agnos event loop.
>
> ⛔ **This file described an empty scaffold ("0.1.0, no releases yet, initial scaffold only") until
> 2026-08-02, five releases after that stopped being true.** A state file that is wrong is worse than a
> missing one, because it is what a cold start reads first.
>
> ⛔ **IT HAPPENED AGAIN, AND THE SECOND TIME WAS WORSE — rot in a detailed file, not neglect of an
> empty one.** From 2026-08-07 to 2026-08-26 this file went untouched across **eleven** releases
> (0.4.4 → 0.4.15) while asserting a `6.5.5` pin (live: `6.5.35`), six dep versions that had **all**
> moved, a `Next` item that shipped in 0.4.6 on 2026-08-08, a retired codename (`anu`), and
> "Never run on iron" — after crab had burned on iron twice. A file confident enough to carry ⛔
> markers is read as authoritative, so the second rot cost more than the first.
>
> ⇒ **Refresh this in the same commit as the CHANGELOG entry, not after it.** Nothing enforces it:
> `cyrius audit` does not gate state currency, so it lives in CLAUDE.md's Process step 5 and nowhere
> else.

## Version

**0.10.1 IS CUT** — `VERSION` reads `0.10.1` and the CHANGELOG header agrees, 2026-09-14.
⚠ 0.10.0 released (tagged). Re-run `git log --oneline -3` and `git tag --list` before restating this.

⛔⛔ **M7's INDEX AND TAGS ARE BLOCKED IN A SIBLING — daimon does not build for agnos.** MEASURED:
`cyrius build --agnos src/main.cyr` on daimon 2.1.3 gives **53 errors, 36 distinct undefined symbols**
— 50 in `lib/syscalls_linux_common.cyr` (a VENDORED stdlib file being compiled for the wrong target,
while `lib/syscalls_x86_64_agnos.cyr`'s own header says it is STANDALONE and does not include it) and
3 in daimon's own `src/agent.cyr` (`SYS_EXECVE`, `SYS_WAIT4`). ⚠ Likely root, not confirmed: daimon
pins cyrius **6.6.2** to crab's **6.6.4** and the two vendored agnos peers DIFFER. ⇒ Filed at
`daimon/docs/development/issues/2026-09-14-daimon-does-not-build-for-agnos.md`; the first failure
class (four `sys_unlink` arity errors) is FIXED there as `daimon_unlink`, prepared and uncommitted —
daimon's own rule forbids bumping its VERSION. ⛔ crab will not fake an index it cannot back.

**0.10.1 contents — the sidebar can reach its own rows.** Two defects found while designing M7's
smart-folder section, cut on their own rather than bundled.
⛔⛆ **(1) FOUR ROWS BELOW THE FOLD, AND NOTHING EVER SCROLLED THE SIDEBAR.** `crab_sblst` appears
seven times in ui.cyr and not one was a scroll call, while the panes and the context column have been
followed for releases. MEASURED: an ordinary desktop is 7 places + 2 headers + 2 volumes = **11 rows**
against **7 that fit** in the shipped 380x220 (174 px band / 22 px rows). `crab_sb_step` walks all
eleven, so the cursor reaches a row the list never shows and Enter sends the pane somewhere the
operator never saw selected. ⚠ Invisible on a headless box: `crab_places_build` stat-checks, so this
box builds THREE places — 7 rows, fitting exactly.
⛔ **(2) A sidebar click that did nothing said nothing** — the pointer arm had no `else` while its
keyboard twin has said "nothing to go to" since 0.9.2. Unreachable today (every row has a path);
reachable the moment one is pathless, which is what a smart folder is. Fixed before the feature.
**2503 / 0**, render_test **55** checks (53 → 55).
⛔⛆ **The scroll fix was UNASSERTED when it first landed and mutation caught it**: deleting the line
left BOTH suites green. The suite proves the arithmetic — the motive — not that the list moves; only a
laid-out tree can, and that lives in `render_test`. Now mutation-proven both ways (exit 1 / exit 0).

**0.10.0 contents.** ⭐⭐ **THE DAIMON RULING: DECLARED.** `[deps.daimon]` at 2.1.3; `cyrius deps`
resolves 7, `--verify` 50/50. This was the OLDEST open item in the roadmap and it gated M7 and M8
entirely. ⛔ **Declared, NOT linked**: daimon is a binary with no `dist/`, so there is no module to
fold and there must not be — crab talks to the AF_UNIX socket it binds per agent, over agnos's
`sock_connect` #47 / `sock_listen` #56 / `sock_accept` #57. No `modules` key, deliberately.
⛔ **crab still runs without it** — the index is an enrichment, not a precondition; the M7 surfaces
report the index unavailable rather than failing.
⭐⭐ **DUPLICATE DETECTION (`Shift+D`)** — the half the roadmap said crab could do alone ("daimon, **or
a content hash crab could do alone**"), and the half that works with no daimon on the box. Marks every
duplicate except the newest of each group. ⛔ **IT MARKS, IT DOES NOT DELETE** — the operator presses
the verb, and the delete prompt then counts and names the set as it does for any hand-marked one.
⛔ Not `d`: a scan one un-shifted key from an irreversible verb gets pressed by accident.
⭐ The size pass is the pre-filter and costs ZERO syscalls (every entry is already stat'ed by the
deferred sweep), so only a size COLLISION is opened. ⚠ A hash match is NOT a guarantee — FNV-1a is 64
bits — so `crab_dup_same` requires equal sizes too, the read is bounded at 64 KiB, and the word is
"duplicate", never "identical".
⛆ **The deferral sweep**: `net` removed from `stdlib` (dead for six minor versions, explicitly queued
as its own change) — ⚠ and the note was loose: NOT byte-identical, both binaries grew 16 bytes and
182,246 differ from the fold-order shift; what is clean is no undefined symbols, deps, deny and the
suite. Four stale comments cut: the A/B strip's "no caller, not hit-tested" (false since 0.8.5),
`CRAB_COL_CHARW`'s "the day crab stops passing font = 0" (that day was 0.9.0), and the 🦀 button's
gate — which said M5 proportional text and **0.9.0 shipped it without unblocking the button**; the
real gate is an ICON PATH, because `dh_draw_text_ink` walks one byte per glyph in BOTH branches,
`rekha_char_to_glyph` is BMP-only and Liberation Sans has no crab.
**2490 / 0**, three mutations. ⚠⚠ **TWO SURVIVED and are recorded at the assertions**: bypassing the
size pre-filter, and bypassing the `CRAB_KIND_FILE` guard — both change what crab OPENS, not what it
MARKS, so no mark-based assertion can see them. A directory opens and then fails to READ, so its hash
stays 0 either way. Both predicates are pinned exhaustively; their USE in the scan is not.

**0.9.7 contents — H1.** `crab_walk_reroot_dtree` turns a cancelled tree copy into a RECURSIVE DELETE
of the destination root. Its own comment named the invariant that made that safe (`crab_walk_begin`
refused an existing destination, so the root was always one crab had just made) **and named the
consequence of losing it**: *"if that guard is ever relaxed to allow merging … THIS FUNCTION BECOMES A
DATA-LOSS BUG and must be deleted in the same change."* 0.8.7 relaxed exactly that guard and did not
touch the delete. ⇒ Cancelling a copy that MERGED into a folder the operator already had deleted that
folder wholesale.
⭐⭐ **MEASURED ON IRON BOTH WAYS** (`crab-h1-test.py`, `debugfs` readback after shutdown): shipped →
`/zzkeep` GONE, **0/3** of the operator's files survived; fixed → folder intact, **3/3** survived,
`crab: transfer cancelled rc 23`.
⇒ `CRAB_OP_DMADE` is written at the only place that knows (the `mkdir` in `crab_walk_begin`) and
`crab_cancel_may_remove(dmade, root_len)` is the decision, lifted out of the walk so the suite drives
it. ⚠ Only an explicit `1` authorises removal — truthiness would let a stale record authorise deleting
the operator's folder. A cancelled merge now reports `CRAB_FS_EMERGED` — *"what was already copied
into that folder is still there"* — instead of implying a clean undo. **2451 / 0**, four mutations.

**0.9.6 contents — a drag cannot outlive its listing.** Two data-loss defects found during 0.9.5 and
deliberately not bundled into it. One root cause: **a drag is a claim about a ROW INDEX and nothing
kept the listing still** — `dragging`/`drag_pane` were touched in exactly three places and NOTHING in
the key dispatch consulted drag state.
⛔⛆ **(1) A drop could move a file while a DELETE PROMPT was on screen.** The release was gated on the
LEFT BUTTON and nothing else — neither `crab_pointer_blocked` nor `crab_pointer_modal`, though the
click has asked since 0.8.0 and the wheel since 0.8.5. Press-hold, move 4 px, press `d` (the prompt
asks about entry X), drag across, release: the drop moved a file, RELISTED BOTH PANES, cleared marks
and CLAMPED both selections — and the `y` deleted something else. **MEASURED on iron**:
`crab: prompt … delete anuenue?` then `crab: drop refused 2`.
⛔⛆ **(2) The keyboard could re-list the source pane mid-drag** — Enter/Backspace descend or ascend
the pane being dragged FROM while the button is down, and `crab_sort_entries` permutes in place even
without navigating. ⇒ **THE NAME IS THE IDENTITY**: the press captures the row's name, the drop
verifies it and refuses out loud. Same discipline as the delete queue's *"queue by NAME first"*.
⚠ **(3) A press on a pane HEADER left the previous drag armed** — arming sat inside `if (hr >= 0)`
with no else, and `crab_hit` records headers as row -1. Nothing else ever disarms a drag: the
compositor DROPS a release outside the content rect, so releasing on the titlebar left `dragging = 1`.
⇒ `crab_drop_ok` is the decision, lifted out of the `#ifdef`. ⛔ The question is asked BEFORE the
target (an operator with a prompt up must not be told "dropped nowhere"), but `dragging` outranks
even that so an ordinary click stays silent. **2440 / 0**, five mutations, each the shipped behaviour.
⭐⭐ **QEMU `crab-drop-test.py` ARM 1 PASSES** (twice, separate runs). ⚠ ARM 2 is honestly UNMEASURED:
its own Backspace re-lists the pane so each retry starts from a different layout and the loop does not
converge — it shows nothing moved; the name check is proven in the suite by mutation instead.
⛔ The drag needed TWO generous moves to promote, not one nudge: `crab_drag_started` needs 4 px but
promotion happens in the POINTER_MOVE arm, so it needs a motion EVENT delivered while the button is
down — and aethersafha dedupes motion.

**0.9.5 contents — pointer polish, and the roadmap's premise was wrong.** The item read *"the middle
mouse button does nothing"*; it has **DISMISSED popups since 0.8.5** — both `CRAB_PA_DISMISS` returns
are button-blind and never test `btn` — and three doc sites claimed otherwise while no test pinned the
one behaviour it had. ⭐⭐ **MIDDLE NOW MARKS the row under the pointer**, an ALIAS of `Space`
(`crab_pa_accel` answers `0x2C`, the arm synthesises that one key through the one binding table, so
there is no second `crab_mark_toggle` call site) — and *"middle is Space, not `d`"* is a HOST
assertion instead of a literal inside the agnos `#ifdef`. ⛔⛔ Deliberately the safest gesture
available: X11 numbers buttons left/**middle**/right, so X11 muscle memory aims middle where crab's
RIGHT lives and `Delete` is a row in the menu right opens — a mark is self-inverse and moves nothing.
Middle stays refused on a popup row, a bar cell, the door, the strip and the sidebar.
⭐⭐ **THE POPUP HIGHLIGHT FOLLOWS THE POINTER.** ⛔ It does not engage until the pointer has MOVED:
the popup is placed AT the pointer and `dh_place_at_point` FLIPS it above the anchor when it would
overhang (which at 380x220 it usually does), so the cursor that opened the menu sits mid-list over a
row nobody aimed at. ⛔⛆ And `-1` from `dh_list_index_at` means BOTH "inert row" and "outside the
list": inside-on-inert HOLDS (the keyboard's rule), OUTSIDE RESTORES the open-time choice — otherwise
sweeping off a menu leaves `Enter` armed on the last row crossed, and the natural exit is
down-and-right through `Delete`.
⛔⛆ **Fixed: the context menu opened on an EMPTY pane with NO HIGHLIGHT AT ALL** — `menu_sel = 0` is
`CRAB_MI_OPEN`, which `crab_menu_enabled` refuses when `count <= 0`, so `dh_list_select` refused the
inert row and `Enter` did nothing. MEASURED: `enabled(OPEN,0) = 0`, first enabled = 5. `crab_menu_first`
is the twin `crab_mb_item_first` has had since 0.8.6; fixed at BOTH open sites.
⛆ Cut four false claims in `crab_mlst`'s comment (all falsified by 0.8.5, left standing four releases)
and corrected the README. **2421 / 0**, five mutations; one expiry fired and was INVERTED
(`"a pane, middle: nothing yet"` → `CRAB_PA_MARK`).
⭐⭐ **QEMU `crab-button-test.py` PASSES, four arms** — and **button 3 is observed for the FIRST TIME
anywhere in this stack**: mask 4 → `crab: mark by middle click`. The hover trace
`menu 3→2→1→0→1→2→3→4→5→0` shows the sweep up a flipped menu, back down all six rows, and the final
`→0` as the pointer leaves and `Enter` means `Open` again.
⛔ **Three harness lessons, each of which gave a WRONG answer first**: a `-2000` home with 0.3 s
settles found nothing (needs `-4000` and 0.8 s — the pin needs its own frame) and would have produced
a false cross-repo blocker; aethersafha's diagnostic is ONE-SHOT so press ORDER decides what it can
tell you; and a popup left open by one probe decides the next in the wrong branch (middle with a menu
up is DISMISS, not MARK).

**0.9.4 contents — the preview shows the SELECTION, and costs nothing.** The roadmap asked for one
thing (*"the 64 KiB dimension+EXIF read is on the idle tick"*) and moving it exposed that the whole
column was reading process-wide *"last touched"* state instead of anything derived from the selection.
⛔⛆ **Two of those readers were wrong and shipping, both measured on the host BEFORE any change:**
(1) the **thumbnail lagged one file behind** — the keypress frame asked `crab_thumb_pixels()` ("what
the last STEP was about") and the tick's gate `if (tafter != tbefore)` skips on `OK -> OK`, so **no
frame was drawn at all** and the previous picture stayed under the new name. That is 0.8.2's bug,
whose own fix comment names the mechanism, obeyed by **1 of 8 render sites**. (2) **`CAMERA: Canon
EOS R5` stayed under a TEXT FILE's name** — `crab_preview_dims` returned at its `is_image` gate
before clearing the process-wide EXIF buffers, and the CAMERA/SHOT rows sit outside the column's
`is_image` block. ⇒ Both die by construction: `crab_pvc_*` (128 path-keyed slots in ui.cyr, 56,320 B
once) holds dims AND EXIF together, `crab_pv_publish` is the ONE publisher for all eight sites and
CLEARS as well as sets, and `crab_pv_redraw_due` makes the SLOT part of the redraw answer.
⭐ **The read is on the tick** (`crab_pv_step`, at most one file per tick, gated on
`crab_preview_fit` — the EFFECTIVE state, so a too-narrow window now reads nothing). MEASURED:
selection path **4 µs per keystroke, the SAME with an empty cache as a warm one** — a miss is a
lookup, never a read. ⚠ The host number understates agnos by ~100x; crab's own recorded figure for
the cheaper *stat* sweep is ~1.1 ms/entry, which is why that was deferred too. Arrowing BACK is now
free (the old memo held ONE entry). Also fixed: a bare `sys_read` that memoised a wrong negative on a
short read, and the **mascot**, the one render site of eight that published no preview state at all.
**2384 / 0**, five mutations; a sixth SURVIVED and is recorded at its assertion (a regular file never
short-reads, so the loop is unprovable with a file fixture).
⭐⭐ **QEMU `crab-preview-test.py` PASSES and is mutation-proven** — new `crab: pv` oracle emitted by
the DRAIN and nothing else; 137x42 and 320x200 each resolved correctly, every file read **exactly
once**, and the 48 non-images produced no line. Against a planted re-read it reports files read up to
**36 times each**.
⛔⛆ **AND THE LAYERING GATE WAS HALF A GATE.** `render_test` includes `ui.cyr` alone to stop the
render path calling up — but an undefined **constant** is `error:` (build fails) while an undefined
**function** is only `warning:` (build exits 0, `53 checks, 0 failed`). Enforced for enums, NOT for
functions — the likelier mistake. `ci.yml` now fails on the warning, mutation-proven both ways.
⛔ Its first draft had a second hole: `cyrius build … | tee` hides the build's exit status, so a
FAILED build ran the STALE binary and reported its old green count. Caught doing exactly that.

⭐⭐⭐ **H1 IS CLOSED (0.9.7).** This section carried it as OPEN for four releases; the fix, the
mutation proofs and the iron measurement are in the 0.9.7 block above. `crab_walk_reroot_dtree` now
asks `crab_cancel_may_remove`, which authorises removing the destination root only when
`CRAB_OP_DMADE` says crab's own `mkdir` created it. Reproduced by the operator before the fix
(`tests/zz_h1_repro.tcyr`, since removed): `POST-CANCEL precious.txt exists = 0 *** DELETED BY
CRAB ***` — and reproduced again by `crab-h1-test.py` against a planted revert, **0/3 surviving**.

**0.9.7 contents — H1.** `crab_walk_reroot_dtree` turns a cancelled tree copy into a RECURSIVE DELETE
of the destination root. Its own comment named the invariant that made that safe (`crab_walk_begin`
refused an existing destination, so the root was always one crab had just made) **and named the
consequence of losing it**: *"if that guard is ever relaxed to allow merging … THIS FUNCTION BECOMES A
DATA-LOSS BUG and must be deleted in the same change."* 0.8.7 relaxed exactly that guard and did not
touch the delete. ⇒ Cancelling a copy that MERGED into a folder the operator already had deleted that
folder wholesale.
⭐⭐ **MEASURED ON IRON BOTH WAYS** (`crab-h1-test.py`, `debugfs` readback after shutdown): shipped →
`/zzkeep` GONE, **0/3** of the operator's files survived; fixed → folder intact, **3/3** survived,
`crab: transfer cancelled rc 23`.
⇒ `CRAB_OP_DMADE` is written at the only place that knows (the `mkdir` in `crab_walk_begin`) and
`crab_cancel_may_remove(dmade, root_len)` is the decision, lifted out of the walk so the suite drives
it. ⚠ Only an explicit `1` authorises removal — truthiness would let a stale record authorise deleting
the operator's folder. A cancelled merge now reports `CRAB_FS_EMERGED` — *"what was already copied
into that folder is still there"* — instead of implying a clean undo. **2451 / 0**, four mutations.

**0.9.6 contents — a drag cannot outlive its listing.** Two data-loss defects found during 0.9.5 and
deliberately not bundled into it. One root cause: **a drag is a claim about a ROW INDEX and nothing
kept the listing still** — `dragging`/`drag_pane` were touched in exactly three places and NOTHING in
the key dispatch consulted drag state.
⛔⛆ **(1) A drop could move a file while a DELETE PROMPT was on screen.** The release was gated on the
LEFT BUTTON and nothing else — neither `crab_pointer_blocked` nor `crab_pointer_modal`, though the
click has asked since 0.8.0 and the wheel since 0.8.5. Press-hold, move 4 px, press `d` (the prompt
asks about entry X), drag across, release: the drop moved a file, RELISTED BOTH PANES, cleared marks
and CLAMPED both selections — and the `y` deleted something else. **MEASURED on iron**:
`crab: prompt … delete anuenue?` then `crab: drop refused 2`.
⛔⛆ **(2) The keyboard could re-list the source pane mid-drag** — Enter/Backspace descend or ascend
the pane being dragged FROM while the button is down, and `crab_sort_entries` permutes in place even
without navigating. ⇒ **THE NAME IS THE IDENTITY**: the press captures the row's name, the drop
verifies it and refuses out loud. Same discipline as the delete queue's *"queue by NAME first"*.
⚠ **(3) A press on a pane HEADER left the previous drag armed** — arming sat inside `if (hr >= 0)`
with no else, and `crab_hit` records headers as row -1. Nothing else ever disarms a drag: the
compositor DROPS a release outside the content rect, so releasing on the titlebar left `dragging = 1`.
⇒ `crab_drop_ok` is the decision, lifted out of the `#ifdef`. ⛔ The question is asked BEFORE the
target (an operator with a prompt up must not be told "dropped nowhere"), but `dragging` outranks
even that so an ordinary click stays silent. **2440 / 0**, five mutations, each the shipped behaviour.
⭐⭐ **QEMU `crab-drop-test.py` ARM 1 PASSES** (twice, separate runs). ⚠ ARM 2 is honestly UNMEASURED:
its own Backspace re-lists the pane so each retry starts from a different layout and the loop does not
converge — it shows nothing moved; the name check is proven in the suite by mutation instead.
⛔ The drag needed TWO generous moves to promote, not one nudge: `crab_drag_started` needs 4 px but
promotion happens in the POINTER_MOVE arm, so it needs a motion EVENT delivered while the button is
down — and aethersafha dedupes motion.

**0.9.5 contents — pointer polish, and the roadmap's premise was wrong.** The item read *"the middle
mouse button does nothing"*; it has **DISMISSED popups since 0.8.5** — both `CRAB_PA_DISMISS` returns
are button-blind and never test `btn` — and three doc sites claimed otherwise while no test pinned the
one behaviour it had. ⭐⭐ **MIDDLE NOW MARKS the row under the pointer**, an ALIAS of `Space`
(`crab_pa_accel` answers `0x2C`, the arm synthesises that one key through the one binding table, so
there is no second `crab_mark_toggle` call site) — and *"middle is Space, not `d`"* is a HOST
assertion instead of a literal inside the agnos `#ifdef`. ⛔⛔ Deliberately the safest gesture
available: X11 numbers buttons left/**middle**/right, so X11 muscle memory aims middle where crab's
RIGHT lives and `Delete` is a row in the menu right opens — a mark is self-inverse and moves nothing.
Middle stays refused on a popup row, a bar cell, the door, the strip and the sidebar.
⭐⭐ **THE POPUP HIGHLIGHT FOLLOWS THE POINTER.** ⛔ It does not engage until the pointer has MOVED:
the popup is placed AT the pointer and `dh_place_at_point` FLIPS it above the anchor when it would
overhang (which at 380x220 it usually does), so the cursor that opened the menu sits mid-list over a
row nobody aimed at. ⛔⛆ And `-1` from `dh_list_index_at` means BOTH "inert row" and "outside the
list": inside-on-inert HOLDS (the keyboard's rule), OUTSIDE RESTORES the open-time choice — otherwise
sweeping off a menu leaves `Enter` armed on the last row crossed, and the natural exit is
down-and-right through `Delete`.
⛔⛆ **Fixed: the context menu opened on an EMPTY pane with NO HIGHLIGHT AT ALL** — `menu_sel = 0` is
`CRAB_MI_OPEN`, which `crab_menu_enabled` refuses when `count <= 0`, so `dh_list_select` refused the
inert row and `Enter` did nothing. MEASURED: `enabled(OPEN,0) = 0`, first enabled = 5. `crab_menu_first`
is the twin `crab_mb_item_first` has had since 0.8.6; fixed at BOTH open sites.
⛆ Cut four false claims in `crab_mlst`'s comment (all falsified by 0.8.5, left standing four releases)
and corrected the README. **2421 / 0**, five mutations; one expiry fired and was INVERTED
(`"a pane, middle: nothing yet"` → `CRAB_PA_MARK`).
⭐⭐ **QEMU `crab-button-test.py` PASSES, four arms** — and **button 3 is observed for the FIRST TIME
anywhere in this stack**: mask 4 → `crab: mark by middle click`. The hover trace
`menu 3→2→1→0→1→2→3→4→5→0` shows the sweep up a flipped menu, back down all six rows, and the final
`→0` as the pointer leaves and `Enter` means `Open` again.
⛔ **Three harness lessons, each of which gave a WRONG answer first**: a `-2000` home with 0.3 s
settles found nothing (needs `-4000` and 0.8 s — the pin needs its own frame) and would have produced
a false cross-repo blocker; aethersafha's diagnostic is ONE-SHOT so press ORDER decides what it can
tell you; and a popup left open by one probe decides the next in the wrong branch (middle with a menu
up is DISMISS, not MARK).

**0.9.4 contents — the preview shows the SELECTION, and costs nothing.** The roadmap asked for one
thing (*"the 64 KiB dimension+EXIF read is on the idle tick"*) and moving it exposed that the whole
column was reading process-wide *"last touched"* state instead of anything derived from the selection.
⛔⛆ **Two of those readers were wrong and shipping, both measured on the host BEFORE any change:**
(1) the **thumbnail lagged one file behind** — the keypress frame asked `crab_thumb_pixels()` ("what
the last STEP was about") and the tick's gate `if (tafter != tbefore)` skips on `OK -> OK`, so **no
frame was drawn at all** and the previous picture stayed under the new name. That is 0.8.2's bug,
whose own fix comment names the mechanism, obeyed by **1 of 8 render sites**. (2) **`CAMERA: Canon
EOS R5` stayed under a TEXT FILE's name** — `crab_preview_dims` returned at its `is_image` gate
before clearing the process-wide EXIF buffers, and the CAMERA/SHOT rows sit outside the column's
`is_image` block. ⇒ Both die by construction: `crab_pvc_*` (128 path-keyed slots in ui.cyr, 56,320 B
once) holds dims AND EXIF together, `crab_pv_publish` is the ONE publisher for all eight sites and
CLEARS as well as sets, and `crab_pv_redraw_due` makes the SLOT part of the redraw answer.
⭐ **The read is on the tick** (`crab_pv_step`, at most one file per tick, gated on
`crab_preview_fit` — the EFFECTIVE state, so a too-narrow window now reads nothing). MEASURED:
selection path **4 µs per keystroke, the SAME with an empty cache as a warm one** — a miss is a
lookup, never a read. ⚠ The host number understates agnos by ~100x; crab's own recorded figure for
the cheaper *stat* sweep is ~1.1 ms/entry, which is why that was deferred too. Arrowing BACK is now
free (the old memo held ONE entry). Also fixed: a bare `sys_read` that memoised a wrong negative on a
short read, and the **mascot**, the one render site of eight that published no preview state at all.
**2384 / 0**, five mutations; a sixth SURVIVED and is recorded at its assertion (a regular file never
short-reads, so the loop is unprovable with a file fixture).
⭐⭐ **QEMU `crab-preview-test.py` PASSES and is mutation-proven** — new `crab: pv` oracle emitted by
the DRAIN and nothing else; 137x42 and 320x200 each resolved correctly, every file read **exactly
once**, and the 48 non-images produced no line. Against a planted re-read it reports files read up to
**36 times each**.
⛔⛆ **AND THE LAYERING GATE WAS HALF A GATE.** `render_test` includes `ui.cyr` alone to stop the
render path calling up — but an undefined **constant** is `error:` (build fails) while an undefined
**function** is only `warning:` (build exits 0, `53 checks, 0 failed`). Enforced for enums, NOT for
functions — the likelier mistake. `ci.yml` now fails on the warning, mutation-proven both ways.
⛔ Its first draft had a second hole: `cyrius build … | tee` hides the build's exit status, so a
FAILED build ran the STALE binary and reported its old green count. Caught doing exactly that.

⭐⭐⭐ **H1 IS CLOSED (0.9.7)** — this line said OPEN for four releases. `crab_walk_reroot_dtree` now
asks `crab_cancel_may_remove`, which authorises removing the destination root only when
`CRAB_OP_DMADE` records that crab's own `mkdir` created it. Measured on iron both ways: shipped →
0/3 of the operator's files survived and the folder itself was gone; fixed → 3/3 and intact.

**0.9.3 contents**: crab can **see a symlink**. ⛔ readdir never could — agnos's
`ext2_readdir_at_sys` sets byte 63 with `if (ftype == 2) { t = 1; }`, one bit, so a link arrived
indistinguishable from a file. ⭐ The stat sweep already visits every entry, so `lstat` costs **no
extra syscall**; the kind goes into the type byte crab already had (2 = link). Marked `@` (ASCII, so
both faces agree), KIND says **Link**, and the size column shows the link's own size as `ls -l` does.
⚠ `lstat` first, `stat` as the fallback — `lstat`#102 is **ext2-only** and crab lists FAT volumes.
⛔⛔ **It found SEVEN broken readers, and one of them was data loss.** The single-entry delete verb
read `if (ddir != 0)` and handed anything non-zero to `crab_walk_begin(CRAB_OP_DTREE, …)`, which
type-checks nothing — so a link pointing at a directory became the **root of a recursive delete** and
the walk went through it into the target. MEASURED: `crab: delete zzlink -> done`, the link still on
disk, **0 of 3 files left in what it pointed at**. ⚠ Older than the symlink work — `crab_stat_one`
used to call `stat`, which follows a link, so a link to a directory was already type `1` and already
took that branch. The other six: the delete **prompt** calling a link a FOLDER *"and everything in
it"*; the transfer planner tree-copying one at both call sites; Enter doing nothing and saying
nothing; two thumbnail readers; and `crab_fs_delete` making links undeletable.
⛔ **The first audit reported itself complete and had missed six of them** — it grepped `== 1` and
`!= 1`, and the whole `!= 0` family is where the damage was. The second **enumerated all 22 reads of
`CRAB_REC_TYPE`** and traced each to its decision. ⇒ Widening a field is only safe where every reader
agrees how to ask, and the way to know they agree is to list them.
⇒ Three decisions moved out of the `#ifdef` into functions the suite can drive: **`crab_delete_plan`**,
**`crab_transfer_plan`** (now takes the kind, not a boolean each caller derived), and `crab_kind_mark`.
⭐ And the obvious fear is not real: a link cannot make the walk loop, because the walk descends only
on `1` and a link is `2` — the same one bit that hid links is what bounded the walk. ⇒ **Delete and
move PRESERVE a link; copy DEREFERENCES** (as `cp -r` does), recorded as **ADR 0004**. Recreate is
possible (`symlink`#63 + `readlink`#70, both with peers) and deferred: both are ext2-only and crab
copies between volumes. **2345 / 0**, six mutations.
⭐⭐ **AND AN IRON ARM** — `crab-symlink-test.py` (new): a real ext2 symlink via `mkfs.ext2 -d`, driven
by keyboard, and the image read back with **`debugfs`** afterwards rather than crab asked its own
opinion. PASSes against the fix, reports the data loss above against the planted defect. New oracle
`crab: prompt <text>` — the prompt is the last thing before an irreversible verb and nothing outside
the screen knew what it said.
⛆ **And the render gate was broken for most of the release**: `CRAB_KIND_*` were declared in
`app.cyr`, which INCLUDES `ui.cyr`, so `src/render_test.cyr` — which includes `ui.cyr` alone to
enforce that the render path never calls up — stopped compiling. The gate worked; nothing ran it,
because `cyrius test` discovers `tests/*.tcyr` and does not build it. Values moved to `path.cyr`'s
`CrabRec`, beside `CRAB_REC_TYPE`. ⇒ **Build `render_test.cyr` explicitly; a green suite is not
evidence it compiles.**

**0.9.2 contents — `Go` is filled**, empty on the bar since 0.8.0. Its rows are every sidebar
destination, picked through **one navigator** (`synth_goto`, mirroring `synth_u`) that the sidebar's
Enter now shares — because `crab_menu_accel` has nothing to return for a path and inventing a key
would make the menu a second implementation of navigation.
⛔⛆ **AND IT CLOSED A KEY LEAK WORSE THAN RECORDED.** The roadmap said `d` was not consumed by the
drop arm; in fact the arm handled six keys and let **every** other one through with `u` intact, so
`c`, `m`, `r`, `n`, Backspace and Space all acted under a popup painted over them. The delete case
defeats `crab_del_prompt`'s whole reason for existing — *"THE PROMPT NAMES WHAT DIES"*, written after
five system binaries left an iron box. `crab_mb_drop_key` eats the mutating set, and an assertion
ties that set to `crab_sb_key`'s so two surfaces that borrow the keyboard cannot drift.
⛔ **The drop-down had no HEIGHT rule** — `Go` is the first menu whose length is the model's. Six rows
fit at 380×220 (220 − bar 22 − status 22 − margin, over a 26 px row); longer is refused, not cut.
⚠ **Parent is a VERB, not a destination**, so it is absent everywhere rather than dead at `/` — the
roadmap's condition met by a better argument than the one it gave. ⚠ Volumes are labelled by
**prefix**, closing *"two FAT volumes render as two identical rows"*. **2314 / 0**, four mutations.
⭐⭐ **QEMU, `crab-go-test.py`, PASS** — and it caught a bug the suite could not: the navigator sat
ABOVE the key dispatch, so a pick made from inside the dispatch was not consumed until the NEXT key.
On target that read as `Go` doing nothing (`crab: go 3 destinations`, no `crab: place`); the suite was
green throughout because every line of it is inside the agnos `#ifdef`. Now: `F10 → Right ×2 → Enter
→ Enter` sends the pane to `/` and lists 8 entries, and `d` ×3 with a menu open does nothing at all.

**0.9.1 contents — the DOOR.** A mark in the status line reveals the menu row, so `F10` is no longer
the only way in — and for seven releases it was **no way at all** (aethersafha claimed the key until
0.16.25). ⛔ **Zero rows**: the status line became a `BOX_H` of [door][text], the shape `crab_pane`
already uses for the A/B strip. ⛔⛆ **The toggle is the Z-ORDER, not a state bit** — the door sits
BELOW the bar branch, so a press with the bar shown never reaches its arm and `DISMISS` closes it.
⛔⛆ **IT IS NOT A CRAB GLYPH, AND THE ROADMAP SAID 0.9.0 WOULD MAKE IT ONE — that claim was wrong
three times over**: `rekha_char_to_glyph` returns 0 above U+FFFF ("format 4 is BMP-only"), the
shipped face has no format-12 cmap, and `dh_draw_text_ink` walks ONE BYTE per glyph in both branches.
U+1F980 is 128,896. ⭐ **CANVAS is the open road** to a real crab — crab already draws thumbnails
through `dh_canvas_new`; an icon is a glyph with no font.
⚠ Using no font also sidesteps a 0.9.0 consequence: **byte 0xF0 is `≡` through kashi and `ð` through
rekha**, so crab's drawable alphabet is printable ASCII.
⛔⛆ **AND A SERIAL LINE MUST BE ONE WRITE** — measured, not reasoned: the console is shared
unserialised by three processes and spliced both new oracles (`crab: font /fonts/default.ttf 0`;
`crab: door ptrscan: first sample handed to ring 3`). ⇒ `crab_line_*` composes, then emits once.
**2257 / 0**, four mutations each caught.
⚠ **On target crab reports `crab: door 0 196 22 22` every run — but a PRESS on it is UNMEASURED**,
and the harness says so rather than PASS or FAIL. Seven runs went into aiming one: the rect is in
crab's surface coordinates, the monitor moves a RELATIVE pointer in screen coordinates, and homing is
unreliable (delivery proved at four different points across four runs of an unchanged script). ⛔ **The commit, the tag and the push are the operator's.** ⚠ The operator commits while
work is in flight; `git log --oneline -3` is the authority on what is in, not this file.

**0.9.0 contents — ⭐⭐ A REAL FACE, the M5 item that has been open longest.** `crab_face()` opens
agnos 1.57.2's kernel-owned **`/fonts/default.ttf`** — Liberation Sans Regular 2.1.5, unmodified,
410,820 B, **SIL OFL 1.1 (the licence travels with any redistribution)** — once, before the first
frame, and all eight `crab_render` sites draw with it. ⛔ Read front-to-back in one pass (`lseek` is
-1 on a `VFS_MEMFILE`); opened **outside any draw**, because dhancha 0.10.0 scopes sadish's allocation
hook to one `dh_draw_text_ink` and `rekha_font_open` follows it; a full read buffer is treated as a
**truncation**, since with no `lseek` that is the only signal there is.
⛔⛆ **THE COINCIDENCE THAT WOULD FOOL A VERIFIER**: Liberation Sans's `n` is 1139/2048 em, which at
16 px rounds to **exactly kashi's 9** — so every derived width is numerically identical to the bitmap
face's. The proof is `i=4 m=13`. *The columns did not move; what goes in them did.*
⭐ **And the `~` marker finally measures.** `crab_name_cell` truncated at a CHARACTER COUNT — ten
`m`s "fit" a ten-character column and measure 160 px in 120. `crab_name_cell_px` measures greedily,
reserving the marker's own width **before** any name byte, so the `~`'s claim is true. At `font = 0`
it reduces exactly to the old arithmetic; the bitmap build is unchanged.
⭐ `crab_say_u` — arena-free integer logging, which crab had none of. **2213 / 0.**
⭐⭐ **QEMU, `crab-face-test.py`, PASS**: `crab: font /fonts/default.ttf 410820 bytes adv=9 upem=2048
i=4 m=13`, exactly one load per session, navigation and view switching under the face, no faults and
no allocator failure — ⛔ the last of which **dhancha 0.10.0 is what makes passable**.
⚠ **One mutation PASSED and is recorded**: deleting `crab_face`'s memo leaves the host green (the
load fails at its first step either way), so the memo's gate is ARM 2 on the target, not the suite.

**0.8.11 contents**: the **6.6.4 stack** — cyrius 6.6.2 → **6.6.4**, sadish 0.5.4 → **0.5.5**, rekha
0.3.7 → **0.3.10**, kashi 1.0.7 → **1.0.8**, dhancha 0.9.29 → **0.10.0** (rupa, setu and chitra had
not moved). ⛔ sadish and rekha are a **floor**, not company: without them dhancha 0.10.0 refuses the
build — `2 reachable undefined function(s)` (`sd_alloc_set`, `sd_canvas_blit_at`). ⚠ **6.6.3 silently
corrupts even-length string literals ≥ 64 KB** — agnos found it generating the embedded face and made
its kernel hash-verify the bytes; crab is past it on 6.6.4.
⭐⭐ **BOTH BLOCKERS ON `0.9.0` CLOSED, within a day of being filed.** **dhancha 0.10.0** routed the
scalable canvas through the frame arena (with sadish 0.5.5's `sd_alloc` hook and rekha 0.3.10's
scoped outline scratch), so **crab's expiry assertion fired and is now INVERTED — a warm frame under
a real face costs the global heap exactly 0.** ⚠ dhancha's hand-off predicted crab's failure *by name
and to the byte* from measurements against crab 0.8.10 (`arena_capacity_total == cap0`, got 468,040,
expected 16,384) — which is what a filing with a gate behind it buys. **agnos 1.57.2 + rekha 0.3.8**
ship a kernel-owned `/fonts` namespace: **`/fonts/default.ttf`**, Liberation Sans Regular 2.1.5
unmodified, 410,820 B, SIL OFL 1.1 (licence travels with redistribution), embedded kashi-style and
FNV-1a-64 verified at boot. ⛔ Read-only `VFS_MEMFILE`, **`lseek` is -1 — one front-to-back pass**.
✅ **Check four re-run**: overrides off, lock **3 → 7 commit-pinned**, both binaries byte-identical,
2198 / 0 in that tree. **2198 / 0.** ⇒ **`0.9.0 · A real face` is unblocked**; what remains is crab's
own, and must be measured on **QEMU** because that file exists on no host.
⭐ **0.8.7, 0.8.8 and 0.8.9 are COMMITTED AND TAGGED locally** (`22f7f53` · `60cc05b` · `eeb6a8b`) and
**0.8.10's code is committed at `c0c7724`, not yet tagged** — the multi-release backlog earlier
entries here describe is cleared. ⚠ The operator commits WHILE work is in flight, so a
committed/uncommitted claim in this file is true as of its writing and `git log` is the authority.
⚠ `git ls-remote --tags` answered empty when this was written, so whether those tags are PUSHED is
unknown from here. Stated rather than assumed: this file has asserted a remote state before and the
rule it learned was to say which half it actually checked.

**0.8.10 contents**: the unblocked half of `0.9.0 · A real face`. Seven widths read
`CRAB_COL_NAME_MIN = 90;  # 10 chars` — the governing fact in the comment and the face-specific
number in the code; ⇒ the **counts** are now the constants and the widths are derived
(`crab_col_name_min()` and six siblings; functions, because a Cyrius enum member must be a literal).
At `font = 0` each returns the literal it replaced and **render_test's 53 pixel checks pass
unchanged**. ⭐⭐ **The suite builds a synthetic PROPORTIONAL face** (head/maxp/hhea/hmtx/cmap, no
`glyf` — a width question never rasterises), which retires 0.8.8's own admission that no host test
could tell *"asks the font"* from *"divides by the constant"*. Its advances are deliberately unequal,
so `"nn"` and `"nm"` differ in width at equal length — the thing `length × advance` cannot see.
⛔⛆ **And it found that the zero-allocation gate measures the branch that is not running**: every
render in it passes `font = 0`; the scalable path opens `sd_canvas_new(surface_w, surface_h)` per
label per frame from the bump allocator, so crab's M1.5 headline would have gone false on the first
frame with a face **with nothing noticing**. That cost is now measured, with an assertion that must be
INVERTED when dhancha fixes it. ⚠ The Latin-1 limit is written down: one **byte** per glyph, a display
limit owned by dhancha/rekha — crab cannot type such a byte (`crab_key_char` tops at `'z'`) but can
display one from a readdir record. **2196 / 0.** ⚠ No QEMU arm: nothing agnos-only changed.
⛔⛔ **`0.9.0` is BLOCKED on two things, neither crab's, and both are now FILED** — (a) there is no
TrueType face anywhere in the stack and nothing stages one onto the target; ⭐ **operator ruling
2026-09-13: *"rekha is that thing... but has yet to get Kernel support"*** — so rekha IS the answer
and the gate is an **agnos** arc, not a font-picking question. ⚠ The obvious template reads a **host**
Arch path that does not exist on AGNOS and would look finished while falling back silently. (b)
dhancha's scalable draw allocates a full-surface canvas per label per frame outside the arena
(**dhancha**). Its roadmap "blocked by" cell read "—" until this release went looking.
⇒ **Both are filed IN THE OWNING REPOS**, with crab copies beside them:
`agnos/docs/development/issues/2026-09-13-no-proportional-face-on-the-target.md` and
`dhancha/docs/development/issues/2026-09-13-scalable-text-allocates-per-call-outside-the-frame-arena.md`. ⚠ 0.8.4's and 0.8.3's CHANGELOG headers still
read *"unreleased"*; both are records and are left alone.

**0.8.9 contents**: a **Shift latch** — names can hold capital letters. `crab_key_char` has taken the
flag since the field was built; its one production call site passed a literal `0` under *"there is no
shift state on the wire yet."* ⛔ **The wire was never the problem and half that claim was wrong**:
`mods` carries only the press/release edge, but a modifier's OWN edge arrives as its own key event
and aethersafha 0.16.25 exempts those from the chord swallow deliberately — *"a client that wants
Shift state has no other way to learn it."* ⛔⛔ **A MASK, not a boolean**: hold LeftShift, hold
RightShift, release LeftShift — still shifted, which a flag loses mid-word. The release-clear is a
**guarded** subtraction because an unpaired release is measured here (*"a claimed key's release is
forwarded while its press is not"*); unguarded it drives the mask to −1 and capitalises everything
forever. ⛔⛆ **It closed a second recorded gap**: the batch sheet advertises `# = number, * = old
name` and **neither could be typed** — both live on the shifted number row, which answered 0. The row
is filled, all ten, and the suite pins `#`/`*` against `crab_batch_name` itself so the keyboard and
the language cannot drift. ⚠ The overwrite policy's *"`R` is unreachable"* second reason expired and
is corrected, not dropped — the design stands on the reason that was load-bearing anyway (a capital
is an **invisible mode** on a prompt where the next key destroys a file). ⭐ `crab: edit open <label>`
is new: the sheet was the one interactive surface crab opened in silence. **2158 / 0.**
⭐⭐ **QEMU, 2026-09-13** — `crab-shift-test.py`, PASS. `Shift+A Shift+B Shift+3 Shift+8` committed
exactly `AB#*`; `Shift+A` then `b c` committed exactly `Abc`; ten shift edges arrived and crab acted
on zero; no faults. ⛔ **The ORDER is what only QEMU can see**: the latch is fed the raw edge BEFORE
`crab_key_is_modifier` zeroes it — backwards, every Shift press looks like a release, the latch never
sets, and the suite stays green. ⚠ The first run measured `nnnAB#*` and that was the **harness** — its
blind `n` retry typed literal `n`s into the name, which is what earned `crab: edit open`.

**0.8.8 contents**: the **COLUMNS view** — `g` cycles a fourth time and the active pane grows a
narrow **context column** naming the parent with the current directory marked. ⛔⛔ **It is a view
mode of ONE pane, and that is a safety decision**: N navigable miller panes make `active_pane`
something other than a 0/1, and `active_pane` is a 0/1 the **write layer** resolves every copy, move
and delete against — a drag from column k to k+1 would plan a move of a directory into its own
subtree. K=2 with one driven column answers it by construction. The context column holds no focus,
is not in `crab_hit`'s walk, is dropped rather than squeezed below 186 px, and shows nothing at all
at `/`. ⛔ **And it first found the guard meant to prevent exactly this**: `crab_view_is_grid`, added
in 0.8.2 claiming to be asked "in all three places", was asked only in `main.cyr`'s arrow sites —
`crab_pane` and the scroll round-trip still tested `view != CRAB_VIEW_LIST`, so a fourth id would have
**rendered and scrolled as a GRID with LIST arrows**. *A negation is not a predicate.*
⭐ **The 9 px advance now has one reader** (`crab_char_w` / `crab_text_w`): crab still passes
`font = 0` so it renders identically, which is why it was done on its own — what is left of
proportional text is passing a real face. 🗺 **The roadmap has an order**: a ladder to 1.0.0 whose
every entry is a VERSION, named by what an operator can newly do when it is cut. **2104 / 0.**
⭐⭐ **QEMU, 2026-09-13** — `crab-columns-test.py`, PASS on its first run: `g` reaches the fourth
view, the parent is listed and titled `/`, redraws **do not** re-list (the memo arm — a readdir is
not a render-path operation), the root's refusal is memoised, leaving the view stops the listing, no
faults. ⛔ The listing could not be gated any other way: `crab_readdir_into`'s body is inside
`#ifdef CYRIUS_TARGET_AGNOS` with no `#else`, so on the host it returns 0 entries for every path.

**0.8.7 contents**: the **overwrite policy** — a collision stops the walk and asks per file
(`r` replace · `s` skip · `k` keep both · `Esc` stop, with `a` arming an *all*); directories MERGE and
only files ask; keep-both suffixes before the extension; replace unlinks first because the two targets
disagree about `O_EXCL`. ⛔ **And it found a shipped defect**: a `DT_UNKNOWN` retry did `return 0`,
which **is** `CRAB_FS_OK`, so that arm ended a delete and reported *"done"* on a half-emptied tree.
Plus a **flag surface** (`--help` / `-h` with the key list, `--about` on the
Ben-Stein line, an unknown flag named and refused with rc 2; no `--version`, because `VERSION` is the
only source of truth and nothing can inject it); the **roadmap cleaned** — 631 → 466 lines, M5/M6
collapsed into the shipped table, 42 stale claims corrected by audit; `u` relists both panes (selection kept by NAME, marks cleared, refused
during a transfer) and rebuilds PLACES and VOLUMES (sidebar cursor re-seated by exact path or
dropped, section-aware — `/` is both a place and a volume); `View ▸ Refresh` is the bar's fifth
item, which makes the separator guard load-bearing. ⛔ **The refresh relist KEEPS the thumbnail
cache** (`crab_relist_keep_thumbs`): a review found the first draft re-charging the permanent
decode budget on every press. **1879 / 0.** ⭐⭐ **QEMU, 2026-09-13**: right-click arrives as button **2**, crab opens the context
menu, a pointer pick runs an entry, a press off the popup dismisses, `u` relists twice per press,
`g`/`b` answer. ⛔⛔ **The compositor claimed Esc, Tab and F4–F10 — measured: crab acted on zero, Esc
quit the desktop — and ⭐ aethersafha 0.16.25 (prepared, operator direction) moved chrome onto Ctrl:**
Ctrl+Q / Ctrl+Tab / Ctrl+F4–F10, bare keys forwarded. Measured again on QEMU with 0.16.25: **crab's
F10 menu bar opened, `View` was driven from the keyboard, bare Esc and Tab reached crab, Ctrl+Q ended
the desktop.** crab's one required change: a modifier's own edge is not a keystroke
(`crab_key_is_modifier`) — it used to answer the delete prompt. **1887 / 0.**

**0.8.6 contents** (2026-09-13): the menu bar's **`View` is filled** — Cycle view · Cycle sort ·
Preview · Sidebar (`g` · `s` · `p` · `b`), display ids 6..9 above `CRAB_MI_COUNT` so the context menu
never lists them, always live, the same one-implementation rewrite as every other menu entry. `Go`
stays empty with its reasons on record. **The six M6 interaction gaps are all closed.** 1838 / 0,
all nine gates green, check four byte-identical. ⚠ Not run on QEMU or iron.

**0.8.5 contents** (2026-09-12/13): all seven deps re-pinned to their 6.6.2 releases (chitra
1.0.1 → **1.0.3** is the one with content — its P-1 sweep closed a SIGSEGV on the first PNG a
memory-pressured process decodes, which is crab's shape); `lib/sankoch.cyr` brought to the 6.6.2
snapshot (the 0.8.4 bump had left that transitive leaf at 2.7.14); check four re-run — 7 / 0, lock
3 → 7 commit-pinned, both binaries byte-identical. Then the pointer routes: `crab_pointer_action`
decides one arm per press in a pinned z-order (popup, bar, strip, sidebar, panes), a right press on
a pane opens the context menu over the row, a left press on a popup row synthesises the entry's
accelerator through the one binding table, `crab_menu_item_at` inverts the separator shift, and
`crab_pointer_blocked` is split out of `crab_pointer_modal` (Phase 0's last piece). ⛔⛆ **Two holes
closed on the way**: the wheel had NO modal guard (a scroll between `d` and `y` moved the
selection — 0.8.0's click hole, one input kind over) and the sidebar arm never consumed the
double-click pair. Then the **sidebar's *you are here* marker** — `crab_sb_here`, containment with
the deepest containing row winning and ties to the lowest row, painted muted when unfocused and
never when focused without a cursor — with Phase 0's last piece under it (both model builders strip
trailing slashes as they store; `crab_path_within` moved down to `path.cyr` so the render path could
share it) and a latent layout fix beside it (a 64-byte volume prefix lost its terminator to `BSIZE`;
the record is 112 bytes now). **1790 / 0**, all nine gates green, check four byte-identical.
⚠ **Not run on QEMU or iron** — the pointer arm and the `mountlist` path are agnos-only; six new
`crab: … by pointer` oracle lines exist for the run that will. ⭐ **aethersafha 0.16.24 is on the
remote, CI green** — the compositor half of right-click is released.

**0.8.3 contents** (2026-09-09): the M6 interaction gaps — **two of six closed**, plus a
shipped bug none of them named. `VERSION` reads `0.8.3` and the CHANGELOG header agrees.
⭐ **Cut checks done**: all nine gates green (**1695 / 0**, render_test 53/0, fuzz 100k, coverage
88 %, vet/deny 0, fmt clean), and **check four re-run with all four `path` overrides disabled** — 7
deps / 0 errors, lock 3 → 7 commit-pinned, and both binaries **byte-identical** to the path-resolved
build (host **1,045,288 B**, agnos **1,081,768 B**).
⛔ **Nothing is committed, tagged or pushed** — the operator handles every git operation.

⛔⛆ **AND ONE THING WAS ASKED FOR AND COULD NOT BE DONE: the aethersafha button blocker.** crab
cannot tell a right-click from a left one, and the loss is aethersafha's — it masks button
transitions with `1` and forwards the number hardcoded. The fix there is small. **aethersafha does
not build on any available toolchain**: its `6.5.33` pin is uninstallable (*"Package registry not yet
available"*) and flagged critical by the toolchain itself, and under 6.6.0/6.6.1 it throws **57
errors, none in its own source** — `sigil` / `agnostik` / `agnodrm` disagreeing about `result_*`
arities and never updated for the `: stack` multi-return. All three are already at their highest tag.
⇒ **Three upstream repos need the 6.6.x language first.** Filed in full at
[`issues/2026-09-09-aethersafha-forwards-only-the-left-button.md`](issues/2026-09-09-aethersafha-forwards-only-the-left-button.md).
⚠ **aethersafha was left exactly as found**, at tag `0.16.22` with its `6.5.33` pin — nothing was
hand-edited into its vendored `lib/`, and no unverified change was pushed to a repo that cannot be
compiled.
⭐⭐ **`Open` WAS DEAD ON BOTH MENU SURFACES.** Both arms rewrite `u` to the chosen entry's key and
fall through to the one implementation of that command — but they sat BELOW the binding table, and
`CRAB_MI_OPEN` rewrites to `0x28`, handled above them. `r`/`n`/`d`/`c`/`m` worked, and nothing made
that true but their line numbers. ⇒ Both arms hoisted; the map they each copied is `crab_menu_accel`.
⭐ **Closed**: the PLACES sidebar's keyboard route (`Tab`, arrows, Enter — with the mutating verbs
EATEN so `d` cannot delete from a pane the keys have left), and the menu bar's fit rule (plus the
**second** rule its drop-downs needed — there is a band of widths where the bar fits and `Edit`'s
menu opens under the word `File`).
✅ **CLOSED in 0.8.5**: the bar/switcher and context-menu pointer routes, together (the upstream
gate closed first — aethersafha 0.16.24, on the remote, forwards every button in the numbering
crab's filing proposed — and crab reads `POINTER_BTN`'s `a` now), and the sidebar's *you are here*
marker. ✅ **CLOSED in 0.8.6**: `View`'s items. **All six M6 interaction gaps are closed.**

**0.8.2** (2026-09-09), on operator direction: the 6.6.1 pin, the
documentation-currency repair, and **the audit backlog's eight correctness bugs — all closed, each
mutation-proven**. `VERSION` reads `0.8.2`. See [`../../CHANGELOG.md`](../../CHANGELOG.md).
⚠ **Nothing is committed, tagged or pushed** — the operator handles every git operation.

⭐⭐ **AND THE SWEEP FOUND A NINTH DEFECT, LARGER THAN THE EIGHT IT WENT LOOKING FOR: RECURSIVE COPY
AND RECURSIVE DELETE HAD NEVER RUN, IN ANY SHIPPED BUILD.** The idle tick called `crab_copy_step`
(the single-file chunk loop) instead of `crab_op_step` (the dispatcher), so every `CTREE`/`DTREE`
died on its first tick reading a descriptor that was not open. `d` on a folder took the `y` and
deleted nothing; a tree copy left an empty directory wearing the source's name. `crab_op_step` had
**zero callers**, under a comment reading *"THE single entry point the idle tick calls"*.
⛔⛆ **THE SUITE WAS GREEN THE WHOLE TIME, BECAUSE EVERY WALK TEST DROVE `crab_op_step`** — the right
entry point — while the only caller that ships drove the wrong one. ⇒ **A test that calls a
different function than the shipping caller is not testing the shipping path.** Keep this above the
individual fixes; it is the transferable part.

⛔⛆ **THIS FILE ROTTED A THIRD TIME, AND THE THIRD TIME IT SURVIVED ITS OWN ⛔ MARKERS.** From
2026-09-02 to 2026-09-08 it asserted *"0.8.0 in preparation"* and *"0.7.7 is the last RELEASED
version"* across **two** tagged releases (0.8.0, 0.8.1), a pin that had already moved twice
(6.5.41 → 6.6.0 → 6.6.1), a dependency table naming **rekha 0.3.5** and **dhancha 0.9.26** while the
manifest declared **0.3.6** and **0.9.28**, a **"6 deps"** count against seven, and *"VOLUMES —
enumeration still open"* after 0.8.1 shipped VOLUMES on agnos `mountlist`#104.
⇒ The two prior rots were written up directly above and did not prevent the third. **The header's
rule — refresh in the same commit as the CHANGELOG entry — is the only thing that ever would have,
and nothing enforces it.**

⭐ **M6 ships everything buildable**: the PLACES sidebar (`b`), sidebar **VOLUMES** with capacity
bars, the menu bar (`F10`), the A/B view switcher, 🦀 Bueller's status-bar line, and pane-header
focus — the last M1–M4 residue.
⛔ **Two M6 items remain and both are GATED, not deferred**: the 🦀 chrome button (CP437 has no crab
glyph; needs an icon path or proportional text) and the held-key repeat number (agnos-runtime, no
host test can see it). ⚠ *A milestone closing with gated items is the normal shape here.*
⭐ **VOLUMES is no longer one of them.** The blocker crab filed on 2026-09-02 was answered: agnos
minted `mountlist`#104 rather than widening `mount`#11, crediting crab's filing by name, and 0.8.1
enumerates mounts instead of probing three hardcoded prefixes.
⛔⛔ **AND 0.8.0 FIXED TWO FEATURES THAT HAD NEVER BEEN VISIBLE** — the context menu and the rename
sheet, shipped in 0.7.5, were laid out entirely below the window. See the CHANGELOG.

⭐ **0.7.7 is a REPAIR cut — no roadmap item advanced.** Five defects that had already shipped were
found by reading code and closed with mutation-proven tests; the toolchain pin moved
**6.5.36 → 6.5.41**; and CI went from one step to nine, closing the three long-open CI gaps.
⚠ **Every figure below has been re-measured at 0.7.7 unless it says otherwise.**

⭐ **M5 is substantially in** — the preview column, thumbnails, EXIF, and the GRID and GALLERY
views — and **every finding of the 2026-08-31 security audit is closed**. ⚠ **Semver would normally
make new user-facing features a MINOR**; this is the fourth time this project has put feature work in
a patch by operator ruling. ⛔ **CORRECTED 2026-09-01: 0.7.6 IS committed, tagged (`26f38ed`) and pushed.** This line said
"Nothing is committed, tagged or pushed" — true when written, false by the time it was read, and
nothing gated it (nothing gates this file's currency, verbatim). The operator still drives every version decision;
what changed is that this one was already made.

⛔ **0.7.2 exists because 0.7.1's CHANGELOG section was being edited after its tag was pushed.**
Three commits landed past `4ac21eb` while their notes were still being written into the released
section. A released section is not a scratchpad: editing one after its tag makes the tag and the
notes disagree, and the notes are what a consumer reads. 0.7.1's section is now byte-identical to
the tag and everything since is under 0.7.2.

⭐ Refreshed **in the same commit as the CHANGELOG entry**, which is the rule the header below
demands and the one this file broke twice.

## Toolchain

- **Cyrius pin**: `6.6.2` (in `cyrius.cyml [package].cyrius`) — moved **2026-09-11 at the 0.8.4
  cut, by the operator**, from `6.6.1`, together with the whole sibling stack (every dep, and
  aethersafha's stack behind it, now sits on 6.6.2). 6.6.2 is the repair release for 6.6.0's
  `: stack` value-form change; crab's `src/` needed no edits. ⚠ The bump re-vendored `boxed`,
  `hashmap`, `result` and `tagged` and **left `lib/sankoch.cyr` at 6.6.1's 2.7.14** — the
  transitive leaf the sync never walks — corrected 2026-09-12 by `cyrius deps`, after which the
  whole vendored tree `cmp`s byte-identical to `~/.cyrius/versions/6.6.2/lib`. ⚠ The wrapper on
  PATH is already 6.6.3 (`cycc --version`); `cyrius` resolves the manifest pin, bare `cycc` does not.
- *(history)* `6.6.1` — moved **2026-09-08 on operator direction**, from `6.6.0`. ⭐ **Not cosmetic, and the reason is on crab's shipping target**:
  6.6.1 rebinds `chrono`'s AGNOS monotonic clock from `sys_uptime_ms` (**#40**, `timer_ticks`) to
  `sys_uptime_us` (**#95**, `rdtsc`). ⛔⛔ **A foreground `run` program on AGNOS executes with IF
  CLEARED** — only `/bin/agnsh` gets IF=1 — so the 100 Hz timer ISR never fires, `timer_ticks` never
  advances, and #40 is **frozen for that program's entire run**. Anything timing itself with it read
  exactly zero, forever, with **no error**. crab is spawned by the compositor, so it is precisely
  that shape of program. Resolution improves as a side effect: µs rather than the 10 ms tick.
  ⚠ **Only two vendored leaves moved** — `lib/chrono.cyr` and `lib/sankoch.cyr` (2.7.11 → 2.7.14,
  an `out_max` output ceiling threaded through the deflate path, reached transitively via chitra).
  The other five stdlib files 6.6.1 touched — `ganita`, `math`, `niyama`, `patra`, `sakshi` — are
  **not in crab's graph** and did not land.
  ⭐ **`cyrius deps` refreshed both leaves correctly this time**, verified by hashing `lib/` against
  `~/.cyrius/versions/6.6.1/lib/` rather than by trusting the command — the 6.5.41 bump's hand-copy
  is why that check is written down. Binary: host **1,032,848 → 1,036,944 B** (+4,096), agnos
  **1,068,976**. Gates: **1462/0**, render_test **53 checks / 0**, fuzz 100,000 rounds, coverage 87 %.
- *(history)* `6.6.0` — the pin between 6.5.41 and 6.6.1; carried the 0.8.0/0.8.1 cuts. It was never
  written into this section, which is part of the third rot recorded above.
- *(history)* `6.5.41` — moved 2026-09-02 at the 0.7.7 cut, on operator direction. ⭐ **Not cosmetic**: 6.5.37 shipped `sys_statfs` and `sys_lstat` (closing the VOLUMES *capacity* gate outright and turning the symlink gap into a decision), and 6.5.39 added the `lib/hashseed.cyr` leaf. ⚠ Before the bump the manifest said 6.5.36 while `cycc` was already 6.5.41, so every build printed `toolchain drift` and the pin was a false declaration. It no longer warns. ⛔ **`cyrius lib sync` walks only the DECLARED stdlib set** — three transitive thread leaves stayed at 6.5.36 content and were copied by hand; `lib/hashseed.cyr` arrived untracked. **Diff the whole vendored tree against the snapshot after any bump.**
- *(history)* `6.5.36` — moved 2026-08-30 with the defect
  and M4 work, per the standing rule that a repaired repo does not stay on a stale pin.
  ⭐ **This bump retired `CRAB_SYS_READDIR_AT = 101`**: 6.5.36 vendors `sys_readdir_at`, so crab no
  longer hardcodes a syscall number, and crab is the first consumer to actually call that wrapper.
- Trail: `6.4.71` → `6.5.5` (0.4.3) → `6.5.9` → **6.5.21** (0.4.9, one language version across the
  desktop stack) → **6.5.27** (0.4.11) → **6.5.28** (0.4.13) → **6.5.35** (0.4.15) →
  **6.5.36** (0.7.1).
- ⚠ The pin is documentation, not enforcement — `cyrius build` compiles with the **installed** `cycc`,
  warns `toolchain drift`, and carries on. ⛔ It is not cosmetic: **CI installs the toolchain from this
  pin** (`grep '^cyrius = ' cyrius.cyml`, both `.github/workflows/*.yml`), so while the pin lagged, a
  cold CI build and a local build compiled crab with *different compilers* and neither run said so.
- ⛔ **6.5.35 is not a cosmetic bump, unlike 6.5.21 → 6.5.27 which 0.4.11 measured byte-identical.**
  The binary is **377,288 B on both 6.5.28 and 6.5.35 — the same SIZE** — but **240,284 of those bytes
  differ (63.7 %)**, from the .35 linear-scan register-allocator rework. ⚠ An identical size is exactly
  the shape of evidence that gets mistaken for "nothing changed". It is not.
- ⭐ **HOW TO BUILD crab CORRECTLY, given the note below**: fetch the pinned **released tarball**,
  extract it, create `$D/versions/<v>/{lib,bin}` symlinks, and run
  `CYRIUS_HOME=$D PATH=$D/bin:$PATH $D/bin/cyrius …`. ⭐ **This is verifiable, not just hygienic**:
  built that way at the 0.7.0 tree, `build/crab` comes out **398,504 B** and `build/crab_agnos`
  **406,992 B — byte-identical to the artifact that burned on iron**. The contaminated local
  snapshot produced 398,520 / 407,040 instead. **A byte delta of 16/48 against the documented sizes
  is the signature of a poisoned toolchain, not of a source change.**
- ⛔⛔ **HISTORY, NOT A LIVE INSTRUCTION — AND ITS REMEDY IS NOW DESTRUCTIVE. DO NOT FOLLOW IT.**
  On 2026-08-28 the local `~/.cyrius` **6.5.35** snapshot was found overwritten with 6.5.36 stdlib,
  and this entry told the reader to check `grep -c SYS_READDIR_AT lib/syscalls_x86_64_agnos.cyr`
  **must be 0** and to run `git checkout -- lib/ cyrius.lock` to undo any change.
  ⛔ **AT THE 6.5.41 PIN THAT GREP MUST BE NON-ZERO** — 6.5.36 onward vendors `sys_readdir_at`, and
  crab calls it in three places. Measured 2026-09-02: it returns **2**, correctly. Following the old
  instruction would revert *correct* vendoring and reintroduce a stdlib that does not match the pin.
  ⇒ **The durable lesson survives and the recipe does not**: a `~/.cyrius/versions/<v>/lib` directory
  can hold a stdlib that is not that version's, so **diagnose by CONTENT, never by the version
  string in the path**. The check that actually works is to compare the vendored tree against the
  pinned snapshot file by file — which is how 0.7.7's bump was verified (0 files drifting), and how
  it caught three transitive leaves the sync had left behind.
  ⚠ *A byte delta of 16/48 against a documented size is the shape of a vendored-`lib/` change — it
  may be poisoning OR a legitimate stdlib move. 0.7.7's bump produced exactly +16 host / +48 agnos
  and was legitimate. The delta tells you to look; it does not tell you which.*
- `lib/` is vendored from the pinned snapshot by `cyrius lib sync`, **not** by `cyrius deps` — a
  toolchain bump without a `lib sync` leaves the stdlib behind. ⚠⚠ **THE SYNC WALKS THE DECLARED `[deps].stdlib` SET ONLY, AND THAT IS A REAL GAP, MEASURED.**
  At the 0.7.7 bump it copied **29** files and left **three** behind — `lib/thread_agnos.cyr`,
  `lib/thread_local.cyr`, `lib/thread_macos.cyr` — all transitive, none named by any declaration.
  They were copied by hand, and the tree then verified byte-identical to the 6.5.41 snapshot.
  ⛔ `lib/atomic.cyr` — the leaf this note has always named — **escaped only by luck**: it happens to
  be byte-identical between 6.5.36 and 6.5.41. It will not always be.
  ⇒ **Diff the WHOLE vendored tree against the snapshot at every bump**, not the declared subset,
  and reconcile `git status lib/` *and* the lock's entry count — 6.5.41 also ADDED a leaf
  (`lib/hashseed.cyr`), which arrived untracked.

## Source

**8,092 lines** across **six** files, plus **4,486** in `tests/` *(0.7.7; 7,915 at 0.7.6, 5,368 at
0.7.5, 2,227 at the 0.7.0 cut)*.
⛔ **THE PER-FILE COUNTS THAT USED TO SIT IN THE HEADINGS BELOW ARE DELETED, NOT UPDATED.** They were
`main.cyr (1,287)` against a real 1,494 and `app.cyr (2,484)` against a real 3,007 — understated by
523 lines in the largest file in the project — while this very section warned three lines down never
to trust them. A number nothing gates does not survive being corrected; it survives being removed.
**Re-derive with `wc -l src/*.cyr`.**
⚠ This section read "2019 lines" and per-file counts from **before** the 0.7.0 cut — it was already
stale by ~200 lines when the cut landed. Re-derive with `wc -l src/*.cyr`, never trust the numbers
here.
⚠ +2,416 over 0.7.5 *(shipped in 0.7.6)*: the render-state record, the preview column, the header-only
dimension parser and the memoised read behind it — plus the tests, which are the larger half.

- `src/main.cyr` — ⛔ **`main()` AND `_entry()` AND NOTHING ELSE, as of 0.6.0.** It ends in
  `_entry();`, so a test that included it would RUN THE APP — which is why everything testable was
  moved to `src/app.cyr`. **Do not add a function here.** What remains is the dhancha client
  lifecycle, the shm present path and the frame loop.
  ⚠ The **present path stays hand-rolled on purpose**: a LIVE shared buffer (create once, rewrite in
  place) against `dh_client_present`'s per-frame ATTACH+COMMIT. Different models — swapping them is
  not a rename.
  ⚠ **No arena setup here any more** (0.6.0). It lived in `main()` and that was a gap: deleting it
  broke no test while restoring a 77 KB-per-frame leak. `crab_render` owns it.
- `src/app.cyr` — ⭐ **NEW at 0.6.0**, and M3's centre of gravity; M4's write layer lives
  here too (`crab_fs_*`, the per-target syscall shim, `crab_name_ok`). The application layer lifted out of `main.cyr`: the
  readdir parser (`crab_readdir_into` and its cap clamp), the stat layer, `crab_descend` /
  `crab_ascend`, `crab_surface_flags`, the serial logging.
  ⛔ It exists for the same reason `path.cyr` does — **none of it was reachable from any test** while
  it lived in `main.cyr`, in a program whose two shipped defects were both found on iron. 0.6.0 added
  17 assertions against it, all mutation-proven.
  ⚠ `CRAB_MAX_ENTRIES = 1024` (256 until M3) and `crab_surface_flags()` returning `SETU_SURF_PREMULTIPLIED`
  **unconditionally** — no flag, no arm, no env var — both live here.
- `src/path.cyr` — the readdir record layout and the **bounded** cstring/path helpers
  (`crab_cstr_len`, `crab_streq_n`, `crab_strcpy_n`, `crab_join_n`).
  ⛔ Extracted at 0.5.0 for the reason `app.cyr` was extracted at 0.6.0: a memory-safety fix that
  cannot be asserted on is a fix held on trust.
- `src/ui.cyr` — dual-pane file browser: a pane is a **`dh_list`** (0.4.10), plus size/mtime
  formatting, row build, status line.
  ⛔ **`crab_render` OWNS THE PER-FRAME ARENA** (0.6.0) — creates it on first use, installs it, and
  calls `dh_frame_begin()`. Placed here rather than in `main.cyr` so every caller is correct without
  remembering, and so the suite can reach it at all.
  ⛔ **Every per-frame allocation here goes through `dh_falloc`, not `alloc`.** For `disp` and the
  status buffer that is a *lifetime* requirement: `dh_widget_set_text` stores the pointer and does
  not copy, so a global-alloc string on an arena widget outlives its widget forever.
  ⛔ **`crab_render` TAKES the surface, it does not create one**, and reads `w`/`h` back off it — so
  they cannot disagree with the surface they describe. The returned surface is **reused**; two frames
  at once need two surfaces.
  ⛔ `CRAB_ROWS_CAP = 7` is **gone** and deliberately not replaced by a bigger number — it was never a
  display limit, it made entries past the 7th unselectable.
  ⚠ **A row sets no background at all.** dhancha paints selection and focus from the list's own state;
  a row that keeps a background paints over the toolkit's highlight and selection silently stops
  showing.
- `src/render_test.cyr` — a standalone harness: renders the production surface and dumps BGRA to
  `build/crab-render.bin`. **53** `check()` assertions *(0.7.6; 26 at 0.7.5)*.
  ⭐ **IT NOW PRINTS ITS OWN CHECK COUNT.** It returned `g_fails` and printed only its dump line, so
  it exited **0** whether it ran 26 checks or none — while this file and the handoff quoted a count
  for three cuts that nothing in the program printed. `crab render_test: 35 checks, 0 failed`.
  ⚠ **It renders at 640x220, not 380** — every assertion in it is about two panes side by side, and
  below 600 crab now draws one. The solo layout has its own checks at 380.
  ⚠ It creates **two** `DhSurface`s on purpose — it holds two frames live and dumps the first, and
  since dhancha 0.9.14 one surface would hand both renders the same target.
  ⛔ **It is not run by CI or by `cyrius test`** — see Known gaps.
- `src/test.cyr` — ⚠ **deliberately empty, with a warning in it.** Bare `cyrius test`
  auto-discovers `tests/*.tcyr` and does **not** run the `[build].test` hook.

⭐ `cyrius coverage` reports **137/159 fns referenced (86 %)** at 0.7.5, 6/6 files — **the v1.0
criterion, met.** It dipped to 73 % mid-cut as M4's write layer landed faster than its tests, and was
recovered by writing the assertions that were genuinely missing (`crab_entry_cmp` directly, the
result messages, the notice channel, `crab_lower`, the listing accessors) rather than by naming
functions to move the number. ⛔ The roadmap now carries this as a **per-release gate**: a criterion
checked only at v1.0 gets further away at every cut that adds code.

⭐ At the 0.7.0 cut it reported **44/54 fns referenced (81 %)**, 6/6 files — up from 75 % at 0.6.1, 53 %
at 0.5.0 and 23 % at 0.4.15. ⭐ **This is the first cut above the v1.0 criterion of 80 %.**
⚠ Still a floor rather than a correctness proof: reference coverage counts a function as covered when
something references it, so `main.cyr` reads 1/1 because its only function is `main` itself. The 0.6.0
jump was the `app.cyr` extraction making a whole layer reachable; M3's is ten new `app.cyr` functions
that were written with their tests.
⚠ **13** lines exceed 120 characters (12 in `main.cyr`, 1 in `ui.cyr`) — up from 9 at 0.6.1, all in
the render call sites the deferred-stat drain added.

## Proven

### ⭐⭐ As of 2026-09-13 — QEMU, a real agnos kernel, aethersafha 0.16.24, one compositor

`agnos/scripts/harness/crab-pointer-test.py` (new), run 4, **PASS**: crab launched from the F2
launcher (deterministically — one held DOWN, one held Enter, the outcome read), ascended to `/` on
Backspace, resolved a left click to a pane, **opened the context menu on a right press that arrived as
button 2**, ran a menu entry from a left press on a popup row (`crab: copy tree bin` — row 1, refused,
`/bin` exists), dismissed the popup on a press beside it, **refreshed on `u` four times** (eight
listings), cycled the view on `g`, toggled the sidebar on `b` with VOLUMES from `mountlist`, and
answered keys throughout. No faults. ⛔ **And it measured the compositor consuming Tab and F10 and
quitting on Esc** — crab acted on none of the three. Verdict log beside the serial log in
`agnos/build/crab-pointer/`. ⚠ Runs 1 and 3 were contaminated by the harness's own doing (a stray
Enter, then a pointer pick of Open on `/bin/aethersafha` — each spawned a second compositor) and are
discarded; the lessons are in the harness header.
⚠ **Not on target**: the *you are here* marker (a pixel claim; the rule is host-pinned), the sidebar
keyboard route (unreachable — Tab is claimed), the menu bar and `View` (unreachable — F10 is
claimed), and iron.

### As of 0.7.5 — what is actually verified, and how

| claim | evidence | where |
|---|---|---|
| the write layer works | **real syscalls against a real filesystem** — mkdir, unlink, rename, open, read, write all exist on the host, so refusals, the bounded join, the overwrite guard and multi-chunk copying are exercised for real | `tests/crab.tcyr` |
| the recursive walk works | ⭐ `crab_walk_readdir` has a host arm over `getdents64`, whose `d_off` is a seek cookie with **exactly** `#101`'s semantics — so the tree state machine runs against real directories | `tests/crab.tcyr` |
| the render is correct at the pixel | 26 `check()` assertions over the production `crab_render` | `src/render_test.cyr` ⛔ **run by no gate** |
| a frame costs the global heap nothing | `alloc_used()` deltas across back-to-back renders | dhancha `arena_test` + crab's own |
| the sort is off the keystroke path | measured: 182.9 ms → 414 µs at the cap, reverse-sorted | `tests/crab.bcyr` |
| the declared dep graph resolves | check 4 — every `path` override disabled, **byte-identical binaries** | re-run at 0.7.4 |

⛔ **What is NOT proven: anything since 0.7.0 on iron.** See *Next*.
⛔ **And the agnos-only paths are proven only under QEMU** — the `#101` walk, the event loop, the
idle tick, spawn. Every `#ifdef CYRIUS_TARGET_AGNOS` region is invisible to the host suite by
construction; that is why the decisions inside them (`crab_readdir_stalled`, `crab_walk_cursor_for`,
`crab_truncation_note`) are lifted out as predicates the suite *can* assert.

### The historical retractions — kept because they warn against re-deriving a wrong answer


⛔ **The transport every pre-2026-08-03 agnos claim rested on is RETIRED — as the WRONG PRIMITIVE for
local display IPC, not as something that never worked.** A local display protocol has nothing to route,
nothing to checksum, no window to negotiate, and no business owning a port.

> ⚠ **Naming supersession (2026-08-26).** This block used to name the replacement "the agnos socket
> (`anu`)". **The codename is retired** — operator ruling 2026-08-05, *"a name is a distribution fact"*:
> a band that only ever appears as a prefix inside one kernel has no repo boundary to cross. The
> replacement is syscall **`#97 chan_op`**, `VFS_CHAN = 11`, kernel prefix `chan_*`
> (agnos `docs/development/planning/ipc.md` §9/§10). `anu` resolves to nothing in the kernel.

⛔ **Retracted — the "proven on agnos" lineage before 2026-08-02 is a FALSE GREEN.** The
setu-descend / setu-stat / readdir end-to-end observations all ran on the image staged by the deleted
`aethersafha-setu-smoke.sh`, which built the kernel with `AETHERSAFHA_SETU_SELFTEST=1` — that hook
assigned `net_ip = 0x7F000001`, the only reason the loopback handshake closed in that era. Before
`net_src_for` (agnos 1.56.34) an ordinary boot could not complete it.

✅ **Real, un-rigged, and NOT retracted** (2026-08-02, QEMU `-smp 1`, agnos 1.56.34+): the honest
harness `agnos/scripts/harness/aethersafha-clients-test.py` — which byte-scans `build/agnos` and
hard-exits if the kernel carries any selftest hook — reached **`connected: 2, presented: 2`**: crab and
setu's `present_probe` (`/bin/puka`). That harness is what CAUGHT the earlier rigging. It rode the
now-retired TCP path, so it is not a current capability claim — but it is evidence that the retirement
was architectural, not a failure verdict.

✅ **The re-establishment gate is CLEARED (0.4.5, 2026-08-07).** This section used to end "crab's agnos
standing must be re-established before anything is called proven again". It was: crab is the **second**
client in agnos 1.56.40's ipc bite 7 — the compositor mints a channel, endows one end, and spawns crab
already connected; crab dials nothing and `setu_connect` reads `AGNOS_CHAN`. Proven under QEMU
**`-smp 4`** alongside `present_probe`, both presenting, framebuffer-confirmed. ⚠ That also supersedes
the old "`-smp 4` fault-kills" note, which was true of the 2026-08-02 TCP-era image only.

⛔ **"Never run on iron" was FALSE, and stood in this file for eighteen days.** crab has burned on iron
at least twice, and both burns found real defects:

- **2026-08-08** — aethersafha's F4 removed the window from its own vector and told nobody, so crab was
  left **orphaned alive**, still holding its `#97` channel end and its `#86` GPU-visible shm slot.
  There are only **16 such slots system-wide**. Fixed in 0.4.6 by handling `SETU_CLOSE` (kind 7), which
  had been in the protocol from the start and was never sent by any compositor nor handled by any
  client.
- **2026-08-19** — a real `/` on iron held **114 entries**; the pane listed 32 and dropped 82 silently
- ⭐ **2026-08-30** — a real `/` on iron now holds **122 entries**, cross-checked against the shell's
  own `ls -a` in the same capture. Every "114" elsewhere in the docs is that older burn's number;
  the memory and allocation work was measured against it.
  (0.4.13, `CRAB_MAX_ENTRIES` 32 → 256), and the per-entry stat tracing that revealed it was itself the
  performance regression the operator reported (0.4.14).

✅ **QEMU, 2026-08-26, crab 0.5.0 on a real agnos kernel at `-smp 4`** — the 0.5.0 repairs, verified
rather than argued for:

- `agnos/scripts/harness/crab-listing-cap-test.py` — **PASS**. The `/bin` pane listed **45 of 45**
  entries, no truncation warning, no fault. Drives the repaired path layer on real ext2.
- `agnos/scripts/harness/puka-terminal-test.py` — **PASS**, background exit **95**, 2 compositor
  presentations. crab connected over the current channel-band transport, presented, and left its loop
  with `crab: compositor closed the window -- exiting` — the 0.5.0 `WINDOW_CLOSE` path on a real
  compositor.

✅ **QEMU, 2026-08-27, crab 0.6.0 + dhancha 0.9.16** — the first on-target run of the 0.6.0 tree and
of the M2 poll change:

- `crab-listing-cap-test.py` — **PASS**, exit 0, `/bin` listed **45 of 45**. The `app.cyr` extraction,
  the reused render target and the per-frame arena did not disturb the readdir/stat path on real
  ext2. ⚠ This harness never reaches the compositor, so it says nothing about the event loop.
- `puka-terminal-test.py` — **PASS**, background exit **95**, 2 presentations; serial carries
  `presented over setu` then `compositor closed the window -- exiting`. crab connects, presents and
  leaves cleanly **through the hoisted-scratch poll**. ⚠ The compositor closes crab's window quickly
  here, so it still **cannot distinguish loop-lifetime behaviour** — see the open item below.

⛔ **And QEMU EARNED ITS KEEP: it caught a regression the host suite could not see.** A 0.5.0 draft
idled with `sys_sleep_ms`, which `preempt_disable()`s — so while crab slept **nothing else could be
scheduled** and the compositor never presented at all (placed 2, presented **0**, `--clients` never
returned). The host suite was **37/37 green for that build**, because the loop is inside
`#ifdef CYRIUS_TARGET_AGNOS`. Shipped primitive is `sys_pause` (#14), which yields to a ready proc
first. ⇒ **Any change to the agnos event loop needs a QEMU run before it is claimed.**

⚠ **Not exercised on agnos:** `crab_descend` / `crab_ascend`. Neither harness drives navigation keys,
so the bounded-join *refusal* path has host assertions only.

## Shipped

| milestone | version | headline |
|---|---|---|
| M1 | 0.5.0 | the P-1 sweep; `src/path.cyr` extracted so the suite could reach it |
| M1.5 | 0.6.0 | **a rendered frame costs the global heap zero bytes**; `src/app.cyr` extracted |
| M2 | 0.6.1 | resize, pointer, wheel, held-key repeat |
| M3 | 0.7.0 | sorting, selection memory, argv paths, deferred statting, columns, `#101 readdir_at`, `on-accent` |
| M4 | 0.7.1 – 0.7.5 | the whole write layer: copy · move · delete · open · mkdir · rename, recursion, multi-select, the transfer tray, drag, the context menu, the batch sheet |

⚠ **Per-release detail lives in [`../../CHANGELOG.md`](../../CHANGELOG.md), not here.** This file had
109 lines of shipped-milestone prose when it was cleaned at 0.7.5; a state file that grows a section
per release stops being readable exactly when a cold start needs it most.

## Dependencies

Declared in `cyrius.cyml`. ⭐ **Re-verified 2026-09-12: all SEVEN declared tags equal that repo's
highest tag on its remote** (`git ls-remote --tags`, `sort -V`), every sibling tree clean on its tag,
and the declared graph resolves with **all four `path` overrides disabled** — 7 deps / 0 errors,
lock **3 → 7 commit-pinned**, `deps --verify` **50 verified / 0 failed**, host and `--agnos` both
build, **1748/0**.
⭐⭐ **AND THE OVERRIDE-DISABLED BUILD IS BYTE-IDENTICAL TO THE OVERRIDE BUILD** — host
**1,045,296 B** `d75c35a9…`, agnos **1,085,872 B** `9ca89ea3…`, both ways (measured at the dep
bump, before the pointer routes landed on top).
⛔ **0.8.4 SHIPPED WITH THE DIVERGENCE THIS CHECK EXISTS FOR.** Its `lib/` carried dhancha 0.9.29,
rupa 0.1.7 and setu 0.8.9 through `path` while the manifest declared 0.9.28 / 0.1.6 / 0.8.8 — so CI
compiled the old tags and the local build the new ones, green both ways, and nothing said so. All
seven are re-pinned in `[Unreleased]`; only chitra 1.0.1 → 1.0.3 carries behaviour (a decoder P-1
sweep), the rest are 6.6.2 pin moves with header-only dist changes. That is the strongest form check 4 can take: it
says not merely *"the declared graph resolves"* but *"the declared graph is what the local build has
been compiling all along."* ⚠ It holds only because every sibling working tree sits exactly on its
tag, clean; re-derive it rather than assuming it, since `path` is what makes it possible to drift.
⚠ **The older "6 deps" count in this section was wrong** — chitra made seven on 2026-08-31.

| dep     | tag    | `path`? | why crab needs it                                   |
|---------|--------|---------|-----------------------------------------------------|
| sadish  | 0.5.4  | no      | 2D vector — the surface everything else draws into   |
| rupa    | 0.1.7  | yes     | shared theme tokens + **`on-accent`** and contrast   |
| rekha   | 0.3.7  | no      | text; references `sd_*`. ⭐ adds the advance widths   |
| kashi   | 1.0.7  | yes     | CP437 8×16 glyph data for `dh_draw_text` (font=0)    |
| dhancha | 0.9.29 | yes     | widgets, `dh_list_new_h` (menu bar), `dh_theme_*`    |
| chitra  | 1.0.3  | **no**  | **thumbnails** — PNG/JPEG/GIF/BMP decode. ⛔ see gaps |
| setu    | 0.8.9  | yes     | client transport — channel-band, reads `AGNOS_CHAN`  |

⛔ **THIS TABLE WAS FICTION FOR PART OF 2026-08-28, AND THAT IS THE FAILURE MODE TO REMEMBER.** The
manifest named `rupa 0.1.5` and `dhancha 0.9.20` while **neither existed on any remote** — both were
local-only, and rupa's was not even committed. Every local build was green because `path` wins;
`cyrius deps` on the declared graph gave `4 deps resolved, 2 errors`. dhancha 0.9.20 was unusable by
*anyone* for the same reason (it pinned the same phantom `rupa 0.1.5`). Both are now genuinely
released — rupa `27e8385`, dhancha `61a1e39`.
⚠ **A `## [x.y.z]` CHANGELOG heading is not a release. A local tag is not a release.** Only
`git ls-remote --tags <url> | sed 's|.*refs/tags/||' | sort -V | tail` proves it — **`sort -V`**, or
`1.56.9` outranks `1.56.50`.
⚠ **`dhancha 0.9.19` never existed** — no tag, no CHANGELOG entry, no commit. `dh_theme_on_accent`
landed in **0.9.20**. Four crab files cited the phantom version and were corrected on 2026-08-28.
⚠ rupa and dhancha moved their **toolchain** pin to `6.5.35` (matching crab) as part of that repair.
sadish, rekha, kashi and setu remain on `6.5.27` and agnos on `6.5.28` — the operator's call: bump
them only when a repair lands in that repo.

⭐ **dhancha 0.9.13 / 0.9.14 / 0.9.15 are the per-frame allocation gate, 0.9.16 is the per-poll half
(together the render/input loop allocates nothing in steady state), and 0.9.17 is M2's
`dh_surface_resize`.** The allocation gate is CLOSED — the
dead pixel buffer deferred, the sadish render target reused, then the widget tree and layout scratch
moved onto a per-frame arena. **746,440 → 0 B per steady-state `crab_render` frame**; see Known gaps.
⚠ Two of the three are contract changes: 0.9.14's `dh_surface_render` may return the same surface
twice, and 0.9.15's `dh_frame_begin` clears dhancha's retained widget pointers as well as rewinding
the arena.

⛔ **THE TAG WAS VERIFIED FOUR WAYS BEFORE THE MANIFEST MOVED, AND ONLY THE FOURTH IS EVIDENCE.**
`path` wins over `tag`, so a green local build says nothing about the declared graph:

1. sibling `VERSION` = `0.9.17`;
2. `git rev-parse 0.9.17` == `HEAD` in `../dhancha` (`b297604`), working tree clean;
3. `git ls-remote --tags` shows `refs/tags/0.9.17` at that same commit;
4. ⭐ **`path` DISABLED so `cyrius deps` actually cloned the tag from the remote** — the resulting
   `lib/dhancha.cyr` hashed to `7b99ec62…`, identical to the `path` build and to
   `git show 0.9.15:dist/dhancha.cyr`. **The first three would each have passed 0.4.13**, the release
   that shipped a manifest naming a library the build never compiled.

⚠ **And check 4 is doing even more work than "path wins" implies** — see Known gaps: `lib/` is not
what compiles at all while `path` is set. Re-run all four at every cut. Automating it is roadmap
still open — see the roadmap's 0.8.0 batch.

⛔ **`path` WINS over `tag`, so a green local build is not evidence that the declared graph resolves.**
That is the drift 0.4.13 caught and closed. **Four of six** carry `path` — the table's own column says
which; the manifest's old "every other dep in this stack carries both" was false and is corrected as of
0.4.15. At every cut, re-verify each tag three ways: the sibling's `VERSION`, `git rev-parse <tag> ==
HEAD` in the sibling tree, and the newest tag actually published
(`git ls-remote --tags … | sort -V | tail -1`).

⚠ **crab consumes `dist/` bundles, not `src/`.** Five deps — sadish, rupa, rekha, dhancha, setu — are
`modules = ["dist/<name>.cyr"]`, so a fix reaches crab only after `cyrius distlib` runs **in that
sibling**; a local `path` override alone is not enough. **kashi is the exception**: it is
`modules = ["src/font_data.cyr"]`, the freestanding core, deliberately — the library face costs
+183,360 B (+50 %) for a runtime font registry crab never calls, and `CYRIUS_DCE=1` reclaims none of it.

⚠ **The `net` stdlib leaf is now REDUNDANT, not load-bearing.** The manifest claimed "`net` stays until
setu moves off TCP" — setu moved off TCP in **0.8.4 (2026-08-07)** and crab has pinned past it since
0.4.5. Measured 2026-08-26: with `net` deleted from `[deps].stdlib` **and** `lib/net.cyr` removed,
`cyrius deps` re-creates the leaf (30,092 B) from setu's `dist/setu.deps` sidecar and `cyrius build` is
OK at the same 377,288 B. ⇒ Dropping the declaration is a real cleanup — held back from 0.4.15 as a
separate change, not bundled into a version bump.

## Tests

- `tests/crab.tcyr` — the only suite `cyrius test` discovers. **2,012 passed / 0 failed**
  *(unreleased; 1,838 at 0.8.6, 1,790 at 0.8.5, 1,695 at 0.8.3/0.8.4, 1,230 at 0.7.7, 253 at 0.7.0)*
  ⭐ **+41 unreleased — `t_refresh`**: the selection rule, the exact re-seat asserted AGAINST the
  containment rule and against the `/` place/volume alias, the menu entry, the sidebar gate, the
  kept thumbnail cache with the write-op relist as control; six mutations each caught.
  ⭐ **+48 at 0.8.6** — `t_view_menu` (the display ids sit above the context menu's bound, every
  View slot is a display item, the accelerator column agrees with the key sent, none is eaten by
  sidebar focus, always live, the drop fits at 380 px) plus the View drop-down rendered; five
  mutations, each caught.
  ⭐ **+95 at 0.8.5** — `t_pointer_routes` (+53: the blocked/modal split, the button numbers pinned
  against X11, the separator inverse's round trip for every item, the z-order of
  `crab_pointer_action` — 31 assertions, seven mutations each caught) and `t_sb_here` (+42: the
  trailing-slash trim, containment-deepest-wins with the tie pinned, the 64-byte prefix layout —
  six mutations plus one on the layout, each caught, one of them through the render wiring).
  ⭐ **+92 at 0.7.7, all of them pinning defects that had already shipped**, in five new groups:
  `t_dir_transfer`, `t_queue_refusals`, `t_transfer_plan`, `t_menu_highlight`, `t_tray_height`.
  ⛔ **Each was mutation-proven** — the guard removed and the suite watched to FAIL (9, 4, 5, 3 and
  5 failures respectively) — because this project has shipped three tests that could not fail in
  their first draft.
  ⛔ **NEW GROUPS GO IN THEIR OWN FUNCTIONS.** `main` reached 2,517 lines and **279 locals**, and one
  more group pushed its stack frame past what the process could touch: the suite **segfaulted**
  (exit 139) part-way through, having printed the group header, with every prior assertion passed.
  It looked exactly like a bug in the code under test. `t_img_dims`, `t_preview` and
  `t_preview_dims` are separate functions; anything added from here on should be too. (119 in M3 —
  44 sorting 12 selection memory 10 argv 28 deferred statting 25 real
  columns; 55 mid-0.6.0, 37 at 0.5.0, 11 at 0.4.15). ⭐ It now includes **`src/app.cyr`**, not `ui.cyr`, so the application
  layer is reachable for the first time.
  Covers the AE-6 premultiplied `#92` contract on the **production** `crab_render` (`a == 255` and
  `c <= a` across all 83,600 pixels, with negative controls); 0.5.0's repairs (bounded path helpers,
  the size ladder at every boundary, the date-formatter clamps, the name-truncation marker); 0.6.0's
  surface reuse (identity, frame independence over a full 334,400-byte compare, a scribbled-sentinel
  coverage check); 0.6.0's zero-cost frame (twenty renders moving the global heap by exactly 0 bytes,
  the arena holding one frame not twenty, the spill path when an arena cannot fit a frame); and the
  application layer (descend's four refusals including the over-long join, ascend's root and
  one-segment cases, the stat-failure default of -1, the unconditional surface flag).
  ⭐ **Mutation-proven throughout** — 7 mutations against the app layer and arena ownership, 3 against
  surface reuse, 5 against dhancha's arena, each producing a named failure.
  ⛔ **THREE TESTS COULD NOT FAIL IN THEIR FIRST DRAFT, AND ONLY MUTATION SAID SO.** The truncation
  test at 0.5.0 mirrored `crab_row`'s loop and asserted on the copy. At 0.6.0: the residue check
  rendered trees that repainted every pixel, so deleting dhancha's `sd_clear` left it green; and the
  convergence check used a 256 KiB arena against a three-entry fixture, so deleting `dh_frame_begin()`
  **entirely** left the whole suite green — an arena that is merely big enough never touches the
  global heap whether it is rewound or not. ⇒ **Size a fixture against the mechanism, not for
  comfort**, and prove the assertion can be observed to fail.
  ⚠ **The zero-cost gate has TWO independent guarantors and no single mutation fails it** — deleting
  dhancha's `sd_clear` leaves it green (crab's opaque root still covers), making crab's root
  transparent leaves it green (the clear still covers); only removing **both** fails, at 7,744
  surviving bytes. Correct for a property test, but **a green suite here is not evidence that the
  toolkit still clears** — that is pinned in dhancha's `programs/draw_test.cyr`.
  ⚠ **`main()` itself remains uncovered and that is irreducible** — a suite that included `main.cyr`
  would run the app. It is now down to the event loop alone; every other line moved to `app.cyr`, and
  the one setup step `main()` used to own (the frame arena) moved into `crab_render`.
- `tests/crab.bcyr` — ⚠ **measures the sort**, not `bench_noop` (that note was stale from 0.7.0).
  Current: merge **88.6 µs** vs insertion **5.66 ms** at 256 scrambled; merge **38.3 µs** vs
  insertion **1.28 ms** at the real iron 122. ⛔ Still nothing writes `docs/benchmarks.md` from it —
  that is the v1.0 criterion, and it is the half that is missing.
- `tests/crab.fcyr` — ✅ **A REAL FUZZ HARNESS as of 0.7.6.**
  It was a scaffold that read none of its input, so `cyrius fuzz` PASSED against anything. It now
  drives **100,000 rounds** (⛔ measured and printed by the harness itself since; this line said
  60,000, as five others did) over mutated format headers, random bytes, degenerate/negative lengths,
  and arbitrary bytes through `crab_name_ok` / `crab_is_image` / `crab_cstr_len` — deterministic
  from a fixed seed, asserting an invariant rather than only the absence of a crash.
  ⛔⛔ **ITS OWN FIRST DRAFT WAS VACUOUS AND THAT IS THE LESSON.** An LCG's low bits have period 2^k,
  so the format selector returned **only 1 and 2 across 20,000 rounds** — PNG and JPEG were never
  seeded and `crab_jpeg_dims` was entered **zero** times while it printed `fuzz: ok`. Caught only by
  planting a known bug and watching the fuzzer pass. ⇒ **Plant a bug in a new fuzzer before
  believing it.**
- `vet` / `deny` take a source argument (`cyrius vet src/main.cyr`), not a bare invocation —
  ⚠ the old row here implied otherwise. `fmt --check` clean across `src/` and `tests/`.

## Targets

| target       | status                                                    |
|--------------|-----------------------------------------------------------|
| x86_64 linux | ✅ builds, **1,084,832 B** *(0.9.3; 1,084,744 at 0.9.2, 1,080,480 at 0.9.1)* ⚠ 0.8.5's size did not move across the marker while its hash did — `cmp`, never `ls -l` |
| `--agnos`    | ✅ builds, **1,129,864 B** *(0.9.3; 1,129,760 at 0.9.2, 1,121,344 at 0.9.1)* — ⭐ **and it draws in Liberation Sans on a real kernel** (`crab-face-test.py`, 2026-09-14) | — the real target, **CI builds it**, and ⭐ **it ran on a real kernel under QEMU on 2026-09-13, three times** (the pointer, columns and shift harnesses — see *Proven*) |
| `--win`      | ⛔ fails: `sys_socket` / `sys_connect` undefined            |

⚠ The `--win` failure is **pre-existing, not a regression** — the 0.4.14 tree on the 6.5.28 toolchain
fails with the identical two symbols. ⛔ **And the obvious cause is the wrong one.** It is not the
retired `net` transport: `lib/net.cyr` names neither symbol (it uses the generic
`syscall(NSYS_SOCKET, …)` form, which emits no named reference). The only callers are `lib/setu.cyr`
:971 / :973 / :1022, and they are **AF_UNIX / SOCK_SEQPACKET** — they exist *because* setu already left
TCP. The real cause is target-table coverage: both are defined in `lib/syscalls_linux_common.cyr`
(:470, :515), `lib/syscalls.cyr` routes `CYRIUS_TARGET_WIN` to `lib/syscalls_windows.cyr`, and that
file defines `sys_socketpair` but neither of these. Windows is not a declared crab target.

## Known gaps

> ⛔⛔ **THIS SECTION HAD ROTTED A THIRD TIME WHEN IT WAS REWRITTEN AT 0.7.5.** It still claimed
> *"five open defects, none fixed"* (all five closed at 0.7.1), *"crab is read-only — no copy, move,
> rename, delete or mkdir"* (the entire M4 write layer had shipped), *"the window is a compile-time
> 380×220 and crab is keyboard-only"* (resize and pointer shipped in **M2**, two milestones earlier),
> and *"the MODIFIED column never shows at the default window size"* (fixed at 0.7.2).
> ⚠ The file's own header has warned about this twice before. **It is not neglect — it is that
> "known gaps" is written when a gap is found and never revisited when it closes.** Re-read this
> section at every cut and delete what is no longer true; a closed gap left here sends the next
> session to fix something that already works.

### Real, and open right now

- ✅ **The security audit exists** — [`../audit/2026-08-31-audit.md`](../audit/2026-08-31-audit.md),
  the first. ⛔ **Its headline (F1) is that GALLERY VIEW CHANGED THE TRUST MODEL**: crab decodes every
  image in a folder the operator merely opened, so ~22,500 lines of `chitra` + `sankoch` now run
  in-process on attacker-chosen bytes. Measured at 8 decodes per folder opened against 1 per entry
  selected. The per-image and session budgets bound **memory**; nothing bounds the code paths a
  crafted 64x64 PNG reaches.
  ✅ **Every finding is CLOSED in 0.7.6.** F1 by parsing only what is on screen (`crab_grid_visible`);
  F2 by inverting the order — crab spawns first and reads the ELF magic only to explain a failure, so
  there is no check left to race; F4 by re-dispatching an untyped directory instead of reporting a
  failure. ⚠ agnos has no `fexecve`/`execveat`/fd-spawn, which is why F2 is fixed by removing the
  check rather than by spawning the checked descriptor.
  ⛔⛔ **F3's first draft was a FALSE FINDING and the audit keeps it, corrected.** It claimed an
  unbounded read in `crab_batch_name` that the code cannot perform — the destination cap bounds the
  read below `CRAB_NAME_MAX`. Caught by planting the implied mutation and watching the suite stay
  green. What was real: no fuzz coverage (now closed), and a safety that depends on
  `CRAB_EDIT_CAP <= CRAB_NAME_MAX`, two constants that can move independently — now asserted.
- ✅ **CLOSED at 0.7.7 — `render_test`'s 53 pixel assertions RUN IN CI**, as their own step, with
  the exit code (the failed-check count) as the assertion. ⚠ Still true, and why the
  step is explicit rather than discovered: `cyrius test` does not find it, and `[build].test` is
  inert. ✅ **CLOSED at 0.7.7**: `[build].test` now reads `tests`, and the orphaned
  `src/test.cyr` is deleted. ⚠ Repointing it did not make it a GATE — measured by pointing it at a
  file that exits 1 and watching `cyrius test` still return **rc=0**. It names where the tests are;
  it does not run them.
- ✅ **CLOSED at 0.7.7 — CI builds `--agnos`.** `release.yml` gates on `ci.yml` via
  `uses:`, so a tag inherits it. ⛔ The flag is `--agnos`: `CYRIUS_TARGET=agnos` is **silently
  ignored** and produces a byte-identical host binary at exit 0.
  ⚠ **The BLIND SPOT ITSELF IS NOT CLOSED, only the build of it**: 1,221 of `src/main.cyr`'s 1,494
  lines are inside that one `#ifdef` with no `#else`, and nothing includes `main.cyr` — so the whole
  key-dispatch table now COMPILES under the gate but is still executed by no test on any target.
  0.7.7's defect fixes answered that by moving the decisions out into `crab_transfer_plan`,
  `crab_menu_row`, `crab_tray_h` and `crab_fs_isdir`, which the suite can reach.
- ✅ **CLOSED — the fuzz harness now reads its input.** **100,000** rounds
  over mutated headers, random bytes and degenerate lengths, deterministic from a fixed seed. It
  caught a real segfault in `crab_img_dims` the day it was written.
  ⚠ **What it still does NOT cover**: `crab_readdir_into` (agnos-only, needs a syscall), the write
  layer's path joins, and `crab_batch_name`'s pattern expansion. Those are the next targets.
- ⛔ **The AI arc is promised in three shipped documents and declared nowhere.** The package
  description, the `[deps]` comment and the README all commit to daimon; `cyrius.cyml` declares no
  daimon dep, and **daimon 2.1.2 exists locally**.
- ✅ **CLOSED in 0.7.6 — `crab_render` takes one record, not 32 positional parameters.** (The
  count here said 33; it was 32.) `crab_rs_pane` / `_op` / `_chrome` / `_preview` / `_dims` fill it,
  max arity 11, and `crab_rs_reset` owns the three `-1`-means-unknown defaults that 23 call sites
  used to spell by hand.
- ⚠ **The idle-tick wiring is untested and structurally untestable on the host.** The walk, the copy
  stepper and the queue are all driven by hand in the suite; that they are *called* from the tick is
  agnos-only event-loop code. Same irreducible gap `main.cyr` has always had.
- ✅ **Shift — CLOSED (0.8.9).** ⚠ The line here said *"Shift is not on the wire"*, and only half of
  that was ever true: `mods` does carry press/release and nothing else — but the modifier's OWN edge
  arrives as its own key event, and aethersafha exempts those edges from the chord swallow on purpose
  (*"a client that wants Shift state has no other way to learn it"*). ⇒ `crab_shift_track` folds
  usages 0xE1/0xE5 into a two-bit mask and the field reads `crab_shift_held()`. **Names can hold
  capital letters**, and the shifted number row is filled — which also makes `#` and `*`, the batch
  sheet's own two operators, typeable into the field that advertises them.
- ⚠ **crab cannot recreate a symlink** — but the GATE on it is gone. A recursive copy copies whatever
  `open`+`read` yields through one. ⭐ **`lstat`#102 GOT its cyrius peer**: 6.5.37 shipped `sys_lstat`
  and crab vendors it as of the 0.7.7 pin — 3-arg on agnos, 2-arg on the host, the same `#ifdef`
  arity split `crab_fs_exists` already resolves. ⇒ **This is now a decision, not a limit**: what
  crab should DO with the answer (refuse? report? recreate?) is unanswered, and that is the work.
- ✅ **CLOSED in 0.7.6 — `README.md` § Status.** It said *"Shipping, and read-only"*, which M4
  falsified, and quoted the retired 256-entry cap. Now states the write layer, the preview column,
  and what is genuinely absent. ⚠ **It has been wrong in both directions now**; the replacement text
  carries that warning itself.
- ⚠ **Focusing a pane by its header does not work** — the header is a sibling of the list, so
  `crab_hit` resolves a header click to no pane. Clicking a row is correct.
- ⛔⛔ **`crab_fs_open_w` BEHAVES DIFFERENTLY ON THE TWO TARGETS, AND THE TARGET THAT SHIPS IS THE
  PERMISSIVE ONE.** The host arm is `O_WRONLY|O_CREAT|O_EXCL` — M4's overwrite guard, which refuses
  an existing file and returns `EEXIST` — while the agnos arm is `AO_WRONLY|AO_CREAT|AO_TRUNC` with
  **no `AO_EXCL`**, so on agnos the same call TRUNCATES an existing file. Every host assertion about
  "crab will not overwrite" is therefore a claim about the host only. Found during 0.7.6 while
  testing the preview's dimension cache; pinned by a host assertion, **not changed** — altering
  write semantics is an operator decision, and a recursive copy is what would notice.
- ⛔⛔ **EVERY THUMBNAIL DECODE IS PERMANENT, AND THAT — NOT THE BINARY SIZE — IS THE LIVE
  CONSTRAINT.** chitra makes 31 `alloc()` calls, `chitra_image_free` is a **no-op**, and cyrius's
  `alloc` has no `free()`. Measured: **~2.5x the RGBA size per decode, never reclaimed**, and a
  re-decode of the same file costs it again. crab bounds it two ways — a per-image pre-check
  (`CRAB_THUMB_MAX_RGBA`, 4 MB; a refusal costs **16 bytes** against up to 26.6 MB unbudgeted) and a
  session ceiling (`CRAB_THUMB_TOTAL_MAX`, 32 MB). ⚠ **Those two constants are the first thing that
  should be deleted** the day the allocator gains a `free()` or chitra takes an arena.
  ⚠ **The session ceiling's WIRING is not pinned by a test** — only its boundary arithmetic is
  (`crab_thumb_over_budget`). Proving `crab_thumb_step` still consults it needs a test that really
  spends 32 MB; the guard is held by review and by the ⛔ at the call site.
- ⚠ **crab's binary more than doubled**: host 466,056 -> **1,003,168 B (+115.2 %)**, agnos 491,856 ->
  **1,026,872 (+108.8 %)**, ~399 KB of which is the `sankoch` inflate leaf PNG requires. Operator
  ruling 2026-08-31, taken with the measurements in hand.
- ✅ **chitra 1.0.1 pinned 2026-08-31** (`b777d34`, verified on the remote). The >16 MiB PNG failure
  is now cheap and correctly named, and the sidecar dropped three unused stdlib leaves: host
  1,011,496 → **1,007,320 B**, agnos 1,035,200 → **1,031,008**.
- ⛔ **TWO EXIF GUARDS ARE CAUGHT BY NOTHING, AND THE FUZZ HARNESS SAYS SO.** `crab_ex_ifd`'s
  per-entry bound (`e + 12 > len`) is the single most important line in that parser, and deleting it
  leaves both gates green: the overread lands in the poison tail, reads as tag `0x7E7E`, matches
  nothing, and returns nothing — **the bytes only reach control flow**, and a detector that watches
  the output cannot see them. The `CRAB_EX_MAX_ENTRIES` cap is bounded by that same line, so it
  limits work rather than reach. Both are held by review.
  ⚠ **This is why an out-of-bounds read is hard to fuzz here at all**: cyrius's allocator is a bump
  allocator over a large mapped heap, so reading past a buffer returns garbage instead of faulting.
  The poison tail catches overreads that reach the OUTPUT; nothing catches the rest.
- ⛔ **`crab_thumb_draw`'s clip test is unexercised defence, and `render_test` says so.** Deleting
  the clip comparisons leaves the suite green: dhancha's BOX_V compresses its children rather than
  overflowing them, so no cheap fixture produces a canvas laid out partially outside its column.
  Two attempts to build one failed to discriminate. The guard stays — measured, not assumed.
- ⚠ **The preview's dimension read is on the selection path, not the idle tick.** It is memoised on
  (directory, name) and capped at 64 KiB, so it costs one open/read/close per newly-selected image
  and nothing otherwise — but a directory of huge JPEGs arrowed through quickly still pays per
  entry. The idle-tick stepping that thumbnails will need is the same machinery that would move it.
- ⚠ **The zero-allocation gate covers the states its fixture renders, and nothing else.** It caught
  nothing for three cuts while `crab_overlay` leaked 32 B per frame with a menu open, because the
  fixture never opened one. Arms now exist for the menu, the sheet and the preview. **A new
  render-path branch without an arm here is a new blind spot, not a covered feature.**

- ✅ **CLOSED by aethersafha 0.16.25 — the compositor's chrome keys are Ctrl chords, bare Esc/Tab/F-keys
  arrive.** The gap (measured 2026-09-13: crab acted on zero of Tab/F10/Esc, Esc quit the desktop) is
  the filing at [`issues/2026-09-13-aethersafha-claims-esc-tab-f10.md`](issues/2026-09-13-aethersafha-claims-esc-tab-f10.md),
  closed the same day. Measured again with 0.16.25: F10 opens the bar, `View` drives from the keyboard,
  bare Esc/Tab reach crab, Ctrl+Q quits. ⚠ crab's required half: a modifier's own edge is ignored
  (`crab_key_is_modifier`). ⚠ Still: F2/F3 stay bare (not in the ruling). ✅ **Shift for capital
  letters was "a road, not a feature yet" here; 0.8.9 walked it** — the forwarded modifier edges are
  read as STATE by `crab_shift_track`, exactly as this line predicted.

### Hazards that are permanent, not gaps

- ⛔ **`path` wins over `tag`**, and four of six deps carry an override. A green local build is not
  evidence the declared graph resolves — only a resolve with every `path` line disabled is.
  **sadish and rekha are tag-only**, which makes them the only two whose remote resolution a local
  build genuinely exercises. Do not add overrides for them.
- ⛔ **`lib/` is not what compiles** while a `path` override is set — the sibling's `dist/` is. A
  cross-repo change must be followed by `cyrius distlib` in that repo, and for dhancha by
  `sh scripts/sync-deps-sidecar.sh` after it.
- ⛔ **Build both targets, every time.** See the `--agnos` gap above.
- ⛔ **`src/ui.cyr` is below `src/app.cyr`.** The render path must never call up; `render_test`
  includes `ui.cyr` alone. This was violated three times during M4.

## Consumers

_None — top-level application._

## Next

⭐ **0.8.2 CLOSED THE AUDIT BACKLOG'S CORRECTNESS SECTION — all eight, plus the recursive-walk
defect none of them had noticed.** See the CHANGELOG and the roadmap's *Unfinished from earlier
stages*. ⭐ **The six M6 interaction gaps are CLOSED** (2026-09-13): 0.8.3 gave the sidebar its
keyboard route; 0.8.5 gave every surface its pointer route — right-click, popup pick and dismiss,
bar and switcher clicks — on aethersafha 0.16.24's button numbering, and lit the sidebar row the
active pane is in; 0.8.6 filled `View`; `[Unreleased]` adds the **REFRESH key** and **proves the
pointer routes on QEMU**. What remains from that audit: two absent affordances (no flag surface, no
overwrite policy) and the *Recorded as facts* list. M6's two GATED items stand (the 🦀 chrome button,
the held-key repeat number). ⭐ The upstream keys gate that opened and closed on 2026-09-13 (aethersafha 0.16.25) made the menu
bar, `View`, the sidebar keyboard route and every Esc real on the target.

**M4 is complete. Every UNGATED M5 item is in, and M6 is closed but for two gated items.**
0.8.0 shipped M6's sidebar, menu bar, switcher and Bueller; **0.8.1** closed the VOLUMES gate on
agnos `mountlist`#104 and hardened the delete prompt after the `/bin` incident.
⚠ **0.7.7 was a repair cut** — five shipped defects closed, the pin moved, CI went from one step to
nine — and is kept here only because the *reasoning* below still refers to it.

⭐ **Done in 0.7.6**: the `crab_render` parameter cleanup, the preview pane, header-only image
dimensions, **thumbnails**, and **EXIF** (camera + shot) — plus (the fuzz harness)
closed, a shipped per-frame leak in `crab_overlay` fixed, and chitra at 1.0.1. ⚠ *dhancha was
0.9.26 at that cut; it is **0.9.28** now.*

⛔⛔ **THE 6.5.41 PIN RETIRED TWO GATES THIS SECTION USED TO LIST AS BLOCKED, and neither needed
crab work to unblock:**
- **Sidebar VOLUMES — capacity**: cyrius **6.5.37 shipped `sys_statfs`**, crab vendors it, and
  cyrius's issue is archived. ⚠ agnos-only (no host arm, so no host test can exercise it), and **no
  `STATFS_*` field offsets are vendored** — the frozen 32-byte layout must come from agnos's docs.
  ⭐⭐ ***Enumeration* IS NOW CLOSED TOO** — this line said it was open "because `mount`#11 /
  `umount`#24 are no-op stubs", which was true when written. crab filed it upstream on 2026-09-02;
  agnos **minted `mountlist`#104** rather than widening `mount`#11, adopting both halves of the
  filing's advice, and **0.8.1 consumes it**. ⇒ Another gate that closed without crab waiting.
- **Symlink detection**: **`sys_lstat`** ships on both targets. What crab should DO with the answer
  is now the open question, not whether it can ask.
⇒ **Both had been written as OPEN for four cyrius releases.** Same failure as the idle-poll buffer, carried OPEN for
nine. ⛔ **Re-derive a gate before believing it — including the ones in this file.**

✅ **M5's columns closed at 0.8.8** — as a view mode of one pane, not N miller panes; the gate was
never dhancha and was only half the truth about crab, too. The rest of this paragraph is the record
of how it was carried before it closed, kept because the *reason* still governs anything N-deep:
⛔ **What remained in M5 until 0.8.8**: **columns/miller**, gated on crab's own two-pane model — a design
question, not a dependency (`dh_columns_new` is absent from dhancha, but columns was never a dhancha
gate: it is a `BOX_H` of `LIST`s). And **proportional text**, whose gate is MIS-STATED in the
roadmap as "rekha + dhancha font plumbing": the plumbing exists and crab already forwards `font`.
What was missing was **advance widths** — ⭐ **and that upstream half CLOSED on 2026-09-02**:
rekha **0.3.6** adds `rekha_advance_width` / `rekha_char_advance_px` and dhancha **0.9.27** consumes
them in `dh_text_advance`, keeping the old hard-coded `advf = (h * 6) / 10` only as the no-metrics
fallback. crab declares both floors. ⛔ **What remains is crab-side and is not a dependency**: it
still passes `font = 0`, and `CRAB_COL_CHARW = 9` plus the five constants derived from it must move
first — see the roadmap, where that constant is load-bearing correctness rather than cosmetics.
⭐ `dh_grid_new` and `dh_list_new_h` are both present and resolvable; GRID is consumed, the
horizontal strip is not.

**Available now, in no particular order — sequencing is the operator's:**

- ⭐ **Adopt dhancha 0.9.24's stable keys and delete three workarounds.** crab hand-rolls its own
  press tracking, its own drag state and its own edit buffer because dhancha identified widgets by
  pointer and crab's arena invalidates pointers every frame. 0.9.24 fixes that at the cause —
  `dh_widget_set_key` + `dh_surface_set_root` re-binding — so `dh_dispatch`, drag-under-arena and
  `dh_text_attach` are all usable now. ⚠ crab uses none of it yet.
  ⛔ **Adopting `dh_dispatch` would relitigate the 2026-08-27 operator ruling** (*crab owns its
  interaction state*). That ruling was made because of the pointer problem, which is now fixed — but
  it is still a ruling, and reversing it is the operator's call, not a consequence of the fix.
- **The `crab_fs_open_w` target divergence** — the shipping target has no overwrite guard, because
  agnos has no `AO_EXCL`. Filed (⚠ at `0x2000`; the first filing proposed `0x400`, which is
  `AO_APPEND`). See *Known gaps*.
- ✅ **The Gallery view is IN.** `g` cycles list → grid → gallery → columns (0.8.8). ⛔ **The view never triggers a
  decode — the idle tick does, one per tick** — so opening a gallery of a thousand files costs one
  frame and the pictures land progressively. Backed by a 64-slot (~1.07 MB, allocate-once) cache
  that stores results **and refusals**. ⚠ Measured: 8 real PNGs for 1,075,160 B of permanent spend,
  well inside the 32 MB ceiling.
  ⛔ **The cache lives in `ui.cyr`, not `app.cyr`** — the gallery looks one up per visible cell while
  building the tree, which is the render path. **Fifth time that rule has decided placement.**
- ✅ **The Grid view is IN** (`g`), on dhancha 0.9.25's GRID kind. Cells are derived from the NAME
  column's own floor, so the view changes only how many entries fit; no column header, because a
  grid shows only names. ⛔ In grid mode the ARROWS navigate and `h`/`l` keep switching panes —
  list mode is unchanged. ⚠ **Not a thumbnail gallery**: 40 cells at 256x256 is ~28 MB of permanent
  decode against a 32 MB ceiling, so cells are names and the preview column carries the one
  thumbnail. That is a budget decision, not a layout one.
- ⛔ **The "dhancha COLUMNS" gate is FALSE — the fourth on record.** Columns is a `BOX_H` of
  `LIST`s: each already has its own selection, scroll and toolkit-painted highlight, so it clears
  neither bar of dhancha's kind rule. **What it is actually gated on is crab's two-pane model** —
  the source/destination pairing the whole M4 write layer rests on. That is a design question and
  belongs to crab, not to dhancha.
- ⛔ **BOTH M6 GATES WERE WRONG, AND IN DIFFERENT WAYS — re-derived 2026-08-31.**
  - **The sidebar's "Gate: dhancha TREE" is FALSE — the fifth false gate on record.** Every piece
    exists: `LIST` (scroll, selection, toolkit-painted highlight), `DH_FLAG_INERT` (section headers
    the keyboard steps over), `PROGRESS` (capacity bars), padding (indent). Expansion state is app
    state either way — crab owns its interaction state by the 2026-08-27 ruling — and the small
    window's drawer is `dh_place_pinned` plus the overlay layer, both shipped in 0.9.23.
    ⇒ **The sidebar is buildable in crab today, with no dhancha work at all.**
  - **The menu bar's gate was REAL but MIS-NAMED.** What was missing was not a `MENU BAR` kind but a
    **horizontal selectable strip**: composing one from a `BOX_H` of labels makes the app paint the
    current item's highlight, which means naming `accent`, which ADR 0001 forbids. dhancha 0.9.26
    adds `dh_list_new_h` — the same container laid the other way — which serves a menu bar, a tab
    strip, a toolbar and crab's own A/B switcher. ✅ **crab pins 0.9.28 as of 2026-09-07**, and
    check four passes against the declared graph. ⭐ **crab now CONSUMES it** — this line read
    *"crab consumes none of it yet"*, true on 2026-09-01 and false since **0.8.0**, which built both
    the `F10` menu bar and the A/B view switcher on `dh_list_new_h`.
  ⇒ **SIX false gates now** — proportional text is the sixth, and it was wrong in a new way again: not about existence or price, but about **which half was missing**. dhancha's font plumbing and rekha's glyph path both exist; what does not is `rekha_advance_width`, so a session would wire a font, watch text render, and only then find every glyph 0.6 em wide.
  ⇒ **Five before it.** A gate is a claim about another repository, and this one was wrong
  about *existence* (TREE), about *price* (thumbnails), and about *what was actually missing*
  (MENU BAR). Re-derive all three before believing a line.

### What is verified, and what is not

Stated as fact, not as a priority. ⚠ The last on-target run was **2026-08-30, against the 0.7.0
tree**. Everything since — the write layer, the tray, recursion, the menu, the edit field, the
render-state record, the preview, thumbnails and EXIF — has run on the host and under QEMU only.
Every `#ifdef CYRIUS_TARGET_AGNOS` region is invisible to the host suite by construction. agnos has
moved 1.56.53 → 1.56.55 underneath.

⛔ **Before believing any gate, re-derive it.** Three false gates on record: M4's *"Gate: agnos write
syscalls"* (real since agnos 1.41.3), drag's *"gated on nothing"* (it was gated, elsewhere), and
thumbnails' *"no image decoder"* — where chitra existed, and the real obstacle turned out to be a
**cost** no gate line mentioned. ⇒ A gate can be false in both directions.

Two decisions are settled and shape everything downstream:
[ADR 0001](../adr/0001-compositor-owns-theming.md) (the compositor owns theming; crab ships no
palette) and [ADR 0002](../adr/0002-semantic-find-is-a-mode.md) (semantic find is a mode over any
view, not a view of its own).
