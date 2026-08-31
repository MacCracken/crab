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

**0.7.5** (2026-08-31) — see [`../../CHANGELOG.md`](../../CHANGELOG.md).

⛔ **THERE IS UNRELEASED WORK ON TOP OF 0.7.5 AND `VERSION` DELIBERATELY DOES NOT SAY SO.** M5's
first slice — the render-state record, the preview column, header-only image dimensions, a real fuzz
harness and a per-frame leak fix — is in the tree under `[Unreleased]` in the CHANGELOG. **The
operator drives version and toolchain bumps**; nothing here is committed or tagged.
⚠ Every number below marked *(unreleased)* is the working tree's, not 0.7.5's. Re-derive rather than
trust: `wc -l src/*.cyr`, `cyrius test`, `stat -c%s build/crab`.

⭐ M4 is now substantially in: `open`, the small-window ratification, and drag between panes, on top
of 0.7.1's write layer. ⚠ **Semver would normally make new user-facing features a MINOR**; this is
the third time this project has put feature work in a patch by operator ruling (M2 shipped in 0.6.1,
and 0.7.1 carried M4's first slice). **M4's completion still targets v0.8.0** — the roadmap's ladder
is unchanged, and these numbers were ruled, not derived.

⛔ **0.7.2 exists because 0.7.1's CHANGELOG section was being edited after its tag was pushed.**
Three commits landed past `4ac21eb` while their notes were still being written into the released
section. A released section is not a scratchpad: editing one after its tag makes the tag and the
notes disagree, and the notes are what a consumer reads. 0.7.1's section is now byte-identical to
the tag and everything since is under 0.7.2.

⭐ Refreshed **in the same commit as the CHANGELOG entry**, which is the rule the header below
demands and the one this file broke twice.

## Toolchain

- **Cyrius pin**: `6.5.36` (in `cyrius.cyml [package].cyrius`) — moved 2026-08-30 with the defect
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
- ⛔⛔ **THE LOCAL `~/.cyrius` 6.5.35 SNAPSHOT IS POISONED (2026-08-28)** — overwritten with 6.5.36
  stdlib. The **released** 6.5.35 tarball has **0** hits for `v6.5.36` and **0** for `SYS_READDIR_AT`;
  the local copy has both. ⇒ **Every `cyrius build` / `cyrius test` re-vendors 6.5.36 stdlib into
  crab's tracked `lib/` and rewrites `cyrius.lock`.** After any build:
  `grep -c SYS_READDIR_AT lib/syscalls_x86_64_agnos.cyr` must be **0**, and `git status lib/` should be
  clean; `git checkout -- lib/ cyrius.lock` undoes it. Root fix (ask the operator first — they may be
  developing 6.5.36 on purpose):
  `curl -sSf https://raw.githubusercontent.com/MacCracken/cyrius/main/scripts/install.sh | CYRIUS_VERSION=6.5.35 sh`
- ⭐ **6.5.36 IS RELEASED (verified 2026-08-30)** — tagged on the remote. This meets the documented
  expiry condition for `CRAB_SYS_READDIR_AT = 101`. ⚠ **The pin has NOT moved and must not move
  here** — the operator drives version and toolchain bumps — so the hardcoded constant stays until
  it does. This line said "UNRELEASED" until 2026-08-30; that was true when written.
- `lib/` is vendored from the pinned snapshot by `cyrius lib sync`, **not** by `cyrius deps` — a
  toolchain bump without a `lib sync` leaves the stdlib behind. ⚠ The sync walks the **declared**
  `[deps].stdlib` set: **29 leaves**, while crab actually vendors **30**. The odd one out is
  `lib/atomic.cyr`, a transitive leaf (`alloc`, `io`, `fmt` and three `syscalls_*` include it) that no
  declaration names. Check it by hand at every bump.

## Source

**6,829 lines** across **six** files *(unreleased; 5,368 at 0.7.5, 2,227 at the 0.7.0 cut)*.
⚠ This section read "2019 lines" and per-file counts from **before** the 0.7.0 cut — it was already
stale by ~200 lines when the cut landed. Re-derive with `wc -l src/*.cyr`, never trust the numbers
here.
⚠ +1,461 over 0.7.5 *(unreleased)*: the render-state record, the preview column, the header-only
dimension parser and the memoised read behind it — plus the tests, which are the larger half.

- `src/main.cyr` (1,287) — ⛔ **`main()` AND `_entry()` AND NOTHING ELSE, as of 0.6.0.** It ends in
  `_entry();`, so a test that included it would RUN THE APP — which is why everything testable was
  moved to `src/app.cyr`. **Do not add a function here.** What remains is the dhancha client
  lifecycle, the shm present path and the frame loop.
  ⚠ The **present path stays hand-rolled on purpose**: a LIVE shared buffer (create once, rewrite in
  place) against `dh_client_present`'s per-frame ATTACH+COMMIT. Different models — swapping them is
  not a rename.
  ⚠ **No arena setup here any more** (0.6.0). It lived in `main()` and that was a gap: deleting it
  broke no test while restoring a 77 KB-per-frame leak. `crab_render` owns it.
- `src/app.cyr` (2,484) — ⭐ **NEW at 0.6.0**, and M3's centre of gravity; M4's write layer lives
  here too (`crab_fs_*`, the per-target syscall shim, `crab_name_ok`). The application layer lifted out of `main.cyr`: the
  readdir parser (`crab_readdir_into` and its cap clamp), the stat layer, `crab_descend` /
  `crab_ascend`, `crab_surface_flags`, the serial logging.
  ⛔ It exists for the same reason `path.cyr` does — **none of it was reachable from any test** while
  it lived in `main.cyr`, in a program whose two shipped defects were both found on iron. 0.6.0 added
  17 assertions against it, all mutation-proven.
  ⚠ `CRAB_MAX_ENTRIES = 1024` (256 until M3 *#02*) and `crab_surface_flags()` returning `SETU_SURF_PREMULTIPLIED`
  **unconditionally** — no flag, no arm, no env var — both live here.
- `src/path.cyr` (131) — the readdir record layout and the **bounded** cstring/path helpers
  (`crab_cstr_len`, `crab_streq_n`, `crab_strcpy_n`, `crab_join_n`).
  ⛔ Extracted at 0.5.0 for the reason `app.cyr` was extracted at 0.6.0: a memory-safety fix that
  cannot be asserted on is a fix held on trust.
- `src/ui.cyr` (2,067) — dual-pane file browser: a pane is a **`dh_list`** (0.4.10), plus size/mtime
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
  `build/crab-render.bin`. **40** `check()` assertions *(unreleased; 26 at 0.7.5)*.
  ⭐ **IT NOW PRINTS ITS OWN CHECK COUNT.** It returned `g_fails` and printed only its dump line, so
  it exited **0** whether it ran 26 checks or none — while this file and the handoff quoted a count
  for three cuts that nothing in the program printed. `crab render_test: 35 checks, 0 failed`.
  ⚠ **It renders at 640x220, not 380** — every assertion in it is about two panes side by side, and
  below 600 crab now draws one. The solo layout has its own checks at 380.
  ⚠ It creates **two** `DhSurface`s on purpose — it holds two frames live and dumps the first, and
  since dhancha 0.9.14 one surface would hand both renders the same target.
  ⛔ **It is not run by CI or by `cyrius test`** — see Known gaps.
- `src/test.cyr` (13) — ⚠ **deliberately empty, with a warning in it.** Bare `cyrius test`
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

Declared in `cyrius.cyml`. ⭐ **Re-verified 2026-08-28: every declared tag equals that repo's highest
tag ON ITS REMOTE**, and the declared graph resolves with all `path` overrides disabled (**6 deps / 0
errors**, host and `--agnos` both build, **253/0**).

| dep     | tag    | `path`? | why crab needs it                                   |
|---------|--------|---------|-----------------------------------------------------|
| sadish  | 0.5.2  | no      | 2D vector — the surface everything else draws into   |
| rupa    | 0.1.5  | yes     | shared theme tokens + **`on-accent`** and contrast   |
| rekha   | 0.3.5  | no      | text; references `sd_*`                              |
| kashi   | 1.0.6  | yes     | CP437 8×16 glyph data for `dh_draw_text` (font=0)    |
| dhancha | 0.9.24 | yes     | widgets, **columns/`dh_table_*`**, `dh_theme_*`      |
| chitra  | 1.0.0  | **no**  | **thumbnails** — PNG/JPEG/GIF/BMP decode. ⛔ see gaps |
| setu    | 0.8.8  | yes     | client transport — channel-band, reads `AGNOS_CHAN`  |

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

- `tests/crab.tcyr` — the only suite `cyrius test` discovers. **966 passed / 0 failed**
  *(unreleased; 757 at 0.7.5, 253 at the 0.7.0 cut)*
  ⛔ **NEW GROUPS GO IN THEIR OWN FUNCTIONS.** `main` reached 2,517 lines and **279 locals**, and one
  more group pushed its stack frame past what the process could touch: the suite **segfaulted**
  (exit 139) part-way through, having printed the group header, with every prior assertion passed.
  It looked exactly like a bug in the code under test. `t_img_dims`, `t_preview` and
  `t_preview_dims` are separate functions; anything added from here on should be too. (119 in M3 —
  44 sorting *#33*, 12 selection memory *#34*, 10 argv *#11*, 28 deferred statting *#03*, 25 real
  columns *#32*; 55 mid-0.6.0, 37 at 0.5.0, 11 at 0.4.15). ⭐ It now includes **`src/app.cyr`**, not `ui.cyr`, so the application
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
- `tests/crab.fcyr` — ✅ **A REAL FUZZ HARNESS as of the unreleased work (*deferral #12* CLOSED).**
  It was a scaffold that read none of its input, so `cyrius fuzz` PASSED against anything. It now
  drives **60,000 rounds** over mutated format headers, random bytes, degenerate/negative lengths,
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
| x86_64 linux | ✅ builds, **1,003,168 B** *(unreleased; 453,304 at 0.7.5, 398,504 at the 0.7.0 cut)* |
| `--agnos`    | ✅ builds, **1,026,872 B** *(unreleased; 474,944 at 0.7.5)* — the real target |
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

- ⛔ **No security audit exists, and crab now writes.** `docs/audit/` is empty. Since 0.7.1 crab
  creates, renames, copies, moves, **deletes recursively**, and **spawns processes**. That is the
  v1.0 criterion that moved furthest while nobody was looking at it.
- ⛔ **`src/render_test.cyr`'s 26 pixel assertions run under NEITHER CI NOR `cyrius test`.**
  `cyrius.cyml` sets `test = "src/test.cyr"`, which `cyrius test` does not run, and auto-discovery
  covers `tests/*.tcyr` only. It is crab's strongest test and no gate executes it. *Deferral #14.*
- ⛔ **CI never builds `--agnos`** — the target that ships. Every `#ifdef CYRIUS_TARGET_AGNOS` region
  is unguarded by the gate, and 0.7.5 proved the cost: a brace error there compiled clean on the host
  and failed only on `--agnos`. *Deferral #15.*
- ✅ **CLOSED (unreleased) — the fuzz harness now reads its input.** *Deferral #12.* 60,000 rounds
  over mutated headers, random bytes and degenerate lengths, deterministic from a fixed seed. It
  caught a real segfault in `crab_img_dims` the day it was written.
  ⚠ **What it still does NOT cover**: `crab_readdir_into` (agnos-only, needs a syscall), the write
  layer's path joins, and `crab_batch_name`'s pattern expansion. Those are the next targets.
- ⛔ **The AI arc is promised in three shipped documents and declared nowhere.** The package
  description, the `[deps]` comment and the README all commit to daimon; `cyrius.cyml` declares no
  daimon dep, and **daimon 2.1.2 exists locally**. *Deferral #18.*
- ✅ **CLOSED (unreleased) — `crab_render` takes one record, not 32 positional parameters.** (The
  count here said 33; it was 32.) `crab_rs_pane` / `_op` / `_chrome` / `_preview` / `_dims` fill it,
  max arity 11, and `crab_rs_reset` owns the three `-1`-means-unknown defaults that 23 call sites
  used to spell by hand.
- ⚠ **The idle-tick wiring is untested and structurally untestable on the host.** The walk, the copy
  stepper and the queue are all driven by hand in the suite; that they are *called* from the tick is
  agnos-only event-loop code. Same irreducible gap `main.cyr` has always had.
- ⚠ **Shift is not on the wire.** `mods` carries press/release, so typed names are lower-case until
  the compositor forwards modifiers. `crab_key_char` already takes the flag.
- ⚠ **crab cannot recreate a symlink.** A recursive copy copies whatever `open`+`read` yields through
  one. `sys_symlink` exists but crab cannot *learn* a source is a link without `readlink`'s ambiguous
  negative. Moves when agnos `lstat`#102 gets its cyrius peer.
- ✅ **CLOSED (unreleased) — `README.md` § Status.** It said *"Shipping, and read-only"*, which M4
  falsified, and quoted the retired 256-entry cap. Now states the write layer, the preview column,
  and what is genuinely absent. ⚠ **It has been wrong in both directions now**; the replacement text
  carries that warning itself.
- ⚠ **Focusing a pane by its header does not work** — the header is a sibling of the list, so
  `crab_hit` resolves a header click to no pane. Clicking a row is correct.
- ⛔⛔ **`crab_fs_open_w` BEHAVES DIFFERENTLY ON THE TWO TARGETS, AND THE TARGET THAT SHIPS IS THE
  PERMISSIVE ONE.** The host arm is `O_WRONLY|O_CREAT|O_EXCL` — M4's overwrite guard, which refuses
  an existing file and returns `EEXIST` — while the agnos arm is `AO_WRONLY|AO_CREAT|AO_TRUNC` with
  **no `AO_EXCL`**, so on agnos the same call TRUNCATES an existing file. Every host assertion about
  "crab will not overwrite" is therefore a claim about the host only. Found (unreleased) while
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
- ⚠ **chitra 1.0.1 is prepared but NOT pinned here** — the tag must exist on a remote first (the
  2026-08-28 phantom-tag rule). It makes the >16 MiB PNG failure cheap and correctly named; crab's
  per-image budget already keeps it clear of that path, so nothing is blocked.
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

**M4 is complete. M5's three ungated items are IN.**

⭐ **Done (unreleased)**: the `crab_render` parameter cleanup, the preview pane with real metadata,
header-only image dimensions, and **thumbnails** — plus *deferral #12* (the fuzz harness) closed, a
shipped per-frame leak in `crab_overlay` fixed, and dhancha bumped to 0.9.24.

**Start here, in this order:**

1. ⛔⛔ **BURN ON IRON. This is the oldest and largest untested stretch in the project's history, and
   it just got larger.** The last iron run was **2026-08-30 against the 0.7.0 tree**. Everything
   since — the whole write layer, the tray, recursion, the menu, the edit field, the record refactor,
   the preview and now a decoder that **more than doubled the binary** — has run only on the host
   and under QEMU. **Both defects crab has ever shipped were iron-only.**
   ⚠ agnos moved 1.56.53 → 1.56.55 underneath, including a rewrite of `is_user_range`.
   ⛔ **And the thumbnail path is the most iron-sensitive thing crab has ever shipped**: it is the
   first code whose failure mode is *memory exhaustion over time* rather than a wrong pixel, and
   QEMU is explicitly not a control for pressure-dependent behaviour.
2. ⚠ **Pin chitra 1.0.1 once it is pushed**, then re-run check 4. It is prepared locally: the
   >16 MiB PNG refusal becomes cheap and correctly named (`CHITRA_ERR_INFLATE_LIMIT`), and its
   sidecar drops three unused stdlib leaves. Nothing is blocked on it.
3. ⭐ **Adopt dhancha 0.9.24's stable keys and delete three workarounds.** crab hand-rolls its own
   press tracking, its own drag state and its own edit buffer because dhancha identified widgets by
   pointer and crab's arena invalidates pointers every frame. 0.9.24 fixes that at the cause —
   `dh_widget_set_key` + `dh_surface_set_root` re-binding — so `dh_dispatch`, drag-under-arena and
   `dh_text_attach` are all now usable. ⚠ **crab uses none of it yet.** That is its own change.
4. **The `crab_fs_open_w` target divergence** — the shipping target has no overwrite guard, because
   agnos has no `AO_EXCL`. Filed; see *Known gaps*.
5. Then the gated pair — grid/columns and the sidebar, both **dhancha**. Re-derived 2026-08-31:
   `dh_grid_new`, `dh_columns_new`, `dh_tree_new` and a menu BAR are all genuinely absent.
   ⭐ **These two gates are REAL**, unlike the three false ones this project has recorded.

⛔ **Before believing any gate, re-derive it.** Three false gates on record: M4's *"Gate: agnos write
syscalls"* (real since agnos 1.41.3), drag's *"gated on nothing"* (it was gated, elsewhere), and
thumbnails' *"no image decoder"* — where chitra existed, and the real obstacle turned out to be a
**cost** no gate line mentioned. ⇒ A gate can be false in both directions.

Two decisions are settled and shape everything downstream:
[ADR 0001](../adr/0001-compositor-owns-theming.md) (the compositor owns theming; crab ships no
palette) and [ADR 0002](../adr/0002-semantic-find-is-a-mode.md) (semantic find is a mode over any
view, not a view of its own).
