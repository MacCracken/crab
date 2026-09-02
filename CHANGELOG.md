# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

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
