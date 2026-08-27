# 0002 — Semantic find is a mode over any view, not a view of its own

**Status**: Accepted
**Date**: 2026-08-26

## Context

The design canvas's direction **1b** ("Meaning-first, dark shell") draws semantic search as the home
screen: a natural-language query bar, REFINE facets, a result list ranked by meaning with a MATCH
column, a WHY IT MATCHED panel, an APPEARS IN panel, and duplicate grouping within the result set.

The canvas raises the architecture question itself, in its Open questions: *"Is semantic search a
view (1b) or a mode over any view? Cheapest to build as a view; better to use as a mode."*

This is a real fork. crab's other two directions (1a's dual-pane shell, 1c's dense operator view)
render *a list of files*. So does 1b. The difference is where the list comes from and what extra
per-row information exists — not how a row is drawn.

Two facts make the choice consequential rather than stylistic:

1. The ranked-results affordances are not search-specific. `2 LIKELY DUPLICATES IN THIS SET` is
   useful in a plain directory listing. Suggested tags appear in 1a's *browse* pane
   (`SUGGESTED TAGS · src → + toolchain + cyrius + wip`), not in 1b. Sorting by relevance is a sort
   order like any other.
2. crab has already paid twice for the same class of mistake — a rule implemented in one place that
   should have been implemented once for everything (row selection painting, fixed in 0.4.10; input
   transport, fixed in 0.4.12). A second result path is that shape.

## Decision

**Semantic find is a mode over the existing views.** There is **one result model** — an ordered list
of entries, each optionally carrying a match score, a match explanation, and a duplicate-group id —
and list, grid, columns and gallery all render it.

Typing a natural-language query switches the *active pane* into semantic mode. The view (list / grid
/ columns / gallery) is unchanged and remains the operator's choice. Clearing the query returns the
pane to its directory listing.

Concretely:

- The pane's content source becomes an interface with two implementations: `readdir` and `query`.
- Match score, match reason and duplicate-group are **optional per-entry fields**, populated by the
  query source and empty for a directory listing. Any view that can show them, shows them.
- 1b's chrome — the query bar, REFINE facets, WHY IT MATCHED, APPEARS IN — is a **surface** that
  appears when the mode is active, not a screen with its own file list underneath it.

## Consequences

- **Positive** — dupe grouping, suggested tags and relevance sorting work in a plain listing, which
  is where the canvas actually draws two of the three.
- **Positive** — one selection model, one keyboard map, one set of file operations. Copy, rename and
  delete work identically on a search result and a directory entry, because they are the same thing.
- **Positive** — no second rendering path to keep in sync, which is the failure this project has
  already fixed twice.
- **Negative** — more up-front cost. A separate 1b screen could be built against a hardcoded fixture
  and shipped before the index exists; a mode requires the entry model to carry the optional fields
  from the start, which touches M3's column work before M7's index work begins.
- **Negative** — the entry record grows. Today it is the syscall's fixed 64-byte readdir record;
  match metadata cannot live there, so crab needs its own entry representation with the readdir
  record as one source. That is a real refactor and it lands in M3, not M7.
- **Neutral** — 1b's dark shell is not part of this decision. Light/dark is a compositor state
  ([ADR 0001](0001-compositor-owns-theming.md)); 1b is drawn dark because the canvas needed to show
  both states somewhere, not because search has its own shell.

## Alternatives considered

- **Build 1b as a separate view now, promote it to a mode later.** Rejected. It is genuinely cheaper
  and it is what the canvas notes as the cheap option — but "promote it later" means two result paths
  in the meantime, and the promotion is exactly the refactor being avoided, only larger and with
  shipped behaviour depending on both paths. The canvas's own judgement is *"better to use as a
  mode"*, and the deciding factor is that the affordances belong in browse too.
- **Semantic results as a filter over the current directory only.** Rejected: it is a much smaller
  feature that does not match the canvas, where results span `~/documents/finance/2026/q2`,
  `~/pictures/scans` and `~/downloads` in one list.
- **A separate application.** Rejected — crab's stated identity is *a view onto a shared memory
  substrate*, not a browser with a search tool beside it.
