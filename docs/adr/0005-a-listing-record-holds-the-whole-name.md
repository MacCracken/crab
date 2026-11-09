# 0005 — A listing record holds the whole name, filled from getdents

**Status**: Accepted
**Date**: 2026-11-08 (0.11.4)

## Context

crab listed directories through agnos's `readdir#81` and `readdir_at#101`, whose fixed 64-byte records cut a name at
62 bytes and say nothing. Every action joined the directory and the listed bytes, so a longer name either opened nothing
or, beside a sibling named exactly its first 62 bytes, acted on the sibling. Under QEMU, a delete aimed at a 70-byte name
removed the sibling and left the file (`docs/development/issues/2026-11-08-agnos-readdir-cuts-long-names.md`).

`getdents#29` returns whole names, in variable-length records, from a directory fd.

## Decision

Pane listings read `#29`, and a listing record is crab's own 257-byte layout: 255 bytes of name, its NUL, and the type
byte. Every reader already indexes records through `CRAB_REC_SZ`, `CRAB_REC_TYPE` and `CRAB_NAME_MAX`.

`#101` stays for two jobs `#29` cannot do: explaining why a directory would not open, because `open` answers only -1, and
the tree walk. The walk runs in steps and holds no fd between them, `#29` cannot resume without one, and `lseek#58`
refuses a directory. Its 64-byte records are read at their own stride and converted by `crab_krecs_widen`.

## Consequences

- **Positive** — a name of up to 255 bytes lists, opens, renames, deletes, copies and moves as itself. The listing
  parser is a function the host suite reaches, which the agnos listing never had.
- **Negative** — about 1.0 MB more permanent heap at full use, and sorting a full pane takes about twice as long, still
  under a millisecond. A folder copy or delete still stops at a name over 62 bytes until agnos lets `lseek#58` reposition
  a directory fd.
- **Neutral** — the path limit stays at 256 bytes, so a long name in a deep directory lists and its actions refuse.
  Raising it touches six fields of the transfer record and every path buffer, and is its own change.

## Alternatives considered

- **A name pool, with records holding a pointer.** Cheaper in memory, but about 100 places read a record as a string.
- **An overflow table for long names only.** Two code paths for one job, and anything reading the record's inline prefix
  would get a long name's extension wrong.
- **Waiting for agnos to mark a cut name.** It would let crab refuse a cut row, not act on it.
- **One directory fd per level of the tree walk.** The kernel allows 32 fds and a walk can go 127 levels deep.
