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

- ✅ **Does the second pane ever leave? — ANSWERED, RATIFIED 2026-08-30 in M4.** Yes, below 600 px,
  where two panes could no longer each show the full column set. See M4 for the derivation and why
  the canvas's 420 px was not copied in as the number.
- ✅ **Which accent roles does the compositor hand over? — ANSWERED, gate closed.** rupa **0.1.5**
  publishes **`on-accent`**, dhancha **0.9.20** binds it (`dh_theme_on_accent()`) and paints the
  focused selection's *text* with it, and crab's `render_test` confirms it at the pixel.
  ⛔ **THE HIGHLIGHT WAS ERASING THE ROW IT HIGHLIGHTED.** The selected row is filled with `accent`
  and its label was drawn in the theme's primary `ink` — on MUDRA dark, `0xE7E9EF` on `0x00E5FF` is a
  contrast ratio of **1.27:1**. The one row the operator is looking at was the one row that could not
  be read.
  ⭐ **All four grounds now clear the WCAG AA floor** (12.72 / 4.64 / 11.27 / 5.49 : 1), and rupa's
  tests assert the *floor* rather than the numbers, plus that the no-token fallback — plain `ink` on
  the accent — **fails** it, so the guarantee is not vacuous.
  ⛔ **The obvious heuristic is wrong.** A 299/587/114 luma against a 128 threshold picks the LIGHT
  ink for MUDRA light's `0x0090A6` accent, where the dark ink measures better (4.64 vs 3.57). rupa
  ships `rupa_contrast` / `rupa_ink_on` comparing real contrast instead.
  ⚠ **A second, older defect fell out of the same change**: dhancha's scalable-font path blitted
  hardcoded white and ignored the theme, so it was unreadable on both light grounds and had been
  since it was written. The bitmap path (what everything ships with) always honoured the theme, which
  is why nothing caught it.
  ⚠ **`on-accent` is deliberately the same value as `bg` on MUDRA dark** (`0x0B0C0E`) — the legible
  ink on a bright cyan fill just *is* the carbon void. So "this pane has no on-accent pixels" cannot
  be written as a pixel check; it finds background. `render_test` says so at the call site.

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
⭐ **Re-derived at the 0.7.0 cut and the ladder held**: M3 shipped as **v0.7.0**, a minor, as mapped.
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

### M3 — A browser you would actually use (v0.7.0) — ⭐ **SHIPPED 2026-08-28**

⭐ **Four of seven items shipped in 0.7.0**: sorting (*#33*), selection memory (*#34*), argv start
paths (*#11*), and deferred statting (*#03*). Each is host-tested and mutation-proven, and each was
confirmed on a real agnos kernel under QEMU.
⛔ **THE OTHER THREE ARE GATED UPSTREAM, NOT UNFINISHED** — real columns (*#32*) on a dhancha TABLE
widget, directories past the 256-entry cap (*#02*) on a resumable agnos `readdir`, and `on-accent` on
rupa. They are listed below with their gates, and none of them is crab-side work being deferred.
⚠ **This mirrors M2, which shipped 5 of 7 with 2 gated.** A milestone closing with gated items is the
normal shape here, and calling it "done" would be the lie; calling it "not shipped" would be another.

⭐ **The dependency graph was verified four ways at this cut, not three.** crab's manifest sets
`path = "../X"` for rupa, kashi, dhancha and setu, and **`path` wins over `tag`** — so a green local
build is not evidence the declared tags resolve. Beyond matching sibling `VERSION`s, `git rev-parse
<tag> == HEAD`, and the tags existing on the remote (checked with `curl` against the GitHub API —
`git ls-remote` cannot authenticate in the sandbox, which is *inconclusive*, not a failure), the
manifest was copied with **every `path` line disabled** so `cyrius deps` actually cloned the tags.
Both targets built and all 228 tests passed against that graph, and the resulting binaries are
**byte-identical** to the path-resolved ones. Only that last check is evidence.

- ✅ **Sorting — DONE.** Name (case-insensitive) / size (largest first) / modified (newest first) /
  kind (extension, then name), **directories first on every key**, `s` cycles the mode and re-sorts
  BOTH panes. Applied to the initial listings and to every descend and ascend. *Deferral #33.*
  ⛔ **DOTFILES ARE NOT HIDDEN, DELIBERATELY.** Hiding them is only safe when there is a way to reveal
  them, and crab has no such affordance and no settings surface to put one on — so hiding would make
  files that exist unreachable and unmentioned. A file manager that silently omits files is worse than
  one that shows too many. They sort naturally: `.` is 0x2E, ahead of every letter.
  ⛔ **Directories-first is unconditional and outranks the key.** Interleaving them by name means
  hunting for the one folder among a hundred files; a directory's "size" is the entry, not its
  contents, so size order with them mixed in is meaningless anyway.
  ⭐ **`s` now follows the selection** through the re-sort — that was *#34*'s work and it has landed.
  It reset the selection at first, which was honest but lossy; leaving `sel` alone would have silently
  selected a **different** file.
  ⭐ **Host-tested (44 assertions) and mutation-proven six ways**; QEMU confirms the key reaches crab
  and all four modes cycle. ⚠ The **order itself** is asserted only on the host — crab does not log
  entry names, deliberately, because the per-entry stat trace WAS the 2026-08-19 slowness.
  ⚠ One guard is labelled as unexercised rather than sold as a fix: the `-1` (unstattable) handling in
  `crab_entry_cmp` fails no test, because -1 is the smallest value and both orders are descending, so
  it sinks on its own. Mutation showed that; an earlier comment claimed the opposite. It is kept for
  the smallest-first order that column headers (*#32*) will bring.
- ✅ **Real columns — DONE, gate closed.** NAME · SIZE · MODIFIED with headers. *Deferral #32.*
  ⭐ **dhancha 0.9.20 adds `src/table.cyr`** — a **shared column-width spec** read by the header and
  by every row. That sharing is the whole feature: dhancha could already draw a `BOX_H` of
  fixed-width labels, but nothing made the header and the rows agree on where column 2 starts, and
  rows laid out independently drift the moment one cell's text changes. A table whose columns drift
  is worse than no columns — the reader trusts the alignment to say which value belongs to which
  heading.
  ⭐ **The 13-character name column is gone.** NAME is the **remainder** column, so it grows with the
  pane instead of being pinned at a width chosen in 2026-08.
  ⛔ **COLUMNS THAT DO NOT FIT ARE DROPPED, NOT SQUEEZED.** crab's default window is 380x220, so a
  pane is ~187 px — about 20 characters at the system font's 9 px advance. MODIFIED is
  `2026-08-28 11:04`, 153 px on its own: showing it there would leave four characters for the
  filename, which is not a file manager. The pane picks its column set from its own width, so a wider
  window earns more columns.
  ⛔ **The header sits OUTSIDE the LIST**, which is what keeps it from scrolling away and from ever
  being selectable. A header the operator can select is a file that does not exist.
  ⚠ **Truncation still says it was truncated** — the `~` marker survives the port, now at the
  COLUMN's width rather than a hardcoded 13. That was the 0.5.0 defect and it is not being re-earned.
  ⭐ **dhancha also gained `DH_W_FG`** (per-widget text colour) so the header reads as a label.
  ⛔ **It does not outrank the selection highlight**: a cell's colour is a preference, on-accent on a
  selected row is a legibility guarantee — otherwise a table could reintroduce the unreadable-row
  defect that had just been fixed. Mutation-proven.
  ⭐ **253 host assertions** (was 228) and **12 dhancha RUN suites**; the alignment test asserts on
  laid-out **bounds**, because a spec that stored the right numbers and laid out wrongly would pass
  an accessor-only test and still be useless.
  ⛔ **TWO REGRESSIONS THE EXISTING SUITE CAUGHT, both of which would have shipped.** Building the
  column spec inside the render path allocated **1600 B over twenty frames** — reopening the
  per-frame allocation gate that dhancha 0.9.13-0.9.16 and crab 0.6.0 spent four releases closing;
  `dh_cols_reset` exists so a spec is allocated once and reused. And the new header shifted every row
  down, which silently invalidated the **hardcoded pixel coordinates** in both the click test and
  `render_test` — they now read the row's position from the layout instead.
  ⚠ **And one test was fragile rather than wrong**: `rt_row_has` samples a single scan line, which is
  fine for a filled rectangle and unreliable for text, since whether a glyph lights a given row
  depends on the letterform. Text assertions scan the row's whole band.
- ✅ **Selection memory on ascend — DONE.** Backspace lands on the directory you just left instead of
  at the top of the parent, and `s` now **follows** the selected entry through the re-sort rather than
  resetting it. *Deferral #34.*
  ⛔ **THE LEAF IS CAPTURED BEFORE THE ASCEND, NOT AFTER.** `crab_ascend` rewrites `path` **in place**,
  so reading the leaf afterwards reads the parent's name. Likewise both panes' selected names are read
  **before either sort**, because `crab_sort_entries` permutes in place and the first sort invalidates
  the second pane's index.
  ⛔ **A MISSING NAME REPORTS -1 AND THE CALLER FALLS BACK TO ROW 0.** The directory can genuinely be
  gone — deleted or renamed between the two readdirs. Guessing a nearby row would select a *different*
  file silently.
  ⭐ **Mutation-proven three ways**: root reporting a leaf (2 failures), prefix matching in
  `crab_index_of` (1), and a missing name returning 0 instead of -1 (4).
- ✅ **Start where the operator chose — DONE.** `crab [LEFT] [RIGHT]`. `/bin` and `/` were hardcoded
  smoke-test targets and `args` sat declared in `[deps].stdlib` and never called. *Deferral #11.*
  ⛔ **AN UNUSABLE PATH IS REFUSED AND SAID OUT LOUD, not quietly swapped for the default.**
  `crab_readdir_into` takes absolute paths; a relative one lists **nothing**, and an empty pane with no
  message reads as a broken filesystem rather than as a rejected argument.
  ⚠ **THE DEFAULTS STAY, and they are the normal case**: on agnos the compositor spawns crab through
  the launcher with a path and no arguments, so the desktop always takes the fallback. Every existing
  QEMU harness is unaffected for the same reason.
  ⚠ **argv on agnos is not the Linux mechanism.** `lib/args_agnos.cyr` reads the init rsp that cycc
  parks in **r15** at entry (the kernel caps argc at 8); on the host it parses `/proc/self/cmdline`.
  That is compiler plumbing, so a green host test says nothing about the agnos target — it was proven
  separately in QEMU.
  ⭐ **Host-tested (10 assertions, both sides of the length boundary) and mutation-proven three ways**:
  dropping the absolute check (2 failures), dropping the null guard (the binary faults), and `>` for
  `>=` on the cap (1). ⛔ A **fourth** mutation deleted the explicit empty-path guard and **nothing
  failed** — `""` cannot begin with `/`, so the absolute check already subsumed it. The dead line was
  removed rather than kept as decoration.
  ⭐ **QEMU-proven on agnos** (`agnos/scripts/harness/crab-listing-cap-test.py`): `crab /bin /bin`
  lists `/bin` twice and `/` **not at all**, and `crab relative` announces the refusal on the console.
  The oracle is an **absence**, because a presence oracle would pass whether or not argv landed — and
  the bare-`crab` run in the same boot reports `/=7, /bin=45`, the exact shape that trips its FAIL
  branch, so the broken-argv output is on record in every run.
- ✅ **Directories larger than the cap — DONE, gate closed.** *Deferral #02.*
  ⭐ **agnos 1.56.50 adds `#101 readdir_at(path, buf, max, &cursor)`** — `#81` with a cursor, so a
  directory bigger than the caller's buffer can be read in batches. The cursor is the **byte offset
  into the directory file** (POSIX `telldir`'s cookie); an entry index would force a re-walk from the
  top every call, which is the O(n²) resumability exists to avoid. `0` starts, `-1` means exhausted,
  and passing `-1` back is a no-op rather than a restart.
  ⛔ **A SEPARATE SYSCALL NUMBER, NOT A 4th ARGUMENT ON `#81`.** Its callers pass three; the 4th
  register holds whatever they left there, and this argument is a **pointer the kernel writes
  through**. agnos states the rule itself at `#55`: *"unused syscall argument registers are not
  zeroed by the compiler, so arity is part of a syscall's ABI here."*
  ⭐ **`CRAB_MAX_ENTRIES` 256 → 1024** — the canvas asks for `812 items` in a pane, so 256 was below
  what the design itself wants. 64 KB per pane buffer, paid once at startup.
  ⛔ **RAISING THE CAP DOES NOT MAKE THE CAP HONEST, which was the actual defect.** A directory
  bigger than any cap still exists. What changed is that once the pane buffer fills, crab **keeps
  walking without storing, purely to count** — so the warning is now `showing 1024 of 1200` instead
  of `has more entries than are shown`.
  ⛔ **THE `#81` FALLBACK IS NOT OPTIONAL.** Against a kernel older than 1.56.50, `#101` is an
  unknown syscall and the dispatcher returns `-1` — without the fallback crab would show an **empty
  pane** on every older kernel and read as a filesystem fault.
  ⭐ **Kernel mutation-proven four ways**, each a full rebuild and boot: resuming at the block start
  instead of mid-block (never terminates), the budget checked after the record is consumed (a batch
  overruns `max`), exhaustion writing `0` instead of `-1`, and the alignment guard removed.
  ⭐ **QEMU-proven end to end** against a seeded **1200-entry** directory: `crab /big /` reports
  `showing 1024 of 1200`. Mutation-proven by deleting the counting walk — the harness then finds no
  counted warning at all.
  ⚠ **The paged path has NO host test and cannot have one** — it is entirely inside
  `#ifdef CYRIUS_TARGET_AGNOS`. The QEMU arm is the only coverage it has.
- ✅ **Got the stat storm off the keystroke path — DONE.** *Deferral #03.*
  ⭐ **MEASURED BEFORE BUILT, because this cost was misattributed once already.** The 2026-08-19 iron
  report ("responding slower from inputs") looked like the listing and was in fact the per-entry
  **narration**. So `crab_stat_all` was first made to report its own cost on the one line per readdir
  that already existed: **50 ms for 45 entries in `/bin`, 10 ms for 7 in `/`** — about **1.1 ms per
  entry**, linear, i.e. **~280 ms of blocking syscalls** at the 256-entry cap, on the keystroke that
  descends. ⚠ QEMU figures; its disk emulation is far slower than iron. The **per-entry linearity** is
  the durable finding, the absolute number is an upper bound.
  ⭐ **What makes the deferral safe: dir-ness comes from the READDIR RECORD, not from stat**
  (`CRAB_REC_TYPE`). Directories-first, the `/` row marker, NAME order and KIND order therefore need
  **no stat data at all**. Only the size column, the status line, and SIZE/MTIME ordering do.
  ⇒ A listing marks every entry pending and returns; the event loop drains bounded batches (32/tick,
  idle only); and a sort to SIZE or MTIME forces the sweep itself, because those orders cannot be
  computed from partial data without visibly reshuffling as batches land.
  ⛔ **PENDING (-2) IS NOT UNSTATTABLE (-1)**, and both the size and date columns render them
  differently (`?` vs `-`). Conflating them would make a drain that never completes look like a
  filesystem fault and get debugged as one.
  ⭐ **Host-tested (28 assertions) and mutation-proven five ways**: a batch ignoring its budget (4
  failures), a batch re-statting settled entries (6), `crab_stat_reset` running one past `count` (1),
  NAME wrongly declared to need stats (3), and the negative-start clamp (1).
  ⛔ That fifth one **passed at first** — the obvious assert read whatever the arena left in front of
  the array, found no `-2`, and walked forward to the right answer anyway. The array is now placed
  eight entries into a block whose front is filled with the pending marker, so a missing clamp returns
  a **negative index**. A test that passes when the code is broken is worthless.
  ⭐ **QEMU-proven on agnos, in two harnesses, because one cannot show both halves.**
  `crab-listing-cap-test.py` sees **zero `stat-cost` lines where it previously saw six** totalling
  ~230 ms — but it runs crab with no compositor, so crab exits before the event loop and the drain
  never runs there at all. `crab-resize-test.py` runs the real desktop and reports **`deferred stat
  drain completed: True`**. ⚠ Reporting only the missing `stat-cost` lines would have shown that crab
  stopped statting, not that it moved the work — and those look identical right up until the sizes
  never arrive.
  ⚠ In the desktop run the sort to SIZE cost **nothing**: the idle drain had already finished, so the
  forced sweep found nothing pending. That is the design working, not the sweep failing to fire.
- **`on-accent` token** — **Gate: rupa.** Forced here: selected rows need legible ink.

### M4 — File operations (v0.8.0) — ⭐ **STARTED in v0.7.1**, completes at v0.8.0

⭐ **Shipped in 0.7.1**: copy, move and delete (keys `c` / `m` / `d`, the last behind a `y`
confirmation), the empty-pane states, and the whole `crab_fs_*` write layer with its per-target
syscall shim. `crab_fs_rename` / `crab_fs_mkdir` are implemented and asserted but **not key-bound**.
⚠ **Those features rode a PATCH number by operator ruling** (2026-08-30), the same way M2 rode
0.6.1. The milestone→version ladder below is unchanged; 0.7.1 is not M4's number.
⛔ **Still open for v0.8.0**: rename/mkdir key bindings (need dhancha `TEXTINPUT` wired into the
event loop's focus and escape routing — the widget exists, since 0.9.7), the transfer tray
(**genuinely gated** on a dhancha PROGRESS widget, and it is what blocks recursive copy and delete),
context menu, batch-rename sheet, drag between panes, and the small-window ratification.

#### ⛔ v0.8.0 CARRIES A COVERAGE GATE, AND IT IS A RELEASE GATE RATHER THAN A v1.0 ASPIRATION

**73 % at 0.7.1 (65/89 fns), down from 81 %, against a v1.0 criterion of 80 %.** The number has now
fallen at two of the last three cuts, and both falls were the same mechanism: a release adds
functions faster than it adds references to them. ⇒ **A criterion checked only at v1.0 gets further
away every release.** Check it at the cut, in Process step 5, alongside the CHANGELOG and state.md.

⛔ **DO NOT CHASE THE NUMBER.** This is *reference* coverage — a function counts as covered the
moment any test names it. `assert(crab_say("x") > 0)` would raise the percentage and prove nothing,
and this file would rather carry 73 % honestly than 85 % that means less. **23 functions are
currently untouched by `tests/` and `render_test`; only the first group below is worth writing.**

**Group 1 — behaviour that can be WRONG, and nothing would say so. Write these.**

| function | what an assertion pins |
|---|---|
| `crab_entry_cmp` | ⭐ **the highest-value gap.** The comparator is exercised only *through* `crab_sort_entries`, so every mode's rules — dirs-first outranking the key, the -1/-2 sentinels, NAME as universal tiebreak — are asserted only in aggregate. A direct table of (a, b, mode) → sign pins each rule separately, and would have caught the `<= 0` stability bug without needing 16 identical rows. |
| `crab_fs_msg` | every `CRAB_FS_*` code maps to a distinct non-empty string. Same shape as the `crab_pane_state_label` assertions 0.7.1 added — and the same failure it prevents: a refusal the operator cannot tell from another. |
| `crab_status_str` | the status line's text for a selected row, an empty pane, a pending stat (`?`) and an unstattable entry (`-`). It is what the operator reads most and nothing asserts it. |
| `crab_set_notice` / `_clear` / `_get` | the notice outranks the row, and a stale notice is cleared. 0.7.1 asserts this only at the pixel, in `render_test`, which CI does not run. |
| `crab_lower` | case folding — ASCII, the boundaries either side of `A`/`Z`, and non-letters passed through. One line, and `crab_name_cmp` is built on it. |
| `crab_stat_one` | ⭐ **testable for real on the host** — Linux `stat` exists, so this can assert a real size and mtime, a directory, and a failure on a path that does not resolve. |
| `crab_fs_*_raw` (4) | the per-target shims. The host arm is real; asserting it directly separates "the shim is wrong" from "the operation is wrong", which today are one failure. |
| `crab_listing_total` / `crab_listing_err` | that they report the LAST listing and are reset per call — the property `main.cyr` depends on when it captures pane state, and the one that breaks silently if a future edit moves the reset. |

**Group 2 — transitively exercised; leave them.** `crab_pane`, `crab_spec_init` run inside every
`render_test` render, and `crab_sort_scratch` / `crab_copy_buf` inside every sort and copy. They are
covered in the sense that matters and uncovered in the sense the tool counts. ⚠ Naming them in a
test would move the number without adding a claim.

**Group 3 — diagnostics.** `crab_say`, `crab_log_cd`, `crab_log_extent`, `crab_stat_trace_init`.
Writes to fd 1. ⚠ Asserting these is how a coverage number becomes theatre; if they are ever wrapped
in logic worth testing, that logic gets a test and not the writer.

⭐ **DONE at 0.7.1 — Group 1 landed and coverage is back to 80 % (74/92).** It took ~30 assertions,
and the number moved as a side effect of writing claims rather than as its object. ⚠ What remains
for v0.8.0 is to keep it there: check coverage at the cut, in Process step 5.

crab is a read-only browser. Enter on a file does nothing, silently.

⭐ **ALL FIVE M3 DEFECTS ARE CLEARED in v0.7.1 (2026-08-30)** — see the CHANGELOG. The two
that were load-bearing for M4 are the two that mattered: the **non-terminating `#101` readdir walk**
(both loops, now guarded on a stalled cursor *and* capped) would have become a hang *while a write
was in flight*, and the **O(n²) sort on the keystroke path** is now a bottom-up merge over an index
array — **182.9 ms → 414 µs** at the cap, reverse-sorted, measured. That is the latency budget the
transfer tray was going to compete for.
⚠ **One of the five was fixed differently than the review prescribed.** The false truncation at
exactly the cap was to be fixed by "guard on `cur`" — and that does not work: `ext2.cyr:2412` *parks*
the cursor on the record it declined to take, so `cur != -1` at exactly the cap. The count is the
oracle, not the cursor.

- ⭐ **Copy · move · delete · open — DONE (2026-08-30). THERE WAS NO GATE.** The line here read
  *"Gate: agnos write syscalls"* and it was false: every arm has been real and mount-routed since
  agnos **1.41.3** — `open`#7 (`syscall.cyr:8496`), `mkdir`#9 (`:8547`), `rmdir`#10 (`:8565`),
  `unlink`#30 (`:8849`), `rename`#31 (`:8879`) — and crab's **already-pinned cyrius 6.5.35** vendors
  a wrapper for every one. No pin move was needed and none is.
  ⛔ **WHERE THE FALSE GATE CAME FROM, so it is not re-derived wrongly**: agnos's own
  `agnos-userland-abi.md` still describes mkdir and rmdir as *"🔧 stub → 0"* and open as
  *"initrd_open ONLY"* — contradicted by the dispatcher arms in the same repo. **Verify against the
  dispatcher, never the table.**
  ⚠ **rename and mkdir are implemented and tested but NOT yet bound to keys** — both need a name
  typed, and wiring dhancha's `TEXTINPUT` (which exists, since 0.9.7) into the event loop's focus
  and escape routing is the next slice. `crab_fs_rename` / `crab_fs_mkdir` are done and asserted.
  ⚠ **`Deferral #10` is cited here and defined nowhere** — there is no deferral registry in the repo.
  Either write one or drop the reference.
- **Transfer tray** (1c) — active operations with progress, rate and ETA.
  **Gate: dhancha** PROGRESS widget.
- **Context menu**, **inline rename**, **batch-rename sheet** — the canvas's own "not yet drawn" list.
  **Gate: dhancha** context menu + modal sheet.
- ⛔⛔ **Drag between panes — BLOCKED UPSTREAM, and this line's "gated on nothing" was wrong.**
  dhancha does ship `DRAG_START`/`DRAG_MOVE`/`DRAG_DROP`/`DRAG_END`, but **its drag API and its
  frame-arena API are mutually exclusive**: `dh_frame_begin` calls `dh_reset_input()`, which zeroes
  `_dh_drag_src`, **every frame**. A drag spans press → move → release, i.e. many frames, so any app
  using the frame arena can receive `DRAG_START` and then never `MOVE`, `DROP` or `END`.
  ⚠ **The clear is correct and must not simply be removed** — after an arena rewind that pointer
  addresses memory about to be handed out again. The conflict is structural: drag identity must
  outlive a frame, and it is stored as a pointer that cannot.
  ⇒ Filed: [`2026-08-30-dhancha-drag-api-cannot-survive-a-frame-arena.md`](issues/2026-08-30-dhancha-drag-api-cannot-survive-a-frame-arena.md),
  and **fixed upstream in dhancha 0.9.21** — a press on a draggable widget under a frame arena is now
  simply a click, and `dh_drag_available()` answers the question up front. Nothing half-fires.
  ⛔⛔ **THAT FIX DOES NOT UNBLOCK THIS ITEM, AND THE REASON IS A STANDING RULING, NOT A GAP.** Drag
  is synthesized inside `dh_dispatch`, and **crab does not use `dh_dispatch`** — operator ruling
  2026-08-27, recorded at `src/app.cyr:215`, for this same cross-frame-identity reason. dhancha
  0.9.21 makes the toolkit honest; it does not give crab events crab never asked for.
  ⇒ **The remaining choice is crab-side and needs a ruling**: implement drag in crab's own
  `(pane, row)` model — which is exactly what crab already does for double-click and the wheel, and
  which works today with no dependency — or drop drag from M4. Adopting `dh_dispatch` to get it is
  the one option the 2026-08-27 ruling forecloses.
  ⚠ **If crab implements it**: press on a row, track with the pointer state crab already owns,
  release over the other pane, then reuse `crab_fs_copy` / `crab_fs_move`. The plumbing is ~30 lines
  because the hit-testing and the file operations both already exist.
- ⛔ **The genuinely gated part of M4 is the transfer tray** (dhancha PROGRESS), and it is what
  blocks recursive copy and recursive delete: neither may go behind a keypress without a surface
  that shows what is happening and a way to stop it.
- ⭐ **Empty pane states — DONE (2026-08-30).** A pane with no rows had one appearance and four
  meanings; it now says which. ⛔ **There is no "permission-denied" state and this line should stop
  asking for one**: agnos is single-user always-root, `ext2_readdir_at_sys` has no `EACCES` arm, and
  `getuid`#15 is a literal `return 0`. The states the kernel can actually report are EMPTY / GONE /
  NOTDIR / UNREAD. A DENIED state belongs here the day agnos grows a credential model, not before.
  ⚠ The 2026-08-30 burn is what motivated this: `/` held two entries (`/sl_s`, `/lp`) that could be
  listed but not stat'd, and every layer above reported the failure as an absence.
- ⭐ **Ratify the small-window question — DONE (2026-08-30). The second pane DOES leave.**
  ⛔ **The threshold is derived, not the canvas's 420 px copied in**: two panes are worth it only
  when **each** can honestly show the full column set (NAME + SIZE + MODIFIED), so `panew >= 297`,
  i.e. a window of **600 px**. A pixel number lifted from a mockup is one nobody can re-derive when
  the font, the row height or the column set changes. The canvas's 420 sits below 600 and gets one
  pane, which is what the canvas draws — the rule agrees with the drawing without quoting it.
  ⭐ **The switcher is the keys crab already has** — Left/Right (h/l) set `active_pane`, which in
  solo mode changes which pane is drawn. No new binding, and the header gains an `A` / `B` label
  because side-by-side is no longer what distinguishes them.
  ⭐⭐ **AND IT FIXES THE MODIFIED COLUMN AT THE DEFAULT WINDOW.** At 380x220 two panes got 187 px
  each, so `crab_cols_for_width` returned 2 and MODIFIED never appeared — while the README and the
  CHANGELOG headline both read "NAME · SIZE · MODIFIED". One pane at 380 is 374 px and shows all
  three. The ⚠ the handoff carried about that is now just fixed.

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
- `crab --about` closing on `…anyone? …anyone?` — M3's argv handling has landed (*#11*), so this is
  now unblocked; crab parses positional paths only and has no flag surface yet.
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
