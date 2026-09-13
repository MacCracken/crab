# aethersafha claims Esc, Tab and F4–F10 — crab's `Esc`, `Tab` and `F10` bindings never receive a keypress on agnos

> ⚠ **This is crab's COPY. The canonical filing is in aethersafha**, at
> `docs/development/issues/2026-09-13-claimed-keys-never-reach-a-client.md` — an issue about another
> repository that lives only here is one nobody who could act on it will ever read.
> ⛔ **What it means for crab, in one line**: on the real desktop the menu bar (`F10`), the sidebar's
> keyboard route (`Tab`) and every `Esc` cancel are **unreachable**, and `Esc` ends the session. The
> pointer routes (0.8.5) are the only road to the popup; `View`'s items (0.8.6) are reachable by
> nobody. Measured on QEMU 2026-09-13 with `agnos/scripts/harness/crab-pointer-test.py`.


**Status:** 🔴 **OPEN — a DESIGN decision, not a bug in either repo's code. MEASURED on QEMU
2026-09-13** (`agnos/scripts/harness/crab-pointer-test.py`), not only read. Filed by crab while
putting crab on QEMU for the first time since 0.7.0.
⚠ Found in passing: `println(lnch_name_at(lsel))` after *"launching from the launcher:"* prints the
name's ADDRESS (`9671447`), not the name — a harness cannot read which app the launcher started.
**Severity:** High for crab's keyboard surface (three shipped affordances are unreachable), and the
Esc arm is destructive: a client's operator pressing Esc — the universal "cancel" — ends the session.

## What was found

`src/input.cyr`'s `input_map` claims **Esc** (`IA_QUIT`), **Tab** (`IA_FOCUS_NEXT`), **F4** (close),
**F5** (maximize), **F6** (minimize) and **F7–F10** (move); `src/main.cyr` reads **F2** (launcher)
and **F3** (theme) before `input_handle`. The frame loop then says, and does, exactly this:

> ⚠ CLAIMED KEYS ARE CONSUMED, NOT FORWARDED. A focus key that also reached the client would type a
> literal tab into the window you just switched away from, and a move key would scroll a file list
> while its window slid out from under it. (`src/main.cyr`, the `kdone` block)

Only `kdone == 0` keys reach `setu_srv_forward_key`. That is a reasonable rule for a compositor
with **no modifier state on the wire** — there is no Super or Alt to hang chrome keys on — but no
client was told, and no client can find out at run time.

## What it costs crab (the first client that binds any of them)

| crab binding | shipped | what the compositor does with it | what the operator sees |
|---|---|---|---|
| `Esc` — dismiss the context menu / close a bar drop-down / cancel a transfer / abandon the rename sheet / leave sidebar focus | 0.7.3 → 0.8.3 | **`IA_QUIT` → `running = 0`** — the desktop exits | every open surface in crab, and crab, and the compositor, gone |
| `Tab` — sidebar focus (the PLACES keyboard route) | 0.8.3 | `IA_FOCUS_NEXT` — cycles window focus | with one window: nothing; with two: the keys leave crab |
| `F10` — reveal the menu bar | 0.8.0 | `IA_MOVE_DOWN` — slides crab's window down | the window moves; no bar |

The menu bar is collapsed by default and **F10 is its only door**, so the bar and everything under
it (File · Edit · Go · View, 0.8.0–0.8.6) is unreachable on the real desktop. The sidebar keyboard
route is unreachable. Every `Esc` binding is unreachable, and the key is destructive. ⚠ crab's host
suite pins all of these as working because it tests crab's dispatch table, which is correct; the
wire never delivers the key. Both suites are green and both are blind to this.

⚠ crab already knew about **F5/F6** (`src/main.cyr`: *"aethersafha takes F5 for maximize, so a
client binding it would never see the key"*) and avoided them. Esc, Tab and F10 were not
recorded anywhere, and the pointer routes (0.8.5) are the only reason the popup is reachable at all.

## Measured (QEMU, 2026-09-13, `scripts/harness/crab-pointer-test.py`)

aethersafha 0.16.24 (`041ac85`) · crab 0.8.6 + the refresh key · agnos `build/agnos` 2026-09-11 · one
compositor, one crab window (a second compositor in run 1 was the harness's own doing — see the
harness header — and that run is discarded). `crab: key press` is what crab ACTED on:

```
Tab x6:  compositor answered 5 time(s) (`a TAB reached the compositor`, `TAB ignored -- fewer than two windows`); crab ACTED on 0 key(s)
F10 x6:  compositor answered 1 time(s) (`F7-F10 moved the focused window`, a one-shot);             crab ACTED on 0 key(s)
Esc x4:  compositor quit: True (`aethersafha: quit on a key`); crab ACTED on 0 key(s)
control: `u` x6 -> crab acted 4 times (`crab: refresh`), `g` and `b` answered — unclaimed keys arrive
```

⚠ crab's `crab: key received` count DOES move on Tab — by the RELEASES. `input_map` returns
`IA_NONE` for a release, so a claimed key's release is forwarded while its press is not. A client
counting received events would conclude the key arrives; only the press edge is gone.

## The decision that needs making

Three shapes, each with a cost; crab has no standing to pick one:

1. **Forward claimed keys to a focused client that asked for them** — a setu surface flag (the
   `SETU_SURF_FULL_KEYS` precedent), e.g. `SETU_SURF_CHROME_KEYS`: the compositor keeps Esc/Tab/F-keys
   for itself only when the focused surface did NOT set it. Puka would keep today's behaviour; crab
   would set it. Cost: a client that sets it can no longer be closed/moved by keyboard while focused
   — which is what every other desktop accepts, because chrome keys carry a modifier there.
2. **Move chrome onto a modifier** — needs modifier state on the wire (bhumi/setu have none today;
   crab's own note: "Shift is not on the wire"). The right long-term answer and the largest change.
3. **Rebind in every client** — crab moves the menu bar to F1, sidebar focus to a letter, and gives
   up Esc for a letter. Cheap, but it teaches operators that Esc kills the desktop, which no key
   should do without a modifier.

⛔ Whatever is chosen, **Esc → quit is worth reconsidering on its own**: it is the one claimed key
whose consequence is not undoable, and `--clients` already ignores it (`ae_probe`), so the compositor
already has a mode where it is not needed.

## What crab does in the meantime

Nothing that pretends: the bindings stay (they are correct the day the key arrives), the harness
records what the desktop actually does, and crab's docs say which keyboard affordances are
pointer-only on agnos today. The REFRESH key (0.8.7) was placed on `u` because F5 is claimed.
