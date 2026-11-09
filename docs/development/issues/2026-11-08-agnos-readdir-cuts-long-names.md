# agnos cuts a listed name at 62 bytes and says nothing — a row can name another file

> ⚠ **This is crab's COPY. The canonical filing is in agnos**, at
> `agnos/docs/development/issues/2026-11-08-readdir-cuts-a-long-name-and-says-nothing.md` — an issue about
> another repository that lives only here is one nobody who could act on it will ever read. The canonical
> text is written from agnos's side; this copy says what it means for crab.
> ⛔ **What it means for crab, in one line**: since 0.11.4 only the tree walk still sees a cut name, so a folder copy or
> delete stops at a name over 62 bytes.

**Status:** 🟡 **PARTLY WORKED AROUND in crab 0.11.4 (2026-11-08); the rest is open upstream.** crab's panes now list
through `getdents#29` and hold whole names. Its tree walk still reads `#101`, so a folder copy or delete stops at a name
over 62 bytes until agnos lets `lseek#58` reposition a directory fd. Filed from crab 0.11.3 against agnos 8927c965 (1.57.17).

## What was measured

A ring-3 probe under QEMU, on ext2. With a 70-byte name beside a sibling named its first 62 bytes, `#81` and `#101`
list two identical rows, and opening either reads the sibling. `unlink#30` on the listed name removed the sibling and
left the 70-byte file. Two names sharing their first 62 bytes list as two identical rows that open nothing.
`getdents#29` listed every name whole in the same run.

## What it means here

Since 0.11.4 crab lists panes through `#29`, so every row holds its whole name. What remains is the tree walk
(`crab_walk_readdir`): it holds no fd between steps, `#29` cannot resume without one, and `lseek#58` refuses a directory.
One fd per level would need up to 127 of the kernel's 32.

## What would close it here

**agnos lets `lseek#58` reposition a directory fd** (the filing's third ask): the walk then reads `#29` as well, and
`crab_krecs_widen` and the `CRAB_KREC_*` constants go.
