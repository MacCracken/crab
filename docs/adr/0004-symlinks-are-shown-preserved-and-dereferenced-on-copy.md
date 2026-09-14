# 0004 — Symlinks are shown, preserved by delete and move, and dereferenced by copy

**Status**: Accepted
**Date**: 2026-09-14 (0.9.3)

## Context

The roadmap carried symlinks as a decision rather than a dependency: *"`sys_lstat` is vendored and
deliberately never called, so symlinks stay invisible to the write layer. The gate closed upstream at
cyrius 6.5.37; what remains is a decision about what crab should DO with the answer — **refuse,
report, or recreate**."*

Three facts shaped the answer, and two of them were only found by reading the kernel:

1. ⛔ **readdir cannot tell crab a link is a link.** agnos's `ext2_readdir_at_sys` writes a 64-byte
   record and sets byte 63 with `var t = 0; if (ftype == 2) { t = 1; }` — one bit, DIR or not. So
   `EXT2_FT_SYMLINK` arrives indistinguishable from a regular file. crab has never had the
   information to show.
2. ⭐ **The stat sweep already visits every entry.** `crab_stat_batch` walks 32 entries per idle tick
   until none is pending, whatever the sort mode — the synchronous storm in `crab_stat_for_listing`
   is only for a SIZE or MTIME sort, which cannot order anything without the data. So asking `lstat`
   instead of `stat` costs **no extra syscall at all**.
3. ⛔⛔ **The obvious hazard is not real, and the same one bit is why.** A symlink pointing at an
   ancestor does not make the recursive walk loop forever, on either target: the walk descends only
   where the type byte says 1, and neither agnos (`ftype == 2` only) nor the host path (`dt == 4`
   only) ever sets it for a link. `CRAB_PATH_MAX` bounds it a second time — `crab_join_n` refuses at
   depth 127 before `CRAB_WALK_DEPTH_MAX` can fire.

## Decision

**Symlinks are SHOWN. Delete and move PRESERVE them. Copy DEREFERENCES them.**

- **Shown** — the stat sweep calls `lstat` and writes `2` into the type byte crab already had
  (`0` file, `1` dir). The listing marks a link with `@`, the KIND column says `Link`, and the size
  column shows the link's own size, which is what `ls -l` reports.
- **Preserved by delete** — `unlink` removes the LINK, never its target. That is the POSIX rule and
  agnos's `unlink`#30 follows it.
  ⛔⛔ **THIS ADR ASSERTED THAT BEFORE THE CODE DID IT, AND THE GAP WAS DATA LOSS.** The single-entry
  delete verb read `if (ddir != 0)` and handed anything non-zero to `crab_walk_begin(CRAB_OP_DTREE,
  …)`, which type-checks nothing — so a link pointing at a directory became the **root of a recursive
  delete** and the walk enumerated the TARGET's contents. Measured on iron: `crab: delete zzlink ->
  done`, the link still on disk, and 0 of 3 files left in what it pointed at. ⚠ It is **older than
  this ADR**: `crab_stat_one` used to call `stat`, which follows a link, so a link to a directory was
  already stored as type `1` and already took that branch. Fixed by `crab_delete_plan(kind)` and held
  by `agnos/scripts/harness/crab-symlink-test.py`, which reads the image back with `debugfs` rather
  than asking crab whether crab did the right thing.
  ⇒ **The lesson is about this document, not only that function.** An ADR that states what the
  primitives *would* do is describing a design; what earns the word "Decided" is a gate. Every verb
  above now has one.
- **Preserved by move** — `rename` moves the link itself.
- **Dereferenced by copy** — `open` + `read` copies the target's bytes as a regular file, which is
  what `cp -r` does. It is now a decision rather than an accident, and the operator can see that an
  entry is a link **before** acting on it, which they could not before.

## Consequences

- **Positive** — crab can see links for zero extra syscalls, and two of the three verbs were already
  correct by construction; the decision mostly documents what the platform's primitives already do.
- ⛔ **It found seven real defects in the write layer, one of them destructive.** `crab_fs_delete`
  chose `rmdir` on `is_dir != 0` — a two-valued test on a field that now has three values — so every
  link would have been **undeletable**. The delete verb walked one. The delete **prompt** called one a
  FOLDER and offered to delete everything in it. The transfer planner tree-copied one, at both call
  sites. Enter on one did nothing and said nothing. Two thumbnail readers offered one a preview.
  ⚠ **Widening a field is only safe where every reader agrees how to ask — and the way to know they
  agree is to ENUMERATE them.** The first sweep grepped `== 1` and `!= 1`, missed the entire `!= 0`
  family, and reported itself complete; the damage was all in the part it did not look at. The second
  listed all 22 reads of `CRAB_REC_TYPE` and traced each to its decision. `crab_kind_mark`,
  `crab_delete_plan`, `crab_transfer_plan` and `== CRAB_KIND_DIR` are that agreement now.
- **Negative** — a recursive copy still reads through a link, so a tree containing one copies data
  from wherever it points. That is `cp -r`'s behaviour and is not novel, but it is worth knowing.
- **Neutral / open** — **recreate is possible and is not done.** agnos has `symlink`#63 and
  `readlink`#70, both with cyrius peers, so a copy could recreate a link rather than follow it. ⛔ Both
  are **ext2-only**: FAT and exFAT cannot represent a symlink, and crab's whole two-pane premise is
  copying between volumes. A recreate that works on one side of a copy and fails on the other is a
  partial answer that needs its own design — what a link becomes when it lands somewhere that cannot
  hold one. Deferred deliberately, with the primitives confirmed present.

## Alternatives considered

- **Refuse to copy a link.** Safe and simple, and rejected: one link makes a whole tree uncopyable,
  which is a worse failure than the one it prevents, and no other file manager does it.
- **Recreate on ext2, refuse on FAT.** The honest full answer, and the reason it is deferred rather
  than rejected — see above. It needs a decision about the cross-filesystem case that this ADR does
  not have enough evidence to make.
- **Mark links from readdir instead of the stat sweep.** Impossible without a kernel change: the
  record carries one bit. Filing for a wider dirent type was considered and not pursued, because the
  stat sweep already answers at no cost and a kernel ABI change to save a syscall crab was already
  making would be asking for the wrong thing.
