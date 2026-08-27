# Handoff — 0.5.0 is shipped. **dhancha 0.9.13 is published and consumed; crab's side is uncommitted.**

> **Written 2026-08-26, at 0.5.0. Updated 2026-08-26** after the first cut at the dhancha per-frame
> allocation gate, and again once dhancha 0.9.13 was published. Read this, then [`CLAUDE.md`](../../CLAUDE.md), then
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
| Version | **0.5.0**, committed as `cbac9d8 repair release`. ⚠ Working tree is **NOT** clean — `CLAUDE.md`, `cyrius.cyml`, `cyrius.lock`, `lib/dhancha.cyr` and four docs carry the gate work below. |
| Toolchain | cyrius pin **6.5.35**, installed `cycc` 6.5.35 — no drift |
| Build | x86_64 **381,536 B** · `--agnos` **381,592 B** · `--win` fails (pre-existing, not crab's) |
| Tests | `cyrius tests` **37 / 0** · fuzz PASS · bench PASS · vet 0 untrusted · deny 0 · fmt clean |
| Coverage | **14/26 fns (53 %)** — reference coverage, a floor not a proof |
| Source | 941 lines: `main.cyr` 394 · `ui.cyr` 309 · `render_test.cyr` 134 · `path.cyr` 91 · `test.cyr` 13 |
| Deps | sadish 0.5.2 · rupa 0.1.4 · rekha 0.3.5 · kashi 1.0.6 · **dhancha 0.9.13** · setu 0.8.7 — all six at their latest published tag |
| Mid-arc work | ⚠ **Step 1 of the allocation gate is done and consumed** — see below. dhancha 0.9.13 is committed (`c273159`) and tagged; **crab's own working tree is not committed**. Next slot is still **M2 (v0.6.0)**. |

---

## Step 1 of the allocation gate — done, measured, and consumed

**dhancha 0.9.13** (`c273159`, tagged and on the remote): `dh_surface_new` no longer allocates a
`w*h*4` pixel buffer that nothing in the stack ever wrote or read.

| | per `crab_render` frame |
|---|---:|
| before (dhancha 0.9.12) | **746,440 B** |
| after (dhancha 0.9.13) | **412,040 B** |
| saved | **334,400 B — 44.8 %** |

Measured with a host probe taking `alloc_used()` deltas around five back-to-back `crab_render` calls
at 380×220 with 114 entries per pane. `dh_surface_new(380,220)` itself: 334,440 B → **40 B**.

**Verified**: dhancha 10/10 `programs/*_test.cyr`, crab `cyrius test` 37/0, `src/render_test.cyr`
0 failed checks, host and `--agnos` both build at unchanged sizes (381,536 / 381,592 B).

⚠ **It was NOT the "one line" three documents called it.** Storing 0 at `DH_S_PIXELS` breaks dhancha's
`programs/event_test.cyr` S): `dh_surface_present` returns `_NO_SURFACE` when pixels are 0 and
`_UNSUPPORTED` otherwise, so a permanently-zero field silently downgrades 0.9.5's "refuse loudly and
diagnosably" contract into "bad surface". The shipped fix **defers** the allocation into
`dh_surface_pixels` (allocate on first call, cache), so any external holder of the published accessor
still gets an owned `w*h*4` buffer. Mutation-verified both ways: the naive variant exits 1.

⭐ **The `path`-vs-`tag` check was done properly for once, and the fourth step is the one that matters.**
`[deps.dhancha]` carries both `path` and `tag`, and `path` wins — so before moving the tag: sibling
`VERSION` = 0.9.13; `git rev-parse 0.9.13` == HEAD, tree clean; `git ls-remote --tags` shows the tag at
that commit; **and `path` was temporarily disabled so `cyrius deps` actually cloned the tag** — the
resulting `lib/dhancha.cyr` had the identical SHA-256 (`0972d27b…`) as the `path` build. ⚠ Only that
fourth step would have caught 0.4.13's manifest naming a library the build never compiled; the first
three would all have passed it. **Deferral #19 (automate this) is still open** — this is the recipe.

⚠ **`lib/` is tracked**, so `cyrius deps` re-vendored the new bundle into `lib/dhancha.cyr` (+43/−8)
and `cyrius.lock` records its new hash. That is the tool's output, not a hand-edit of `lib/`.

### ⚠ The gate is NARROWED, NOT CLOSED — the M2-onward ordering does not move

1. **`dh_surface_render`'s own `sd_surface_new` — 334,432 B, now 81 % of the frame.** ⛔ **This one is
   NOT purely upstream**, contrary to what `docs/architecture/001` said before today. A reused render
   target has to outlive the frame, and `crab_render` builds a **fresh `DhSurface` every call**
   (`src/ui.cyr`), so a dhancha-side cache keyed on the surface would save crab nothing. Closing it
   needs a matching crab change, and it changes `dh_surface_render`'s contract from *returns a fresh
   surface* to *may return the same surface twice* — which `src/render_test.cyr` (two renders, dumps
   the first) can observe. **Operator decision, not a drive-by.**
2. **The widget tree, ~58 KB/frame** (~236 records @ 248 B) — needs dhancha to take an arena hook.
3. **crab's own scratch, ~19 KB/frame** — local, and under 5 % of the problem.

⚠ **Do not add a continuously-repainting element yet.** 412,040 B at 60 Hz is still 24 MB/s into an
allocator with no `free()`. The rule was never about the factor.

⚠ **The 412,040 B figure is not re-derivable by anyone else.** The probe that produced it was a scratch
harness, not a file in this repo. Its natural home is `tests/crab.bcyr`, which currently times
`bench_noop` — that is roadmap deferral #13, and it now has a concrete first job.

⚠ **Noticed in passing, NOT fixed** (out of scope, and nobody asked): `../dhancha/cyrius.cyml` pins
`cyrius = "6.5.27"` while the installed `cycc` is **6.5.35**. dhancha's CI installs the toolchain from
that pin, so its CI and every local dhancha build are on different compilers — the same drift crab's
`state.md` documents for itself. `cyrius distlib` prints the warning and carries on.

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

> ⛔ **The dhancha per-frame-allocation gate is HALF-CLOSED and still blocks every milestone after
> M2.** Step 1 is done and consumed — see the section above: 746,440 → 412,040 B/frame. ⚠ It was **not**
> the "one line" this file called it: storing 0 breaks dhancha's `event_test` by downgrading
> `dh_surface_present`'s refusal code, so the fix defers the allocation into `dh_surface_pixels`
> instead (mutation-verified both ways). **What is left is bigger than what was fixed** — step 2 is
> 81 % of the remaining frame and needs a change on *both* sides. Full measurement, breakdown and the
> corrected three-step fix in
> [`../architecture/001-every-frame-allocates-and-nothing-is-freed.md`](../architecture/001-every-frame-allocates-and-nothing-is-freed.md).
>
> ⇒ **Do not add a continuously-repainting element** — the idle mascot line, transfer progress, index
> progress — until it closes. At 60 Hz this is still **24 MB/s** after step 1. It will look fine in
> QEMU and exhaust memory on iron.

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

- `CLAUDE.md` still carries two `cyrius init` placeholders — the identity line (`**crab** — crab —
  TODO`) and the whole `## Goal` section. **It needs the operator's mission statement**, which is why
  0.5.0 left it.
- `README.md` § Status still opens **"Scaffold."** and lists the dual-pane GUI under *Planned scope* —
  stale since 0.2.0 (2026-07-10). It also names the retired `anu` codename and claims rekha TrueType
  text crab does not render.
- CI runs only build + `cyrius test`. It **never builds `--agnos`** (the real target), never runs
  `render_test.cyr`'s ten pixel assertions, and runs no fuzz/bench/lint/fmt/vet/deny/coverage.
- The fuzz harness reads **none** of its input; the bench harness times an empty function. Both are
  scaffolds, so both are green against anything.
