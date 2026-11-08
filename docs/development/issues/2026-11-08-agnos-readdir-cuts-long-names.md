# agnos cuts a listed name at 62 bytes and says nothing — a row can name another file

> ⚠ **This is crab's COPY. The canonical filing is in agnos**, at
> `agnos/docs/development/issues/2026-11-08-readdir-cuts-a-long-name-and-says-nothing.md` — an issue about
> another repository that lives only here is one nobody who could act on it will ever read. The canonical
> text is written from agnos's side; this copy says what it means for crab.
> ⛔ **What it means for crab, in one line**: a name over 62 bytes lists as its first 62, and crab builds every
> path from the listed bytes, so the row opens nothing, or acts on a sibling with exactly that name.

**Status:** 🔴 **OPEN — upstream, filed 2026-11-08 from crab 0.11.3 against agnos 8927c965 (1.57.17).**

## What was measured

A ring-3 probe under QEMU, on ext2. With a 70-byte name beside a sibling named its first 62 bytes, `#81` and `#101`
list two identical rows, and opening either reads the sibling. `unlink#30` on the listed name removed the sibling and
left the 70-byte file. Two names sharing their first 62 bytes list as two identical rows that open nothing.
`getdents#29` listed every name whole in the same run.

## What it means here

crab lists through `#101`, falling back to `#81` (`src/app.cyr`, `sys_readdir_at` and `sys_readdir`). Every action then
joins the directory and the listed bytes:

- **A long name alone:** a row whose size, Open, Rename and Delete all fail.
- **A long name beside a sibling named its first 62 bytes:** two identical rows, and an action on either lands on the sibling.

## What crab already does

Since 0.11.2 it refuses to create a name over 62 bytes (`CRAB_NAME_CHARS`), and its duplicate scan never pairs two rows
of one name. Neither helps with a long name already on disk.

## What would close it here

- **agnos marks a cut name** (the filing's ask): crab masks the new bit in byte 63 wherever it reads the type, and
  refuses to act on a marked row, out loud.
- **Or crab moves to `#29`**, which returns whole names today. crab's record is 64 bytes (`CRAB_REC_SZ`) and holds 62 of
  them, so this means widening the record or adding a name pool, across every reader of a listing. Not started.
