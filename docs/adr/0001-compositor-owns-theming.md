# 0001 — The compositor owns theming; crab ships no palette

**Status**: Accepted
**Date**: 2026-08-26

## Context

The design canvas ([`Crab File Manager Mockups.dc.html`](../../Crab%20File%20Manager%20Mockups.dc.html))
draws crab in both a light and a dark shell. The obvious reading of two shells is two crab settings,
and the obvious follow-on is a theme switcher in a preferences pane — which is what nearly every
file manager ships.

That reading is wrong for this stack, and the canvas says so in its own brief: *"The compositor owns
theming. crab ships no theme switcher and no palette of its own; it reads surface, text and accent
from aethersafha and declares only which parts of the UI accent is permitted to tint. Light and dark
shells here are two compositor states, not two crab settings."*

The machinery already exists and crab already uses it. `dh_theme_bg` / `_panel` / `_widget` / `_line`
/ `_ink` / `_mute` / `_accent` / `_alert` (`lib/dhancha.cyr:2111-2118`) each resolve through
`rupa_theme_active()` — the shared desktop theme-token core. crab calls `dh_theme_panel()` and
`dh_theme_accent()` in `crab_pane` today and has never owned a colour constant.

The decision is worth recording because the *pressure* to reverse it is predictable: a user asks for
a dark mode toggle, and adding one looks like a small local change to a settings menu.

## Decision

**crab owns no colour, no palette, and no theme UI.** It reads every colour from `dh_theme_*` and
declares only which surfaces accent is permitted to tint. A theme switcher is out of scope for v1.0
and is not a deferral — it is a thing crab will not have.

Light and dark are compositor states. Changing theme is an aethersafha action, and every client
follows because every client reads the same rupa tokens.

## Consequences

- **Positive** — one theme change re-skins the whole desktop consistently, and crab cannot drift from
  its neighbours. crab carries no palette to maintain, no contrast pairs to re-check, no settings
  surface to build, and no persistence for any of it.
- **Positive** — it keeps the rule in one place. crab has now twice paid for a rule living in three
  apps instead of one (row selection painting in 0.4.10; input transport in 0.4.12), and both times
  the fix was to delete crab's copy and defer to the toolkit.
- **Negative** — crab cannot express a colour the tokens do not have. This is not hypothetical: rupa
  publishes `accent`, `active`, `held` and `faint` but **no `on-accent`** — the ink colour guaranteed
  legible *on* an accent fill. Until it does, a selected row cannot safely carry white text, which
  the canvas's selected rows assume. That is now a named gate on rupa (roadmap M3), and the right
  fix is upstream rather than a local override.
- **Negative** — crab cannot ship a theme its users want ahead of the compositor shipping it.
- **Neutral** — the canvas's own open question, *"which accent roles does the compositor actually
  hand over — one accent, or accent + on-accent + a hover step?"*, becomes a rupa question rather
  than a crab one.

## Alternatives considered

- **A crab-local theme with its own palette.** Rejected: it guarantees drift from every other client
  on the desktop, and it re-creates exactly the "rule in three apps" defect this project has already
  fixed twice.
- **Read the tokens but allow a local override.** Rejected: an override that exists will be used, and
  the first use will be to work around a missing token instead of adding it upstream — which is how
  the `on-accent` gap would have been papered over rather than closed.
- **Ship a switcher that drives the compositor.** Deferred, not rejected — but it is an aethersafha
  control surface that happens to be rendered by a client, and it belongs to whichever client owns
  desktop settings. That is not the file manager.
