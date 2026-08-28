# Handoff — **0.6.1 is cut: M2 shipped.** The window answers pointer, wheel, releases and repeat.

> **Written 2026-08-26 at 0.5.0; rewritten 2026-08-27 at 0.6.0, then updated as M2 opened.** Read this, then [`CLAUDE.md`](../../CLAUDE.md), then
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
| Version | **0.6.1** (2026-08-27). ⚠ Uncommitted. Every dependency is released and current. |
| Toolchain | cyrius pin **6.5.35**, installed `cycc` 6.5.35 — no drift |
| Build | x86_64 **385,816 B** · `--agnos` **390,184 B** · `--win` fails (pre-existing) |
| Tests | `cyrius test` **134 / 0** · `render_test` 11 checks, 0 failed · fuzz + bench still scaffolds |
| Coverage | **27/36 fns (75 %)**, 6/6 files — v1.0 wants 80 % |
| Source | 1,512 lines across **six** files: `main.cyr` 453 · `ui.cyr` 442 · `app.cyr` 368 · `render_test.cyr` 145 · `path.cyr` 91 · `test.cyr` 13 |
| Deps | agnos 1.56.49 · bhumi 1.4.3 · setu 0.8.8 · sigil 3.12.12 · dhancha 0.9.18 · aethersafha 0.16.22 · sadish 0.5.2 · rupa 0.1.4 · rekha 0.3.5 · kashi 1.0.6 — all released |
| Mid-arc work | **None.** M2 shipped as 0.6.1. The two open M2 items are **gated upstream** (dhancha keysym space; a GPU carveout), not unfinished. |

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

## M2 — started. *Deferral #09* is closed: the loop allocates nothing

**dhancha 0.9.16** (`68c60f8`, tagged and consumed). `dh_setu_poll_event` opened with `setu_msg_new()` — an
80-byte `alloc` — and only *then* asked whether a frame was pending. With no `free()` under it, an
idle desktop grew the heap once per wakeup forever, and a client repainting without input grew it at
the repaint rate: ~4.8 KB/s at 60 Hz.

The message is pure scratch — `setu_client_poll_input` fills it and `dh_setu_map_input` reads it to
build a **separate** `DhEvent` — so it is now one hoisted buffer per process, handed out zeroed.

⭐ **This was the last unbounded per-cycle allocation.** 0.9.13–0.9.15 took a rendered frame to zero;
this is the other half. **crab's whole render/input loop now allocates nothing in steady state**, and
the "no continuously-repainting element" rule is lifted **outright** rather than moved — the idle
mascot line (*deferral #29*), M4's transfer tray and M7's index progress are unblocked.

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

⚠ It lives in `agnos/scripts/harness/` beside its siblings; *deferral #16* (move crab's harnesses into
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

## ✅ And `src/main.cyr` is testable## ✅ And `src/main.cyr` is testable## ✅ And `src/main.cyr` is testable — the gap that hid two of the above

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
Re-run all four at every cut; automating it is deferral **#19**, still open.

### ⭐ The repaint rule is LIFTED — and immediately replaced

"Do not add a continuously-repainting element" stood for three releases and was right at 45 MB/s. The
frame is free now, so the idle mascot line (*deferral #29*), M4's transfer tray and M7's index
progress are no longer blocked by it.

⛔ **But `dh_setu_poll_event` still calls `setu_msg_new()` BEFORE it knows whether anything is
pending** — ~80 B per poll, on the global heap, never reclaimed. Continuous repaint implies continuous
polling, so at 60 Hz that is ~4.8 KB/s of permanent growth. Four orders of magnitude better than what
it replaces, and still unbounded. **Closing *deferral #09* (M2, gate: dhancha) is now the precondition
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
harness is deferral **#13**; `tests/crab.bcyr` still times `bench_noop`.

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

## Next slot — M2 (v0.6.1), with its gate already closed

[`roadmap.md`](roadmap.md) sequences eight milestones to 1.0. Next is **M2 — the window is real**:
resize (`WINDOW_CONFIGURE` is decoded by dhancha and dropped on the floor), pointer input (dhancha
synthesizes eight kinds; crab consumes none), key release, and routing through `dh_dispatch`.

> ✅ **The dhancha per-frame-allocation gate is CLOSED and no longer blocks anything** — see the
> section above: **746,440 → 0 B per steady-state frame**, at 114 and at 256 entries per pane. Full
> accounting in
> [`../architecture/001-every-frame-allocates-and-nothing-is-freed.md`](../architecture/001-every-frame-allocates-and-nothing-is-freed.md).
>
> ⛔ **The repaint constraint has MOVED, not vanished. M2's own *deferral #09* now carries it.**
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
