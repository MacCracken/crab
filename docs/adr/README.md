# Architecture Decision Records

Decisions about crab — what we chose, the context, and the consequences we accept. Use these when a future reader would reasonably ask *"why did we do it this way?"*

## Conventions

- **Filename**: `NNNN-kebab-case-title.md`, zero-padded to four digits. Never renumber.
- **One decision per ADR.** If a decision supersedes a prior one, add a new ADR and set the old one's status to `Superseded by NNNN`.
- **Status lifecycle**: `Proposed` → `Accepted` → (optionally) `Superseded` or `Deprecated`.
- Use [`template.md`](template.md) as the starting point.

## ADR vs. architecture note vs. guide

| Kind | Lives in | Answers |
|---|---|---|
| ADR | `docs/adr/` | *Why did we choose X over Y?* |
| Architecture note | `docs/architecture/` | *What non-obvious constraint is true about the code?* |
| Guide | `docs/guides/` | *How do I do X?* |

## Index

| ADR | Title | Status |
|-----|-------|--------|
| [0001](0001-compositor-owns-theming.md) | The compositor owns theming; crab ships no palette | Accepted |
| [0002](0002-semantic-find-is-a-mode.md) | Semantic find is a mode over any view, not a view of its own | Accepted |
| [0003](0003-kashi-freestanding-core-over-the-library-face.md) | Take kashi's freestanding core, not its library face | Accepted |
