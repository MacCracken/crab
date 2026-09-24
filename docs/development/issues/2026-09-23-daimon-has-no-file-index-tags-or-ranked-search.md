# daimon has no file index, no tags and no ranked file search — M7 and M8 wait on a surface, not a build

> ⚠ **This is crab's COPY. The canonical filing is in daimon**, at
> `daimon/docs/development/issues/2026-09-23-crab-needs-a-file-index-tags-and-ranked-search.md` —
> an issue about another repository that lives only here is one nobody who could act on it will
> ever read. The canonical text is written from daimon's side; this copy says what it means for crab.
> ⛔ **What it means for crab, in one line**: the remaining half of M7 (the disk-wide index, tags,
> smart folders) and all of M8 (assisted search) cannot be built, because daimon 2.4.3 holds no index
> of files, no tags and no ranked results — on any target. Building for agnos was necessary, and
> daimon has done it; it was never sufficient.

**Status:** 🔴 **OPEN — upstream.** Filed 2026-09-23 at crab's daimon pin move (2.1.4 → 2.4.3).

## The gate moved, and crab's roadmap had it wrong

From 0.10.0 to 0.10.3 crab's roadmap recorded M7's blocker as *"daimon does not build for agnos"*,
and its gate table called the index, tags and smart folders **UNGATED** once daimon was declared.
Both statements were about daimon's *build*. Neither checked whether daimon offered anything M7
could use. **It does not**, and it did not at 2.1.3 either:

- none of the 25 path branches in daimon's `http_route` is about files;
- its per-agent memory store has no route and no tag field (tag lookup is a substring search, by its
  own note);
- its RAG pipeline keeps nothing on disk, embeds by a 32-slot character-sum hash (`stop`, `pots` and
  `tops` are one token), and answers a query with a prompt template rather than with files.

⇒ The **eighth false gate** on crab's record, wrong about *which half was missing* — the same way
proportional text was (the plumbing existed; the advance widths did not). Here the build now exists;
the surface does not.

## What crab does meanwhile

Nothing for M7's remaining half, deliberately. No crab-side disk-wide index (it would silo the shared
index crab's design says it reads), and no tag or smart-folder rows that could only ever answer
"unavailable" (the VOLUMES rule). Duplicate detection within a listing (0.10.0) needs no daimon and
stays as it is.

## What the transport will have to live with, when there is something to ask

Recorded now because it is measured now, and it is crab's problem when the surface lands:
- daimon listens on the NIC's own address on agnos; TCP to 127.0.0.1 is dropped, so crab dials
  `sys_net_ip()` port 8090 and sends `Host: 127.0.0.1` — what daimon's `tests/agnos/http_client.cyr`
  does;
- `sock_connect`#47 and `sock_send`#48 hold the CPU while they wait, and a local send over 2 KB can
  stop the machine; `sock_recv`#49 never reports end of stream, so a reply is framed by
  `Content-Length`; the machine has 8 TCP slots and daimon serves at most 5 — five agnos filings,
  all daimon's, all open;
- crab's event loop may not block (`sys_pause`#14, never `sys_sleep_ms`#41), so every call must be
  interleavable with frames.

The canonical filing tells daimon crab has no attachment to HTTP; if daimon picks another reach for
local GUI clients, crab follows it.
