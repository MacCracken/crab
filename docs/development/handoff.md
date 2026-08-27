# Handoff — 0.5.0 is shipped. **The allocation gate is 89.6 % closed; dhancha 0.9.14 is uncommitted.**

> **Written 2026-08-26, at 0.5.0. Updated 2026-08-26** after the first cut at the dhancha per-frame
> allocation gate, again once dhancha 0.9.13 was published, and again after step 2 landed. Read this, then [`CLAUDE.md`](../../CLAUDE.md), then
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
| Version | **0.5.0** at `19c6e8c docs`. ⚠ Working trees are **NOT** clean in **either repo** — crab carries step 2 plus the README pass; `../dhancha` carries an untagged 0.9.14. |
| Toolchain | cyrius pin **6.5.35**, installed `cycc` 6.5.35 — no drift |
| Build | x86_64 **381,568 B** · `--agnos` **381,632 B** · `--win` fails (pre-existing, not crab's) |
| Tests | `cyrius test` **45 / 0** · `render_test` 11 checks, 0 failed · fuzz PASS · bench PASS (both scaffolds) · fmt clean |
| Coverage | **14/26 fns (53 %)** — reference coverage, a floor not a proof |
| Source | 982 lines: `main.cyr` 407 · `ui.cyr` 326 · `render_test.cyr` 145 · `path.cyr` 91 · `test.cyr` 13 |
| Deps | sadish 0.5.2 · rupa 0.1.4 · rekha 0.3.5 · kashi 1.0.6 · **dhancha 0.9.13 declared, 0.9.14 compiled (unpublished)** · setu 0.8.7 |
| Mid-arc work | ⛔ **YES.** Steps 1 and 2 of the allocation gate are done and measured; **dhancha 0.9.14 is uncommitted and untagged**, and crab compiles against it via `path`. Next slot is still **M2 (v0.6.0)**. |

---

## ⛔ The allocation gate — steps 1 and 2 done, step 3 open, and half of it is uncommitted

| | per `crab_render` frame |
|---|---:|
| baseline (dhancha 0.9.12) | **746,440 B** |
| + step 1 (dhancha 0.9.13, published `c273159`) | **412,040 B** |
| + step 2 (dhancha **0.9.14, UNCOMMITTED** + crab) | **77,568 B** |
| total | **−89.6 %** |

Measured with a host probe taking `alloc_used()` deltas around five back-to-back `crab_render` calls
at 380×220 with 114 entries per pane, into **one** surface — the lifetime `src/main.cyr` now uses.

**Step 1** — `dh_surface_new` no longer allocates a `w*h*4` pixel buffer that nothing in the stack
ever wrote or read; the allocation is deferred into `dh_surface_pixels`. ⚠ Not "one line": storing 0
breaks dhancha's `event_test` by downgrading `dh_surface_present`'s `_UNSUPPORTED` refusal to
`_NO_SURFACE`. Mutation-verified.

**Step 2** — `dh_surface_render` caches its sadish render target on the `DhSurface` (`DH_S_SDS` at
+40; the struct grew 40 → 48 B) and reuses it. ⛔ **This half was NOT purely upstream**, contrary to
what two documents said: `crab_render` was minting a fresh `DhSurface` **every call**, so a
per-DhSurface cache would have been thrown away each frame and saved crab nothing. crab now creates
**one surface for the session** in `src/main.cyr` and passes it in; `crab_render` reads `w`/`h` back
off the surface, so the signature went **18 → 17 parameters** for gaining one, and `w`/`h` can no
longer disagree with the surface they describe.

⚠ **Step 2 IS A CONTRACT CHANGE.** `dh_surface_render` used to return a fresh surface every call and
now returns the same one. A caller wanting two frames at once needs two `DhSurface`s.
`src/render_test.cyr` is exactly that caller — it asserts on frame 1, renders frame 2, then dumps
frame 1 — so it now creates two, guarded by `check(sds2 == sds, 0)`.

**Verified**: dhancha 10/10 `programs/*_test.cyr` (lint + fmt clean across `src/` and `programs/`),
crab `cyrius test` **45/0**, `render_test` 0 failed checks, host **381,568 B** and `--agnos`
**381,632 B**. No new lines over 120 chars (still 9 across `src/`, unchanged from HEAD).

### ⛔ Two things must happen together, and one is a trap

1. **Commit and tag dhancha 0.9.14**, then bump crab's `[deps.dhancha] tag` to `0.9.14`.
   `path` wins over `tag`, so **the local build already compiles 0.9.14 while the manifest declares
   0.9.13**. The tag is left behind on purpose — an unpublished tag fails CI loudly, whereas the
   alternative is CI silently resolving a different library than the local tree. There is a ⛔ block
   at the tag in `cyrius.cyml`.
   ⭐ When the tag moves, re-run the **four-way** check: sibling `VERSION`; `git rev-parse <tag>` ==
   HEAD; `git ls-remote --tags`; **and disable `path` so `cyrius deps` actually resolves the tag**,
   comparing `lib/dhancha.cyr`'s SHA-256. Only the fourth would have caught 0.4.13's manifest naming
   a library the build never compiled. Deferral #19 (automate it) is still open; this is the recipe.
2. ⛔ **`cyrius distlib` MUST be followed by `sh scripts/sync-deps-sidecar.sh` in dhancha.** distlib
   rewrites `dist/dhancha.deps` and re-adds `kashi_font_data`, which is VENDORED, not stdlib — every
   consumer then hard-fails `cyrius deps` with *"dep dhancha requires 'kashi_font_data' but it is not
   in the cyrius stdlib"*. I hit this mid-session by running distlib alone. dhancha's CI gates it;
   a local run does not.

### ⚠ What is left — step 3, and it is now most of the problem

- **The widget tree: ~236 records @ 248 B = 58,528 B, now 75 % of the frame.** Needs an arena hook in
  dhancha — the stdlib ships `arena_new`/`arena_reset` documented for exactly this "per-frame reuse"
  pattern, but dhancha allocates via plain `alloc()` at 19 sites, so crab cannot redirect it.
- **crab's own row/status scratch, ~19 KB** — local, and was under 5 % of the problem when the pixel
  buffers dwarfed it. It is now roughly a quarter.
- ⚠ **The repaint rule STILL STANDS.** 77,568 B at 60 Hz is 4.7 MB/s into an allocator with no
  `free()`; the rule was never about the factor. ⭐ The margin has changed though — an element
  repainting a few times a minute is now clearly affordable, where at 750 KB a frame it was not.

### ⚠ Two test findings worth carrying forward

1. **A residue check over an opaque full-surface tree cannot fail, and mutation testing is the only
   thing that says so.** The first draft of both the dhancha and the crab reuse tests rendered trees
   that repainted every pixel, so deleting `sd_clear` outright left both suites green. dhancha's
   `draw_test` now uses a root that paints nothing (`bg = -1`) above a short child; crab's suite
   scribbles a sentinel over the whole buffer instead.
2. ⛔ **crab's sentinel check has TWO independent guarantors, so no single mutation fails it** —
   deleting dhancha's `sd_clear` leaves it green (crab's opaque root still covers) and making crab's
   root transparent leaves it green (the clear still covers); **only removing both fails**, at 7,744
   surviving bytes. That is correct for a property test, but it means **a green crab suite is not
   evidence that the toolkit still clears.** That is pinned in dhancha's own `draw_test`.

⚠ **Owed at the 0.6.0 cut**: crab's CHANGELOG has no entry for any of this. `VERSION` is still 0.5.0
and the work is mid-arc, so per CLAUDE.md's Process the CHANGELOG entry and version sync land with the
release, not now. Do not let that be the reason it goes unrecorded.

⚠ **The 77,568 B figure is still not re-derivable by anyone else.** The probe is a scratch harness,
not a file in this repo. Its natural home is `tests/crab.bcyr`, which times `bench_noop` — roadmap
deferral #13, with a concrete first job.

⚠ **Noticed in passing, NOT fixed** (out of scope, and nobody asked): `../dhancha/cyrius.cyml` pins
`cyrius = "6.5.27"` while the installed `cycc` is **6.5.35**. dhancha's CI installs the toolchain from
that pin, so its CI and every local dhancha build are on different compilers — the same drift crab's
`state.md` documents for itself.

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

## Next slot — M2, and the gate that dominates it

[`roadmap.md`](roadmap.md) sequences eight milestones to 1.0. Next is **M2 — the window is real**:
resize (`WINDOW_CONFIGURE` is decoded by dhancha and dropped on the floor), pointer input (dhancha
synthesizes eight kinds; crab consumes none), key release, and routing through `dh_dispatch`.

> ⚠ **The dhancha per-frame-allocation gate is 89.6 % CLOSED and still blocks every milestone after
> M2.** Steps 1 and 2 are done — see the section above: **746,440 → 77,568 B/frame**. ⚠ Neither was
> the "one line" this file called it, and step 2 was not purely upstream. **What is left is step 3,
> the widget tree, now 75 % of the frame**, and it needs an arena hook in dhancha that crab cannot
> supply. Full measurement, breakdown and the corrected three-step fix in
> [`../architecture/001-every-frame-allocates-and-nothing-is-freed.md`](../architecture/001-every-frame-allocates-and-nothing-is-freed.md).
>
> ⇒ **Do not add a continuously-repainting element** — the idle mascot line, transfer progress, index
> progress — until it closes. At 60 Hz this is still **4.7 MB/s**, into an allocator with no `free()`.
> It will look fine in QEMU and exhaust memory on iron. ⭐ But the margin has moved: something that
> repaints a few times a minute is now clearly affordable.

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
