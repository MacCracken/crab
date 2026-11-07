# `mountlist`#104 backend 4 (ZFS) shows as "unknown" in the VOLUMES sidebar

**Filed by:** agnos 1.57.11 (2026-10-09) · **Severity:** cosmetic · **Status:** OPEN

## What changed in agnos

agnos 1.57.11 ships read-only ZFS. A pool the kernel imports is mounted at **`/mnt/zfs`** as `FsBackend`
**`FS_ZFS = 4`**, so `mountlist`#104 now returns records with `backend == 4` (prefix `/mnt/zfs`, and `/` when no
ext2/FAT/exFAT filesystem took the root). Contract: agnos `docs/development/agnos-userland-abi.md` §3.6 and the
`#104` row ("1 ext2 · 2 FAT · 3 exFAT · 4 ZFS").

## What crab does

`src/ui.cyr` `crab_vol_label(backend)` maps 1/2/3 and returns `"unknown"` for anything else, so a ZFS volume is
listed as **unknown**. Nothing breaks: `statfs`#103 answers for `/mnt/zfs` (f_bsize = 2^ashift; see ABI §3.6), and
every write verb on the volume answers -1 (the pool is read-only until agnos 1.57.12).

## Suggested change

```
if (backend == 4) { return "ZFS"; }
```

and the comment at `src/ui.cyr:1207` listing the backend ids. A file manager may also want to grey out
write actions on a backend-4 volume while it is read-only (every write verb answers -1 today).
