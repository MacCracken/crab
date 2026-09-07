# Five contiguous `/bin` entries were deleted on AGNOS iron — mechanism unproven

**Status:** 🔴 **OPEN — damage confirmed and reproducible in its SHAPE; the mechanism is NOT
established. Filed as evidence so it is not re-derived from scratch.**
**Reported by:** agnos (operator), archaemenid iron burn, 2026-09-03, agnos **1.56.60**, crab **0.8.0**.
**Severity:** High — it bricked the boot. AGNOS's `/bin` is not a user directory; deleting from it
makes the machine unbootable to userland, and the operator did not find out until the next reboot.

## What is established

An AGNOS iron box was browsed with crab on the desktop. Some time later every boot fell to the
in-kernel emergency shell instead of reaching `/bin/agnsh`. The agnos-fs was mounted on the host and
diffed against the staged rootfs. **Exactly five files were missing, and they are exactly rows 1–5 of
a sorted `/bin` listing, contiguous:**

```
1  aethersafha   MISSING
2  agnsh         MISSING     <- this is what bricked the boot
3  anuenue       MISSING
4  bnrmr         MISSING
5  cmdrs         MISSING
6  cp            present
7  crab          present
8  cyim          present
   ... all 46 remaining entries present
```

⭐ **No filesystem fault produces that pattern.** Corruption does not pick a contiguous alphabetical
prefix and stop cleanly at row 6. A *selection* does. `e2fsck -fn` on the volume is clean, the
directory is intact, and the surviving 46 entries are byte-correct. Timestamps corroborate a live
session: `.agnsh_audit.log` 10:08, `60run.txt` 10:06, `.modeset-armed` 10:24.

⛔ **The kernel is exonerated, independently.** agnos 1.56.60 boots to `agnsh` 3/3 times on a
persistent QEMU image, and deleting only `/bin/agnsh` from that image reproduces the operator's exact
console signature (`kybernet: emergency shell (exec rc=-1)`).

## What is NOT established — and one hypothesis already refuted

**The obvious theory is wrong.** Marks in crab are INDEX-based, so stale marks surviving a directory
change would apply to whatever now occupies those indices — which would produce exactly this damage.
**It does not happen:** every real navigation site clears marks —
`src/main.cyr:525`, `:535` (double-click descend), `:935`, `:946` (keyboard Enter descend),
`:1338`, `:1350` (Backspace ascend) each call `crab_mark_clear` immediately after a successful
`crab_descend`/`crab_ascend`. Verified by auditing all six call sites. **Do not re-file this theory.**

Still open, in the order worth checking:
1. **Any path that re-lists WITHOUT navigating.** `crab_relist` (`src/app.cyr:1710`) invalidates the
   preview and thumbnail caches but does **not** clear marks — deliberately, since it runs after
   crab's own writes. If any code path re-lists a *different* directory through `crab_relist` rather
   than `crab_descend`/`crab_ascend`, index-based marks would survive into a listing they never
   referred to. That is the same failure the navigation sites are guarded against, at an unguarded door.
2. **The batch sheet**, which `crab_relist`'s own comment names as ending in a relist.
3. **Ordinary operator error** — five rows marked and `y` pressed. Possible, and the operator does not
   recall doing it. Worth ruling in or out rather than assuming either way.

## Checked and ruled out (crab 0.8.1, 2026-09-07)

Audited against the **tagged `0.8.0`** — the code that was on the box — not against HEAD.

| hypothesis | verdict |
|---|---|
| Stale index marks surviving a re-list (the issue's #1) | **Refuted.** Every `crab_relist` site in the burned tree clears marks within 3 lines, and each clears *its own* pane's — the cross-pane shape was checked too. |
| Marks surviving a re-SORT, where the permutation repoints indices | **Refuted.** The `s` handler already cleared both panes' marks at 0.8.0. |
| `crab_mark_clear` bounded by the live count, leaving high indices alive | **Refuted.** All 24 call sites pass `CRAB_MAX_ENTRIES`. |

⚠ **One real finding that does NOT explain the shape.** The tagged 0.8.0 has no `crab_pointer_modal`
— a click during the delete confirmation could re-list the pane and change what `y` deleted, and
that was live on the burned box. But it yields **one** wrong entry or one wrong tree, not five
contiguous siblings. ⇒ An unguarded path to a wrong delete, closed in 0.8.1, but not this mechanism.

⛔ **Remaining hypotheses, unchanged**: the batch sheet's relist, and ordinary operator error. The
contiguous-prefix shape (rows 1–5, stopping cleanly at 6) is what *marking the first five entries*
produces; nothing found so far manufactures that pattern without the operator.

## ✅ The ask is implemented (crab 0.8.1)

Both preventions shipped, independent of mechanism:

* **The prompt names what dies** — `SYSTEM DIR! delete 5 marked: agnsh, boot, cp +2 more?` The count
  and the first three names are in front of the keypress now. ⚠ Three then a remainder: the status
  line is one row.
* **`/bin`, `/boot`, `/sbin` and `/lib` say `SYSTEM DIR!` first.** ⛔ Exact match on the directory,
  not a prefix test — a guard that swallowed the whole subtree would make it undeletable and teach
  the operator to route around it.
* ⚠ **Still no undo, and no refusal** — the guard warns, it does not block. Blocking deletion under
  `/bin` outright is an operator decision, not crab's to take unilaterally.

## The ask, independent of mechanism

⛔ **`crab_fs_delete` has no guard for system paths and no undo.** Whatever the trigger, five system
binaries left the disk on one confirm, and nothing surfaced it until the machine would not boot. Two
things would each have prevented the outcome:

* **Refuse or double-confirm deletes under `/bin`** (and `/boot`) on an AGNOS target. AGNOS is
  single-user with no package manager — `/bin` is the OS.
* **Name what is about to die.** The confirm is a bare `y`; a count and the first few names would have
  made "5 files including agnsh" visible before the keypress rather than after the reboot.

⚠ **Marks are index-based**, which is what makes any un-cleared-marks path this destructive. Queuing
by name at delete time (`crab_queue_marked`, `src/main.cyr:743`) removes the *ordering* hazard within
one delete, and its comment says so — but it does not help if the marks referred to a different
listing in the first place.
