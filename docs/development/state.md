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

**0.11.3 IS CUT** — `VERSION` reads `0.11.3` and the CHANGELOG header agrees, 2026-11-08. 0.11.2 is
tagged on the remote (`e8d1b38`). Re-run `git describe --tags` and `git log --oneline -3` before restating this;
the commit, the tag and the push are the operator's.

⭐⭐ **daimon NOW HAS THE FILE SURFACE M7 AND M8 WAITED ON — M8's ranked by words, not meaning. Nothing in
crab uses it, and it does not unblock M7 by itself.**
✅ daimon builds and runs on agnos — re-derived for **2.5.5** on 2026-11-07 (`--agnos` from its tag,
**0 errors**; every CI job on the 2.5.5 commit passed, *"AGNOS guest test"* among them).
⭐ **Since 2.5.4, `/v1/files`** (daimon ADR-010, `src/api_files.cyr`): an index of the files under the
operator's roots and its state (`none` / `building` / `catching_up` / `paused` / `fresh`), tags and
ratings per file with counts and suggestions, six smart folders as saved queries, and duplicates by a
whole-content SHA-256 — the filing's items 1–4, M7. **2.5.5** adds item 5: `GET /v1/files?q=…`, ranked,
each result with a `why`, facets and the folders it appears in.
⛔ **What is still not there:**
- **meaning.** It ranks by WORDS (`cost.method` says so; `receipt` does not find `invoice`), and M8's
  canvas says *"vector + name match"*. No local embedding model exists. The filing is 🟡, **open**.
- **a running index on agnos.** It is OFF unless daimon is started with `serve --index-root DIR`, and
  nothing in agnos does that: only its smoke scripts start daimon at all (`git grep` over agnos, 2026-11-07).
  Reading contents is a second opt-in. The API has **no caller authentication until daimon 2.6.x**.
- **battery.** On agnos a walk does not pause on battery; the kernel has no power source to read (filed
  with agnos by daimon).
- **a client.** crab vendors no `net` leaf, so M7's first crab-side step is an HTTP client to
  127.0.0.1:8090.
The filing: `daimon/docs/development/issues/2026-09-23-crab-needs-a-file-index-tags-and-ranked-search.md`
(crab's copy in `issues/`). ⛔ crab will not fake an index it cannot back, nor build one of its own.

**0.11.3 contents — dhancha 0.12.0.** A pin release, and no line of crab's code moved. Text on a real face now draws at
the 16 px crab measures it at, so a name stops at its column (69 overrun pixel columns on Liberation Sans → 0); a resize
re-uses dhancha's render target; a polled event is free and valid only until the next poll. Four of the five upstream
items 0.11.2 listed are closed; agnos's 62-byte name cut remains. Host **390,400 B**, agnos **663,336 B** under DCE.
**2543 / 0 and 180 / 0**, render_test 55, the 640x220 dump `cmp`-identical to 0.11.2's.

**0.11.2 contents — the P(-1).** Six read-only lenses, one per surface; every finding read against the code; every
repair a host test can reach mutation-proven (53 mutants). 11 HIGH, 21 MEDIUM, 31 LOW:
[`../audit/2026-11-08-0.11.2-p-1.md`](../audit/2026-11-08-0.11.2-p-1.md). No overflow in crab's own code and the parsers
are sound; the serious findings are actions landing on something the operator did not name: a cancel that deleted an
earlier file, a Replace or a stepped move losing the only copy, prompts left armed while a transfer rewrote their rows,
a refused verb replacing the running queue, a held key moving the selection under a menu, a hang on long names. ⭐ One
frame flag and one publisher replace seven copies; a key release, a modifier edge and a no-op motion draw nothing. ⛔ Five
items are upstream: see the audit's *Upstream*. Host **390,368 B**, agnos **659,208 B** under DCE. **2543 / 0 and 176 / 0**
(`tests/hardening.tcyr`), render_test 55, the 640x220 dump `cmp`-identical. QEMU on the DCE agnos binary: ten harnesses PASS,
resize PARTIAL, door and drop's second arm UNMEASURED and the same on 0.11.1's binary; a scratch harness for the collision
prompt, a withdrawn question and the held-key gate PASS.

**0.11.1 contents — toolchain 6.7.9 → 6.7.18, every dep to its latest tag.** No code line moved; the
comments that the new pins made false were corrected, and both binaries are `cmp`-identical across
those edits. sadish 0.12.1, rupa 0.2.1, rekha 1.1.2, kashi 1.3.0, dhancha 0.11.1, setu 0.9.1, chitra
1.3.1, daimon 2.5.5 — the newest tag of each, and each that repo's cut for 6.7.18; nine commit pins
(sankoch 2.8.9 through chitra). ⛔ The 6.7.10 – 6.7.18 migration notes (~6,700 lines) were listed item
by item, and every item that could reach crab-shaped code was checked against the stripped source or
settled by the build: zero sites. Host **385,664 B**, agnos **654,600 B** under DCE (377,312 / 642,016
at 0.11.0): the toolchain −4,080 / −3,944, and **rekha 1.1.2's audit +12,432 / +16,528, the whole of the
growth**; sadish, rupa, dhancha and daimon move no byte. ⭐ **chitra 1.3.1 decodes in ~2.2x its RGBA
where 1.1.3 took ~2.9x** (re-measured; the table is in `src/app.cyr`). ⭐ **CI's lock step fails on a
`cyrius.lock keeps` note**: `--locked` alone passed a lock holding the old tags' rows beside the new,
which is what `cyrius deps --lock` writes. **2543 / 0**, render_test 55, the 640x220 dump `cmp`-identical
to 0.11.0's. QEMU on the DCE agnos binary: face (the same font line as 0.11.0's), preview, button and
pointer PASS, pointer on its first run; resize PARTIAL, its QEMU ceiling.

**0.11.0 contents — toolchain 6.6.20 → 6.7.9, the language minor, every dep to its 6.7.9 tag.** No
code line moved. sadish 0.12.0, rupa 0.2.0, rekha 1.1.0, kashi 1.2.0, dhancha 0.11.0, setu 0.9.0,
chitra 1.1.3, daimon 2.5.0 — each the newest tag whose own pin is `6.7.9`; nine commit pins (sankoch
2.8.7 joins through chitra). ⛔ **6.7 can change what code MEANS without a diagnostic**, so the
migration notes were checked against the stripped source, not only the build: zero lone `!`, zero
unary `-` on a name, zero `~`, zero `*T`, zero newly reserved names. ⭐ **Seven of the eight dep
moves changed no byte** (byte-identical builds with chitra held back); **chitra 1.1.0's `zlib`
profile is the whole drop**: host **377,312 B**, agnos **642,016 B** under DCE (521,544 / 784,200 at
0.10.5; the toolchain alone −64). ⭐ **CI's lock guard is `cyrius deps --locked`**: the old
`deps` + `--verify` went green over the stale 0.10.5 lock (it rewrites first); `--locked` refuses it.
⭐ **`path` is a dev override since 6.7.6** — a default build builds the tag (measured; the lines stay
commented). ⚠ **0.11.0 was the ladder's number for Assisted search**; M8 reads 0.12.0 now.
**2543 / 0**, render_test 55, the 640x220 dump `cmp`-identical to 0.10.5's. QEMU on the DCE agnos
binary: face, preview (PNG through the new inflate), pointer and button PASS; resize PARTIAL, its QEMU
ceiling.

**0.10.5 contents — toolchain 6.6.6 → 6.6.20, every dep to the tag its repo cut for 6.6.20.** No code
line moved. sadish 0.11.3, rupa 0.1.9, rekha 1.0.0, kashi 1.1.1, dhancha 0.10.5, setu 0.8.14,
chitra 1.0.4, daimon 2.4.4 — the newest tag of each whose own `[package].cyrius` is `6.6.20` (every
sibling has since moved to 6.7.18). Each verified on its remote; all **eight** commit-pinned in the
lock. ⛔ **Every `path` line is DORMANT** (dhancha's convention): the sibling checkouts are on their
6.7.18 lines, and the uncommitted tree this cut started from had already vendored setu 0.8.13, rupa
0.1.8 and kashi 1.2.1 through them under a manifest declaring 0.8.9 / 0.1.7 / 1.0.10.
⭐ **`[deps.daimon] modules = []`** (6.6.13's declared-not-linked spelling) clones and pins the tag,
so a phantom daimon pin FAILS `cyrius deps` now (`9.9.9` → rc=1) — it passed every gate through
0.10.4. ⛔ kashi >= 1.1.0 is a hard floor (dhancha 0.10.5's `dh_text_cp437` → `kashi_cp437_from_unicode`;
1.0.10 is refused on both targets, measured). ⭐ kashi 1.1.1 alone is −88,576 B on both targets and
ends the `large static data` advisory: host **521,544 B**, agnos **784,200 B** under DCE (608,040 /
878,376 at 0.10.4). ⚠ **The coverage floor moved 85 → 82**: cyrius 6.6.8 made the tool honest and the
unchanged tree read 82 %; `[coverage] programs` puts `render_test` in the corpus (84 %, 347/413), and
the floor sits two under it, the rule `ci.yml` states. **2543 / 0**, render_test 55, the 640x220 dump
`cmp`-identical to 0.10.4's, QEMU `crab-face-test.py` PASS on the DCE agnos binary.

**0.10.4 contents — daimon 2.1.4 → 2.4.3.** Declared, not linked: both DCE targets
**byte-identical** to 0.10.3 (`f6a223f9…` / `8102a82e…`), lock unchanged, check 4 byte-identical — so
0.10.3's QEMU face PASS covers these exact bytes. ⛔ The transport the manifest named — daimon's
per-agent AF_UNIX socket (`agent_ipc_new`) — was **removed at daimon 2.3.3**; crab reaches daimon over
its HTTP API (2.3.0: *"crab reaches it over HTTP"*). Through 0.10.4 nothing verified the daimon tag
(no `modules`, never fetched); 0.10.5 closed that.

**0.10.3 contents — names are measured by CHARACTER, and the release ships under DCE.**
⭐ `crab_char_adv(s, at, n, lenp)` — `dh_text_decode` (dhancha 0.10.4's own decoder) plus the per-arm
advance the draw uses — is the one reader both `crab_text_w` and `crab_name_cell_px` walk by. `Über.txt`
measures eight cells not nine and fits an eight-cell column unmarked; `—` is one cell; a cut lands on
a character boundary (`C3 9C 7E`, never `C3 7E`); the scalable arm prices the SCALAR (`Ü` 12 in the
fixture face, not its lead byte's 8, not the two bytes' 16). ⛔ The bound is a LENGTH, not a NUL, so
within four bytes of it the decoder is fed a NUL-terminated copy and a crossing length is clamped —
⚠ both guards are UNOBSERVABLE by any width and are held by review, recorded at the assertion.
**2543 / 0** (+40), `t_utf8_0103` in its own function; the fixture face gained a mapped `Ü` because an
all-`.notdef` face could not tell "decoded, then asked" from "asked about a byte" — the lead-byte
mutation SURVIVED until it could. Six mutations caught (21 / 7 / 12 / 1 / 2 and the 0.9.0 expiry
inverted); the render dump is byte-identical to 0.10.2's. ⭐ **`CYRIUS_DCE=1` on `release.yml` AND
`ci.yml`'s two target builds** (operator ruling): host **608,040 B**, agnos **878,376 B**; every gate
re-run against the DCE binaries. Three stale "the draw walks one byte per glyph" comments cut (the
🦀 glyph is closed at TWO levels now, not three).

**0.10.2 contents — toolchain 6.6.6, and the draw-path deps taken to their tags.** A pin-only
release; no byte of `src/` moved. cyrius **6.6.4 → 6.6.6**; sadish 0.5.5 → **0.11.2**, rekha
0.3.10 → **0.9.0**, kashi 1.0.8 → **1.0.10**, dhancha 0.10.0 → **0.10.4**, daimon 2.1.3 → **2.1.4**;
rupa, setu, chitra already at their highest tags. Every tag verified on its remote; check 4 re-run
with every `path` disabled — both targets **byte-identical** to the override build.
⭐⭐ **The production render is PIXEL-IDENTICAL across the move** — the 640x220 `render_test` dump
from the 0.10.1 tree on 6.6.4 and from this tree `cmp` clean (563,200 B). ⭐⭐ **QEMU
`crab-face-test.py` PASSES** on the new agnos binary: the face loads through rekha 0.9.0
(`adv=9 i=4 m=13`), no fault, no allocator failure. The button and pointer harnesses: every crab arm
that received a press answered correctly (middle marks, right opens the menu, middle on a popup row
fires nothing, the hover follows, F10/View/Esc/Tab answer); the red verdicts in two of three button
runs were press DELIVERY misses the 0.10.1 binary also hit — see *Proven*.
⚠ **The binary is +49 %**: host 1,097,664 → **1,636,136 B**, agnos 1,146,920 → **1,681,192 B** — a
third from the toolchain's stdlib (sankoch 2.7.15 → 2.8.0, +279 KB of Brotli), the rest from the
sadish (471 KB) and rekha (551 KB) bundles, all linked whole because 0.10.2 built without DCE.
`CYRIUS_DCE=1` returned **608,040 / 874,280 B**, `render_test` 55/0 under it — and 0.10.3 turned it
on for CI and the release, on the operator's ruling. ⚠ New advisory on every build: `large static data (143024 bytes)` — the
compiler's 128 KiB threshold, crossed by the bundles' tables (crab declares no top-level array).
⛔⛆ **The committed 0.10.1 tree did not build on this box**: `../dhancha` had moved to 0.10.4 and
`path` won — 3 undefined `sd_flatten_*` against the pinned sadish 0.5.5. The converse of the
manifest's hazard, now written into it. The 0.10.1 baseline was rebuilt from its TAGS and reproduces
the shipped artifacts byte-identical (`7ee03c10…` / `a5ba0476…`).
⚠ **Inherited from dhancha 0.10.4 — one glyph per CHARACTER, while crab still MEASURED per byte**
(`crab_text_w`, `crab_name_cell_px`): a non-ASCII name was over-measured and a cut could land inside
a UTF-8 sequence. Deliberately not bundled into the pin bump; **closed by 0.10.3 above**. **2503 / 0**,
render_test 55, all gates green.

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
fold and there must not be. (The plan then was daimon's per-agent AF_UNIX socket; daimon removed it
at 2.3.3, and crab reaches daimon over HTTP — see 0.10.4.) No `modules` key, deliberately.
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

- **Cyrius pin**: `6.7.18` (in `cyrius.cyml [package].cyrius`) — moved **2026-11-07 at the 0.11.1
  cut, on operator direction**, from `6.7.9`: the newest release, nine on.
  ⛔ Its migration notes run to ~6,700 lines, mostly struct values, generics, vectors, closures and
  multi-value returns — none of which crab's source uses (plain fns, untyped vars, enums, globals,
  `load` / `store`). Every item was listed, and each that could reach crab-shaped code was checked
  against the stripped source or settled by the build: **zero sites**. The silent ones that matter for
  this shape: a global declared twice now takes its constant from program start (6.7.14 – 6.7.16) —
  crab declares none twice, and lib/'s five pairs sit in opposite `#ifdef` arms; agnos `sys_close(-1)`
  answers -1 (6.7.11) — crab never reads `sys_close`'s answer. What 6.7.x newly refuses (an enum
  without separators, a statement headed by `load8`, a top-level `return`) the build reports as
  nothing: both targets, the suite, render_test, fuzz and bench compile with zero diagnostics.
  ⭐ **Built from the released tarball** (`1411e773…`, equal to its `.sha256`); the installed
  `~/.cyrius/versions/6.7.18/lib` is byte-identical to it. **The toolchain alone** (0.11.0's tree on its
  tags): host −4,080 B, agnos −3,944 B under DCE; plain +16 / +152 — DCE removes more, the code is not
  smaller.
  ⭐ `lib/` re-vendored from no lock: the same 40 files as 0.11.0, each stdlib leaf `cmp`-identical to the
  tarball and each bundle to its tag; lock 40 entries, **9** commit-pinned. ⚠ A moved tag's old row
  stays in an existing lock (fail-closed, deliberate); `rm cyrius.lock && cyrius deps` is the re-pin.
  `cyrius deps --lock` keeps both rows, and CI now fails on the note that says so.
  ⚠ `cyrius lint`'s content rules (6.7.11's literal write length, the `load*(&a + K)` overrun) report
  nothing in any of crab's eight files.
- *(history)* `6.7.9` — moved **2026-11-07 at the 0.11.0
  cut, on operator direction**, from `6.6.20`: onto the 6.7 language minor.
  ⛔ **The first pin move where the language changes MEANING**, not only what it refuses: 6.7.3 made
  `!` an operator (the lexer had dropped a lone `!`), 6.7.8 made `*p` through a narrow `*T` read the
  element, 6.7.9 made unary `-` / `~` on a constant bind tighter. Checked against `src/` and `tests/`
  with comments and strings stripped: **zero** sites of any shape cyrius's `ecosystem-migration.md`
  lists for 6.7.0 – 6.7.9; both targets link with zero diagnostics.
  ⭐ **Built from the released tarball** (`b517a004…`, equal to its `.sha256`); the installed
  `~/.cyrius/versions/6.7.9/lib` is byte-identical to it. **The toolchain alone** (0.10.5's tree on its
  tags): −64 B on each target.
  ⭐ `lib/` re-vendored from no lock: **32** stdlib leaves, every one `cmp`-identical to the tarball;
  10 left with sankoch (`sankoch`, `mmap`, `sync` ×3, `thread` ×5), `sankoch-zlib.cyr` arrived through
  chitra. 40 files, lock 40 entries, **9** commit-pinned.
  ⭐ What the move changed in crab's own files: `ci.yml`'s lock guard is `cyrius deps --locked` (6.7.6),
  and comments that said `path` wins over `tag` (6.7.6 builds the tag — measured).
- *(history)* `6.6.20` — moved **2026-11-07 at the 0.10.5
  cut, on operator direction**, from `6.6.6`: the close of the 6.6.x line, fourteen releases.
  crab's `src/` needed **no code edits** — 0 uses of the names 6.6.20 reserves (`sizeof`, `mulh64`,
  `fncall0`..`8`), 0 `duplicate fn` (6.6.20 binds to the LAST definition), 0 `struct` (6.6.16),
  0 `f64_le`/`_ge`/`_trunc` (6.6.13) and no vendored `math.cyr`.
  ⭐ **Built from the released tarball** (`38814a81…`, equal to its `.sha256`);
  `~/.cyrius/versions/6.6.20/lib` is byte-identical to it.
  ⭐ **The toolchain's effect, isolated** (the 0.10.4 tree on its old tags, compiled by 6.6.20): host
  608,040 → 605,360 B, agnos 878,376 → 868,048 B under DCE. The rest of 0.10.5's −86 / −94 KB is the
  deps, and 88,576 B of that is kashi 1.1.1 alone.
  ⭐ `lib/` re-vendored from no lock (`rm -rf lib cyrius.lock && cyrius deps`): **0 of 42** stdlib
  files drift from the tarball — every file `cmp`'d. `lib/poison.cyr` arrives (6.6.18's seam);
  `lib/net.cyr` leaves (setu's sidecar). Lock 49 entries, **8** commit-pinned.
  ⭐ What the move changed in crab's own files: `[deps.daimon] modules = []` (6.6.13), `[coverage]
  programs` and the coverage floor 85 → 82 (6.6.8 / 6.6.17), and the comments that said `cyrius test`
  ignores `[build].test` (6.6.17 runs it — measured).
- *(history)* `6.6.6` — moved **2026-09-21 at the 0.10.2
  cut, on operator direction**, from `6.6.4`. Two repair releases (6.6.5: call ABI, aggregate
  layout, `cyrlint --strict-deferrals`; 6.6.6: the Windows `O_APPEND`/`O_TRUNC` corruption, CVE-44
  and CVE-45, nine new refusals — a redeclared global, a top-level block `var` read after its block,
  a function-like `#define` in a comment). crab's `src/` needed **no edits**: it has no `#define`, no
  `struct`, no PE target, no duplicated global; the build was the gate and it is green.
  ⭐ **The bump's effect was MEASURED IN ISOLATION** (the 0.10.1 tree, its old dep tags, compiled
  by 6.6.6): host 1,097,664 → **1,277,920 B**, agnos 1,146,920 → **1,327,080 B** — +180 KB each,
  and none of it codegen: the **stdlib snapshot** moved `lib/sankoch.cyr` 2.7.15 → **2.8.0**
  (665,879 → 945,063 B, Brotli decode for rekha's WOFF2 — a leaf crab reaches only through chitra's
  PNG inflate), doubled `lib/bench.cyr`, and grew every `syscalls_*` peer. That is also what crosses
  the compiler's 128 KiB **`large static data`** advisory (138,848 B with the old deps, 143,024 with
  the new): crab declares no top-level array; the tables are the bundles'.
  ⭐ `lib/` re-vendored from scratch (`rm -rf lib && cyrius deps`, CI's shape): **0 of 42** stdlib
  files drift from `~/.cyrius/versions/6.6.6/lib` — every file `cmp`'d. `lib/alloc_cx.cyr` arrives
  (new in 6.6.6); `lib/flags.cyr` and `lib/hashmap_fast.cyr` leave (nothing names them; no target
  misses a symbol). Lock 50 → 49 entries.
- *(history)* `6.6.4` — moved **2026-09-14 at the 0.8.11 cut, by the operator**, from `6.6.2`, with
  the whole stack (sadish 0.5.5, rekha 0.3.10, dhancha 0.9.29 → 0.10.0). ⛔ **It was never written
  into this section** — for seven releases (0.8.11 → 0.10.1) this bullet asserted `6.6.2` against a
  manifest that read `6.6.4`. The same rot the header above records twice already; the CHANGELOG's
  0.8.11 entry is where that bump lives.
- *(history)* `6.6.2` — moved **2026-09-11 at the 0.8.4
  cut, by the operator**, from `6.6.1`, together with the whole sibling stack (every dep, and
  aethersafha's stack behind it, then sat on 6.6.2). 6.6.2 is the repair release for 6.6.0's
  `: stack` value-form change; crab's `src/` needed no edits. ⚠ The bump re-vendored `boxed`,
  `hashmap`, `result` and `tagged` and **left `lib/sankoch.cyr` at 6.6.1's 2.7.14** — the
  transitive leaf the sync never walks — corrected 2026-09-12 by `cyrius deps`, after which the
  whole vendored tree `cmp`'d byte-identical to `~/.cyrius/versions/6.6.2/lib`.
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
  **6.5.36** (0.7.1) → **6.5.41** (0.7.7) → **6.6.0** (0.8.0) → **6.6.1** (0.8.2) → **6.6.2** (0.8.4)
  → **6.6.4** (0.8.11) → **6.6.6** (0.10.2) → **6.6.20** (0.10.5) → **6.7.9** (0.11.0) → **6.7.18** (0.11.1).
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

**14,311 lines** across **five** files, plus **8,557** in `tests/` *(0.10.3, `wc -l`; 14,257 / 8,406 at
0.10.1–0.10.2, 8,092 / 4,486 at 0.7.7, 7,915 at 0.7.6, 5,368 at 0.7.5, 2,227 at the 0.7.0 cut)*.
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
  `build/crab-render.bin`. **55** `check()` assertions *(0.10.1; 53 at 0.7.6, 26 at 0.7.5)*.
  ⭐ **IT NOW PRINTS ITS OWN CHECK COUNT.** It returned `g_fails` and printed only its dump line, so
  it exited **0** whether it ran 26 checks or none — while this file and the handoff quoted a count
  for three cuts that nothing in the program printed. `crab render_test: 35 checks, 0 failed`.
  ⚠ **It renders at 640x220, not 380** — every assertion in it is about two panes side by side, and
  below 600 crab now draws one. The solo layout has its own checks at 380.
  ⚠ It creates **two** `DhSurface`s on purpose — it holds two frames live and dumps the first, and
  since dhancha 0.9.14 one surface would hand both renders the same target.
  ⛔ **`cyrius test` does not run it** (it is neither a `tests/*.tcyr` nor the `[build].test`
  entry); CI's own `render_test` step builds it, fails on any `undefined function` in the build
  log, and runs it. Since 0.10.5 it is also in `cyrius coverage`'s corpus (`[coverage] programs`).

⭐ `cyrius coverage` reports **370/440 fns referenced (84 %)** at 0.11.2 (347/413 at 0.10.5 – 0.11.1), against a CI floor of 82 —
the first reading by cyrius 6.6.8's honest tool, which counts `public fn` and reads whole files (the
same tree read 82 % without `render_test` in the corpus, against the 85 floor set when the old tool
read 87 %). ⚠ No test was written to move it; `cyrius coverage` lists the 66 unreferenced fns.
⭐ It reported **137/159 fns referenced (86 %)** at 0.7.5, 6/6 files — **the v1.0
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

### ⭐⭐ As of 2026-11-08 — the 0.11.3 DCE binary (663,336 B, `b764d50f…`; cyrius 6.7.18, dhancha 0.12.0, the other deps as 0.11.2) under QEMU, agnos 1.57.17, the same `aethersafha_agnos` as 0.11.2's run

- All 13 harnesses, with the same verdict on each as 0.11.2: **PASS** button, columns, face, go, h1, listing-cap, pointer,
  preview, shift, symlink. **PARTIAL** resize, its QEMU ceiling. **UNMEASURED** door and drop's second arm.
- ⭐ The scratch harness for the collision prompt, a withdrawn question and the held-key gate, **PASS** on this binary: it
  drives the event path dhancha 0.12.0 changed.
- ⚠ **Not on the target**: a real grow. QEMU cannot back a larger window, so `resize` proves the refusal and dhancha's
  target reuse through a grow is proven on the host.

### ⭐⭐ As of 2026-11-08 — the 0.11.2 DCE binary (659,208 B, `dbfc15f9…`; cyrius 6.7.18, same deps as 0.11.1) under QEMU, agnos 1.57.17, aethersafha 0.17.7

- All 13 harnesses. **PASS**: button, columns, face (`410820 bytes adv=9 upem=2048 i=4 m=13`), go, h1, listing-cap, pointer,
  preview, shift, symlink. **PARTIAL**: resize, its QEMU ceiling. **UNMEASURED**: door and drop's second arm, and
  identically on 0.11.1's binary on the same kernel, so they are limits of those harnesses, not regressions.
- ⭐ A scratch harness (session scratchpad) for what the 13 do not reach, **PASS**: a merge asks once on its collision; `d`,
  `n`, Enter and Space under the question do nothing; `s` resumes and `debugfs` reads the operator's file intact; a delete
  prompt left open as a transfer ends is withdrawn and `y` deletes nothing; a held key repeats 0 times under the menu bar
  and 3 with nothing open.
- ⚠ **Not on the target**: H4's FAT and exFAT close path (no harness has a full FAT volume), M13's edge release, M12's refusal.

### ⭐⭐ As of 2026-11-07 — the 0.11.1 DCE binary (654,600 B, `5720097b…`; cyrius 6.7.18, sadish 0.12.1, rekha 1.1.2, kashi 1.3.0, dhancha 0.11.1, setu 0.9.1, chitra 1.3.1 + sankoch-zlib 2.8.9) under QEMU, agnos 1.57.17, aethersafha 0.17.7

- `crab-face-test.py` **PASS**: `410820 bytes adv=9 upem=2048 i=4 m=13` — the same line, character
  for character, as 0.11.0's, so rekha 1.1.2's audit measures the shipped face as 1.1.0 did; one
  `font` line, navigations 1, view changes 2, no fault, no allocator-failure text.
- `crab-preview-test.py` **PASS** — chitra 1.3.1's in-place unfilter on the target: the idle tick read
  `zzpic1.png` → 137x42 and `zzpic2.png` → 320x200, each file exactly once; every arm line identical
  to 0.11.0's; no fault.
- `crab-button-test.py` **PASS**, one run: middle marks a pane, middle on a popup row fires nothing,
  the hover follows across 5 rows and does not arm before the pointer moves.
- `crab-pointer-test.py` **PASS on the first run**: right-click menu, pick ran `cd /bin`, dismiss,
  6 refreshes, bare F10 / Esc / Tab crab's, Ctrl chords the compositor's, no faults.
- `crab-resize-test.py` **PARTIAL — the harness's QEMU ceiling**: the extent was unbackable (no GPU
  carveout), crab refused it and answered 16 keys after; wheel 8 scrolls from 8 events, FULL_KEYS
  12 / 6, held-key repeat 2 while held and 0 after release, all four sort modes, the stat drain
  completed, no fault. (0.11.0's run: wheel 7 of 8, repeat 4 — TCG timing, the same verdicts.)
- ⚠ **Not on target for this binary**: the drop, H1, symlink, door, go, columns, shift and
  listing-cap harnesses. No crab code moved and the host pixel diff is byte-identical. Of the dep moves
  that changed bytes, rekha (face) and chitra + sankoch (preview) ran on the target; kashi's clamp sits in
  the bitmap fallback, unused once the face loads, and setu's two lines in refusals the band never
  takes.

### ⭐⭐ As of 2026-11-07 — the 0.11.0 DCE binary (642,016 B, `899f6bd9…`; cyrius 6.7.9, sadish 0.12.0, rekha 1.1.0, kashi 1.2.0, dhancha 0.11.0, setu 0.9.0, chitra 1.1.3 + sankoch-zlib 2.8.7) under QEMU, agnos 1.57.17, aethersafha 0.17.7

- `crab-face-test.py` **PASS**: `410820 bytes adv=9 upem=2048 i=4 m=13`, one `font` line,
  navigations 1, view changes 2, no fault, no allocator-failure text.
- `crab-preview-test.py` **PASS** — the one dep that moved bytes, on the target: the idle tick read
  `zzpic1.png` → 137x42 and `zzpic2.png` → 320x200 through chitra 1.1.3 and sankoch's `zlib` profile,
  each file exactly once; no fault.
- `crab-button-test.py` **PASS**, one run, every arm: mask 4 marks a pane, middle on a popup row
  fires nothing, the hover follows across 5 rows and does not arm before the pointer moves.
- `crab-pointer-test.py` **PASS on run 2**. ⚠ Run 1 was INCONCLUSIVE from the console, not crab: its
  serial shows the Backspace delivered and `listed 9 entries in /`, with aethersafha's frame-cost line
  spliced into the middle of `crab: cd /`, so the harness's substring oracle missed and it skipped the
  pick / dismiss arms. Run 2: ascend, right-click menu (wire 2), pick ran `cd /bin`, dismiss, `u` ×6 →
  6 refreshes, bare F10 / Esc / Tab crab's, Ctrl+Tab / Ctrl+F10 / Ctrl+Q the compositor's, no faults.
- `crab-resize-test.py` **PARTIAL — the harness's QEMU ceiling**: the extent was unbackable (no `#86`
  carveout), crab refused it and answered 16 keys after; wheel 7 scrolls from 8 events, FULL_KEYS
  12 / 6, held-key repeat 4 while held and 0 after release, all four sort modes, the stat drain
  completed, no fault.
- ⚠ **Not on target for this binary**: the drop, H1, symlink, door, go, columns, shift and
  listing-cap harnesses. No crab code moved, the host pixel diff is byte-identical, and seven of the
  eight dep moves are byte-identical in the binary; the toolchain and chitra's inflate moved, and the
  five harnesses run cover those.

### ⭐⭐ As of 2026-11-07 — the 0.10.5 DCE binary (784,200 B; cyrius 6.6.20, sadish 0.11.3, rekha 1.0.0, kashi 1.1.1, dhancha 0.10.5, setu 0.8.14) under QEMU, agnos 1.57.17, aethersafha 0.17.7

- `crab-face-test.py` **PASS**: `crab: font /fonts/default.ttf 410820 bytes adv=9 upem=2048 i=4
  m=13` — the face loads through rekha 1.0.0 and measures proportionally; one `font` line;
  navigations 1, view changes 2; **no fault, no allocator-failure text**.
- `crab-pointer-test.py` **PASS**, one run: ascend, left click to a pane, right-click opens the
  context menu (wire 2), a left pick ran `cd /bin`, a press off the popup dismissed it, `u` ×6 → 6
  refreshes, `g` / `b`, bare F10 opens the bar and View drives from the keyboard, bare Esc / Tab ×3
  reach crab, Ctrl+Tab / Ctrl+F10 / Ctrl+Q go to the compositor, no faults.
- `crab-button-test.py` **PASS**, one run, and every arm landed this time: mask 4 → `mark by middle
  click` on a pane, mask 1 → click, mask 2 → context menu; middle on a popup row fires nothing; the
  hover follows the pointer across 5 rows and does not arm before the pointer moves; no faults.
- `crab-resize-test.py` **PARTIAL — the harness's own ceiling under QEMU**, not a crab defect: the
  extent was unbackable (2 MB `#71` slot, no `#86` carveout), crab refused it and stayed at its
  size, then answered 16 keys; the wheel resolved 7 scrolls from 8 events, a click resolved to a
  pane, FULL_KEYS releases delivered and gated (12 / 6), held-key repeat fired 3 times while held
  and 0 after release, the sort key cycled all four modes, the deferred stat drain completed, no
  fault. The ADOPT path needs a machine with a carveout, as before.
- ⚠ **Not on target for this binary**: the drop, H1, symlink, preview, door, go, columns, shift and
  listing-cap harnesses. No crab code moved and the host pixel diff is byte-identical; the
  compiler, setu's transport and kashi's glyph data moved underneath, and the four harnesses run
  are the ones that cover those.

### ⭐⭐ As of 2026-09-21 — the 0.10.2 binary (cyrius 6.6.6, sadish 0.11.2, rekha 0.9.0, dhancha 0.10.4) under QEMU, agnos 1.57.5, aethersafha 0.16.25

- `crab-face-test.py` **PASS**, twice: on the 0.10.2 plain binary (1,681,192 B) and on the **0.10.3
  DCE binary (878,376 B) — the artifact shape the release publishes from 0.10.3 on**:
  `crab: font /fonts/default.ttf 410820 bytes adv=9 upem=2048 i=4 m=13` — the face loads through
  rekha 0.9.0 and measures proportionally; one `font` line; navigations 1, view changes 2; **no
  fault, no allocator-failure text**. The scalable draw path and the frame arena, on the target,
  with every draw-path dep moved and the unreachable half of the binary NOPed.
- `crab-button-test.py`, three runs: **run 3 landed every button** — middle → `crab: mark by middle
  click` (wire 3), left → `crab: click`, right → `crab: context menu opened by pointer`, ARM 2
  (middle on a pane) **marks**, ARM 3 (middle on a popup row) **fires nothing**; run 1 measured the
  hover following the pointer across **6 rows**; run 2 saw the popup-row refusal as
  `crab: press btn 3 no action`. ⚠ Runs 1 and 2 FAILED ARM 2 and the verdict said so — each a press
  the compositor never forwarded (`NO client content under the cursor`, one-shot, later misses
  silent). **Discriminated, not assumed**: the 0.10.1 binary on the same kernel missed all four
  per-mask probes and landed ARM 2 once. The aiming under TCG is a coin; the harness header says so.
- `crab-pointer-test.py`, one run: ascend, left click, `u` ×6 → 3 refreshes, `g`, `b`, **bare F10
  opens the bar and View drives from the keyboard**, bare Esc/Tab reach crab, Ctrl chords go to the
  compositor, **no faults**. Its right-click arm was the same delivery miss (three tries) —
  UNMEASURED there, measured in button run 3.
- ⚠ **Not on target for this binary**: the drop, H1, symlink, preview, door, go, columns, shift and
  resize harnesses — none of their code moved, and the pixel diff on the host is byte-identical, but
  a compiler moved under the event loop and only the face, button and pointer harnesses were run.

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

Declared in `cyrius.cyml`. ⭐ **Re-derived 2026-11-07 for 0.11.1: every tag is its repo's newest**, and
each is that repo's cut for cyrius 6.7.18, the pin — so nothing is pinned ahead of the toolchain.
dhancha 0.11.1 names the same sadish, rupa, rekha, kashi and setu. Each tag verified on its remote
(`git ls-remote --tags` over https) and **commit-pinned in `cyrius.lock` — nine**: the eight declared,
plus sankoch 2.8.9 (`21470ae`), which chitra brings as its own `[deps.sankoch]`
(`dist/sankoch-zlib.cyr`). ⛔ crab must not declare a `sankoch` stdlib leaf: its own would win and link
the full codec set back.
⭐ **Each dep held back in turn, against the final tree** (0.11.1): sadish, rupa, dhancha and daimon
build byte-identical; setu moves 2 bytes of the agnos binary (two refusal lines that now write their
newline); kashi, chitra and sankoch move bytes but not size; rekha 1.1.2 is all of the growth.
⭐ **GIT FIRST SINCE cyrius 6.7.6 — `path` IS A DEV OVERRIDE, NOT A WINNER.** A default build builds the
tag and checks its pin; a `path` is read only under `CYRIUS_LOCAL=1` / `--local`. Measured on crab's
manifest with four `path` lines live: the tags built, `hint: 4 deps have local checkouts not in use`,
and local mode wrote `build/local-deps/` and left `lib/` and the lock alone. The `path` lines stay
**commented in a commit** all the same (dhancha's convention; 0.10.5 made them dormant when `path`
still won). ⭐ CI's guard is `cyrius deps --locked` since 0.11.0: it refuses a lock or `lib/` that
differs from what the tags resolve — the old `deps` + `--verify` went green over a stale lock.
⭐ **And since 0.11.1 it fails on a `cyrius.lock keeps` note.** A moved tag's old row stays in an
existing lock (fail-closed, deliberate), and `--locked` passed such a lock: rc=0, "exactly what the
tags resolve", eight notes (measured on the lock `cyrius deps --lock` wrote at this cut). The re-pin is
`rm cyrius.lock && cyrius deps`.
⭐ **THE DAIMON ROW IS VERIFIED BY THE TOOLCHAIN NOW.** `modules = []` (6.6.13) clones and pins the tag:
`tag = "9.9.9"` fails `cyrius deps` with rc=1 (`Remote branch 9.9.9 not found`). Through 0.10.4 the
same phantom gave "7 deps resolved" and passed every gate.
⚠ dhancha's `0.10.4` was an ANNOTATED tag (`cyrius deps` printed `… is not a commit!` and pinned the
peeled commit); `0.10.5` is a plain commit tag.
⛔⛆ *(history, through cyrius 6.7.5)* **THE CONVERSE HAZARD, OBSERVED 2026-09-21: a sibling that moves
AHEAD of the tags broke the local build while the committed tree was fine** — `../dhancha` at 0.10.4
against the pinned sadish 0.5.5, 3 undefined functions in a tree with no diff. From 6.7.6 a live `path`
is ignored unless local mode asks for it, so it can only bite a `CYRIUS_LOCAL=1` build.
⛔ **0.8.4 SHIPPED WITH THE DIVERGENCE THIS CHECK EXISTS FOR.** Its `lib/` carried dhancha 0.9.29,
rupa 0.1.7 and setu 0.8.9 through `path` while the manifest declared 0.9.28 / 0.1.6 / 0.8.8 — so CI
compiled the old tags and the local build the new ones, green both ways, and nothing said so. All
seven are re-pinned in `[Unreleased]`; only chitra 1.0.1 → 1.0.3 carries behaviour (a decoder P-1
sweep), the rest are 6.6.2 pin moves with header-only dist changes. That is the strongest form check 4 can take: it
says not merely *"the declared graph resolves"* but *"the declared graph is what the local build has
been compiling all along."* ⚠ It holds only because every sibling working tree sits exactly on its
tag, clean; re-derive it rather than assuming it, since `path` is what makes it possible to drift.
⚠ **The older "6 deps" count in this section was wrong** — chitra made seven on 2026-08-31.

| dep     | tag    | `path`?  | why crab needs it                                   |
|---------|--------|----------|-----------------------------------------------------|
| sadish  | 0.12.1 | dormant  | 2D vector — the surface everything else draws into; the flatten budget (0.7.x) dhancha needs; 0.11.1's audit. ⛔ moves WITH rekha (rekha 1.1.2 names 0.12.1) |
| rupa    | 0.2.1  | dormant  | shared theme tokens + **`on-accent`** and contrast   |
| rekha   | 1.1.2  | dormant  | **the face** — `crab_face` opens `/fonts/default.ttf` through it; advances size every column. ⭐ the 1.x freeze of 0.9.0's surface (1.0.0); ⚠ 1.1.2 is a full audit (three out-of-bounds reads; faces it now refuses), +12 / +16 KB |
| kashi   | 1.3.0  | dormant  | CP437 8×16 glyph data for `dh_draw_text` (font=0), the fallback face. ⛔ **>= 1.1.0, hard** (`kashi_cp437_from_unicode`); 1.1.1's byte-sized buffers are −88,576 B |
| dhancha | 0.12.0 | dormant  | widgets, `dh_list_new_h` (menu bar), `dh_theme_*`, the frame arena; one glyph per CHARACTER; ⭐ 0.10.5's CP437 map is kashi's; ⛔ 0.12.0: a polled event lasts until the next poll, and text draws at `DH_TEXT_EM` |
| chitra  | 1.3.1  | dormant  | **thumbnails** — PNG/JPEG/GIF/BMP decode. ⭐ 1.1.0+ brings sankoch's `zlib` profile, not all of sankoch (−144 KB); 1.3.1 decodes in ~2.2x its RGBA (1.1.3: ~2.9x). ⛔ see gaps |
| setu    | 0.9.1  | dormant  | client transport — channel-band, reads `AGNOS_CHAN`; 0.8.11 reads a dead compositor as -6 on Linux |
| daimon  | 2.5.5  | dormant  | **declared, NOT linked** — `modules = []`; reached over its HTTP API, loopback-only on agnos too since 2.4.4 (agnos >= 1.57.7). ✅ builds and runs on agnos. ⭐ `/v1/files` since 2.5.4: index, tags, ratings, smart folders, duplicates; ranked search by WORDS since 2.5.5 — opt-in (`--index-root`), no caller auth until 2.6.x. The M7/M8 filing is 🟡, open on meaning. ⭐ tag verified by `cyrius deps` |

⚠ **This table sat at the 0.8-era tags (sadish `0.5.4`, rekha `0.3.7`, dhancha `0.9.29`, seven rows)
from 2026-09-12 to 2026-09-21** — through the 0.8.11 stack bump, 0.9.0's face and 0.10.0's daimon
row. The table is refreshed at every cut or it is fiction; this is the third time this file has had
to say so about itself.

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

⭐ **Check 4 is the default build since cyrius 6.7.6** (a default build builds the tags) and CI's
`cyrius deps --locked` since 0.11.0. Through 6.7.5 `path` won over `tag` — the drift 0.4.13 caught —
and the four checks above were how a cut proved the declared graph. At every cut, still verify each
tag on its remote (`git ls-remote --tags … | sort -V`) and against the commit the lock pins.

⚠ **crab consumes `dist/` bundles, not `src/`.** Six deps — sadish, rupa, rekha, dhancha, setu,
chitra — are `modules = ["dist/<name>.cyr"]`, so a fix reaches crab only after `cyrius distlib` runs
**in that sibling**; a local `path` override alone is not enough. **kashi is the exception**: it is
`modules = ["src/font_data.cyr"]`, the freestanding core, deliberately — the library face costs
+183,360 B (+50 %) for a runtime font registry crab never calls, and `CYRIUS_DCE=1` reclaims none of
it. **daimon is the other**: `modules = []`, cloned and pinned, nothing folded. ⚠ The sadish and rekha
bundles are 472 KB and 552 KB of source at 0.12.0 / 1.1.0 (86 KB and 41 KB at 0.5.5 / 0.3.10); CI and the release link them under
`CYRIUS_DCE=1` since 0.10.3 — see *Targets*.

✅ **The `net` stdlib declaration is GONE (0.10.0), and since 0.10.5 so is `lib/net.cyr`**: setu
0.8.14's compile-verified sidecar no longer names it, and crab never called it.

## Tests

- ⭐ `tests/hardening.tcyr` (0.11.2) — **176 assertions**: the P(-1) findings, reproduced against real files with
  `crab_op_step` driven as the idle tick drives it, each one seen to fail with its repair removed.
- `tests/crab.tcyr` — the main suite `cyrius test` runs (discovered, and named by `[build].test =
  "tests"`, which cyrius has run since 6.6.17 — once). **2,543 passed / 0 failed**
  *(0.10.3 through 0.11.2 — on cyrius 6.7.18 at 0.11.1, 6.7.9 at 0.11.0; 2,503 at 0.10.1 and 0.10.2, 2,490 at 0.10.0, 2,451 at 0.9.7, 2,440 at 0.9.6, 2,421 at 0.9.5, 2,384 at
  0.9.4, 2,345 at 0.9.3, 2,012 at 0.8.7, 1,838 at 0.8.6, 1,790 at 0.8.5, 1,695 at 0.8.3/0.8.4, 1,230
  at 0.7.7, 253 at 0.7.0)*. ⚠ This line said 2,012 from 0.8.7 to 0.10.1 while the *Version* section
  above carried the live count — two numbers in one file, one of them stale. `render_test` is **55**
  checks (0.10.1; 53 from 0.7.6 to 0.10.0).
  ⭐ **0.11.1's was a pixel diff too**: the same 563,200 B dump on 6.7.18 with the latest tags, `cmp`
  clean against 0.11.0's — rekha 1.1.2's audit and kashi 1.3.0's clamp moved no pixel of it.
  ⭐ **0.11.0's was a pixel diff too**: the same 563,200 B dump on 6.7.9 with the 6.7.9 tags, `cmp`
  clean against 0.10.5's and 0.10.4's.
  ⭐ **0.10.5's was a pixel diff too**: the 640x220 dump from the 0.10.4 tree (6.6.6, its tags) and
  from 0.10.5 (6.6.20, the new tags) — 563,200 B, `cmp` clean.
  ⭐ **0.10.2's test was a PIXEL DIFF, not a count**: the 640x220 BGRA dump `render_test` writes
  `cmp`s byte-identical between the 0.10.1 tree (6.6.4, old pins from their tags) and the bumped
  tree — 563,200 bytes, no difference. A toolchain and three draw-path deps moved and the
  bitmap-font frame did not.
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
| x86_64 linux | ✅ builds, **390,400 B under `CYRIUS_DCE=1`** *(0.11.3; plain 1,127,680. 390,368 at 0.11.2; plain 1,127,648. 385,664 at 0.11.1, cyrius 6.7.18 — CI and the release build with DCE since 0.10.3; plain 1,118,848. 377,312 at 0.11.0: the toolchain alone −4,080, rekha 1.1.2's audit +12,432, every other dep move 0 in size — sadish, rupa, dhancha and daimon byte-identical)*. ⚠ Compare with `cmp`, never `ls -l` — 0.10.3's DCE binary was the same size as 0.10.2's DCE probe with 94,327 bytes differing |
| `--agnos`    | ✅ builds, **663,336 B under `CYRIUS_DCE=1`** *(0.11.3; plain 1,183,528. 659,208 at 0.11.2; plain 1,183,496. 654,600 at 0.11.1; plain 1,178,888. 642,016 at 0.11.0: toolchain −3,944, rekha 1.1.2 +16,528, the other deps 0 in size)* — ⭐ **and it draws in Liberation Sans and decodes PNG previews on a real kernel with the 6.7.18 stack** (QEMU, agnos 1.57.17 + aethersafha 0.17.7: face, preview, pointer and button PASS; resize PARTIAL at its QEMU ceiling) — the real target, **CI builds it**, DCE'd, the artifact the release publishes. No `large static data` advisory since 0.10.5 |
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
- ✅ **CLOSED at 0.10.0 — daimon is DECLARED** (`[deps.daimon]`, 2.5.5 as of 0.11.1;
  `modules = []`), and ✅ it builds and runs on agnos (re-derived at 2.5.5). ⭐ Since 2.5.4 it has the
  file surface the filing (2026-09-23) asked for, ranked by words, not meaning, since 2.5.5. ⛔ What is
  open instead is everything around it: no meaning, nothing on agnos starts it with `--index-root`, no
  caller auth until 2.6.x, and no client in crab — see *Version*.
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
- ✅ **DECIDED at 0.9.3 — [ADR 0004](../adr/0004-symlinks-are-shown-preserved-and-dereferenced-on-copy.md)**:
  a link is SHOWN (`@`, KIND *Link*), delete and move PRESERVE it, copy DEREFERENCES it (`cp -r`).
  This line asked *"refuse? report? recreate?"* for six releases after the answer shipped.
  ⚠ Still true: **crab cannot recreate a symlink** — `symlink`#63 / `readlink`#70 are ext2-only and
  crab copies between volumes, so what a link *becomes* on a filesystem that cannot hold one is its
  own decision, deferred with its reason.
- ✅ **CLOSED in 0.7.6 — `README.md` § Status.** It said *"Shipping, and read-only"*, which M4
  falsified, and quoted the retired 256-entry cap. Now states the write layer, the preview column,
  and what is genuinely absent. ⚠ **It has been wrong in both directions now**; the replacement text
  carries that warning itself.
- ✅ **CLOSED at 0.10.3 — crab measures text by CHARACTER, as dhancha 0.10.4 draws it.** Opened at
  0.10.2 (the pin bump that brought 0.10.4 in), closed one release later by `crab_char_adv`, the one
  reader both measuring loops share. ⚠ What remains true: a name with a character NEITHER face
  carries draws `?` under kashi and `.notdef` under the face — one cell either way, measured as one.
  ⚠⚠ **Two bound guards in `crab_char_adv` are held by review** — the NUL-terminated tail copy and
  the crossing-length clamp protect a READ, and no width can witness a read. Recorded at the test.
- ✅ **CLOSED at 0.11.2 — `crab_fs_open_w` is exclusive on both targets.** The agnos arm is `AO_WRONLY|AO_CREAT|AO_TRUNC|AO_EXCL`. ⚠ This note said agnos had no `AO_EXCL` from 0.7.6 until the audit; it has had one since 1.56.56. The cost of the error was real: FAT and exFAT answer `stat` with -1, so a copy onto an existing file there replaced it without asking (P(-1) H5).
- ⛔⛔ **EVERY THUMBNAIL DECODE IS PERMANENT, AND THAT — NOT THE BINARY SIZE — IS THE LIVE
  CONSTRAINT.** chitra has 30 `alloc()` call sites (1.3.1), `chitra_image_free` is a **no-op**, and
  cyrius's `alloc` has no `free()`. Measured at 0.11.1: **~2.2x the RGBA size per decode, never
  reclaimed** (1.1.3: ~2.9x; the table is in `src/app.cyr`), and a re-decode of the same file costs it
  again. crab bounds it two ways — a per-image pre-check (`CRAB_THUMB_MAX_RGBA`, 4 MB; a refusal costs
  **16 bytes** against 36 MB for a 2048x2048 photo unbudgeted) and a
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
- ✅ **CLOSED at 0.9.4 — the preview's dimension read is on the idle tick** (`crab_pv_step`, one
  file per tick, 128 path-keyed slots); the selection path is a lookup, **4 µs per keystroke with an
  empty cache**. This line described the 0.7.6 shape for five releases after 0.9.4 moved it.
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

- ⭐ **`path` no longer wins over `tag`** (cyrius 6.7.6): a default build builds the tag, and a `path`
  counts only under `CYRIUS_LOCAL=1`. A local-mode build is therefore not evidence the declared graph
  resolves — a default one, or `cyrius deps --locked`, is. Through 6.7.5 `path` won, which is why
  the lines were made dormant at 0.10.5; they stay commented.
  ⛔ **And it cuts the other way (2026-09-21)**: a sibling that moves ahead of its tag breaks the
  local build of an UNCHANGED tree — `../dhancha` at 0.10.4 against the pinned sadish 0.5.5 was 3
  undefined functions. When a green tree turns red with no diff, look at the siblings before the
  source.
- ⛔ **In local mode `lib/` is not what compiles** — `build/local-deps/lib/`, from the sibling's
  `dist/`, is. A cross-repo change must be followed by `cyrius distlib` in that repo (dhancha's
  `scripts/sync-deps-sidecar.sh` is gone since its 0.10.5, with its sidecar).
- ⛔ **Build both targets, every time.** See the `--agnos` gap above.
- ⛔ **`src/ui.cyr` is below `src/app.cyr`.** The render path must never call up; `render_test`
  includes `ui.cyr` alone. This was violated three times during M4.

## Consumers

_None — top-level application._

## Next

⭐ **As of 0.11.3 (2026-11-08), in the order the roadmap's ladder gives them:**
0. ✅ **`0.11.3 · dhancha 0.12.0`** — cut; the one upstream item left from the audit is agnos's 62-byte name cut.
   ✅ `0.11.2 · The P(-1)` before it; the audit is in `docs/audit/`.
1. ✅ **`0.11.1 · The 6.7.18 stack`** — cut; every dep on its latest tag, rekha 1.1.2's audit, chitra's
   cheaper decode, CI failing a lock that keeps a superseded pin. ✅ `0.11.0 · The 6.7.9 stack` before it.
2. **M7's index, tags and smart folders** — ✅ daimon's side exists (2.5.4's `/v1/files`). ⛔ Not yet
   buildable end to end: nothing in agnos starts daimon with `--index-root`, the API has no caller auth
   until 2.6.x, and crab has no HTTP client. ⚠ The release number is open: M7's ladder row is 0.10.0
   (half shipped), and 0.12.0 is pinned to Assisted search — an operator call.
3. **`0.12.0 · Assisted search`** — daimon 2.5.5 answers its shape by WORDS; the canvas promises
   meaning (*"vector + name match"*), so the filing stays open on that. Building it by words, labelled as
   such, or waiting is an operator call.
4. ⚠ Filed against crab by agnos 1.57.11 and not yet taken:
   `issues/2026-10-09-mountlist-backend-4-zfs-shows-unknown.md` — a ZFS volume (`mountlist`#104
   backend 4) is labelled "unknown" in VOLUMES. Cosmetic; not part of a pin release.

Everything below this line is the record of how the earlier queue was carried, kept because its
reasoning still governs.

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
- ✅ **The `crab_fs_open_w` target divergence — closed at 0.11.2**; see *Known gaps*.
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

Stated as fact, not as a priority. ⚠ The on-target runs recorded before this release are dated
**2026-09-14** (the H1, symlink and face harnesses — the handoff says which were iron and which
QEMU); the newest is **2026-11-08, the 0.11.2 DCE binary under QEMU** (agnos 1.57.17, aethersafha
0.17.7) — see *Proven*. Everything since 0.9.7 has run on the host and under QEMU only.
Every `#ifdef CYRIUS_TARGET_AGNOS` region is invisible to the host suite by construction.

⛔ **Before believing any gate, re-derive it.** Three false gates on record: M4's *"Gate: agnos write
syscalls"* (real since agnos 1.41.3), drag's *"gated on nothing"* (it was gated, elsewhere), and
thumbnails' *"no image decoder"* — where chitra existed, and the real obstacle turned out to be a
**cost** no gate line mentioned. ⇒ A gate can be false in both directions.

Two decisions are settled and shape everything downstream:
[ADR 0001](../adr/0001-compositor-owns-theming.md) (the compositor owns theming; crab ships no
palette) and [ADR 0002](../adr/0002-semantic-find-is-a-mode.md) (semantic find is a mode over any
view, not a view of its own).
