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

**0.7.0** (2026-08-28) — see [`../../CHANGELOG.md`](../../CHANGELOG.md).

⭐ Refreshed **in the same commit as the CHANGELOG entry**, which is the rule the header below
demands and the one this file broke twice.

## Toolchain

- **Cyrius pin**: `6.5.35` (in `cyrius.cyml [package].cyrius`)
- Trail: `6.4.71` → `6.5.5` (0.4.3) → `6.5.9` → **6.5.21** (0.4.9, one language version across the
  desktop stack) → **6.5.27** (0.4.11) → **6.5.28** (0.4.13) → **6.5.35** (0.4.15).
- ⚠ The pin is documentation, not enforcement — `cyrius build` compiles with the **installed** `cycc`,
  warns `toolchain drift`, and carries on. ⛔ It is not cosmetic: **CI installs the toolchain from this
  pin** (`grep '^cyrius = ' cyrius.cyml`, both `.github/workflows/*.yml`), so while the pin lagged, a
  cold CI build and a local build compiled crab with *different compilers* and neither run said so.
- ⛔ **6.5.35 is not a cosmetic bump, unlike 6.5.21 → 6.5.27 which 0.4.11 measured byte-identical.**
  The binary is **377,288 B on both 6.5.28 and 6.5.35 — the same SIZE** — but **240,284 of those bytes
  differ (63.7 %)**, from the .35 linear-scan register-allocator rework. ⚠ An identical size is exactly
  the shape of evidence that gets mistaken for "nothing changed". It is not.
- `lib/` is vendored from the pinned snapshot by `cyrius lib sync`, **not** by `cyrius deps` — a
  toolchain bump without a `lib sync` leaves the stdlib behind. ⚠ The sync walks the **declared**
  `[deps].stdlib` set: **29 leaves**, while crab actually vendors **30**. The odd one out is
  `lib/atomic.cyr`, a transitive leaf (`alloc`, `io`, `fmt` and three `syscalls_*` include it) that no
  declaration names. Check it by hand at every bump.

## Source

2019 lines across **six** files (five until 0.6.0). ⚠ +507 in M3, almost all of it in
`app.cyr` (368 → 755) and its tests — sorting, selection memory, argv validation, deferred statting.

- `src/main.cyr` (562) — ⛔ **`main()` AND `_entry()` AND NOTHING ELSE, as of 0.6.0.** It ends in
  `_entry();`, so a test that included it would RUN THE APP — which is why everything testable was
  moved to `src/app.cyr`. **Do not add a function here.** What remains is the dhancha client
  lifecycle, the shm present path and the frame loop.
  ⚠ The **present path stays hand-rolled on purpose**: a LIVE shared buffer (create once, rewrite in
  place) against `dh_client_present`'s per-frame ATTACH+COMMIT. Different models — swapping them is
  not a rename.
  ⚠ **No arena setup here any more** (0.6.0). It lived in `main()` and that was a gap: deleting it
  broke no test while restoring a 77 KB-per-frame leak. `crab_render` owns it.
- `src/app.cyr` (755) — ⭐ **NEW at 0.6.0**, and M3's centre of gravity. The application layer lifted out of `main.cyr`: the
  readdir parser (`crab_readdir_into` and its cap clamp), the stat layer, `crab_descend` /
  `crab_ascend`, `crab_surface_flags`, the serial logging.
  ⛔ It exists for the same reason `path.cyr` does — **none of it was reachable from any test** while
  it lived in `main.cyr`, in a program whose two shipped defects were both found on iron. 0.6.0 added
  17 assertions against it, all mutation-proven.
  ⚠ `CRAB_MAX_ENTRIES = 256` and `crab_surface_flags()` returning `SETU_SURF_PREMULTIPLIED`
  **unconditionally** — no flag, no arm, no env var — both live here.
- `src/path.cyr` (91) — the #81 record layout and the **bounded** cstring/path helpers.
  ⛔ Extracted at 0.5.0 for the reason `app.cyr` was extracted at 0.6.0: a memory-safety fix that
  cannot be asserted on is a fix held on trust.
- `src/ui.cyr` (453) — dual-pane file browser: a pane is a **`dh_list`** (0.4.10), plus size/mtime
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
- `src/render_test.cyr` (145) — a standalone harness: renders the production surface at 380×220
  and dumps BGRA to `build/crab-render.bin`, 11 `check()` assertions.
  ⚠ It creates **two** `DhSurface`s on purpose — it holds two frames live and dumps the first, and
  since dhancha 0.9.14 one surface would hand both renders the same target.
  ⛔ **It is not run by CI or by `cyrius test`** — see Known gaps.
- `src/test.cyr` (13) — ⚠ **deliberately empty, with a warning in it.** Bare `cyrius test`
  auto-discovers `tests/*.tcyr` and does **not** run the `[build].test` hook.

⭐ `cyrius coverage` reports **44/54 fns referenced (81 %)**, 6/6 files — up from 75 % at 0.6.1, 53 %
at 0.5.0 and 23 % at 0.4.15. ⭐ **This is the first cut above the v1.0 criterion of 80 %.**
⚠ Still a floor rather than a correctness proof: reference coverage counts a function as covered when
something references it, so `main.cyr` reads 1/1 because its only function is `main` itself. The 0.6.0
jump was the `app.cyr` extraction making a whole layer reachable; M3's is ten new `app.cyr` functions
that were written with their tests.
⚠ **13** lines exceed 120 characters (12 in `main.cyr`, 1 in `ui.cyr`) — up from 9 at 0.6.1, all in
the render call sites the deferred-stat drain added.

## Proven

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

## M3 — shipped as v0.7.0

⭐ **M3 shipped in 0.7.0.** Four of seven items are done and QEMU-proven; the three that are not are
**gated upstream, not unfinished** — real columns (*#32*) on a dhancha TABLE widget, >256 entries
(*#02*) on a resumable agnos `readdir`, and `on-accent` on rupa.
⭐ **The dep graph was verified with every `path` override disabled** at this cut: `cyrius deps`
cloned the declared tags, both targets built, all 228 tests passed, and the binaries were
**byte-identical** to the path-resolved ones. ⚠ `git ls-remote` cannot authenticate in the sandbox —
that is *inconclusive*, not a failure; the tag SHAs were confirmed with `curl` against the GitHub API.

- ✅ **Sorting (*#33*)** — name / size / modified / kind, **directories first on every key**, `s`
  cycles and re-sorts both panes; applied to the initial listings and to every descend and ascend.
  ⛔ **Dotfiles are NOT hidden** — crab has no reveal affordance, and silently omitting files is worse
  than showing too many.
  ⚠ The **order** is host-asserted only — crab does not log entry names by design.
- ✅ **Selection memory (*#34*)** — Backspace lands on the directory you just left, and `s` follows the
  selected entry through the re-sort instead of resetting it.
  ⛔ **The leaf is captured BEFORE the ascend** — `crab_ascend` rewrites `path` in place. Both panes'
  names are likewise read **before either sort**, since `crab_sort_entries` permutes in place.
  ⚠ A name that has vanished between readdirs reports `-1`; the caller falls back to row 0 rather than
  guessing a nearby row and selecting a different file.
- ✅ **argv paths (*#11*)** — `crab [LEFT] [RIGHT]`. Defaults stay and are the normal case: the agnos
  compositor spawns crab through the launcher with **no arguments**, so the desktop always falls back
  and every existing harness is unaffected.
  ⛔ **A path that is not absolute, or does not fit `CRAB_PATH_MAX`, is refused and ANNOUNCED**, then
  the default is used. `crab_readdir_into` needs absolute paths; a relative one lists nothing, and a
  silent empty pane reads as a broken filesystem rather than a rejected argument.
  ⚠ **argv on agnos is compiler plumbing, not a syscall** — `lib/args_agnos.cyr` reads the init rsp
  that cycc parks in **r15** at entry (kernel caps argc at 8). A green host test says nothing about
  the agnos target, so it was proven separately in QEMU.
  ⭐ **QEMU-proven**: `crab /bin /bin` lists `/bin` twice and `/` not at all; `crab relative` announces
  the refusal. The oracle is an **absence** — a presence oracle would pass either way, since `/bin` is
  also the default left pane.
- ✅ **Deferred statting (*#03*)** — the listing marks entries pending and returns; the event loop
  drains 32 per idle tick; a sort to SIZE/MTIME forces the sweep because those orders cannot be
  computed from partial data.
  ⭐ **Measured first**: ~1.1 ms per entry on agnos under QEMU (50 ms / 45 in `/bin`), so ~280 ms at
  the 256 cap on the keystroke that descends. ⚠ QEMU figures — the per-entry linearity is the durable
  finding, not the absolute number.
  ⭐ **Safe because dir-ness comes from the readdir record, not stat** — directories-first, the `/`
  marker, NAME and KIND order all need no stat data.
  ⛔ **Pending (-2) renders as `?`, unstattable (-1) as `-`.** Conflating them would make a stalled
  drain look like a filesystem fault.
  ⭐ **Proven in two harnesses**: `crab-listing-cap-test.py` now sees **zero** `stat-cost` lines (was
  six, ~230 ms), and `crab-resize-test.py` reports `deferred stat drain completed: True`. The first
  runs crab with no compositor so its drain never runs — one harness cannot show both halves.
- **Next**: M3's remaining items are all gated. Real columns (*#32*) on dhancha, >256 entries (*#02*)
  on agnos, `on-accent` on rupa.

## M2 — shipped as v0.6.1

⭐ **M2 shipped in 0.6.1.** Five of seven items are done and QEMU-proven; the two that are not are
**gated upstream**, not unfinished — see the ⛔ entries below and the roadmap.

- ✅ **Idle-poll allocation (*#09*)** — dhancha 0.9.16. The loop allocates nothing in steady state.
  Host-tested (200 idle polls, 0 bytes) and QEMU-confirmed for no regression.
- ⚠ **Resize (*#01/#04*) — built; REFUSAL path QEMU-proven, ADOPT path not reachable under QEMU.**
  `dh_surface_resize` (dhancha 0.9.17), the `#86` slot **created before the old is closed**,
  `w`/`h`/`stride` as state, re-ATTACH + COMMIT after the next render. Layout reflows for free.
  ⛔ **A new harness (`agnos/scripts/harness/crab-resize-test.py`) found a real bug on its first
  run**: crab closed its only shm buffer before knowing the replacement existed and exited. setu's
  `setu_client_present` closes first but has an inline-pixel fallback; crab's LIVE-buffer path has
  none, so that order was fatal here.
  ⛔ **And the byte cap was invented, not derived** — 16 MB from "the framebuffer's size", when agnos
  caps a `#71` pmm slot at **2 MB** and only a `#86` GPU carveout reaches 32 MB, chosen at runtime.
  The cap is now the absurdity bound; the kernel is the arbiter.
  ⭐ **Measured**: crab launches via F2→DOWN→Enter, receives a CONFIGURE for **2048x2018**, refuses it
  (no carveout under QEMU), stays at its old extent, and **answers 6 keystrokes afterwards** — which
  also settles the loop-lifetime question open since 0.5.0.
  ⭐ **And the harness is proven to go red**: with the `WINDOW_CONFIGURE` branch removed it reports
  FAIL (no CONFIGURE line at all) against PARTIAL for the real build, so "crab refused the ask" and
  "crab ignored the ask" are distinguishable. Without that the QEMU result would mean nothing.
- ✅ **Pointer (*#05*) — click to select, click to focus a pane, double-click to descend.**
  QEMU-proven (`crab: click`, then 5 keystrokes answered). crab is the **first client to decode
  `SETU_INPUT_PTR_MOVE`**.
  ⛔ **crab owns the interaction state; `dh_dispatch` is deliberately NOT used** — it tracks a press
  by widget pointer, and `dh_frame_begin` clears those between a press and its release. crab tracks
  pane + row index; dhancha supplies geometry via `dh_hit_test`. Operator ruling 2026-08-27.
  ⚠ `SETU_INPUT_PTR_BTN` carries no coordinates, so position comes from `PTR_MOVE`.
  ⚠ **Scroll wheel is NOT done** — setu has no wheel message kind. **Gate: setu.**
  ⚠ **Pane headers are not clickable** — the header is a sibling of the list, so the hit walk never
  reaches it.
- ✅ **Key release (*#06* first half) — `SETU_SURF_FULL_KEYS` requested and gated.** QEMU-proven
  exactly: 6 keystrokes → **12 received / 6 acted**. The setu gate is cleared — the compositor
  implements the flag end to end.
  ⛔ Asking without gating makes every key act twice; the compositor measured that on its own chrome
  (2026-08-18). The flag and the gate are coupled and live together in `src/app.cyr`.
- ✅ **Held-key repeat (*#06* second half) — DONE, and NO kernel change was needed.**
  ⛔ **This file previously said it was blocked on agnos for want of a bounded wait. That was wrong**,
  and wrong for a specific reason worth keeping: the claim was made from syscall *signatures*
  (`#14 pause` takes no timeout) without reading `#14`'s own comment, which says a device IRQ **or the
  100 Hz timer** wakes it. Measured on a real kernel: `sys_pause()` returns in **0-4 ms**. Repeat is a
  clock check in the idle path.
  ⚠ Only Up/Down/j/k repeat — Enter and Backspace would walk the tree on a held key. 400 ms delay,
  then 60 ms interval, both gates required.
  ⭐ QEMU-proven: a QMP-held key repeats and **stops on release** (0 repeats after).
  ⚠ **The observed rate is far below the configured interval** (~1 repeat per 1.6 s hold vs ~20
  expected) and is **not diagnosed**. Each repeat re-renders and re-writes 334 KB of shm, so frame
  cost is the likely bound — a hypothesis, not a finding.
- **Untouched**: routing through `dh_dispatch` (#07 — ⚠ the KEY half is safe; adopting it wholesale is
  not, same ruling as the pointer).

## Dependencies

Declared in `cyrius.cyml`, **all six at their latest published tag as of 2026-08-27**:

| dep     | tag    | `path`? | why crab needs it                                   |
|---------|--------|---------|-----------------------------------------------------|
| sadish  | 0.5.2  | no      | 2D vector — the surface everything else draws into   |
| rupa    | 0.1.4  | yes     | shared desktop theme tokens (`RupaMotion` since .3)  |
| rekha   | 0.3.5  | no      | text; references `sd_*`                              |
| kashi   | 1.0.6  | yes     | CP437 8×16 glyph data for `dh_draw_text` (font=0)    |
| dhancha | 0.9.17 | yes     | widgets, `dh_client_poll_event`, `dh_theme_*`        |
| setu    | 0.8.7  | yes     | client transport — channel-band, reads `AGNOS_CHAN`  |

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
deferral **#19**, still open.

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

- `tests/crab.tcyr` — the only suite `cyrius test` discovers. **228 passed / 0 failed** (94 in M3 —
  44 sorting *#33*, 12 selection memory *#34*, 10 argv *#11*, 28 deferred statting *#03*; 55
  mid-0.6.0, 37 at 0.5.0, 11 at 0.4.15). ⭐ It now includes **`src/app.cyr`**, not `ui.cyr`, so the application
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
- `tests/crab.bcyr` — ⛔ benchmark **scaffold**: it times `bench_noop`, i.e. nothing about crab.
- `tests/crab.fcyr` — ⛔ fuzz **scaffold**: `fuzz_main(data, len)` returns 0 without reading a byte
  of `data`, so `cyrius fuzz` would PASS against any input.
- `vet` 1 dep / 0 untrusted / 0 missing · `deny` 0 violations · `fmt --check` clean.

## Targets

| target       | status                                                    |
|--------------|-----------------------------------------------------------|
| x86_64 linux | ✅ builds, 394,248 B                                       |
| `--agnos`    | ✅ builds, 398,648 B — the real target                     |
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

⭐ **All of these are now sequenced in [`roadmap.md`](roadmap.md) with named upstream gates** — 0.5.0
harvested 39 deferrals out of comment prose and folded them into eight milestones. This section lists
only what a cold start must know *before touching the code*; the roadmap is the full inventory.

- ✅ **CLOSED 2026-08-27 — a rendered frame now costs the global heap ZERO bytes.** It was 746,440 B
  per `crab_render`, permanently, into a bump allocator with no `free()`. Three steps, each measured
  with a host probe taking `alloc_used()` deltas around back-to-back renders:

  | | per steady-state frame |
  |---|---:|
  | baseline (dhancha 0.9.12) | 746,440 B |
  | + step 1 (0.9.13) — `dh_surface_new`'s dead pixel buffer, deferred | 412,040 B |
  | + step 2 (0.9.14 + crab) — the sadish render target, reused | 77,568 B |
  | + step 3 (0.9.15 + crab) — the widget tree, arena'd | **0 B** |

  Identical at 114 entries per pane (the iron count for `/`) and at 256 (`CRAB_MAX_ENTRIES`).
  ⚠ **Zero is per-frame, not total** — crab pays a one-time **~597 KB** (334,432 B render target +
  262,144 B arena chunk), allocated on the first frame and reused for the process's life.
  ⚠ Neither step 1 nor step 2 was the "one line" two documents claimed, and step 2 was not purely
  upstream. All three are mutation-verified on both sides.
  ⛔ **Anything added to the render path must allocate through `dh_falloc`, not `alloc`** — a single
  plain `alloc()` in `crab_render` reintroduces a per-frame leak. `tests/crab.tcyr` asserts twenty
  rendered frames move the global heap by exactly 0 bytes, so the suite will say so.
  ⛔ **`dh_frame_begin` also clears dhancha's retained widget pointers** (`_dh_focus` and friends) and
  the two halves cannot be separated — never call `arena_reset` on a frame arena directly. It follows
  that crab **must re-establish focus every frame**, which `crab_pane` does. Full accounting in
  [`../architecture/001-every-frame-allocates-and-nothing-is-freed.md`](../architecture/001-every-frame-allocates-and-nothing-is-freed.md).
- ✅ **CLOSED 2026-08-27 — the "no continuously-repainting element" rule is LIFTED OUTRIGHT.** It had
  two halves and both are now zero: the frame (dhancha 0.9.13–0.9.15, see above) and the **poll**
  (dhancha 0.9.16 — `dh_setu_poll_event` called `setu_msg_new()` for 80 B before it knew whether
  anything was pending; the message is pure scratch and is now one hoisted buffer per process).
  **crab's render/input loop allocates nothing in steady state.** 200 idle polls move the global heap
  by exactly 0 bytes, asserted in dhancha's `poll_test`. *Deferral #09.*
  ⇒ The idle mascot line (#29), M4's transfer tray and M7's index progress are unblocked.
  ⚠ **An EVENT still costs 56 B** (`dh_event_new`) — that is per input, not per cycle, and bounded by
  how fast a human types. Only the idle path is free.
- ⚠ **`lib/` IS NOT WHAT COMPILES, and that sharpens the `path`-wins hazard considerably.** Measured
  2026-08-27: appending garbage to `lib/dhancha.cyr` leaves the build green, while appending it to
  `../dhancha/dist/dhancha.cyr` fails it — the `path` override compiles the **sibling's `dist/`**
  directly, and crab's vendored copy is a committed record that `cyrius deps` refreshes and
  `cyrius.lock` hashes. Appending garbage to `lib/alloc.cyr` is also harmless: the stdlib comes from
  the **installed toolchain**, not `lib/`. ⇒ This file's Toolchain note that "a toolchain bump without
  a `lib sync` leaves the stdlib behind" describes the snapshot, not the compiler input, and the
  stdlib's arena internals **cannot be mutation-tested from this repo**.
- ⛔ **`src/render_test.cyr`'s eleven pixel assertions never run in CI**, and CI **never builds
  `--agnos`** — the target this file calls the real one. Every `#ifdef CYRIUS_TARGET_AGNOS` region is
  uncompiled by the gate. CI also runs no fuzz, bench, lint, `fmt --check`, vet, deny or coverage.
- ⛔ **The fuzz and bench harnesses are both scaffolds** that measure nothing — see Tests above.
- ⛔ **The AI arc is promised in three shipped documents and declared nowhere.** The package
  description, the `[deps]` comment and the README all commit to semantic find / auto-tag / dedup on
  daimon's vector store; `cyrius.cyml` declares no daimon dependency. daimon 2.1.0 exists locally.
- ⚠ **crab is read-only.** No copy, move, rename, delete or mkdir. Enter on a *file* does nothing at
  all, silently — the return is discarded. (Roadmap M4.)
- ⚠ **The window is a compile-time 380×220 and crab is keyboard-only.** `WINDOW_CONFIGURE` is decoded
  by dhancha and dropped on the floor; the eight pointer event kinds dhancha synthesizes are all
  ignored. (Roadmap M2.)
- ✅ **CLOSED 2026-08-26 — the `cyrius init` placeholders and the stale README.** `CLAUDE.md` now
  carries a real identity line and `## Goal`; `README.md` § Status no longer says **"Scaffold."** (it
  had, since before the dual-pane GUI shipped in 0.2.0) and no longer lists that GUI under *Planned
  scope*. The retired `anu` codename and the rekha-TrueType claim are corrected — crab passes
  `font = 0` and draws with kashi's CP437 8×16 bitmap, calling no `rekha_*` function.
  ⚠ **The README's own replacement text over-claimed once before it landed** and was corrected against
  the code: there are **no per-row SIZE/MODIFIED columns**. A row is a single LABEL — 13-char name
  (truncation marked `~`) plus `/` for a directory or a size for a file — and the mtime appears only in
  the status line. Columns with headers are M3, gated on a dhancha table widget.

## Consumers

_None — top-level application._

## Next

⭐ **[`roadmap.md`](roadmap.md) is now a real plan** — the `cyrius init` template it had been through
fifteen releases is gone. Eight milestones from here to 1.0, sequenced against the design canvas at
the repo root, each carrying its named upstream gate.

✅ **M2 shipped as v0.6.1 and M3 as v0.7.0.** Immediately next is **M4 — file operations (v0.8.0)**:
crab is still **read-only** — no copy, move, rename, delete or mkdir, and Enter on a *file* does
nothing at all, silently.
⛔ **M4 is the first milestone that can destroy data**, which makes it a different kind of work from
everything shipped so far: every item before it was recoverable by pressing another key.
✅ Both of M2's memory gates are CLOSED and stayed closed (dhancha 0.9.13–0.9.16 plus the crab half):
a rendered frame costs the global heap zero bytes and the render/input loop is allocation-free in
steady state, so nothing about memory blocks anything downstream.

Two decisions are settled and recorded, and both shape everything downstream:
[ADR 0001](../adr/0001-compositor-owns-theming.md) (the compositor owns theming; crab ships no
palette) and [ADR 0002](../adr/0002-semantic-find-is-a-mode.md) (semantic find is a mode over any
view, not a view of its own).
