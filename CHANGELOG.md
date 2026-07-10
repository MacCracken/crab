# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

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

## [0.3.0] - 2026-07-10 — real filesystem listing (the `readdir` syscall)

crab's panes now show the **actual on-disk contents** of a directory on agnos rather than a
hardcoded name list. Each pane calls the ring-3 `readdir` syscall (#81) — landed in **agnos
1.53.13** and exposed as the named, agnos-gated `sys_readdir(path, buf, max)` stdlib wrapper in
**cyrius 6.4.43** — and renders the live entries in the kashi system font, directories suffixed
`/`. Left pane lists `/bin` (`aethersafha` / `crab` / `puka`), right pane lists `/` (`bin/` /
`lost+found/`). QEMU-proven end-to-end: the compositor spawns crab, both panes list their real
directories, and Up/Down selection + Left/Right pane-switch navigate the live entries.

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
