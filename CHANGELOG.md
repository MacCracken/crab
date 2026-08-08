# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased] — `AE-6`: crab's surface is PREMULTIPLIED, so the desktop composites it on the shader

⭐⭐ **crab now declares `SETU_SURF_PREMULTIPLIED`, unconditionally.** That routes it through agnos
`gpu_shader_op #92` op 0x01 — a real per-pixel `out = src + dst * (1 - src_a)` on the compute units —
instead of `#87`, a byte mover with no ALU that can only copy. The blend is the one operation CP-DMA
structurally cannot do, and until now it had **no client at all**.

**crab satisfies the op's precondition by construction.** sadish's `sd_alpha_of()` maps a bare `sd_rgb`
value's 0 byte to 255 and `draw.cyr` stores that, so every pixel crab produces is **alpha 255** — trivially
premultiplied, since at a == 255 premultiplied and straight alpha are the same bytes.

⚠ **No flag, no arm, no env var.** The client states what its pixels ARE; the compositor decides how to draw
them. A boot whose GPU has no shader envelope falls back to `#87` **per window** in
`ae_gpu_present_frame` — exact at alpha 255 — instead of costing the frame its hardware compositing.

### Added — `tests/crab.tcyr`: the premultiplied contract, per pixel, on the production render

crab's entire test coverage was `assert(1, "true is true")` and `1 + 1 == 2`, while the arc's docs carried
"crab is alpha-255 clean throughout" as a load-bearing fact. It was true by inheritance from sadish, with
nothing pinning it. The suite now renders the **production `crab_render`** and asserts `a == 255` **and**
`c <= a` — the op's actual contract, not a proxy — on all 83,600 pixels, so if crab ever gains a translucent
element the gate fails and names `sd_premul()` as the fix. **11/11**, with negative controls for each check.

⛔ **The negative controls caught a bug in themselves first.** They seeded `pix + 4*3` for "pixel 4, byte 3"
— a pixel is **4 bytes**, so that wrote pixel 3's blue channel, left every alpha at 255, and the scan
correctly found nothing. Without them the suite would have read green while testing nothing.

⚠ `src/test.cyr` is now deliberately empty with a warning in it: bare `cyrius test` auto-discovers
`tests/*.tcyr` and does **not** run the `[build].test` hook, so a gate written there never executes.

## [0.4.5] - 2026-08-07 — the setu handshake fails out loud

### Changed — setu > 0.8.1: crab presents on the agnos channel band

⭐ crab is the **second** client in agnos 1.56.40's ipc bite 7: the compositor mints a channel, endows
one end, and spawns crab already connected — crab dials nothing and `setu_connect` reads `AGNOS_CHAN`.
Proven under QEMU `-smp 4` alongside `present_probe`: both present, framebuffer-confirmed.

Pinned to **setu 0.8.4**, no override. That tag also removes TCP from setu's Linux arm (AF_UNIX /
SOCK_SEQPACKET), so crab speaks a record transport on both targets and its handshake cannot be correct
on one and broken on the other for a framing reason.

⚠ **`chrono` added to `[deps] stdlib`.** crab was missing it and building clean anyway: setu's
`dist/setu.deps` under-reported 8 of 12 leaves, which switched OFF `cyrius deps`' own validation. setu
0.8.4 corrects its sidecar, the check fires again, and this gap became a hard error. Requires
**agnos >= 1.56.40**.

### Added — the three silent `return 1`s in the handshake now say what failed

The CREATE_SURFACE send, the SURFACE_CREATED read, and the reply-kind check each returned bare `1`. A
client that exits 1 with no output is indistinguishable from a client that never ran, and that
ambiguity cost real time: the compositor's serial log showed the reply being *sent*, and only crab
could say whether it arrived. Each now prints before returning.

## [0.4.4] - 2026-08-02

### Changed — setu 0.7.2: crab can actually connect to the compositor on agnos

> ⛔ **SUPERSEDED 2026-08-03 — the transport this entry repairs is RETIRED as the wrong primitive.**
> TCP-on-loopback was the WRONG PRIMITIVE for a local display protocol: it was picked because a TCP
> stack happened to exist, was never put to the operator, and accumulated six accommodations (the
> `sys_net_ip()` dial below being the last of them). A local display protocol has nothing to route,
> nothing to checksum, no window to negotiate, and no business owning a port. The desktop transport is
> now the agnos socket (`anu`) — agnos `docs/development/planning/ipc.md` §9/§10.
>
> ⚠ **What this entry claims is nevertheless TRUE and is NOT withdrawn.** With `net_src_for`
> (agnos 1.56.34) the handshake completes on an ordinary boot, and on 2026-08-02 the honest harness
> `agnos/scripts/harness/aethersafha-clients-test.py` — which byte-scans `build/agnos` and hard-exits if
> the kernel carries any selftest hook — reached **`connected: 2, presented: 2`** with **crab** and
> setu's `present_probe` (`/bin/puka`) as the two clients. Scope it honestly: QEMU at `-smp 1`, never
> shown on iron, `-smp 4` fault-kills. Retirement is architectural — do not restore a TCP dial, but do
> not record this result as a false green either.

Picks up setu **0.7.2**, whose `setu_connect` dials `sys_net_ip()` instead of `127.0.0.1`.

⛔ **Before this, crab could never present on a real agnos boot.** agnos puts `net_ip` in an outbound
SYN's source field, so a connect to 127.0.0.1 produced a SYN-ACK the client's own conn could not
match on the 4-tuple; `sock_connect` #47 returned -1 instantly and crab printed
`crab: setu connect failed`. It looked like a crab fault and was not — the identical failure hit
setu's own `present_probe`. The `AETHERSAFHA_SETU_SELFTEST` kernel hook had been masking it by
assigning `net_ip = 0x7F000001`, which is why the desktop smoke passed while no ordinary boot worked.

No source change here — this is a dependency bump and a rebuild.

⚠ **crab consumes setu's `dist/` bundle**, not its `src/`, so a setu fix only reaches this client
after `cyrius distlib` runs in setu. A local `path` override alone is not enough.

### Verified

Composited as a live window on agnos in QEMU, launched **in the foreground** (`aethersafha`, no `&`)
from the agnsh prompt: the dual-pane file manager rendering real `/bin` contents — checked on the
**framebuffer**, not just the serial log.

### Note — corrected 2026-08-02

This entry originally said *"0.4.3 was never released — `VERSION` read 0.4.3 with no CHANGELOG section."*
That was wrong: a 0.4.3 section existed, misfiled **below** 0.4.2 behind an empty `[Unreleased]` heading,
which is why a search for it came up empty. The ordering is repaired and 0.4.3 now sits in sequence; no
content was invented or dropped.

## [0.4.3] - 2026-08-02

### Changed — cyrius pin 6.4.71 -> 6.5.5; dhancha 0.9.3, sadish 0.5.1, rupa 0.1.2, rekha 0.3.4, kashi 1.0.4, setu 0.7.1

Part of the whole-desktop-stack toolchain catch-up cut on this date.

⭐ crab picks up **setu 0.7.1** and is, as of today, the only setu client in the stack whose surface
is already alpha-255 clean throughout — which makes it the natural first candidate when the
compositor's premultiplied `#92` blend path gets a real consumer.

### Verification

Host + `--agnos` builds green; 1 suite passes.

## [0.4.2] - 2026-07-23

### Changed — setu 0.7.0 (`SETU_SURF_PREMULTIPLIED`) + dep refresh

No behaviour change: the flag is opt-in and this client does not set it, so its surface is still composited
with the opaque `gpu_blit_shm` #87 path.

## [0.4.1] - 2026-07-23

### Changed — setu 0.6.0: client buffers are GPU-visible on agnos

Picks up `setu` **0.6.0**, whose `setu_buf_create` now asks for `shm_create_gpu` **#86** before falling back
to `shm_create` **#71**.

⚠ **Why this matters beyond a version number.** `#71` allocates **system RAM**, which the agnos GPU cannot
reach at all — bus-master is off by design and the engines see only the framebuffer aperture. The kernel
rejects a `#71` slot at both GPU entry points (`gpu_blit_shm` #87: `src_mc == 0 ⇒ the GPU cannot read it`;
`gpu_shader_op` #92: `GPO_E_BADSLOT`). Every shared surface in the desktop was allocated that way, so the
whole iron-proven ring-3 GPU band had **no reachable consumer**. Buffers from this release are eligible for
a hardware blit.

No API change and no call-site change here — the buffer id behaves identically, and `#86` falls back to
`#71` automatically on a machine with no GPU carveout (every QEMU boot).

### Changed — cyrius pin → 6.4.71

## [0.4.0] - 2026-07-12 — follows the shared desktop theme (rupa)

crab now colours itself from the active desktop theme instead of a hardcoded dark
palette, so the file manager matches the compositor chrome and every other dhancha app.
Switch the whole desktop's look with `rupa_theme_set_active_name("shanta-dark")` and crab
re-colours with it — two themes, each dark + light: MUDRA (the seal, default) and SHANTA
(stillness).

### Changed

- **Panes, rows, headers, and the status line draw with `dh_theme_*`** (dhancha's theme
  helpers over the shared **rupa** token core), replacing the hardcoded `sd_rgb(...)`
  literals in `src/ui.cyr`: root → `dh_theme_bg`, pane column → `dh_theme_panel`, list rows
  → `dh_theme_widget`, header + status → `dh_theme_panel`. The **selected** row / active
  header is now the theme **accent** when its pane is focused, and a muted `dh_theme_line`
  tint when it isn't (was a fixed blue). This makes crab legible under the light themes.
- **`[deps.dhancha]` 0.8.0 → 0.9.0** (the `dh_theme_*` API) + new **`[deps.rupa]` 0.1.0**
  (the shared theme tokens). Builds green against published rupa@0.1.0; `test` + `render_test`
  unaffected.

## [0.3.2] - 2026-07-10 — file mtime (status line)

Each entry's modification time joins its size: a Midnight-Commander-style status bar along
the bottom shows the active pane's selected entry — name, size, and mtime date.

### Added

- **mtime status line** — a `BOX_V` root now wraps the panes row plus a bottom status
  `LABEL` (`crab_status_str`) rendering the active selection's `<name>  <size|<dir>>
  <YYYY-MM-DD HH:MM>`. The date comes from the same `stat` syscall (#33 — `st_mtime` @ +40,
  unix seconds), formatted via the civil days→(y,m,d) algorithm (verified against the host
  `datetime` across leap-year boundaries). crab now stats **every** entry (files AND dirs)
  into parallel `sizes[]` + `mtimes[]` per pane. Proven on agnos: the status bar shows
  `aethersafha  14M  2026-07-10 19:18` from the real inode mtime, composited by aethersafha.

  > ⛔ **RETRACTED 2026-08-03 — "Proven on agnos" here is a FALSE GREEN.** The observation came from a
  > setu smoke run against `build/ae-setu-smoke/agnos-ae.img`, the image staged by the now-deleted
  > `aethersafha-setu-smoke.sh`, which built the kernel with `AETHERSAFHA_SETU_SELFTEST=1`. That hook
  > assigned `net_ip = 0x7F000001` before launching the compositor, which is the ONLY reason the
  > loopback handshake completed **in this era** — before `net_src_for` (agnos 1.56.34) an ordinary
  > boot could not. The `stat`/`mtime` work itself may well be sound; what is retracted is the claim
  > that *this run* proved it on agnos. TCP is retired as the desktop transport — as the wrong
  > primitive, not because it never worked (post-`net_src_for` crab connected and presented un-rigged
  > in QEMU at `-smp 1`, 2026-08-02). See agnos `docs/development/planning/ipc.md` §9/§10.

## [0.3.1] - 2026-07-10 — directory navigation + per-entry size

The read-only listing from 0.3.0 becomes a navigable, informative browser: keyboard
directory traversal (Enter/Backspace) and each file's size shown alongside its name.

### Added

- **Directory navigation** — Enter descends into the selected directory (re-`readdir`s the
  child and resets the selection to the top); Backspace ascends to the parent. Pane paths
  are now mutable buffers, so each pane's header tracks the current directory. Path helpers
  (`crab_strcpy`/`crab_join`/`crab_descend`/`crab_ascend`) are host-safe — `sys_readdir` stays
  behind `#ifdef CYRIUS_TARGET_AGNOS`. Proven on agnos via the setu-descend smoke: focus the
  right pane, descend into `/lost+found`, ascend back to `/` (Up/Down + Left/Right unregressed).
- **Navigation serial log** — a successful descend/ascend emits `crab: cd <path>` to serial
  (the smoke's dispositive gate, alongside `key received`).
- **Per-entry file size (richer listing)** — each file's byte size renders right-gapped after
  a 13-char name column, human-readable (`14M` / `299K` / `512`); directories keep the `/`
  marker. Sizes come from the agnos `stat` syscall (#33 — `sys_stat`, `st_size` @ +16 of the
  §4.1 struct): crab stats each listed entry on listing/navigation into a parallel `sizes[]`
  per pane. No new kernel/cyrius work — stat #33 shipped in agnos 1.41.3 and `sys_stat` is in
  cyrius 6.4.43. Proven on agnos (setu-stat smoke): `/bin` lists `aethersafha 14M` / `crab 299K`
  / `puka 79K` from real `stat` sizes, composited by aethersafha. `st_mtime` (@ +40) is
  available from the same syscall but not yet rendered — the ~187px panes don't fit both cleanly.

> ⛔ **RETRACTED 2026-08-03 — both "Proven on agnos" claims in this release are FALSE GREENS.** The
> setu-descend and setu-stat smokes both ran on `build/ae-setu-smoke/agnos-ae.img`, staged by the
> now-deleted `aethersafha-setu-smoke.sh` with `AETHERSAFHA_SETU_SELFTEST=1`. That hook assigned
> `net_ip = 0x7F000001` in the kernel, making a loopback SYN's source and destination agree by
> accident; **in that era (before `net_src_for`, agnos 1.56.34) the compositor↔client handshake could
> not complete on an ordinary boot.** The navigation and `stat` code may be correct — what is retracted
> is that *these smokes* proved it on agnos. TCP is no longer the desktop transport: it is retired as
> the wrong primitive, not as a thing that never worked — after `net_src_for` crab did connect and
> present un-rigged (QEMU, `-smp 1`, 2026-08-02). See agnos `docs/development/planning/ipc.md` §9/§10.

## [0.3.0] - 2026-07-10 — real filesystem listing (the `readdir` syscall)

crab's panes now show the **actual on-disk contents** of a directory on agnos rather than a
hardcoded name list. Each pane calls the ring-3 `readdir` syscall (#81) — landed in **agnos
1.53.13** and exposed as the named, agnos-gated `sys_readdir(path, buf, max)` stdlib wrapper in
**cyrius 6.4.43** — and renders the live entries in the kashi system font, directories suffixed
`/`. Left pane lists `/bin` (`aethersafha` / `crab` / `puka`), right pane lists `/` (`bin/` /
`lost+found/`). QEMU-proven end-to-end: the compositor spawns crab, both panes list their real
directories, and Up/Down selection + Left/Right pane-switch navigate the live entries.

> ⛔ **RETRACTED 2026-08-03 — "QEMU-proven end-to-end" is a FALSE GREEN.** The end-to-end path ran under
> the `AETHERSAFHA_SETU_SELFTEST` kernel hook, which assigned `net_ip = 0x7F000001` so the setu
> loopback handshake would close; **before `net_src_for` (agnos 1.56.34)** no ordinary boot could
> complete it. The hook and its smoke are deleted, and TCP-on-loopback is retired as the desktop
> transport — retired as the **wrong primitive** for local display IPC, not because it never worked:
> after `net_src_for`, crab connected and presented un-rigged (QEMU, `-smp 1`, 2026-08-02). See agnos
> `docs/development/planning/ipc.md` §9/§10. The `readdir` work stands on its own; *this* end-to-end
> proof does not.

### Added

- **Real directory listing via `sys_readdir`** (`src/main.cyr`, `src/ui.cyr`) — each pane
  `readdir`'s its path into a buffer of fixed 64-byte records (name at `+0`, type at `+63`:
  `1` = dir, `0` = file) and renders the live entries; directories get a trailing `/`. The
  selection clamps to the real entry count. Replaces the 0.2.0 hardcoded name list.

### Changed

- **cyrius pin 6.4.34 → 6.4.43** — picks up the agnos-gated `sys_readdir` wrapper; crab now
  calls `sys_readdir(path, buf, max)` instead of the raw `syscall(81, …)` under its own
  `#ifdef CYRIUS_TARGET_AGNOS`.

### Dependencies

- (unchanged) sadish 0.4.1 + rekha 0.3.1 + kashi 1.0.2 + dhancha 0.8.0 + setu 0.4.0.

## [0.2.0] - 2026-07-10 — a real dual-pane file manager on the sovereign desktop

crab graduates from scaffold to a working file manager: a **dhancha widget client** that
presents a **dual-pane** UI over **setu** and is composited by **aethersafha on agnos** — the
first standalone dhancha app to build `--agnos`. Norton-Commander / Dolphin style, navigable
by keyboard, drawn in the kashi system font.

> ⛔ **RETRACTED 2026-08-03 — "composited by aethersafha on agnos" was, *in this arc*, only true under a
> rigged kernel.** Every agnos compositing run in this arc went through the `AETHERSAFHA_SETU_SELFTEST`
> hook, which assigned `net_ip = 0x7F000001` so setu's loopback TCP handshake would match; before
> `net_src_for` (agnos 1.56.34) an ordinary boot could not. The "connects on loopback:7700" transport
> below is RETIRED — as the **wrong primitive** for a local display protocol (nothing to route,
> checksum, or negotiate; no business owning a port), *not* as something that never worked: after
> `net_src_for`, crab connected and presented un-rigged (QEMU, `-smp 1`, 2026-08-02). The replacement
> is the agnos socket (`anu`). See agnos `docs/development/planning/ipc.md` §9/§10.

### Added

- **Dual-pane UI** (`src/ui.cyr`) — two file-list columns (`BOX_H` → two `BOX_V`), each a
  header + rows, rendered by the dhancha toolkit via sadish (2D vector). The active pane's
  header + selection are bright, the inactive pane's dim.
- **Keyboard navigation** — Left/Right (or `h`/`l`) switch the active pane; Up/Down (or
  `j`/`k`) move the selection within it. Routed over setu as `SETU_INPUT_KEY`.
- **setu client transport** (`src/main.cyr`) — connects on loopback:7700, presents a
  shared-buffer surface (CREATE → ATTACH-by-buf → COMMIT), and re-renders live on forwarded
  focus / key input.
- **Real file names** in the kashi system font (full CP437, lowercase) — the same font the
  compositor chrome uses.
- Host layout harness (`src/render_test.cyr`) — dumps the rendered UI for fast iteration.

### Dependencies

- sadish 0.4.1 + rekha 0.3.1 + **kashi 1.0.2** (system font) + **dhancha 0.8.0** (toolkit,
  kashi text path) + setu 0.4.0 (transport). No mabda — on agnos the present goes over setu's
  shared buffer, not a GPU ioctl.

## [0.1.0]

### Added
- Initial project scaffold
