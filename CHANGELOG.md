# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

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
