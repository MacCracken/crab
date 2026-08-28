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

- **Does the second pane ever leave?** At 420px all three directions show one pane plus a Pane A/B
  switcher, so the canvas answers this in the affirmative — but it has not been ratified. Forced by M4.
- **Which accent roles does the compositor hand over?** rupa publishes `accent`, `active`, `held`,
  `faint` — but **no `on-accent`** (the ink colour to use *on* an accent fill). Until it does, a
  selected row cannot carry guaranteed-legible text. **Gate: rupa.** Forced by M3.

---

## v1.0 criteria

- [ ] All three canvas directions absorbed — shell (1a), density (1c), assisted search (1b as a mode)
- [ ] Public surface documented and tested; reference coverage ≥ 80 % — **70 % at 0.6.0**
- [ ] `docs/benchmarks.md` captured from a real bench harness, not the scaffold's `bench_noop`
- [ ] Green on iron, not only QEMU — the two defects crab has shipped were both iron-only
- [ ] CHANGELOG complete from v0.1.0; ADRs written for every ⛔ invariant now living in comments
- [ ] Security audit pass (`docs/audit/YYYY-MM-DD-audit.md`)
- [ ] `docs/examples/` populated

---

## Milestones

### M1 — Hardening (v0.5.0) — ✅ shipped

The P-1 sweep and its repairs. No new features; the point was to stop building on a floor with holes
in it. See the [CHANGELOG](../../CHANGELOG.md) for the full accounting.

- ✅ **P1** bounded path helpers — `crab_join`/`crab_strcpy` had no destination cap and overflowed
  `pathscr`/`lpath`/`rpath` into the *other pane's* buffers on ordinary Enter presses
- ✅ **P1** the event loop no longer ends itself after ~2 s of spinning
- ✅ **P1** the size ladder rounds, carries, reaches G/T, and cannot overflow
- ✅ `src/path.cyr` extracted so the suite can reach the path layer at all
- ✅ 11 → 37 assertions, mutation-proven; reference coverage 23 % → 53 %

### M1.5 — The allocation gate, and a floor under the tests (v0.6.0) — ✅ shipped

Not a planned milestone: it is M2's **gate** plus the test debt closing it exposed, and it took the
version number M2 had reserved. No user-visible features.

- ✅ **A rendered frame costs the global heap ZERO bytes** — 746,440 B → 0, across dhancha 0.9.13,
  0.9.14 and 0.9.15 plus the matching crab halves. See the gate note under M2 below.
- ✅ **`src/app.cyr` extracted from `src/main.cyr`**, which ends in `_entry();` and so could never be
  included by a test. The readdir parser, the stat layer, `crab_descend`, `crab_ascend` and the
  surface flag had **zero reachable coverage** until this. Finishes the extraction 0.5.0 began with
  `src/path.cyr`.
- ✅ **The frame-arena setup moved out of `main()`** rather than being tested around — `crab_render`
  owns it, so there is no step left for `main()` to forget.
- ✅ 37 → 75 assertions; reference coverage 53 % → 70 % against a v1.0 criterion of 80 %.

⚠ **M2 was v0.6.0; this release took it, and M2 moves to v0.6.1 — a PATCH, not a minor.** Operator
ruling 2026-08-27. The ladder from M3 onward is therefore **unchanged** (v0.7.0 … v1.0.0): only M2 was
displaced, and it was absorbed inside the 0.6 line rather than pushing everything down.
⛔ **Re-derive the number at each cut rather than trusting these headings.** The milestone→version
mapping has now been wrong once, and nothing gates it.

### M2 — The window is real (v0.6.1)

crab is a fixed 380×220 rectangle that only understands the keyboard. Everything in the canvas
assumes otherwise.

- **Resize** — ⚠ **BUILT; the REFUSAL path is QEMU-proven, the ADOPT path is not reachable here.**
  `WINDOW_CONFIGURE` is handled: `dh_surface_resize` (dhancha 0.9.17), the `#86` shm slot **created
  before the old one is closed**, `w`/`h`/`stride` as state, a re-ATTACH + COMMIT after the next
  render. Layout reflows for free — `crab_render` reads `w`/`h` off the surface. *Deferral #01, #04.*
  ⛔ **A new harness found a real bug on its first run.** `scripts/harness/crab-resize-test.py`
  (agnos) drives boot → F2 → **DOWN** → Enter → F5, and crab died with
  `buf_create failed on resize -- exiting`: the first draft closed its only buffer before knowing the
  replacement existed. setu's own `setu_client_present` closes first, but it has an inline-pixel
  fallback to land on; crab's hand-rolled LIVE-buffer path has none, so copying that order was fatal.
  ⛔ **And the byte cap was invented rather than derived.** It was 16 MB ("the framebuffer's size");
  agnos actually caps a `#71` pmm slot at **2 MB** and only a real GPU carveout (`#86`) reaches 32 MB,
  with `setu_buf_create` choosing between them at runtime. ⇒ The cap is now the absurdity bound and
  **the kernel is the arbiter**; crab survives a refusal by staying at its old extent.
  ⭐ **The harness is proven to go red**: with the `WINDOW_CONFIGURE` branch removed it reports FAIL
  (no CONFIGURE line at all), against PARTIAL for the real build — so "refused" and "ignored" are
  distinguishable, which is the only thing that makes the QEMU result meaningful.
  ⚠ **What is proven**: crab launches from the launcher, receives a real CONFIGURE for **2048x2018**,
  refuses it because QEMU has no carveout, stays at its old size, and **answers 6 keystrokes
  afterwards**. **What is not**: the adopt path, which needs a machine whose `#86` carveout can back
  the ask. Say so rather than reading the PARTIAL as a pass.

- **Pointer** — ✅ **click to select, click to focus a pane, double-click to descend.** *Deferral #05.*
  ⭐ **QEMU-proven**: `crab: click` on a real kernel, crab resolving a click to a pane and answering 5
  keystrokes afterwards. crab is the **first client to decode `SETU_INPUT_PTR_MOVE`** — aethersafha's
  own note says "no shipped client decodes PTR_MOVE yet", so this wire had never run end to end.
  ⛔ **crab OWNS THE INTERACTION STATE — it does NOT use `dh_dispatch`** (operator ruling 2026-08-27).
  `dh_dispatch` tracks a press by **widget pointer** (`_dh_press`), and crab renders after every
  handled event with `dh_frame_begin()` rewinding the arena and clearing exactly those pointers — so
  a press and its release are separated by a rebuild and the target no longer exists. crab tracks
  **pane index + row index**, which survive a rebuild because they are its own model. dhancha
  supplies geometry only, via `dh_hit_test` over the tree the last render built.
  ⚠ `SETU_INPUT_PTR_BTN` carries **no coordinates** — only button and state — so crab tracks position
  from `PTR_MOVE`. A client that ignores motion has nowhere to put a click.
  ⭐ **SCROLL WHEEL — BUILT ACROSS FIVE REPOS, and the chain was broken at the BOTTOM, not at setu.**
  The gate read "setu has no wheel message kind"; the real defect was that **agnos discarded the wheel
  byte**. `hid_process_mouse_report` read bytes [1] and [2] of the boot-mouse report and left byte [3]
  — documented in its own layout comment as `wheel (s8, optional)` — on the floor, and `#98 ptrscan`'s
  record had no field for it. bhumi had no scroll concept either. A setu patch alone would have been a
  fourth dead wire, after `SETU_CLOSE` and `SETU_CONFIGURE`.
  ⇒ **agnos 1.56.49** reads byte [3] and grows `#98`'s record to 20 bytes **opt-in per call** ·
  **bhumi 1.4.3** carries `BHUMI_EV_SCROLL` and accepts a 16-byte record from an older kernel ·
  **setu 0.8.8** defines `SETU_INPUT_PTR_SCROLL` (kind 12) · **dhancha 0.9.18** maps `POINTER_SCROLL`
  · **aethersafha 0.16.21** forwards it · **crab** moves the selection of the pane under the cursor.
  ⭐ **The wheel byte was QEMU-measured before any of the four layers above it were written**:
  `hid: wheel byte seen, b3=1 resid=0`. `scripts/harness/hid-wheel-test.py` injects the event over
  **QMP** (`input-send-event`) because HMP has no wheel verb.
  ⛔ **crab's wheel moves the SELECTION, not the view.** `crab_render` restores each pane's scroll
  offset and then calls `dh_list_scroll_to_sel`, so a free-scrolled view is snapped back on the next
  frame by the machinery keyboard navigation depends on. A detached view-scroll is a separate change.
  ⭐ **PROVEN TO THE COMPOSITOR; THE LAST HOP IS NOT.** QEMU, 2026-08-27: a QMP-injected wheel reaches
  aethersafha — **agnos → bhumi → compositor is end-to-end proven**. crab logs no scroll, and the
  compositor says why: `got a scroll with NO client window under the cursor`. QEMU's `usb-mouse` is
  RELATIVE, so a harness cannot place the cursor on crab's window; **crab's silence is correct
  behaviour, not a defect.** Closing it needs absolute pointing or reading the window placement.
  ⛔ **The desktop was BLOCKED by sigil, not by aethersafha.** `cyrius deps` prefers a sibling
  checkout, and sigil 3.12.11 added a pipe/fork/execve/waitpid helper with no target guard —
  `SYS_FCNTL` and `WNOHANG` are Linux-only, and Cyrius treats an undefined **variable** as a hard
  error regardless of reachability, so it failed the whole `--agnos` compile. ✅ **sigil 3.12.12**
  guards it and returns `ENOSYS` on agnos; aethersafha builds again.
  ⚠ **An earlier note here blamed a pre-existing `sys_execve` failure. That was wrong** — it was a
  symptom of a `lib/` I had churned, because `cyrius build` re-resolves deps and `git stash` does not
  isolate a tree whose `lib/` the build regenerates. A detached worktree at the same commit built fine.
  ⚠ **NOT DONE: focusing a pane by its header.** The header is a sibling of the list, not inside it,
  so `crab_hit`'s walk never reaches a LIST and the click does nothing. Reasonable to want; not
  claimed.
  ⚠ Row-level precision is asserted on the **host**, against the real rendered tree — the emulated
  mouse is `usb-mouse` (relative), so a harness cannot land on a chosen pixel and does not pretend to.
- **Key release** — ✅ **DONE.** crab requests `SETU_SURF_FULL_KEYS`; the compositor honours it per
  surface (`win_set_keymode` at CREATE_SURFACE, `mods` = 1 press / 0 release). *Deferral #06, first
  half.* ✅ **The setu gate is CLEARED** — the flag is implemented end to end, not just defined.
  ⭐ **QEMU-proven, exactly**: 6 keystrokes → **12 `crab: key received` / 6 `crab: key press`**. The
  first count proves releases are delivered; the second proves they are gated.
  ⛔ **ASKING FOR IT WITHOUT GATING MAKES EVERY KEY ACT TWICE**, and the compositor carries that burn:
  *"three F3 presses produced SIX `theme switched` lines, and the launcher moved its selection twice
  per keypress"* (2026-08-18). For crab it is two rows per Down and Enter descending twice.
  ⚠ The flag and the gate must move together: on a press-only surface `mods` is 0 for a **press**, so
  dropping the flag while keeping the gate would act on nothing. Both live in `src/app.cyr`.
- **Held-key repeat** — ✅ **DONE, and it needed NO kernel change.** *Deferral #06, second half.*
  ⛔⛔ **I CLAIMED THIS WAS BLOCKED ON agnos FOR WANT OF A BOUNDED WAIT. THAT WAS WRONG.** The claim
  came from reading syscall SIGNATURES — `#14 pause` takes no timeout argument, so it looked
  unbounded — and stopping there. Its own kernel comment says the opposite in as many words: *"Do ONE
  SAFE (IF=1) hlt so a device IRQ (e.g. NIC RX) **or the 100 Hz timer** wakes it."*
  ⭐ **MEASURED on a real kernel before building anything: `sys_pause()` returns in 0-4 ms.** It is
  already a bounded wait. An agnos 1.56.50 adding a timed-pause syscall would have duplicated
  behaviour that shipped long ago.
  ⇒ Repeat is a clock check in the idle path. bhumi does not repeat for us and says why —
  *"forwarding a repeat as another press delivers a key the user never pressed again. Repeat is
  policy, not drain."*
  ⛔ **ONLY MOVEMENT REPEATS.** Enter descends and Backspace ascends; repeating those walks the
  operator through the filesystem on one held key, re-readdir'ing and re-stat'ing every step.
  ⚠ Two gates, not one: a 400 ms delay so a deliberate tap is not a burst, then a 60 ms interval.
  ⭐ **QEMU-proven**: a QMP-held key (HMP `sendkey` cannot hold — it sends both edges) gives
  `hold window: 1 press, 0 releases` → repeat fires → **0 repeats after the release**, so the latch is
  cleared on the release edge rather than sticking.
  ⚠ **The observed RATE is far below the configured interval** — about 1 repeat in a 1.6 s hold where
  ~20 was expected. Not diagnosed. Each repeat re-renders and re-writes a 334 KB shm buffer, so the
  rate is plausibly bounded by frame cost under QEMU rather than by the interval, but that is a
  hypothesis and is recorded as one.

- **Route through `dh_dispatch`** — ⛔ **BLOCKED ON AN UPSTREAM TYPE CONFUSION, and adopting it today
  would be actively WRONG rather than merely inert.** *Deferral #07.*
  `dh_dispatch`'s only key-level action is Tab traversal, and it tests `dh_event_a(ev) == DH_KEY_TAB`
  where `DH_KEY_TAB = 9`. dhancha's own enum header says those values are *"ASCII / Unicode
  codepoints, matching the puka keymap's sym space"*. **crab's KEY events carry raw HID usages**, and
  HID usage 9 is **`f`** — so a crab routed through `dh_dispatch` would Tab-traverse whenever the
  operator typed `f`.
  ⛔ **The confusion spans three repos and one number.** aethersafha sends `bhumi_key_usage(ev)` — an
  HID usage. setu's protocol names the field **`keysym`**. dhancha maps it straight into `DhEvent.a`,
  whose constants are **codepoints**. Three names, one field, two meanings, and nothing translates.
  ⚠ puka already pays this toll explicitly — `setuwin__hid_to_evdev` exists precisely because HID
  usages arrive and its engine speaks evdev. dhancha simply never added the equivalent.
  ⇒ **Gate: dhancha** — either its setu mapper translates usage → codepoint, or `DH_KEY_*` moves into
  HID-usage space. Until one happens, crab's own keysym switch is not "around the toolkit", it is the
  only correct place for the mapping to live.
  ⚠ **And the rest of #07 was never going to move anyway**: `dh_dispatch` delivers to the focused
  widget via `dh_propagate`, and crab registers **no widget handlers** — its key semantics (descend,
  ascend, pane switch, selection) are app-level by the same ruling that governs pointer state. Tab was
  the whole prize, and Tab is what the mismatch takes away.

- ✅ **CLOSED 2026-08-27 (dhancha 0.9.16) — the idle leak.** `dh_setu_poll_event` called
  `setu_msg_new()` (80 B) **before** it knew whether anything was pending; the message is pure scratch,
  so it is now one hoisted buffer per process. **200 idle polls move the global heap by 0 bytes**,
  asserted in dhancha's `poll_test`. ⇒ With the frame already at zero, **crab's render/input loop
  allocates nothing in steady state**. *Deferral #09.*
  ⚠ An **event** still costs 56 B (`dh_event_new`) — per input, not per cycle. Only the idle path is
  free.
  ⛔ **Whatever replaces the idle wait must still not be `sys_sleep_ms`** — see the ⛔ below; that is
  unchanged by this. 0.5.0's stopgap is to wait on an interrupt when the
  poll is empty. **Gate: dhancha** — hoist the buffer or accept a caller-owned one. *Deferral #09.*
  ⛔ **Whatever replaces this wait, it must not be `sys_sleep_ms`.** That syscall `preempt_disable()`s,
  so while crab waits nothing else on the machine can be scheduled — a 0.5.0 draft used it and the
  compositor never presented at all (`presented: 0` against a baseline of 2). `sys_pause` (#14) yields
  to a ready proc first and only halts when nothing else is runnable. The host suite is green either
  way; **only a QEMU run distinguishes them**, so this line is a required QEMU gate, not a preference.

> ✅ **Gate: dhancha — per-frame allocation. CLOSED 2026-08-27.** A rendered frame now costs the
> global heap **zero bytes**, at 114 entries per pane and at the 256 `CRAB_MAX_ENTRIES` ceiling alike.
>
> | | per steady-state frame |
> |---|---:|
> | baseline (dhancha 0.9.12) | 746,440 B |
> | + step 1 (0.9.13) — `dh_surface_new`'s dead pixel buffer, deferred | 412,040 B |
> | + step 2 (0.9.14 + crab) — the sadish render target, reused | 77,568 B |
> | + step 3 (0.9.15 + crab) — the widget tree, arena'd | **0 B** |
>
> ⚠ Zero is per-frame, not total: a one-time **~597 KB** (render target + arena chunk) is allocated on
> the first frame and reused for the process's life. That fixed cost instead of a per-keypress one is
> the whole point.
> ⚠ **This file called steps 1 and 2 "one line" and "upstream". Both were wrong** — step 1's naive
> form breaks dhancha's `event_test`, and step 2 needed a matching crab change because `crab_render`
> was minting a `DhSurface` per frame. All three steps are mutation-verified on both sides.
> ⇒ **This gate no longer blocks the milestones after M2.**

> ⭐⭐ **AND THE REPAINT RULE IS LIFTED OUTRIGHT (2026-08-27).** "Do not add a continuously-repainting
> element" stood for three releases and was right at 45 MB/s. It had two halves and **both are now
> zero**: the frame (0.9.13–0.9.15) and the **poll** (0.9.16 — `dh_setu_poll_event` allocated an 80 B
> message before it knew whether anything was pending; it is now one hoisted scratch per process).
> **crab's render/input loop allocates nothing in steady state**, asserted at 200 idle polls moving
> the global heap by 0 bytes. ⇒ The idle mascot line (*deferral #29*), M4's transfer tray and M7's
> index progress are unblocked.
> ⚠ An **event** still costs 56 B (`dh_event_new`) — per input, not per cycle, and bounded by how fast
> a human types.

### M3 — A browser you would actually use (v0.7.0)

- **Sorting** by name / size / modified / kind, directories-first, dotfile handling — the pane
  currently renders raw readdir order, which is on-disk order. *Deferral #33.*
- **Real columns** — NAME · SIZE · MODIFIED with headers, replacing the 13-character name column.
  **Gate: dhancha** TABLE or column-header widget. *Deferral #32.*
- **Selection memory on ascend** — Backspace returns you to the top of the parent instead of to the
  directory you just left. *Deferral #34.*
- **Start where the operator chose** — argv paths; `/bin` and `/` are hardcoded smoke-test targets.
  `args` is declared in `[deps].stdlib` and never called. *Deferral #11.*
- **Directories larger than the cap** — `CRAB_MAX_ENTRIES = 256` is a compile-time ceiling; the
  canvas shows `812 items` in a pane and `41,208 files` indexed. Needs paged or streaming readdir.
  **Gate: agnos** — a readdir that can resume. *Deferral #02.*
- **Get the stat storm off the keystroke path** — one synchronous `sys_stat` per entry per listing.
  *Deferral #03.*
- **`on-accent` token** — **Gate: rupa.** Forced here: selected rows need legible ink.

### M4 — File operations (v0.8.0)

crab is a read-only browser. Enter on a file does nothing, silently.

- Copy · move · rename · delete · mkdir · open. **Gate: agnos** write syscalls. *Deferral #10.*
- **Transfer tray** (1c) — active operations with progress, rate and ETA.
  **Gate: dhancha** PROGRESS widget.
- **Context menu**, **inline rename**, **batch-rename sheet** — the canvas's own "not yet drawn" list.
  **Gate: dhancha** context menu + modal sheet.
- **Drag between panes** — dhancha already has `DRAG_START`/`DRAG_MOVE`/`DRAG_DROP`/`DRAG_END` and
  `dh_widget_set_draggable`/`set_drop_target`. crab consumes none of it.
- **Empty and permission-denied pane states** — also from the canvas's not-yet-drawn list.
- **Ratify the small-window question** — one pane + switcher at 420px.

### M5 — Views (v0.9.0)

- **Grid**, **Columns** (miller), **Gallery** — the canvas's four-way view switcher.
  **Gate: dhancha** GRID / COLUMNS widgets. Note `CANVAS` (0.9.9) exists and could carry app-drawn
  grids, but the toolkit is the right home — "the rule lived in three apps, now it lives once".
- **Preview pane** with real metadata (`42.8 MB · 8192 × 5464`, camera, shot).
- **Thumbnails** — **Gate:** an image decoder; none exists in the stack today.
- **Proportional text** — crab passes `font = 0` (kashi CP437 bitmap) and calls no `rekha_*`
  function, while the README claims rekha TrueType and the canvas assumes Barlow + JetBrains Mono.
  **Gate: rekha** + dhancha font plumbing. *Deferral #26.*

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

## Cross-cutting (not a milestone — do these continuously)

### Testing and CI — the gate is thinner than it looks

- ⛔ **`src/render_test.cyr`'s ten pixel assertions never run in CI.** It is crab's strongest test,
  mutation-proven against four regressions, and CI runs `cyrius test` which does not discover it.
  *Deferral #14.*
- ⛔ **CI never builds `--agnos`** — the target `state.md` calls "the real target". Every
  `#ifdef CYRIUS_TARGET_AGNOS` region is uncompiled by the gate. *Deferral #15.*
  ⛔⛔ **THIS STOPPED BEING HYPOTHETICAL ON 2026-08-27.** With crab's `path` overrides disabled and a
  fresh resolve against the DECLARED graph (dhancha 0.9.17, which predates `POINTER_SCROLL`):
  **host build PASS · `--agnos` build FAIL — `undefined variable 'POINTER_SCROLL'`.** A commit in that
  state goes green through CI and cannot build the thing that ships. ⇒ #15 is not a tidiness item; it
  is the gate that would have caught a broken dependency graph, and it is the reason `path` currently
  masks one.
- CI runs neither `fuzz`, `bench`, `lint`, `fmt --check`, `vet`, `deny` nor `coverage`. *Deferral #36.*
- **The fuzz harness reads none of its input** — `fuzz_main(data, len)` returns 0 without touching a
  byte, so `cyrius fuzz` would PASS against anything. crab parses untrusted readdir records.
  *Deferral #12.*
- **The bench harness times an empty function.** crab has had exactly one performance regression and
  it was reported from iron by an operator, not caught here. *Deferral #13.*
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
- `crab --about` closing on `…anyone? …anyone?` — needs M3's argv handling.
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
