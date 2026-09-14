# Handoff — **0.9.3: crab sees a symlink; a data-loss defect FOUND AND FIXED, and H1 still open.**

> ⛔⛔⛔ **READ THIS FIRST — H1, CONFIRMED, NOT FIXED.** Cancelling a copy that **merged** into an
> existing folder deletes that folder wholesale, **including files crab never wrote**. Reproduced by
> the operator (`tests/zz_h1_repro.tcyr`, since removed):
> `POST-CANCEL precious.txt exists = 0 *** DELETED BY CRAB ***`.
> ⇒ **The source predicted it exactly.** `crab_walk_reroot_dtree` says: *"WHY DELETING HERE IS SAFE…
> rests on one invariant in `crab_walk_begin`: that function REFUSES with `EEXIST` if the destination
> already exists… If that guard is ever relaxed to allow merging into an existing destination, THIS
> FUNCTION BECOMES A DATA-LOSS BUG and must be deleted in the same change."* **0.8.7 relaxed exactly
> that guard** (`crab_walk_begin`: *"THE ROOT MERGES LIKE ANY OTHER DIRECTORY (0.8.7)"*) and did not
> delete the function.
> ⇒ **The fix, named**: a record flag set in `crab_walk_begin` meaning *crab created this root*;
> `crab_walk_reroot_dtree` reroots only when it is set. A cancelled MERGE then unlinks the in-flight
> file and leaves the partial merge — the honest outcome, against deleting data the operator did not
> name, which is the failure this project already paid for once (`/bin`, 2026-09-03).
> ⚠ **Not fixed in 0.9.3 because the operator was actively probing the same file** (`h3probe.cyr`,
> `h5probe.cyr`, `probe_h7.cyr` in the tree) and two concurrent edits to a cancel path is exactly
> where that goes wrong.
>
> ⭐⭐ **2026-09-14. `VERSION` reads 0.9.3; 0.9.2 released.** ⛔ commit, tag and push are the operator's.
>
> 1. ⭐⭐ **crab CAN SEE A SYMLINK.** ⛔ readdir never could: agnos's `ext2_readdir_at_sys` sets byte
>    63 with `if (ftype == 2) { t = 1; }` — one bit, DIR or not — so a link arrived indistinguishable
>    from a file. ⭐ The stat sweep already visits every entry (32/tick until none is pending,
>    whatever the sort mode), so **`lstat` costs no extra syscall**, and the kind goes into the type
>    byte crab already had: `0` file, `1` dir, **`2` link**. Marked `@`, KIND says **Link**, size is
>    the link's own as `ls -l` shows. ⚠ `lstat` first, `stat` as the fallback — `lstat`#102 is
>    **ext2-only** and crab lists FAT volumes, so an lstat-only sweep would leave a whole disk
>    unstatted. It loses nothing: a filesystem that cannot hold a link has none to miss.
> 2. ⛔⛔ **DELETING A LINK DELETED WHAT IT POINTED AT — MEASURED, FIXED, AND OLDER THAN THIS WORK.**
>    The single-entry delete verb read `if (ddir != 0)` and handed anything non-zero to
>    `crab_walk_begin(CRAB_OP_DTREE, …)`, which **type-checks nothing**. So a link pointing at a
>    directory became the ROOT OF A RECURSIVE DELETE and the walk went through it into the target.
>    On iron: `crab: delete zzlink -> done`, link **still on disk**, **0 of 3 files survived** in
>    `zztarget/`. ⚠ **Not a regression from making links visible** — `crab_stat_one` used to call
>    `stat`, which FOLLOWS a link, so a link to a directory was already stored as `1` and already
>    took that branch. Fixed by **`crab_delete_plan(kind)`**; held by `crab-symlink-test.py`.
> 2b. ⛆ **AND THE FIRST AUDIT REPORTED ITSELF COMPLETE HAVING MISSED SIX READERS.** It grepped `== 1`
>    and `!= 1`; the entire `!= 0` / `== 0` family went unlooked-at, and that is where the damage
>    was — the delete verb above, the delete **prompt** (a link announced as *"FOLDER `<name>` and
>    everything in it"*), the transfer planner at both call sites, Enter/Open (did nothing and said
>    nothing), two thumbnail readers, and `crab_fs_delete`. The second audit **enumerated all 22
>    reads of `CRAB_REC_TYPE`** and traced each to its decision. ⇒ Widening a field is only safe
>    where every reader agrees how to ask, and **the way to know they agree is to enumerate them, not
>    to grep for the shapes you remember writing.**
> 3. ⭐ **THE OBVIOUS FEAR IS NOT REAL, and the same one bit is why.** A symlink to an ancestor cannot
>    make the recursive walk loop: it descends only where the type byte says `1`, and neither agnos
>    (`ftype == 2`) nor the host (`dt == 4`) sets that for a link — and 0.9.3 writes `2`. Asserted.
> 4. ⇒ **Delete and move PRESERVE a link; copy DEREFERENCES** (`cp -r`'s behaviour), now a decision
>    rather than an accident — **[ADR 0004](../adr/0004-symlinks-are-shown-preserved-and-dereferenced-on-copy.md)**.
>    ⚠ **Recreate is possible and deferred**: `symlink`#63 and `readlink`#70 both have cyrius peers,
>    but both are **ext2-only** and crab's whole premise is copying between volumes — what a link
>    *becomes* when it lands somewhere that cannot hold one needs its own decision.
>
> **Suite 2345 / 0**, render_test **53 / 0 — and it did not COMPILE for most of this release.**
> ⛔⛆ **THE RENDER GATE WAS BROKEN BY 0.9.3 AND I CLAIMED IT GREEN FROM MEMORY.** `CRAB_KIND_FILE/
> _DIR/_LINK` were declared in `app.cyr`, which **includes** `ui.cyr` — so the moment the render path
> started naming them (`@`, the KIND column, the delete prompt) they were invisible to it, and
> `src/render_test.cyr`, which includes `ui.cyr` ALONE precisely to catch a render path reaching
> upward, stopped building. ⇒ **The architectural gate worked; nobody ran it.** `ci.yml:72` runs it,
> so the 0.9.3 push would have gone red. Fixed by moving the three values into `path.cyr`'s `CrabRec`
> beside `CRAB_REC_TYPE` — the byte's values next to the byte's offset, at the bottom layer every
> reader can see. ⚠ Both `path.cyr` and `ui.cyr` already carried a note saying exactly this
> (*"layering follows the include order, not the other way round"*); the rule was written down and I
> put the enum on the wrong side of it anyway.
> ⇒ **`cyrius test` does not build `render_test.cyr`** — it discovers `tests/*.tcyr` only. A green
> suite is not evidence the render gate compiles; build it explicitly.
> Host **1,084,832 B** · agnos **1,129,864 B**.
> ⭐⭐ **QEMU: `crab-symlink-test.py` PASSES** (2026-09-14) — a real ext2 symlink placed by
> `mkfs.ext2 -d`, crab driven to it by keyboard, and the image read back with **`debugfs`** after
> shutdown rather than crab asked whether crab did the right thing. Gates the prompt's wording, that
> Enter answers out loud, that the link is deleted, and that the target and all three of its files
> survive. Run against the planted `!= 0` it reports the data loss in item 2 — **the harness is
> mutation-proven, not just green.**
> ⛔ **It does not count keypresses.** A first run pressed Down 80 times to "clamp at the bottom",
> landed on `whirl` and deleted it — keys are lost between the compositor's per-frame drains, so N
> presses are not N rows. It homes on `crab: prompt` instead and only confirms once the prompt names
> the right entry.
> ⚠ `crab_stat_one`, `crab_readdir_into` and the delete verb are all inside the agnos `#ifdef` with no
> `#else`, so **no host test reaches any of them** — the decisions are lifted into `crab_stat_kind`,
> `crab_delete_plan` and `crab_transfer_plan` and proven in the suite; the consequence needed iron.
> ⚠ **The 0.9.2 block below is one release stale but its reasoning is current.**

# Handoff — **0.9.2 cut: `Go` is filled, and an open menu stops acting on the pane underneath it.**

> ⭐⭐ **2026-09-14, READ THIS BLOCK FIRST.** `VERSION` reads **0.9.2**; **0.9.1 released**. ⛔ commit,
> tag and push are the operator's; `git log --oneline -3` is the authority.
>
> 1. ⭐⭐ **`Go` IS FILLED**, empty on the bar since 0.8.0. Its rows are every sidebar destination,
>    picked through **one navigator** — `synth_goto`, mirroring `synth_u` exactly: a pointer pick
>    becomes a KEY so one binding table answers it; a `Go` pick becomes a PATH so one navigator does.
>    The sidebar's Enter now files a destination too, so twenty lines of `crab_goto` + pane-state +
>    selection reset exist **once**. ⛔ `crab_menu_accel` has nothing to return for a path, and
>    inventing a key would make the menu a second implementation of navigation.
> 2. ⛔⛆ **AND IT CLOSED A KEY LEAK WORSE THAN THE ONE RECORDED.** The roadmap said `d` was not
>    consumed by the drop arm. In fact the arm handled **six** keys and let **every other key through
>    with `u` intact** — with a menu open, `c`/`m` started a transfer, `r`/`n` opened a sheet,
>    Backspace ascended, Space marked, all under a popup painted over them. The delete case defeats
>    `crab_del_prompt`'s whole reason for existing — *"THE PROMPT NAMES WHAT DIES"*, written after
>    five system binaries left an iron box. ⇒ `crab_mb_drop_key`, the `crab_sb_key` shape, lifted so
>    the suite can reach it — and **an assertion ties the two eat-sets together**, because two
>    surfaces that borrow the keyboard from the panes must refuse the same verbs or one is a hole.
> 3. ⛔ **THE DROP-DOWN HAD NO HEIGHT RULE**, and `Go` is the first menu that needed one: its length
>    is the MODEL's. **Six rows fit at 380×220** — 220 − bar 22 − status 22 − margin, over a 26 px
>    row. Seventeen want 442. ⚠ The status line is subtracted on purpose: `dh_place_at_point` clamps
>    against the SURFACE, so without it a drop sits legally on the line that says what is selected —
>    which since 0.9.1 also holds the door that closes the menu. ⚠ **Refused, not truncated.**
> 4. ⚠ **`Parent` is a VERB, not a destination** — Backspace. The roadmap's condition was *"absent
>    rather than dead at `/`"* and the first draft carried a `hasparent` flag; wrong shape, because a
>    menu holding both verbs and destinations needs two pick paths in one list, which is exactly what
>    kept `Go` empty. It is absent **everywhere**. ⚠ Volumes are labelled by **prefix** (unique by
>    construction), closing *"two FAT volumes render as two identical rows"*.
> 5. ⭐⭐ **QEMU CAUGHT A BUG THE SUITE COULD NOT.** The navigator sat ABOVE the key dispatch, so a
>    pick made from *inside* the dispatch was not consumed until the NEXT key — the menu closed, the
>    pane stayed, and the destination fired later against whatever was pressed next. On target that
>    read as `Go` doing nothing. The suite was green throughout: every line is inside the agnos
>    `#ifdef`. ⭐ `crab-go-test.py` **PASS** — `F10 → Right ×2 → Enter → Enter` sends the pane to `/`
>    and lists 8 entries; `d` ×3 with a menu open does nothing at all.
>    ⚠ **Driven by the keyboard deliberately** — 0.9.1 spent seven runs failing to aim a relative
>    pointer, and `F10 → Right → Enter` is a road `crab-columns-test.py` already drives.
> 6. ⚠ **Two oracles were weak and are now dispositive**, both found by a harness that could not tell
>    two causes apart: `crab: go <n> destinations` (a dead pick and an empty model look identical from
>    outside) and `crab: place <path> ok <n> entries` (the line printed whether or not the listing
>    succeeded, and a refusal only ever reached the status line).
>
> **Suite 2314 / 0**, render_test 53 / 0. Host **1,084,744 B** · agnos **1,129,760 B**.
> ⭐ **Next: `0.9.3 · Symlinks`** — or any of 0.9.3–0.9.5; they are independent.
> ⚠ **The 0.9.1 block below is one release stale but its reasoning is current.**

# Handoff — **0.9.1 cut: the door — the menu row has a second way in.**

> ⭐⭐ **2026-09-14, READ THIS BLOCK FIRST.** `VERSION` reads **0.9.1**; **0.9.0 released**. ⛔ the
> commit, the tag and the push are the operator's; `git log --oneline -3` is the authority.
>
> 1. ⭐⭐ **A MARK IN THE STATUS LINE OPENS THE MENU ROW.** `F10` was the only door — and for seven
>    releases it was **no door**, because aethersafha claimed that key until 0.16.25. ⛔ **Zero
>    rows**: the status line is now a `BOX_H` of [door][text], the shape `crab_pane` already uses for
>    the A/B strip. The text flexes; the mark does not.
> 2. ⛔⛆ **THE TOGGLE IS THE Z-ORDER, NOT A STATE BIT.** The door sits BELOW the bar branch in
>    `crab_pointer_action`, so a press on it with the bar shown never reaches its own arm — the bar
>    branch sees a press off a bar cell and answers DISMISS. Open when closed, close when open, from
>    one hit test. Moving it up is a mutation that returns DOOR where DISMISS is expected.
> 3. ⛔⛆ **IT IS NOT A CRAB GLYPH, AND THE ROADMAP SAID 0.9.0 WOULD MAKE IT ONE. That line was mine
>    and it was wrong three times over**: `rekha_char_to_glyph` returns 0 above U+FFFF in its own code
>    (*"format 4 is BMP-only"*); the shipped face has **no format-12 subtable**; and
>    `dh_draw_text_ink` walks **one byte per glyph in both branches**. U+1F980 is 128,896. Any one is
>    fatal alone. ⭐ **CANVAS is the open road** — crab already draws thumbnails through
>    `dh_canvas_new`; *an icon is a glyph with no font*. The door is three filled boxes.
>    ⚠ No font also sidesteps a 0.9.0 consequence: **byte 0xF0 is `≡` through kashi and `ð` through
>    rekha**, so crab's drawable alphabet is printable ASCII and a text mark would differ between the
>    host build and the target.
> 4. ⛔⛆ **A SERIAL LINE MUST BE ONE WRITE — MEASURED.** The console is shared unserialised by three
>    processes and spliced both new oracles: `crab: font /fonts/default.ttf 0` (digits lost) and
>    `crab: door ptrscan: first sample handed to ring 3` (the kernel, mid-line). A harness reading
>    either gets a parse failure **indistinguishable from the feature being broken** — three QEMU runs
>    went on that before it was the diagnosis. ⇒ `crab_line_*` composes, then emits once. A single
>    `crab_say` is still fine; it is the SEQUENCE that is not atomic.
> 5. ⚠ **THE PRESS ITSELF IS UNMEASURED, AND THE HARNESS SAYS SO.** On target crab reports
>    `crab: door 0 196 22 22` every run. **Seven runs** went into landing a press inside that rect
>    and none did: the rect is in crab's SURFACE coordinates, the monitor moves a RELATIVE
>    `usb-mouse` in SCREEN coordinates, the compositor picks the window origin, and homing is
>    unreliable — an unchanged script proved delivery at (200,80), (100,80), (200,130) and (200,180),
>    and put the pane edge at y=154 one run and y=104 the next. Sweeping the mapped region in both
>    axes delivered nothing, i.e. presses were **dropped**. ⇒ FAIL would assert the door is broken,
>    which nothing shows and nine suite assertions contradict. `crab-door-test.py` returns
>    **INCONCLUSIVE** and carries all seven runs' findings in its header.
>
> **Suite 2257 / 0**, render_test 53 / 0. Host **1,080,480 B** · agnos **1,121,344 B**.
> ⭐ **Next: `0.9.2 · Go`** — the shape is already decided (a thin projection over the sidebar's
> keyboard route, capped by `crab_mb_drop_fit`). 0.9.2–0.9.5 are all independent.
> ⚠ **The 0.9.0 block below is one release stale but its reasoning is current.**

# Handoff — **0.9.0 cut: A REAL FACE — crab draws in Liberation Sans on the target.**

> ⭐⭐ **2026-09-14, READ THIS BLOCK FIRST.** `VERSION` reads **0.9.0**; ⛔ the commit, the tag and the
> push are the operator's. ⚠ The operator commits while work is in flight — `git log --oneline -3` is
> the authority, not this file.
>
> **M5's longest-open item is closed.** Two days ago it was blocked by two things outside crab; both
> were filed in the repo that owned them and both closed within 24 hours.
>
> 1. ⭐⭐ **crab OPENS `/fonts/default.ttf`** — agnos 1.57.2's kernel-owned namespace. Liberation Sans
>    Regular 2.1.5, unmodified, 410,820 B, **SIL OFL 1.1: the licence text must travel with any
>    redistribution.** Loaded once, before the first frame; all eight `crab_render` sites draw in it.
>    ⛔ **Read front-to-back in one pass** — `lseek` is -1 on a `VFS_MEMFILE`, so a reader that seeks
>    gets an error, not a rewind. ⛔ **Opened outside any draw**: dhancha 0.10.0 scopes sadish's
>    allocation hook to one `dh_draw_text_ink` and `rekha_font_open` follows it, so a face opened
>    inside would live on the frame arena and die at its first reset. ⚠ A full read buffer is a
>    **truncation**, not a success — with no `lseek` it is the only signal there is.
> 2. ⛔⛆ **THE COINCIDENCE THAT WOULD FOOL ANYONE VERIFYING THIS.** Liberation Sans's `n` is
>    1139/2048 em; at 16 px that is 8.9, which rounds to **exactly kashi's 9**. So every width crab
>    derives comes out **numerically identical to the bitmap face's** — a check asserting "the advance
>    changed" or "the layout moved" would FAIL against a perfectly working face. ⇒ The oracle prints
>    **`i=4 m=13`**, which a monospace face cannot produce. *The columns did not move; what goes in
>    them did.*
> 3. ⭐ **AND THE `~` MARKER FINALLY MEASURES — this was the whole point of the item.**
>    `crab_name_cell` truncated at a CHARACTER COUNT: ten `m`s "fit" a ten-character column and
>    measure **160 px in 120**, so the name was clipped by the column while carrying a `~` claiming it
>    had been cut to fit — the 0.5.0 defect, reinstated by an estimate. `crab_name_cell_px` measures
>    greedily and **reserves the marker's own width before accepting any name byte** (otherwise the
>    mark saying "this was cut" is the thing clipped off the edge). ⚠ At `font = 0` it reduces exactly
>    to the old arithmetic — the bitmap build is unchanged and the suite's numbers did not move.
> 4. ⭐⭐ **QEMU — `crab-face-test.py`, PASS.**
>    `crab: font /fonts/default.ttf 410820 bytes adv=9 upem=2048 i=4 m=13`; exactly one load per
>    session; navigation and view switching under the face; no faults, no allocator failure — ⛔ the
>    last of which **dhancha 0.10.0 is what makes passable**: before it, every label allocated a
>    full-surface canvas per frame and a session in a face would have exhausted the heap.
>    ⚠ **One mutation PASSED and is recorded**: deleting `crab_face`'s memo leaves the host suite
>    green, because with no `/fonts` the load fails at its first step either way. **The memo's gate is
>    ARM 2 on the target**, not the suite — unmemoised, crab would re-parse 410 KB every frame.
> 5. ⚠ **Two bugs of mine, kept in the source as comments** because both look right: exiting the
>    greedy loop with `j = n` destroyed the count in the same statement that ended the walk (SIGSEGV
>    when the marker was then placed by scanning for a NUL nothing had written); and summing
>    `dh_text_advance` **before** the `crab_font == 0` check dereferences a null font — the bitmap
>    face is a different BRANCH, not a fallback value.
>
> **Suite 2213 / 0**, render_test 53 / 0. Host **1,076,240 B** · agnos **1,121,176 B**.
> ⭐ **Next: `0.9.1 · The 🦀 button`** — and it is no longer chained to anything, because the face it
> needed is here. See [the ladder to 1.0](roadmap.md); every entry there is a version.
> ⚠ **The 0.8.11 block below is one release stale but its reasoning is current.**

# Handoff — **0.8.11 cut: the 6.6.4 stack, and both blockers on a real face cleared within a day.**

> ⭐⭐ **2026-09-14, READ THIS BLOCK FIRST.** `VERSION` reads **0.8.11**; ⛔ the commit, the tag and the
> push are the operator's. ⚠ The operator commits WHILE work is in flight — `git log --oneline -3` is
> the authority on what is in, not this file.
>
> 1. **THE 6.6.4 STACK.** cyrius 6.6.2 → **6.6.4**; sadish **0.5.5**, rekha **0.3.10**, kashi
>    **1.0.8**, dhancha **0.10.0** (rupa/setu/chitra unmoved). ⛔ sadish and rekha are a **floor**, not
>    company — without them dhancha 0.10.0 **refuses** the build (`2 reachable undefined function(s)`:
>    `sd_alloc_set`, `sd_canvas_blit_at`). ⚠ **6.6.3 silently corrupts even-length string literals
>    ≥ 64 KB**; agnos found it generating the embedded face and made the kernel hash-verify the bytes.
>    ✅ **Check four re-run**: overrides off, lock **3 → 7 commit-pinned**, both binaries
>    **byte-identical**, 2198 / 0 in that tree.
> 2. ⭐⭐ **THE EXPIRY FIRED AND THE ASSERTION IS INVERTED.** 0.8.10 asserted a defect in dhancha and
>    wrote *"the day dhancha fixes this, the test FAILS and must be inverted."* **dhancha 0.10.0 fixed
>    it** — sadish 0.5.5's `sd_alloc`/`sd_alloc_set` hook, rekha 0.3.10's scoped outline scratch,
>    dhancha installing `dh_falloc` as that hook for one `dh_draw_text_ink` and sizing the canvas to
>    clip ∩ surface ∩ run. ⇒ **A warm frame under a real face costs the global heap exactly 0.**
>    ⭐ **dhancha's hand-off predicted crab's failure by name and to the byte** from measurements taken
>    against crab 0.8.10 — not `scost > 0` but `arena_capacity_total(farena) == cap0`, *got 468,040,
>    expected 16,384*. That is what a filing with a gate behind it buys.
>    ⛔ **Three rules it taught, now in the suite**: a **warm-up face frame** at the widest run before
>    measuring (a cold one chains ~452 KB of arena chunks — the arena growing, not a leak); the face
>    **opened outside a draw** (`rekha_font_open` follows the scoped hook and would die at the arena's
>    first reset); and the arm placed **below** the bitmap assertions, since a face frame extends the
>    chunk chain and that is exactly how it broke `cap0`.
> 3. ✅ **AND agnos 1.57.2 CLOSED THE OTHER ONE, the same day it was filed.** A **kernel-owned
>    `/fonts` namespace** — the operator's ruling picked rekha, and agnos embedded the face
>    kashi-style rather than staging an asset. **`/fonts/default.ttf`** is the contract
>    (`/fonts/LiberationSans-Regular.ttf` is the same bytes); Liberation Sans Regular 2.1.5
>    unmodified, 410,820 B, **SIL OFL 1.1 — the licence must travel with any redistribution**;
>    FNV-1a-64 verified at boot. ⛔ Read-only `VFS_MEMFILE`, **`lseek` is -1 — one front-to-back pass**.
>    Contract: `agnos-userland-abi.md` §3.5.
>    ⭐⭐ Both filings stated the need and **declined to design the sibling's answer**; both siblings
>    then picked something better than crab would have asked for. *Declining to approximate is what
>    got the right primitive built* — twice.
> 4. ⇒ **`0.9.0 · A real face` IS UNBLOCKED.** What remains is crab's own: open `/fonts/default.ttf`,
>    hand the face to `crab_render`, prove a warm frame still costs zero. ⛔ **On QEMU** — that file
>    exists on no host, and the one existing template in the stack reads a host path and would fall
>    back silently on the target while looking finished.
>
> **Suite 2198 / 0**, render_test 53 / 0, fuzz 100k, coverage 88 %, vet/deny clean, `deps --verify`
> 50 / 0. Host **1,071,688 B** · agnos **1,116,632 B**.
> ⚠ **The 0.8.10 block below is one release stale but its reasoning is current.**

# Handoff — **0.8.10 cut: every width derived from the font, proved against a proportional face — and the two things that actually block a real one.**

> ⭐⭐ **2026-09-13, READ THIS BLOCK FIRST.** ⭐ **0.8.7, 0.8.8 and 0.8.9 are COMMITTED AND TAGGED**
> by the operator (`22f7f53` / `60cc05b` / `eeb6a8b`) — the three-releases-uncommitted backlog that
> every block below this one warns about is **cleared**. **0.8.10's CODE is committed** too
> (`c0c7724`) and is **not yet tagged**; `VERSION` reads 0.8.10 and ⛔ the tag and the push are the
> operator's. ⚠ The operator was committing WHILE this release was written, so treat any
> committed/uncommitted claim in this file as of its writing — `git log --oneline -3` is the answer,
> not a document.
> ⚠ `git ls-remote --tags` answered empty when this was written, so whether those three tags have
> been PUSHED is unknown from here — said rather than assumed, because every block below asserts a
> remote state and this one will not.
>
> ⚠ **THIS IS THE UNBLOCKED HALF OF `0.9.0 · A real face`, NUMBERED HONESTLY.** An operator cannot
> newly read crab in a proportional font; crab is now *correct under one* and the suite proves it.
>
> 1. ⭐⭐ **THE CHARACTER COUNT IS THE CONSTANT; THE PIXEL WIDTH IS DERIVED.** Seven widths read
>    `CRAB_COL_NAME_MIN = 90;  # 10 chars` — the governing fact in the comment, the face-specific
>    number in the code. ⛔ And none failed loudly under another face; they picked a **wrong layout**:
>    a NAME column under ten characters cannot tell `…build.log` from `…build.tmp`, which is the exact
>    confusion `crab_name_cell`'s `~` exists to prevent. Now `crab_col_name_min()` and six siblings.
>    ⚠ Functions, not constants — a Cyrius `enum` member must be a literal.
> 2. ⭐⭐ **A SYNTHETIC PROPORTIONAL FACE IN THE SUITE, WHICH RETIRES 0.8.8'S OWN ADMISSION.** 0.8.8
>    had to write that no host test could tell *"asks the font"* from *"divides by the constant"*.
>    There is still no TTF in the stack — so the suite **builds** one (head/maxp/hhea/hmtx/cmap, the
>    shape rekha's and dhancha's tests use; no `glyf`, and none is needed for a width question).
>    ⛔ Its advances are deliberately **unequal**, so `"nn"` and `"nm"` have equal length and different
>    width — which a `length × advance` implementation cannot tell apart, and every truncation rests
>    on telling apart. Three mutations, each caught.
> 3. ⛔⛆ **THE ZERO-ALLOCATION GATE WAS MEASURING THE BRANCH THAT WAS NOT RUNNING.** Every render in
>    it passes `font = 0` (the bitmap path). The scalable path opens with
>    `sd_canvas_new(surface_w, surface_h)` — a full-surface canvas **per label, per frame** — plus a
>    path per glyph, from the bump allocator with no `free()`. crab's M1.5 headline would have become
>    false on the first frame with a face **and nothing would have noticed.** The fixture makes that
>    branch reachable; the cost is now a measured number. ⚠ **The assertion carries its own expiry** —
>    when dhancha fixes it, that test FAILS and must be inverted.
> 4. ⛔⛔ **AND `0.9.0` IS BLOCKED ON TWO THINGS, NEITHER OF THEM crab's.** Its "blocked by" cell read
>    "—" until this release went looking.
>    **(a) There is no TrueType face in the stack and nothing stages one onto the target** — zero
>    `*.ttf`/`*.otf` across every first-party repo, and the `agnos` repo contains no occurrence of
>    "ttf", "truetype" or "sfnt" anywhere; `build/rootfs` has no `/usr`, no `/share`, no font dir.
>    ⭐⭐ **OPERATOR RULING 2026-09-13: *"rekha is that thing... but has yet to get Kernel support."***
>    ⇒ rekha IS the designated answer and the gate is an **agnos** arc — **not** a font-picking
>    question, and crab must not treat it as one. crab states the need and does not design agnos's
>    answer (the VOLUMES precedent). ⚠ Two things that look missing are not: a 410 KB read is ordinary
>    and `stage-tools.sh` already carries `etc/ssl/cert.pem`.
>    ⛔⛆ **The obvious template is a trap**: the one caller feeding `rekha_font_open` real bytes reads
>    a **host Arch path that does not exist on AGNOS** — copied into crab it works on the host, falls
>    back silently on the target, and looks finished. ⚠ kashi is not the escape hatch and ADR 0003's
>    expiry has not fired — it is about bitmap loading.
>    **(b) dhancha's scalable draw allocates outside the frame arena** — item 3. ⇒ **dhancha** owns it.
>    ⇒ **Both are FILED IN THE OWNING REPOS** (2026-09-13), with crab copies beside them:
>    `agnos/docs/development/issues/2026-09-13-no-proportional-face-on-the-target.md` and
>    `dhancha/…/2026-09-13-scalable-text-allocates-per-call-outside-the-frame-arena.md`.
>
> **Suite 2196 / 0** (+38), render_test **53 / 0 unchanged** — which is the proof that what ships
> still draws exactly as it did. Host **1,071,560 B** · agnos **1,112,424 B**.
> ⚠ **No QEMU arm, stated rather than skipped**: nothing agnos-only changed and no code entered the
> `#ifdef`.
> ⚠ **The 0.8.9 block below is one release stale but its reasoning is current.**

# Handoff — **0.8.9 cut: Shift — capital letters in names, and a sheet that can accept its own language.**

> ⭐⭐ **2026-09-13, READ THIS BLOCK FIRST.** **0.8.6 is still the last RELEASED version** (`249279f`,
> on the remote). **0.8.7, 0.8.8 and 0.8.9 are all CUT and none is committed** — `VERSION` reads
> 0.8.9, the CHANGELOG headers agree, and ⛔ the commit, the tag and the push are the operator's.
>
> 1. ⭐⭐ **A SHIFT LATCH — names can hold capital letters.** `crab_key_char` has taken the flag since
>    the rename field was built; its ONE production call site passed a literal `0` under a comment
>    reading *"there is no shift state on the wire yet."* ⛔ **The wire was never the problem, and half
>    the written claim was wrong the whole time**: `mods` does carry only the press/release edge, but a
>    modifier's OWN edge arrives as its own key event, and aethersafha 0.16.25 exempts those edges from
>    the Ctrl-chord swallow **on purpose** — its source says *"a client that wants Shift state has no
>    other way to learn it."* crab had been receiving Shift since that release and discarding it.
> 2. ⛔⛔ **A MASK, NOT A BOOLEAN.** Hold LeftShift, hold RightShift, release LeftShift — still
>    shifted. A single flag cleared there and the next letters came out lower case mid-word with both
>    hands on the keyboard. Two keys, two bits. ⛔ And the release-clear is a **guarded** subtraction,
>    because an unpaired release is **measured** on this stack, not hypothetical: the 2026-09-13 QEMU
>    investigation recorded that *"a claimed key's release is forwarded while its press is not."*
>    Unguarded, one stray release turns mask 1 into −1 and every letter is capitalised forever.
> 3. ⛔⛆ **AND IT CLOSED A SECOND RECORDED GAP THAT LOOKED UNRELATED.** The batch sheet's label reads
>    `# = number, * = old name` and **neither character could be typed into it** — `#` is Shift+3, `*`
>    is Shift+8, and the whole shifted number row answered 0 under *"the shifted row is symbols crab
>    does not need."* A surface describing a language its own input cannot produce. The row is filled,
>    all ten, and the suite pins those two **against `crab_batch_name` itself** rather than against two
>    literals, so the keyboard and the language cannot drift apart.
> 4. ⛔⛔ **THE ORDERING IS INVISIBLE TO THE SUITE, AND THAT IS WHAT THE HARNESS IS FOR.** In
>    `src/main.cyr` the latch is fed the RAW edge three lines BEFORE `crab_key_is_modifier` zeroes a
>    modifier's `kacts`. Track first, suppress second. Backwards, every Shift PRESS reaches the latch
>    looking like a RELEASE, the latch never sets, and **the suite stays completely green** — all of it
>    is inside the agnos `#ifdef` with no `#else` and nothing includes `main.cyr`.
>    ⭐ `crab-shift-test.py` — **PASS**: `Shift+A Shift+B Shift+3 Shift+8` committed exactly **`AB#*`**
>    (latch, ordering and both operators in one line); `Shift+A` then `b c` committed exactly **`Abc`**
>    (the release clears); ten shift edges arrived and crab acted on **zero**; no faults.
>    ⚠ **Its first run measured `nnnAB#*` and that was the HARNESS.** The retry pressed `n` until the
>    sheet opened and could not tell that it had, so every extra press typed a literal `n` into the
>    name. ⇒ crab now prints **`crab: edit open <label>`** — it was the one interactive surface opened
>    in silence, on the exact boundary where a keystroke stops being a command and becomes text.
> 5. ⚠ **The overwrite policy's second reason expired and is corrected, not dropped.** 0.8.7 armed
>    *all* with `a` partly because `R` was *"unreachable on this stack"*. False now. **The design does
>    not change** — the surviving reason was the load-bearing one: a capital is an **invisible mode**,
>    and this is a prompt where the next keystroke can destroy a file.
>
> **Suite 2158 / 0** (+54), render_test 53 / 0. Host **1,067,496 B** · agnos **1,112,456 B**. Four
> mutations, each caught — including `'0'` surviving a range widened from `0x1E..0x26` to `0x1E..0x27`
> (the planted defect made it come out as `':'`).
> ⭐ **Next was `0.9.0` — a real face.** Its crab-side half shipped as **0.8.10** (above); `0.9.0`
> itself is now **blocked on agnos and dhancha**. See [the ladder to 1.0](roadmap.md).
> ⚠ **The 0.8.8 block below is one release stale but its reasoning is current.**

# Handoff — **0.8.8 cut: the COLUMNS view, one reader for the 9 px advance, and a roadmap with an order.**

> ⭐⭐ **2026-09-13, READ THIS BLOCK FIRST.** **0.8.6 is still the last RELEASED version** (`249279f`,
> on the remote). **0.8.7 and 0.8.8 are both CUT and neither is committed** — `VERSION` reads 0.8.8,
> the CHANGELOG headers agree, and ⛔ the commit, the tag and the push are the operator's.
>
> 1. ⭐⭐ **COLUMNS — M5's last view, and its design question answered.** `g` cycles a fourth time and
>    the active pane grows a narrow **context column** showing the parent with the current directory
>    marked. ⛔⛔ **It is a view mode of ONE pane, and that is a SAFETY decision, not a layout one.**
>    N independently navigable miller panes make `active_pane` something other than a 0/1 — and
>    `active_pane` is a 0/1 that the **entire M4 write layer** resolves every copy, move and delete
>    against. A drag from column k into column k+1 would plan a `crab_fs_move` of a directory **into
>    its own subtree**. K = 2 with one driven column answers it by construction, and the canvas agrees
>    (pane A as columns, pane B as the preview). ⇒ **If miller ever goes N-deep, the gate is the write
>    layer, not the renderer.** The context column holds no focus (dhancha's MUTED-vs-ACCENTED
>    selection is the whole answer to *which listing do my arrows drive*), is not in `crab_hit`'s walk
>    (a pointer path means an answer to *which pane is this* — the 0/1 again), is **dropped rather
>    than squeezed** below 186 px, and at `/` is not drawn at all.
> 2. ⛔⛆ **AND LOOKING FOR THE FOURTH VIEW'S SEAT FOUND THE GUARD THAT WAS MEANT TO PROTECT IT.**
>    `crab_view_is_grid` was added in 0.8.2 under a comment reading *"one predicate, asked in all
>    three places, so a fourth view cannot be added and half-wired."* It was asked in `main.cyr`'s
>    three ARROW sites and **nowhere else** — `crab_pane` and `crab_render`'s scroll round-trip each
>    still tested `view != CRAB_VIEW_LIST`. A fourth id would have **rendered as a GRID, scrolled as a
>    GRID, and taken LIST arrow semantics**: the same defect 0.8.2 fixed, inverted, in the file that
>    introduced the fix. ⇒ ***A negation is not a predicate.*** Ask what a new id would actually do
>    **before** adding one; it is the only time that question is cheap.
> 3. ⭐ **The 9 px advance has ONE reader now.** `crab_char_w()` answers `CRAB_COL_CHARW` for the
>    bitmap font and the **font's own** advance for anything else; `crab_text_w(s)` **measures** a
>    string rather than pricing it at `length × advance`. crab still passes `font = 0`, so it renders
>    identically — which is exactly why it was done on its own. ⚠ **What the suite cannot say is said
>    at the assertions**: at `font = 0` no host test can tell *"asks the font"* from *"divides by the
>    constant"*; deleting `crab_font = font` leaves the suite green and the test records that rather
>    than implying a gate. **What is left of proportional text is passing a real face** — `0.9.0`.
> 4. ⭐⭐ **QEMU: `agnos/scripts/harness/crab-columns-test.py` (new), PASS on its first run.** `g` ×4
>    reaches COLUMNS; the parent of `/bin` is listed and titled `/`; **four redraws re-listed nothing**
>    (the memo arm — a readdir is not a render-path operation, and without the memo this view would
>    readdir plus stat-every-entry on every keypress, pointer move and idle tick); the root's refusal
>    is memoised; leaving the view stops the listing; no faults. ⛔ **The listing could not be gated
>    any other way**: `crab_readdir_into`'s body is inside `#ifdef CYRIUS_TARGET_AGNOS` with no
>    `#else`, so on the host it returns 0 entries for **every** path. A first draft of the suite built
>    a real tree under `build/` and asserted the listing; every assertion failed against an empty
>    listing and a clean error code, which is what that `#ifdef` looks like from a host test.
> 5. 🗺 **The roadmap has an order.** [The ladder to 1.0](roadmap.md) — every entry a VERSION, plus one ruling,
>    each named by what an operator can newly do, with what blocks it and what closes it.
>    ⛔ **The order is the commitment; the number is not** (the milestone→version map has been wrong
>    four times, and both 0.8.7 and 0.8.8 closed milestones **out of order, after all of M6**).
>    ⭐ **Next was Shift** — ✅ shipped as **0.8.9**, above. ⚠ **daimon is deliberately not on the
>    ladder** — it is a ruling the operator owns.
>
> **Suite 2104 / 0** (+88 over 0.8.7), render_test 53 / 0. Host **1,067,440 B** · agnos **1,112,400 B**. Fourteen mutations
> planted; ⚠ **two of them PASSED and both are recorded rather than deleted** — the context column's
> focus grab (the assertions were being carried by CALL ORDER, so the test now asks `crab_col_ctx`
> directly: *an order is not a contract*) and the scroll round-trip (behaviour-identical today because
> `dh_grid_*` on a vertical LIST reduces exactly to `dh_list_*`; fixed anyway, and the test says which
> claim it can make).
> ⚠ **The 0.8.7 block below is one release stale but its reasoning is current.**

# Handoff — **0.8.7 cut: a collision you can answer, a flag surface, `u` refreshes, chrome keys on Ctrl, and crab proven on a real kernel.**

> ⭐⭐ **2026-09-13, READ THIS BLOCK FOR THE CURRENT NUMBERS AND THE ONES BELOW IT FOR THE REASONING.**
> **0.8.6 is the last release** (`249279f`, on the remote). `[Unreleased]` holds two things and no
> cut was asked for; `VERSION` reads 0.8.6; nothing is committed by anyone but the operator.
>
> 1. **The REFRESH key, `u`** — both panes relist (selection kept by NAME via `crab_refresh_sel`,
>    marks cleared, refused out loud during a transfer), PLACES and VOLUMES rebuilt, the sidebar
>    cursor re-seated by EXACT path (`crab_sb_row_for_path`, asserted against `crab_sb_here`'s
>    containment, and section-aware because `/` is both a place and a volume) or dropped.
>    `View ▸ Refresh` is the bar's fifth item — which finally makes the separator-mapping guard
>    load-bearing (`got -1` when mutated). ⛔ Not F5: claimed. ⛔⛔ **The refresh relist KEEPS the
>    thumbnail cache** (`crab_relist_keep_thumbs`) — an adversarial review (23 agents) caught the
>    first draft re-charging the permanent decode budget on every press: four presses on a 1024²
>    image would have ended thumbnails for the session. **1879 / 0**, six mutations caught.
> 2. ⭐⭐ **QEMU, run 4 of `agnos/scripts/harness/crab-pointer-test.py` (new): PASS.** The first
>    on-target run since 0.7.0. Right press → button **2** on the wire → context menu; left press on
>    a row → `verb by pointer` (it ran row 1, Copy, refused — the harness reports which verb ran
>    rather than assuming); press beside the popup → dismissed; `u` ×6 → 4 refreshes / 8 listings;
>    `g`, `b` answer; no faults. Verdict log beside the serial log in `agnos/build/crab-pointer/`.
>
> ⛔⛔ **THE FINDING THAT OUTRANKED BOTH, AND ITS CLOSE:** aethersafha's `input_map` claimed Esc, Tab
> and F4–F10 bare and consumed them — MEASURED: crab acted on 0, Esc ×4 → `quit on a key`. Three
> shipped affordances dead on the target, green on the host. **The operator ruled the same day** —
> *"it was easy for initial testing … now it's time to fix that right"* — and **aethersafha 0.16.25**
> (prepared in the sibling, git theirs) moves chrome onto Ctrl: **Ctrl+Q** quits, **Ctrl+Tab** cycles,
> **Ctrl+F4–F10** close/max/min/move; bare keys are forwarded; a chord is swallowed whole. Ctrl was
> already observable (usages 0xE0/0xE4 — the kernel diffs the modifier byte, bhumi maps it): no
> kernel or bhumi change. MEASURED AGAIN (run 6, ARM 7 rewritten as a gate): **bare F10 opened crab's
> menu bar and `View` was driven from the keyboard for the first time; bare Esc ×3 and Tab ×3 reached
> crab; Ctrl+Tab / Ctrl+F10 were the compositor's with crab acting on 0; Ctrl+Q ended the desktop.**
> crab's required half: `crab_key_is_modifier` — a modifier's own edge (forwarded, always was) is not
> a keystroke; it used to answer the delete prompt. ⚠ F2/F3 stay bare — not in the ruling, open.
> ⚠ Every harness in `agnos/scripts/harness` that sent a bare chrome key sends the chord now;
> `puka-terminal-test` expects its typed Tab to reach puka. Only the two crab harnesses were re-run.
>
> ⚠ **What driving crab under QEMU taught, kept in the harness header:** crab-resize-test's Enter ×8
> launch burst leaks into crab, where Enter is OPEN on the selected row — `/bin`'s first row is
> `aethersafha`, so run 1 spawned a **second compositor** and measured two of them on one mouse; a
> DOWN burst is a coin flip because the launcher wraps; keys are lost because the boot-keyboard
> report is a STATE — `sendkey <key> 400` holds the press across drains and the launch became
> deterministic; a pointer pick's row is not knowable from the harness, so it ascends to `/` first
> (Open = descend) and reports which verb ran. ⚠ `crab-resize-test.py` still has the burst.
> ⚠ `println(lnch_name_at(lsel))` in aethersafha prints the app name's ADDRESS — noted in the filing.
>
> ⭐ **0.8.7 IS CUT** (operator direction) — the overwrite policy, the flag surface, the REFRESH key,
> the chord contract and the QEMU runs. ⛔ Commit, tag and push are the operator's.
> ⚠ **Superseded by 0.8.8 above**: columns and the crab-side plumbing half of proportional text both
> closed there, and "what is next" is now [the ladder to 1.0](roadmap.md) rather than a sentence in
> this block. The ***Small, cheap, unblocked*** list is still open and still rides along (drop the
> redundant `net` declaration; park `--win`; give the stat trace an arm that works on agnos; correct
> the CHANGELOG's harvested-deferral count).
> ⚠ **The 0.8.6 block below is one release stale but its reasoning is current.**

# Handoff — **0.8.6 cut: `View` is filled, and M6's six interaction gaps are all closed.**

> ⭐⭐ **2026-09-13, READ THIS BLOCK FOR THE CURRENT NUMBERS AND THE ONES BELOW IT FOR THE REASONING.**
> **0.8.6 IS CUT on operator direction** — `VERSION` = 0.8.6, CHANGELOG `[0.8.6]`, every gate and
> check four green — **and the commit, the tag and the push are the operator's.** Until
> `git ls-remote --tags` shows `0.8.6`, **0.8.5** (`4344cb9`, on the remote, CI and Release green)
> is the last release; `git describe` answered `0.8.5` exactly before the cut was written.
>
> ⭐ **What 0.8.6 holds — one thing:** the menu bar's **`View` drop-down** — Cycle view · Cycle sort
> · Preview · Sidebar, the `g` · `s` · `p` · `b` keys with labels on them. Display ids **6..9, above
> `CRAB_MI_COUNT`** (the context menu's bound, unchanged) with `CRAB_MI_ALL = 10` as the new
> "is this an item at all" bound; the four maps (`crab_menu_label` / `_key` / `_accel` / `_enabled`)
> answer for them, the bar's `crab_mb_item(CRAB_MB_VIEW, j)` is the slice, and a pick is the same
> rewrite-and-fall-through every other entry uses. Always live (they act on the view, not a row);
> none of the four keys is eaten by sidebar focus; the drop fits at 380 px. `Go` stays empty with its
> reasons on record. ⚠ A note that was wrong by one: the separator mapping's "unexercised defence"
> becomes load-bearing at a FIFTH bar item, not a fourth — index 3 is not above `CRAB_MI_RENAME`;
> the note and its twin in the suite say so, and the suite pins that index 4 would shift.
>
> **Suite 1838 / 0** (+48), render_test 53 / 0, fuzz 100k, fmt clean, coverage 88 % (265/298),
> vet/deny 0, deps --verify 50 / 0. Host **1,049,568 B** `ebe13334…` · agnos **1,090,304 B**
> `59eeae79…`. Five mutations, each caught. Check four: 7 / 0, 3 → 7 commit-pinned, byte-identical.
> ⛔ **NOT run on QEMU or iron** — 0.8.5's pointer arm and this cut's View pick both ride the
> agnos-only synthesised-key road; 0.8.5's six oracle lines still stand for the run that will.
> ⚠ **Sequencing that run is the operator's call.**
>
> ⭐ **M6 is closed but for its two GATED items** (the 🦀 chrome button — no crab glyph in CP437;
> the held-key repeat number — agnos-runtime). **Next, ungated:** the absent affordances — a REFRESH
> key (the source names one as the intended home in two places), a flag surface for `--about`
> (`docs/development/mascot.md` asks for it by name), and the overwrite policy (an operator decision:
> replace / keep-both / skip / rename-on-collision, against a load-bearing overwrite guard). Small
> and cheap: drop the redundant `net` stdlib declaration (its own change; measured same size,
> different layout). Then M5's remainder — columns/miller (a design question on the two-pane model)
> and proportional text (crab-side: the 9 px constants) — and M7's daimon decision, which three
> milestones name and the manifest declares nowhere.
> ⚠ **The 0.8.5 block below is one release stale but its reasoning is current.**

# Handoff — **0.8.5 cut: the 6.6.2 stack, the pointer reaches every surface, the sidebar knows where you are.**

> ⭐⭐ **2026-09-13, READ THIS BLOCK FOR THE CURRENT NUMBERS AND THE ONES BELOW IT FOR THE REASONING.**
> **0.8.5 IS CUT on operator direction** — `VERSION` = 0.8.5, CHANGELOG `[0.8.5]`, every gate and
> check four green — **and the commit, the tag and the push are the operator's.** Until
> `git ls-remote --tags` shows `0.8.5`, **0.8.4** (`7929ae1`, on the remote, CI green) is the last
> release. ⚠ The first half of 0.8.5 is already in as `8bdcbfe` ("left button work") with `VERSION`
> still 0.8.4; `git describe` answered `0.8.4-1-g8bdcbfe` when the cut was written. The pin is
> **6.6.2**, and the whole sibling stack sits on it.
>
> ⭐ **What 0.8.5 holds:**
> 1. **The seven dep tags re-pinned to the 6.6.2 siblings.** 0.8.4 shipped the exact divergence check
>    four exists for (`lib/` at dhancha 0.9.29 / rupa 0.1.7 / setu 0.8.9 through `path`, manifest at
>    the previous tags). Six bumps are header-only; **chitra 1.0.1 → 1.0.3 is real** — a decoder P-1
>    sweep headed by a SIGSEGV on the first PNG a memory-pressured process decodes (crab's shape).
>    `lib/sankoch.cyr` was at 6.6.1's leaf and is corrected. Check four at the cut: 7 / 0, lock 3 → 7
>    commit-pinned, both binaries **byte-identical** (host `fa588e69…` 1,049,480 · agnos
>    `3438489f…` 1,090,216).
> 2. **The pointer routes** (M6): `crab_pointer_action` in `src/ui.cyr` names ONE arm per press in a
>    pinned z-order — popup, bar, strip, sidebar, panes — and every arm consumes. Right press on a
>    pane → focus, select the row under the point, open the context menu there. Left press on a popup
>    row → the entry's accelerator is **synthesised** (`synth_u`) through the one binding table (the
>    `KEY` gate takes two roads in). Bar cell → open / switch / close the drop; A/B strip → focus
>    that pane; anything else while a popup or the bar is up → dismiss. `crab_menu_item_at` inverts
>    the separator shift; `crab_pointer_blocked` is split out of `crab_pointer_modal`. Buttons are
>    **aethersafha 0.16.24's numbering, `wire = kernel_bit + 1` — 1 left, 2 right, 3 middle, NOT
>    X11** — and 0.16.24 is **on the remote, CI green** (`041ac85`); crab's filing is closed.
>    ⛔⛆ **Two holes closed on the way**: the WHEEL had no modal guard (a scroll between `d` and `y`
>    moved the selection — 0.8.0's click hole one input kind over) and the sidebar arm never consumed
>    the double-click pair. A right/middle release can no longer end a left drag.
> 3. **The sidebar's *you are here* marker.** `crab_sb_here`: containment, deepest wins, ties to the
>    lowest row (Root the place over `/` the volume); painted muted when unfocused, never when
>    focused without a cursor (`crab_sb_shown_row`'s 0.8.3 rule, unchanged). Under it, Phase 0's last
>    piece — **both model builders strip trailing slashes as they store** (`$HOME=/home/macro/` never
>    contained `/home/macro`) — and `crab_path_within` moved from `app.cyr` to `path.cyr` so the
>    render path could share the copy guard's containment rather than copy it. Beside it, a latent
>    layout fix: a 64-byte volume prefix's NUL sat at offset 80 = `BSIZE`, and `statfs` overwrote
>    it; the prefix field is 72 bytes, the record 112.
>
> **Suite 1790 / 0** (+95 over 0.8.4), render_test 53 / 0, fuzz 100k, fmt clean, coverage 88 %
> (265/298), vet/deny 0, deps --verify 50 / 0. Fourteen mutations across the release, each caught —
> one of them (the render passing -1 for `here`) through the 0.8.3 render assertion, which was
> updated to the marker's rule rather than deleted.
> ⛔ **NOT run on QEMU or iron, on either side.** crab's pointer arm and its `mountlist` path are
> agnos-only; six new oracle lines exist for the run: `crab: context menu opened by pointer` ·
> `menu pick by pointer` · `verb by pointer` (kept separate from `key press` so the harness's
> received-vs-acted ratio is undisturbed) · `popup dismissed by pointer` · `bar click` · `switcher
> click`; aethersafha prints `forwarded a non-left button press, wire number:` on the first one.
> ⚠ **Sequencing that run is the operator's call.**
>
> ⭐ **Next, ungated:** `View`'s items — the last of the six M6 interaction gaps (every target key is
> reachable since the 0.8.3 hoist; `Go` is refused as a drop-down with its reasons in the roadmap) —
> then the absent affordances: a REFRESH key, a flag surface for `--about`, the overwrite policy
> (an operator decision). Small and cheap: drop the redundant `net` stdlib declaration (its own
> change). ⚠ **The 0.8.3 block below is two releases stale but its reasoning is current**; the
> `Open` lesson — check the caller's POSITION, not just its logic — is the one to carry.

# Handoff — **0.8.3 in preparation: the M6 interaction gaps, and a menu verb that was dead.**

> ⭐⭐ **0.8.3, updated 2026-09-09. `Open` WAS DEAD ON BOTH MENU SURFACES, IN EVERY BUILD THAT SHIPPED
> EITHER.** The context menu and the menu bar both answer Enter by rewriting `u` to the chosen
> entry's key and falling through to the one implementation of that command — but both arms sat
> **below** the binding table, and `CRAB_MI_OPEN` rewrites to `0x28`, which is handled above them.
> `r`/`n`/`d`/`c`/`m` worked, and nothing made that true but their line numbers.
> ⇒ **The same shape as 0.8.2's dispatcher bug: a correct comment, wrong about where the handler
> actually sits.** Check the caller's POSITION, not just its logic.
>
> ⭐ **Closed in 0.8.3**: the PLACES sidebar's keyboard route (`Tab`/arrows/Enter — ⛔ with the
> mutating verbs EATEN, or `d` deletes from a pane the keys have left) and the menu bar's fit rule
> (⛔ plus a SECOND rule for drop-downs: there is a band of widths where the bar fits and `Edit`'s
> menu opens under the word `File`). Suite **1695 / 0**.
> ⛔ **Four M6 gaps remain**, with an adversarially-verified design recorded in the roadmap. The
> pointer routes must land TOGETHER, and the context-menu one is **gated upstream**.
> ⛔⛆ **THAT GATE IS NOW MEASURED, AND IT IS BIGGER THAN IT LOOKED.** aethersafha is the single point
> of button loss (the kernel, bhumi, setu and dhancha all carry the code intact) and the fix there is
> small — but **aethersafha does not build on any available toolchain**: its `6.5.33` pin is
> uninstallable and flagged critical, and under 6.6.0/6.6.1 it throws 57 errors, *none in its own
> source*, from `sigil`/`agnostik`/`agnodrm` disagreeing about `result_*` arities. All three are
> already at their highest tag. ⇒ **Three upstream repos need the 6.6.x language before this can be
> touched at all.** Filed at `docs/development/issues/2026-09-09-aethersafha-forwards-only-the-left-button.md`.
> ⚠ **aethersafha was left exactly as found** — no hand-edited `lib/`, no unverified push. Do not
> guess a button number either; no repo defines one and X11's order is the wrong default here.

# Handoff — **0.8.2: the audit backlog's correctness bugs, and a walk that never ran.**

> ⭐⭐ **READ THIS FIRST, BECAUSE IT IS THE TRANSFERABLE PART: RECURSIVE COPY AND RECURSIVE DELETE HAD
> NEVER RUN IN ANY SHIPPED BUILD, AND THE SUITE WAS GREEN THE WHOLE TIME.** `src/main.cyr`'s idle
> tick called `crab_copy_step` — the single-file chunk loop — instead of `crab_op_step`, the
> dispatcher. A walk's `CRAB_OP_FIN` is `-1` until a file is open, so `sys_read(-1, …)` failed and
> every `CTREE`/`DTREE` died on its FIRST tick with `EIO`. `d` on a folder took the `y` and deleted
> nothing. `crab_op_step` had **zero callers**, under its own comment reading *"THE single entry
> point the idle tick calls"*.
> ⛔⛆ **Every walk TEST drove `crab_op_step` — the right entry point — while the only caller that
> ships drove the wrong one.** ⇒ **A test that calls a different function than the shipping caller
> is not testing the shipping path.** Check the caller, not the function you wish it called.
>
> ⭐ **Updated 2026-09-09. 0.8.2 is IN PREPARATION** (`VERSION` = 0.8.2, nothing committed or
> tagged): the 6.6.1 pin, a documentation-currency repair, and **the audit backlog's eight
> correctness bugs — all closed, each mutation-proven**, plus the walk defect above. Suite
> **1571 / 0**. The roadmap's *Correctness* section is now empty; what remains in that audit is the
> M6 interaction gaps, the absent affordances, and the *Recorded as facts* list.
>
> ⭐ **Previously, 2026-09-08.** **0.8.1** (2026-09-07) is the last release and HEAD is its tag. It closed
> the VOLUMES gate on agnos **`mountlist`#104** — the blocker crab filed on 2026-09-02, which agnos
> answered by minting a new number rather than widening `mount`#11, crediting the filing by name —
> and it hardened the delete prompt after the `/bin` incident. **0.8.0** shipped M6.
> The toolchain pin has since moved **6.6.0 → 6.6.1** (2026-09-08, operator direction).
>
> ⛔⛆ **THE OPEN ITEM THAT OUTRANKS EVERY FEATURE**: five contiguous `/bin` entries were deleted on
> AGNOS iron on 2026-09-03 and **the mechanism is still unproven** —
> `docs/development/issues/2026-09-03-five-contiguous-bin-entries-deleted-on-agnos.md`. Both
> preventions shipped in 0.8.1 (the prompt names what dies; `/bin` says `SYSTEM DIR!` first), but a
> prevention is not a diagnosis, and **there is still no undo**. Re-verified 2026-09-08 at HEAD: all
> 16 `crab_relist` call sites clear marks within four lines, so the stale-marks hypotheses stay
> refuted; what remains is the batch sheet and ordinary operator error.
>
> ⚠ **The 0.7.7 notes below are kept because the SHAPE of those five defects is still the lesson** —
> a decision stated correctly in a comment and implemented wrongly three lines away — but 0.7.7 is
> three releases back now.
>
> ⛔⛔ **READ *Where things stand* AND NOTHING ABOVE IT FOR NUMBERS.** Everything below that table is
> kept because the REASONING is still the reasoning — but **every figure in it is 0.7.5's or
> older**, and several passages describe behaviour 0.7.7 changed. Re-derive before quoting.
>
> ⛔⛔ **THE FIVE DEFECTS, BECAUSE THE SHAPE MATTERS MORE THAN THE FIXES.** Every one was a
> *decision* stated correctly in a comment and implemented wrongly three lines away, or a rule
> written twice where only one copy was updated:
> - `crab_copy_begin` had **no directory guard**, so a folder handed to the single-entry copy
>   created a stray 0-byte file at the destination and then refused forever with `EEXIST`. On agnos
>   the stray is worse than empty: reading a directory yields raw dirent bytes.
> - `crab_queue_advance` **returned on the first refusal** and never tried the next entry — while
>   both `app.cyr` and `main.cyr` carried comments promising *"A REFUSAL DOES NOT STOP THE QUEUE"*.
>   Marking ten files with the first already present transferred **none** of the other nine.
> - **`m` on a folder ran `CRAB_OP_CTREE` — a COPY** — and set the notice `copying folder...`.
>   There is no `CRAB_OP_MTREE`; the operator was told the truth about what happened and the wrong
>   thing about what they asked for.
> - The marked set was read **only in the not-a-folder arm**, so a cursor resting on a folder
>   silently discarded every mark — three lines below a ⭐ comment reading *MARKS OUTRANK THE CURSOR*.
> - The context menu passed a **model** index to `dh_list_select` while a separator made the list one
>   row longer: *Delete* selected the inert separator and painted **no highlight at all**, and
>   *New Folder* highlighted **Delete**.
> - The tray's PROGRESS bar was **0 px tall whenever a rate was known** — the height rule was
>   written twice and only `crab_render`'s copy grew.
> ⇒ **A duplicated rule does not drift symmetrically; it drifts on whichever side someone
> remembered.** Four of the six now live as pure functions the suite can interrogate
> (`crab_transfer_plan`, `crab_menu_row`, `crab_tray_h`, `crab_fs_isdir`) rather than as shapes
> buried in an agnos-only key branch.
>
> ⛔⛔ **AND THE REASON NONE OF THEM WAS CAUGHT: 1,221 of `src/main.cyr`'s 1,494 lines sit inside a
> single `#ifdef CYRIUS_TARGET_AGNOS` with NO `#else`, and nothing includes `main.cyr`.** The whole
> key-dispatch table is uncompiled on the host and unreachable by every test crab has. `render_test`
> stayed green through the menu mutation too. That is why the decisions moved out of it.
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
> 1.56.53 → **1.56.57**. ⚠ **Sequencing verification is the operator's call and is not this file's to
> rank.**

## ✅ RELEASE ORDER HONOURED — rekha 0.3.6, dhancha 0.9.27 and crab 0.7.7 are all pushed (2026-09-02)

The proportional-text chain shipped in order: **rekha 0.3.6** (`hhea`/`hmtx` readers) →
**dhancha 0.9.27** (`dh_text_advance` consumes them) → **crab** (declares both floors, consumes
neither directly — it still passes `font = 0`).

⭐⭐ **CHECK FOUR RE-RUN AGAINST THE FULLY PUBLISHED GRAPH, 2026-09-02 — and this is the first time
since the work began that it proves anything.** A scratch copy with all four `path` lines disabled,
so `cyrius deps` really clones the declared tags:
**7 deps / 0 errors** · lock **7 commit-pinned** (3 with the overrides on, which is the tell they
were really off) · **1230 / 0** · and both binaries **BYTE-IDENTICAL** to the path-resolved ones —
host `61a2dec7…` **1,023,968 B**, `--agnos` `24db0039…` **1,051,928 B**.
*That equality is the evidence; everything else is merely consistent with it.*

⛔ **TWO HAZARDS THIS CHAIN TAUGHT, BOTH CHEAP AND BOTH EXPENSIVE TO RE-LEARN:**

1. **A POPULATED LOCAL DEP CACHE MASKS AN UNPUSHED TAG.** `~/.cyrius/deps/<dep>/<tag>/` persists, so
   once a temporary `path` override has resolved a version, a later `cyrius deps` finds it there and
   reports success — while a fresh CI runner with an empty cache fails. Mid-chain, dhancha reported
   `5 deps resolved, rc=0` against a rekha tag that was not on any remote. ⇒ **Only
   `git ls-remote --tags` answers whether a tag is published.** A local resolve does not.
2. **CHECK `git describe --tags` BEFORE WRITING INTO THE TOP CHANGELOG SECTION.** The dep-bump note
   for this work was drafted straight into a `## [0.7.7]` that was already tagged and pushed —
   `git describe` answered `0.7.7-1-g8ebe9d8`. **That is precisely how 0.7.2 came to exist**: three
   commits landed past 0.7.1's pushed tag while its section was still being edited, and the notes a
   consumer reads stopped describing the artifact they name. Caught and moved to `[Unreleased]`;
   `[0.7.7]` was restored byte-identical to the tag.

⚠ **crab's own 9 px assumption is NOT closed by any of this.** `CRAB_COL_CHARW = 9` and the five
constants derived from it are crab-side work, and they only matter the day crab stops passing
`font = 0`. The UPSTREAM half of the proportional-text gate is what moved.

## Cross-repo work, and where each repo stands

| repo | state |
|---|---|
| **dhancha 0.9.25** | ⭐ **RELEASED** (`2698ae1`, verified) and crab's tag moved to it. **0.9.25 adds the `GRID` kind** — wrapping layout, cell selection, row-wise arrows, keep-visible, hit-test; 66 checks, six mutations. 0.9.24 before it: Stable widget keys: dhancha identified widgets by *pointer*, which a per-frame arena invalidates — so focus, hover, press and drag were all unreachable from an immediate-mode app. Fixed at the cause. **Drag now works under a frame arena** (0.9.21 could only refuse it); `dh_text_attach` lets an app own the edit buffer. 16/16 suites, new `key_test` at 50 checks, six mutations. ⚠ **crab uses none of it yet** — adopting it deletes three hand-rolled workarounds and is its own change. |
| **chitra 1.0.1** | ⭐ **RELEASED** (`b777d34`, verified on the remote) and crab's tag moved to it. A valid PNG past sankoch's 16 MiB inflate ceiling used to spend **26,617,512 bytes** to return a bare `CHITRA_ERR_INFLATE`; it now refuses from the header in **under 64 KiB** with its own `CHITRA_ERR_INFLATE_LIMIT`. Its sidecar drops three unused stdlib leaves — **-4,176 B** in crab. 3,020 tests green, two mutations caught. ⚠ Changes no crab behaviour: the per-image budget already kept crab clear of that cliff. |
| **sankoch** | Issue filed: `DECOMPRESS_MAX_OUTPUT` is an absolute 16 MiB with **no caller override**, and the streaming API enforces it identically — so no entry point can inflate a ~5.6 MP RGB PNG. Only sankoch can change that. |
| **agnos** | Issue filed: `open`#7 has no `AO_EXCL`, so crab's overwrite guard exists on the host only. agnos's own `syscall.cyr:1138` already says so; crab is a second consumer. |
| **cyrius** | ⚠ **An issue I filed earlier this session was WITHDRAWN** — I claimed `cyrius distlib` copies `[deps].stdlib` into the sidecar. It does not; it derives it from `src/lib.cyr`'s includes, which is sound. The symptom was real, the cause I named was not. Corrected in chitra's own issue. |

## What landed in 0.7.6 (on top of 0.7.5) — ⛔ RELEASED, not pending

⛔⛔ **THIS SECTION SAID "`VERSION` is untouched, nothing is committed or tagged" AND POINTED AT A
`[Unreleased]` CHANGELOG SECTION. ALL OF IT WAS FALSE, AND IT IS THE LARGEST ROT THIS FILE HAS
CARRIED.** Corrected 2026-09-01 by measurement: `VERSION` is **0.7.6**, `HEAD` is `26f38ed`,
`git describe --tags --exact-match HEAD` answers **0.7.6**, that tag is **on the remote**, and
`CHANGELOG.md` had **no `[Unreleased]` section at all** — the work was folded into the released
`## [0.7.6]`. ⇒ A cold start read this table as pending work and would have re-done a shipped
release. **The file that warns hardest about staleness is not exempt from it.**
⚠ Post-0.7.6 work now accumulates in a real `[Unreleased]` section, added 2026-09-01.
Full accounting in [`../../CHANGELOG.md`](../../CHANGELOG.md).

| | |
|---|---|
| `crab_render` | **32 positional parameters → one record.** Filled by `crab_rs_pane` (11 params, indexed by pane), `_op`, `_chrome`, `_preview`, `_dims`. ⛔ The point is not brevity: at 32 `i64` arguments across 23 call sites, a miscounted comma shifted everything after it and still compiled. And `crab_rs_reset` now owns the three **`-1` = cannot be said yet** defaults that every one of those sites used to spell by hand — `0` there would make the tray render a real `0 B/s`. |
| Preview column | `p` toggles it. NAME · KIND · SIZE · MODIFIED · DIMENSIONS. ⛔ Width rule **derived** from crab's own column rule (*it may not cost a pane its SIZE column* → 303 px), not lifted from the canvas. Refuses out loud below that, via `crab_set_notice`. |
| Security audit | [`docs/audit/2026-08-31-audit.md`](../audit/2026-08-31-audit.md), crab's first — a v1.0 criterion. **All four findings closed in 0.7.6.** ⛔ F1 was a TRUST-MODEL change: gallery view decoded every image in a folder merely opened (measured 8 decodes vs 1 for a selection), running ~22,500 lines of `chitra`+`sankoch` on attacker-chosen bytes. Now it parses only what is on screen. ⛔ F2's fix is the ORDER: spawn first, read the magic only to explain a failure — there is no check left to race, and agnos has no fd-spawn to close it any other way. ⛔⛔ **F3's first draft was a FALSE finding and is kept with its correction** — an audit reporting a bug that is not there spends the reader's trust. |
| Gallery view | `g` cycles list → grid → gallery → columns (0.8.8). ⛔ **The view never triggers a decode — the idle tick does, one per tick**, so opening a gallery of a thousand files costs one frame. Stops three ways: the walk ends, refusals are cached, the budget refuses once spent. Backed by a 64-slot ~1.07 MB allocate-once cache holding results AND refusals. ⛔ The cache lives in `ui.cyr` because the render path looks one up per cell — **fifth time that rule decided a placement**. |
| Grid view | `g` toggles both panes onto dhancha 0.9.25's `GRID`. ⛔ Cell size DERIVED from the NAME column's floor, so the view changes only how many entries fit. No column header (a grid shows only names). ⛔ **Arrows navigate in grid mode; `h`/`l` still switch panes** — list mode unchanged. ⚠ **Not a gallery**: 40 thumbnail cells is ~28 MB of permanent decode against a 32 MB ceiling. |
| EXIF | CAMERA and SHOT, both byte orders, verified against an independent parser. ⛔⛔ **The most attacker-controlled parser crab has** — byte order, entry count and value offsets are ALL chosen by the file. Sub-IFD followed exactly once, never recursively. ⚠ Its fuzz round was **vacuous at first**: an out-of-bounds read does not crash on a bump allocator over a large mapped heap, so four planted bounds bugs survived. A printable poison tail fixed it — and the mutator had to be stopped from writing the poison byte itself. |
| Thumbnails | 64x64, PNG/JPEG/GIF/BMP, **decoded off the idle tick** — at most one per tick, because chitra's entry point is a single call that cannot be resumed the way `crab_copy_step` can. Memoised on the full path **including a remembered refusal**; a closed preview decodes nothing. ⛔ Four differently-named nothings, because "too large" is a property of the FILE, "budget spent" of the SESSION, and "cannot decode" of this BUILD. |
| Image dimensions | `crab_img_dims` — PNG/JPEG/GIF/BMP from header bytes, **no decoder, no dependency**. Verified against real files against `identify`: 137×42, 1×1, 4096×2160 PNGs, a 91×33 GIF, a 65×17 BMP and its top-down twin, and a real 384×288 JPEG whose SOF sits past an APP0 block. |
| Fuzz harness | ✅ * CLOSED.** **100,000** deterministic rounds — ⛔ *the figure was 60,000 in six documents and wrong in all six; the harness now COUNTS and prints its own rounds (`fuzz: rounds 100000`), because a harness whose output cannot contradict a stale claim about it is one nobody can check.* It caught a real segfault in `crab_img_dims` the day it was written. |
| Leak fixed | ⛔ `crab_overlay` used `alloc(32)` not `dh_falloc(32)` on both the menu and sheet branches — **32 B per frame, shipped in 0.7.5**, invisible because the zero-allocation gate never opened an overlay. |
| Numbers | tests **757 → 1,138/0** · `render_test` **26 → 53** checks · coverage **87 %** (195/223) · source **5,368 → 7,915** lines · host **1,019,784 B** · `--agnos` **1,047,624 B** · fmt clean · both targets build |

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
 stopped being a tidiness item: a swapped pair of pane blocks, and a preview column
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
| Version | ⭐ **0.8.2 IS IN PREPARATION** (`VERSION` = 0.8.2, dated 2026-09-09, on operator direction) — the 6.6.1 pin, the doc-currency repair, and the audit backlog's eight correctness bugs. ⛔ **Nothing is committed, tagged or pushed; the operator handles every git operation.** **0.8.1 is the last RELEASE** (2026-09-07). ⛔⛆ **THIS ROW ONCE SAID *"0.8.0 IN PREPARATION / 0.7.7 IS THE LAST RELEASE"* THROUGH TWO TAGGED RELEASES** — the rot `state.md` has now suffered three times, in the file whose own header says to read this table and nothing above it for numbers. **Re-derive from `git describe` before quoting this row.** Lineage: 0.8.1 closed the VOLUMES gate on agnos `mountlist`#104 and hardened the delete prompt after the `/bin` incident; 0.8.0 shipped M6. ⛔ **KEEP THIS LESSON: 0.7.2 exists only because 0.7.1's CHANGELOG section was still being edited after its tag was pushed** — a released section is a record, not a scratchpad. |
| Toolchain | cyrius pin **6.6.1** (moved 2026-09-08 on operator direction, from 6.6.0; trail 6.5.36 → 6.5.41 → 6.6.0 → 6.6.1). ⭐ **The bump is not cosmetic and it lands on crab's SHIPPING target**: 6.6.1 rebinds `chrono`'s AGNOS monotonic clock from `sys_uptime_ms` (#40, `timer_ticks`) to `sys_uptime_us` (#95, `rdtsc`). ⛔⛆ **A foreground `run` program on AGNOS executes with IF CLEARED** — only `/bin/agnsh` gets IF=1 — so the timer ISR never fires, `timer_ticks` never advances, and #40 reads **exactly zero forever, with no error**. crab is spawned by the compositor, so it is precisely that shape of program. ⚠ Only two vendored leaves moved: `lib/chrono.cyr` and `lib/sankoch.cyr` (2.7.11 → 2.7.14). ⛔ **`cyrius lib sync` walks only the DECLARED stdlib set**, so after any bump diff the WHOLE vendored tree against `~/.cyrius/versions/<pin>/lib` — verified by hash this time, which is how the 6.5.41 hand-copy was caught. |
| Build | x86_64 **1,023,968 B** · `--agnos` **1,047,832 B** at 0.7.7 · `--win` fails (pre-existing, not a regression — two absent syscall stubs, and Windows is not a declared target). ⭐ **Check four re-run in full at the new pin (2026-09-02)**, all four `path` lines disabled so `cyrius deps` really clones the tags: **7 deps / 0 errors**, lock **7 commit-pinned** (3 with the overrides on — that jump is the tell), **1230 / 0**, and both binaries **BYTE-IDENTICAL** to the path-resolved ones (host `5170a452…`, agnos `446b7f6a…`). *That equality is the evidence; everything else is merely consistent with it.* |
| Tests | `cyrius test` **1230 / 0** · `render_test` **53** checks, 0 failed · `cyrius fuzz` **100,000 rounds**, and it prints the number so it cannot agree with a stale claim about itself · bench measures the sort, not `bench_noop`. ⭐ **All four now run in CI** (*#14 / #36*), alongside the `--agnos` build, a per-file `fmt --check` loop, `coverage --min 85`, `vet` and `deny`. ⛔ Each of 0.7.7's five fixes is **mutation-proven** — the guard was removed and the suite watched to fail — because this project has shipped three tests that could not fail in their first draft. |
| Coverage | **199/227 fns (87 %)**, 6/6 files — the v1.0 criterion, met. ⭐ **And no longer met by hand**: `ci.yml` runs `cyrius coverage --min 85`, so a cut that drops below the floor fails before anyone remembers to look (verified the gate can fail: `--min 95` exits 1). ⛔ Still REFERENCE coverage — a floor, not a correctness proof, as the tool itself says. |
| Source | **8,092** lines across **six** files, plus 4,486 in `tests/`. ⚠ Re-derive with `wc -l src/*.cyr`; the per-file numbers that used to sit here went stale at two of the last three cuts, so they are deliberately not repeated. |
| Deps | **SEVEN**: sadish 0.5.3 · rupa 0.1.6 · **rekha 0.3.6** · kashi 1.0.6 · **dhancha 0.9.28** · setu 0.8.8 · chitra 1.0.1. Every build prints `7 deps resolved`. ⚠ *This row named rekha 0.3.5 and dhancha 0.9.26 while the manifest declared 0.3.6 and 0.9.28 — re-read it against `cyrius.cyml`, not from memory.* ⛔ **This row once also listed agnos, bhumi, sigil and aethersafha — none of which is a declared dependency.** agnos is the *kernel* crab runs on and the others are peers; misreading this row as the dep graph is how a tag check checks the wrong things. ⭐ **Check four re-run 2026-09-08 at the 6.6.1 pin**, in a scratch copy with all four `path` lines disabled so `cyrius deps` really clones the tags: **7 deps / 0 errors**, `deps --verify` **49/0**, lock **3 → 7 commit-pinned** (the tell the overrides were off), **1462 / 0**, and both binaries **byte-identical** to the path-resolved ones — host **1,036,944 B**, `--agnos` **1,068,976 B**. *That equality is the evidence.* |
| Mid-arc work | **M4 is complete** (0.7.1–0.7.5) and **M5 is substantially in** (0.7.6): the preview pane, header-only image dimensions, thumbnails, EXIF, and the GRID and GALLERY views. ⚠ **0.7.7 is a repair cut, not a feature cut** — no roadmap item advanced; five shipped defects were closed, the toolchain moved, and the CI gate stopped being one step. ⛔ **What M5 has left**: columns/miller (gated on crab's own two-pane model — a design question, not a dependency) and proportional text (gated on rekha `hmtx` advance widths, *not* on the plumbing, which exists — see the roadmap). |

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
reason** — took a row from one widget to 1 + `ncols`, so the frame chains **7 chunks** at the
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

2. **The cap went 256 → 1024 and re-broke the keystroke path that M3 existed to fix.**
   `crab_sort_entries` (`src/app.cyr:427`) is insertion sort — O(n²) with a 64-byte record swap done
   byte-at-a-time. **Measured on native x86_64** (agnos under QEMU is far slower):

   | n | random order | reverse-sorted |
   |---:|---:|---:|
   | 256 | 6 ms | 14 ms |
   | 1024 | **100 ms** | **200 ms** |

   It runs once per listing — i.e. on every descend/ascend keypress. treated ~280 ms there as
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

## M2 — started. is closed: the loop allocates nothing

**dhancha 0.9.16** (`68c60f8`, tagged and consumed). `dh_setu_poll_event` opened with `setu_msg_new()` — an
80-byte `alloc` — and only *then* asked whether a frame was pending. With no `free()` under it, an
idle desktop grew the heap once per wakeup forever, and a client repainting without input grew it at
the repaint rate: ~4.8 KB/s at 60 Hz.

The message is pure scratch — `setu_client_poll_input` fills it and `dh_setu_map_input` reads it to
build a **separate** `DhEvent` — so it is now one hoisted buffer per process, handed out zeroed.

⭐ **This was the last unbounded per-cycle allocation.** 0.9.13–0.9.15 took a rendered frame to zero;
this is the other half. **crab's whole render/input loop now allocates nothing in steady state**, and
the "no continuously-repainting element" rule is lifted **outright** rather than moved — the idle
mascot line, M4's transfer tray and M7's index progress are unblocked.

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

⚠ It lives in `agnos/scripts/harness/` beside its siblings; (move crab's harnesses into
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
Re-run all four at every cut; automating it is still open — see the roadmap's 0.8.0 batch.

### ⭐ The repaint rule is LIFTED — and immediately replaced

"Do not add a continuously-repainting element" stood for three releases and was right at 45 MB/s. The
frame is free now, so the idle mascot line, M4's transfer tray and M7's index
progress are no longer blocked by it.

⛔ **But `dh_setu_poll_event` still calls `setu_msg_new()` BEFORE it knows whether anything is
pending** — ~80 B per poll, on the global heap, never reclaimed. Continuous repaint implies continuous
polling, so at 60 Hz that is ~4.8 KB/s of permanent growth. Four orders of magnitude better than what
it replaces, and still unbounded. **Closing (M2, gate: dhancha) is now the precondition
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
harness is deferral **; `tests/crab.bcyr` still times `bench_noop`.

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
   - ⚠ This is roadmap deferral ** ("bring crab's agnos/iron harness into crab's own repo")
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

**Done and QEMU-proven**: sorting, selection memory, argv start paths, and
deferred statting. Details and the mutation evidence are in [`roadmap.md`](roadmap.md).

⭐ **The one finding worth carrying**: was **measured before it was built**, and that mattered.
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
> ⛔ **The repaint constraint has MOVED, not vanished. M2's own now carries it.**
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
  mission statement; the identity line and `## Goal` are real.
- ✅ **CLOSED 2026-08-26 — `README.md` § Status**, which had opened **"Scaffold."** and listed the
  dual-pane GUI under *Planned scope* since 0.2.0 (2026-07-10). It now says what works, what does not,
  and defers every volatile number to `state.md`. The retired `anu` codename and the rekha-TrueType
  claim went in the same pass.
  ⚠ **The replacement text over-claimed once before it landed, and was caught by reading `crab_row`
  rather than by a gate**: it said the panes had "size and modified columns". They do not — a row is
  one LABEL (13-char name, `~` on truncation, then `/` or a size) and the mtime lives only in the
  status line. ⇒ **Rewriting a stale claim is exactly when a new one gets introduced**; check the
  replacement against the code, not against the old text.
- CI runs only build + `cyrius test`. It **never builds `--agnos`** (the real target), never runs
  `render_test.cyr`'s ten pixel assertions, and runs no fuzz/bench/lint/fmt/vet/deny/coverage.
- The fuzz harness reads **none** of its input; the bench harness times an empty function. Both are
  scaffolds, so both are green against anything.
