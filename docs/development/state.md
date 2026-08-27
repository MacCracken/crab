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

**0.5.0** (2026-08-26) — see [`../../CHANGELOG.md`](../../CHANGELOG.md).

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

982 lines across five files.

- `src/main.cyr` (407) — entry, **dhancha** client lifecycle (`dh_client_connect` / `dh_client_fd` /
  `dh_client_poll_event` / `dh_client_close`), the readdir+stat layer, the frame loop.
  ⛔ It does **not** open its own setu connection (since 0.4.8) and does **not** run its own
  `setu_poll_input` switch (since 0.4.12) — depending on the toolkit for pixels while bypassing it for
  the transport was the bug, not the style.
  ⚠ The **present path stays hand-rolled on purpose**: a LIVE shared buffer (create once, rewrite in
  place) against `dh_client_present`'s per-frame ATTACH+COMMIT. Different models — swapping them is
  not a rename.
  ⚠ `CRAB_MAX_ENTRIES = 256` and `crab_surface_flags()` returning `SETU_SURF_PREMULTIPLIED`
  **unconditionally** — no flag, no arm, no env var — both live here.
- `src/path.cyr` (91) — the #81 record layout and the **bounded** cstring/path helpers.
  ⛔ It is a separate file for one reason: `tests/crab.tcyr` includes `ui.cyr`, never `main.cyr`
  (which ends in `_entry()`, so including it would run the app). Nothing in `main.cyr` is reachable
  from the suite, and 0.5.0's P1 repair lived exactly there. A memory-safety fix that cannot be
  asserted on is a fix held on trust.
- `src/ui.cyr` (326) — dual-pane file browser: a pane is a **`dh_list`** (0.4.10), plus size/mtime
  formatting, row build, status line.
  ⛔ **`crab_render` TAKES THE SURFACE, it does not create one** (step 2 of the allocation gate). It
  used to call `dh_surface_new` per call, which threw away dhancha 0.9.14's per-DhSurface render-target
  cache every frame. `w`/`h` are read back off the surface rather than passed, so they cannot disagree
  with it. ⚠ The returned surface is **reused** — two frames at once need two surfaces.
  ⛔ `CRAB_ROWS_CAP = 7` is **gone** and deliberately not replaced by a bigger number — it was never a
  display limit, it made entries past the 7th unselectable.
  ⚠ **A row sets no background at all.** dhancha paints selection and focus from the list's own state;
  a row that keeps a background paints over the toolkit's highlight and selection silently stops
  showing.
- `src/render_test.cyr` (145) — a standalone harness: renders the production surface at 380×220 and
  dumps BGRA to `build/crab-render.bin`, 10 `check()` assertions.
  ⛔ **It is not run by CI or by `cyrius test`** — see Known gaps.
- `src/test.cyr` (13) — ⚠ **deliberately empty, with a warning in it.** Bare `cyrius test`
  auto-discovers `tests/*.tcyr` and does **not** run the `[build].test` hook.

⚠ `cyrius coverage` reports **14/26 fns referenced (53 %)** — up from 23 % at 0.4.15, and still a
floor rather than a correctness proof. `cyrius lint` is clean apart from **9** lines over 120 characters (8 in `main.cyr`, 1 in `ui.cyr` — re-derived 2026-08-26; this file said 8).

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

⛔ **And QEMU EARNED ITS KEEP: it caught a regression the host suite could not see.** A 0.5.0 draft
idled with `sys_sleep_ms`, which `preempt_disable()`s — so while crab slept **nothing else could be
scheduled** and the compositor never presented at all (placed 2, presented **0**, `--clients` never
returned). The host suite was **37/37 green for that build**, because the loop is inside
`#ifdef CYRIUS_TARGET_AGNOS`. Shipped primitive is `sys_pause` (#14), which yields to a ready proc
first. ⇒ **Any change to the agnos event loop needs a QEMU run before it is claimed.**

⚠ **Not exercised on agnos:** `crab_descend` / `crab_ascend`. Neither harness drives navigation keys,
so the bounded-join *refusal* path has host assertions only.

## Dependencies

Declared in `cyrius.cyml`. **Five of six are at their latest published tag as of 2026-08-26; dhancha
is not** — `../dhancha` holds an uncommitted, untagged **0.9.14** (step 2 of the allocation gate) and
`path` wins over `tag`, so the local build compiles 0.9.14 while the manifest declares 0.9.13. The tag
is left behind deliberately; see the ⛔ at `[deps.dhancha]` in the manifest.

| dep     | tag    | `path`? | why crab needs it                                   |
|---------|--------|---------|-----------------------------------------------------|
| sadish  | 0.5.2  | no      | 2D vector — the surface everything else draws into   |
| rupa    | 0.1.4  | yes     | shared desktop theme tokens (`RupaMotion` since .3)  |
| rekha   | 0.3.5  | no      | text; references `sd_*`                              |
| kashi   | 1.0.6  | yes     | CP437 8×16 glyph data for `dh_draw_text` (font=0)    |
| dhancha | 0.9.13 | yes     | widgets, `dh_client_poll_event`, `dh_theme_*` ⛔ **local tree is 0.9.14, unpublished** |
| setu    | 0.8.7  | yes     | client transport — channel-band, reads `AGNOS_CHAN`  |

⭐ **dhancha 0.9.13 (2026-08-26) is the per-frame allocation fix** — `dh_surface_new` no longer
allocates a `w*h*4` pixel buffer that nothing in the stack ever wrote or read. See Known gaps for the
measurement. Published as `c273159`, and the tag was verified **four** ways before crab's manifest
moved, because `path` wins over `tag` and a green local build proves nothing about the declared graph:

1. sibling `VERSION` = `0.9.13`;
2. `git rev-parse 0.9.13` == `HEAD` in `../dhancha`, working tree clean;
3. `git ls-remote --tags` shows `refs/tags/0.9.13` at that same commit;
4. ⭐ **and the declared graph was actually resolved** — with `path` temporarily disabled,
   `cyrius deps` cloned the tag from the remote and produced a `lib/dhancha.cyr` with the **identical**
   SHA-256 (`0972d27b…`) as the `path` build and as `git show 0.9.13:dist/dhancha.cyr`. That fourth
   check is the only one that would have caught 0.4.13's manifest naming a library the build never
   compiled; the first three would all have passed it.

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

- `tests/crab.tcyr` — the only suite `cyrius test` discovers. **45 passed / 0 failed** (37 at 0.5.0,
  11 at 0.4.15). The 8 added cover the reused render target: identity, frame independence over a full
  334,400-byte compare, and a scribbled-sentinel coverage check.
  ⛔ **The sentinel check has TWO independent guarantors and neither mutation alone fails it** —
  deleting dhancha's `sd_clear` leaves it green (crab's opaque full-window root still covers), and
  making crab's root transparent leaves it green (the clear still covers); **only removing both
  fails**, at 7,744 surviving bytes. Measured, not assumed. ⇒ A green suite here is **not** evidence
  that the toolkit still clears — that is pinned in dhancha's own `programs/draw_test.cyr`.
  Covers the AE-6 premultiplied `#92` contract on the **production** `crab_render`
  (`a == 255` and `c <= a` across all 83,600 pixels, with negative controls), plus 0.5.0's repairs:
  the bounded path helpers, the size ladder at every boundary, the date-formatter clamps, and the
  name-truncation marker.
  ⭐ **Every 0.5.0 assertion is mutation-proven** — reverting each bound, the overflow fix and the
  truncation marker each produce a named failure.
  ⛔ The truncation test was rewritten because its first draft **could not fail**: it mirrored
  `crab_row`'s loop in the harness and asserted on the copy, so deleting the marker from the
  production function left the suite green. It now drives the real `crab_row` through a real
  `dh_list`. A test that mirrors the code under test is measuring itself.
- `src/render_test.cyr` — 11 further pixel assertions, mutation-proven against four regressions.
  ⚠ It now creates **two** `DhSurface`s, on purpose: it holds two frames live (asserts on the first,
  renders the second, then dumps the FIRST at the end), and since dhancha 0.9.14 one surface would
  hand both renders the same target. The added `check(sds2 == sds, 0)` is the guard — without it, a
  future regression to a shared target would silently make it dump the wrong picture.
  ⛔ **Never runs in CI or under `cyrius test`.**
- `tests/crab.bcyr` — ⛔ benchmark **scaffold**: it times `bench_noop`, i.e. nothing about crab.
- `tests/crab.fcyr` — ⛔ fuzz **scaffold**: `fuzz_main(data, len)` returns 0 without reading a byte
  of `data`, so `cyrius fuzz` would PASS against any input.
- `vet` 1 dep / 0 untrusted / 0 missing · `deny` 0 violations · `fmt --check` clean.

## Targets

| target       | status                                                    |
|--------------|-----------------------------------------------------------|
| x86_64 linux | ✅ builds, 381,536 B                                       |
| `--agnos`    | ✅ builds, 381,592 B — the real target                     |
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

- ⚠ **Every frame allocates ~78 KB and nothing is ever freed — 89.6 % NARROWED, still not closed.**
  Reproduced and then re-measured with a host probe (`alloc_used()` deltas around five back-to-back
  `crab_render` calls at 380×220, 114 entries per pane, into **one** surface):

  | | per frame |
  |---|---:|
  | baseline (dhancha 0.9.12) | 746,440 B |
  | + step 1 (dhancha 0.9.13) | 412,040 B |
  | + step 2 (dhancha 0.9.14 + crab) | **77,568 B** |

  **Step 1** deferred `dh_surface_new`'s pixel buffer, which was allocated on every surface and
  **never written and never read** by anything in the stack. **Step 2** cached the sadish render
  target on the DhSurface — ⛔ **and that half was NOT purely upstream**, contrary to what two
  documents said: `crab_render` was minting a fresh `DhSurface` every call, so a per-DhSurface cache
  would have been thrown away each frame. crab now holds **one surface for the session**
  (`src/main.cyr`) and `crab_render` takes it as a parameter, reading `w`/`h` back off it — 18 → 17
  parameters, shorter for gaining one.
  ⚠ **Step 2 is a CONTRACT CHANGE**: `dh_surface_render` may now return the same surface twice. A
  caller wanting two frames at once needs two `DhSurface`s; `src/render_test.cyr` is exactly that
  caller and creates two, guarded by `check(sds2 == sds, 0)`.
  ⚠ Neither step was "one line", which the roadmap and the architecture note both claimed.
  ⛔ **What remains is the widget tree — ~236 records at 248 B, now 75 % of the frame** — plus crab's
  own ~19 KB of row/status scratch (was under 5 % of the problem, now roughly a quarter). Closing it
  needs an arena hook in dhancha; the stdlib ships `arena_new`/`arena_reset` for exactly this, but
  dhancha allocates via plain `alloc()` at 19 sites.
  ⚠ **The repaint rule still stands.** 77,568 B at 60 Hz is 4.7 MB/s into an allocator with no
  `free()` — the rule was never about the factor. ⭐ What changed is the margin: an element repainting
  a few times a minute is now clearly affordable, where at 750 KB a frame it was not. Full measurement
  in [`../architecture/001-every-frame-allocates-and-nothing-is-freed.md`](../architecture/001-every-frame-allocates-and-nothing-is-freed.md).
- ⛔ **`src/render_test.cyr`'s ten pixel assertions never run in CI**, and CI **never builds
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

Immediately next is **M2 — the window is real** (v0.6.0): resize, pointer input, key release, routing
through `dh_dispatch`, and the event-driven wait. ⛔ **Its dhancha gate — per-frame allocation —
blocks every milestone after it**, so it is the first thing to file upstream rather than the last.

Two decisions are settled and recorded, and both shape everything downstream:
[ADR 0001](../adr/0001-compositor-owns-theming.md) (the compositor owns theming; crab ships no
palette) and [ADR 0002](../adr/0002-semantic-find-is-a-mode.md) (semantic find is a mode over any
view, not a view of its own).
