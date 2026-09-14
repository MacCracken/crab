# There is no proportional face on AGNOS — crab is correct under one and has nothing to load

> ⚠ **This is crab's COPY. The canonical filing is in agnos**, at
> `agnos/docs/development/issues/2026-09-13-no-proportional-face-on-the-target.md` — an issue about
> another repository that lives only here is one nobody who could act on it will ever read.
> ⛔ **What it means for crab, in one line**: `0.9.0 · A real face` cannot be cut. Every crab-side
> half of it shipped in **0.8.10** — every width is derived from the font and proved against a
> synthetic proportional face — and crab still passes `font = 0` because there is nothing to pass.

**Status:** ✅ **CLOSED by agnos 1.57.2 + rekha 0.3.8 (2026-09-13, the same day it was filed).**
The operator picked the mechanism the filing deliberately declined to pick: **a kernel-owned `/fonts`
namespace**, face embedded kashi-style rather than staged as an asset. rekha 0.3.8 generates
`fonts/face_data.cyr` — **Liberation Sans Regular 2.1.5, unmodified, 410,820 bytes, SIL OFL 1.1**
(the licence text must travel with any redistribution) — and the kernel assembles it at boot,
**verifying it by FNV-1a-64 against the generator's hash** before exposing it.
⇒ **`/fonts/default.ttf`** is the stable contract crab should code against;
`/fonts/LiberationSans-Regular.ttf` is the same bytes under the provenance name. Read-only by
construction, a `VFS_MEMFILE` fd — ⛔ **`lseek` is -1, so it must be read front-to-back in one pass**
— and `stat` reports `st_size 410820`. Contract: `agnos-userland-abi.md` §3.5.
⚠ The kernel verifies because **cyrius 6.6.3 silently corrupts even-length string literals ≥ 64 KB**
(found while generating this face, filed in cyrius). crab is on **6.6.4**.
**Original status:** 🔴 OPEN — upstream. Not a bug in crab, and not a bug in rekha.
⭐ **OPERATOR RULING, 2026-09-13:** *"rekha is that thing... but has yet to get Kernel support."*
⇒ **rekha is the designated answer for proportional text.** This is therefore **not** a "choose a
font" question and crab must not treat it as one — it is an agnos-side arc that has not been walked
yet. What follows is the need stated from crab's side, and deliberately **not** a design for agnos's
answer: the VOLUMES precedent is the house rule here — crab filed the need, declined to approximate
it with a probe, and agnos chose to mint `mountlist`#104 rather than widen `mount`#11. *Declining to
approximate is what got the right primitive built.*

**Severity:** Low and patient. Nothing is broken; a roadmap item is blocked, and crab has already
done everything it can do without it.

## What crab needs

One thing: **a TrueType face that `crab_fs_open_r` can open on the target**, at a path crab can know.
That is the whole ask. Whether it arrives as a staged asset, a kernel-owned font service, a rekha
integration into agnos, or something none of this file anticipates is agnos's call to make.

⚠ **Two constraints come from rekha 0.3.7 and are worth knowing before a face is chosen**, because
they narrow the field and are cheap to check up front:

- **`glyf` outlines only.** CFF/OpenType is rejected outright at `rekha_font_open` — it accepts
  sfntVersion `0x00010000` or `'true'` and nothing else.
- **A format-4 BMP `cmap`.** The codepoint-to-glyph path reads that subtable.

## What was checked, and what was found

⛔ **Stack-wide, across nineteen first-party repos: zero `*.ttf`, `*.otf`, `*.ttc` or `*.woff*`.**
The only TrueType files on the machine belong to host packages and to an unrelated project's
`node_modules`.

⛔ **The `agnos` repo contains no occurrence of "ttf", "truetype" or "sfnt"** in any script, manifest
or document. `build/rootfs` is `bin/` (first-party ELF binaries), `etc/ssl/cert.pem`, an empty `fw/`
and a verification PNG — there is no `/usr`, no `/share`, and no font directory.

⚠ **The two things it would be easy to assume are missing are NOT missing**, and saying so keeps the
filing honest about where the gap really is:

- **Reading a large file works.** `lib/io.cyr`'s `file_read_all` loops to a short read with no cap,
  and `file_open` carries the AGNOS `AO_*`/namelen bridge. A 410 KB face is an ordinary read.
- **A staging path exists and already carries a non-binary asset.** `scripts/burn/stage-tools.sh`
  populates `build/rootfs`, and fs-population copies it whole — `mke2fs -d build/rootfs` for
  QEMU, `install-media.sh` by label for iron. `etc/ssl/cert.pem` (185 KB) is already delivered by
  exactly that route.

⇒ So the gap is **not** "crab cannot read it" and **not** "there is no mechanism to put a file
there". It is that the face, and whatever agnos decides owns it, do not exist yet.

## ⛔⛆ The trap, written down because it would look like success

The one caller in the whole stack that feeds `rekha_font_open` real file bytes is
`dhancha/programs/setu_demo_client.cyr`:

```
var fontpath = "/usr/share/fonts/liberation/LiberationSans-Regular.ttf";
var flen = file_read_all(fontpath, fbuf, 1048576);
if (flen > 0) { font = rekha_font_open(fbuf, flen); }
```

That is a **host Arch path**. It does not exist on AGNOS, and `flen` comes back negative there.

**Copied into crab it would work on the host build, fall back silently to the bitmap face on the
target that ships, and look finished.** The `if (flen > 0)` is a correct guard and it is exactly what
makes the failure quiet. ⇒ Any crab-side font load must be **measured on QEMU**, not observed to
compile — the same rule that made `crab-columns-test.py` and `crab-shift-test.py` necessary.

## Why kashi is not the escape hatch

kashi owns AGNOS's **bitmap console fonts** by design — PSF/BDF/PCF import and three built-in CP437
tables. It has no scalable face at all, and rekha's own README states the split: *kashi — bitmap
glyph sources; rekha — outline / Bézier glyph sources.*

⚠ And [ADR 0003](../../adr/0003-kashi-freestanding-core-over-the-library-face.md)'s written expiry
does **not** fire here. It says switch to `dist/kashi.cyr` *"the day crab needs runtime font
LOADING"* and goes on to say, in as many words, *"Not the day proportional text lands — that is
rekha's TrueType path, a different dependency."* crab already links rekha. Taking kashi's library
face would cost +183 KB and buy nothing.

## What crab did instead of waiting

**0.8.10** shipped every half of `0.9.0` that does not need the face:

- All seven character-count widths are **derived from the font** (`crab_col_name_min()` and six
  siblings) rather than written as pixel literals at kashi's 9 px advance.
- The suite **builds a synthetic proportional face** (head/maxp/hhea/hmtx/cmap, unequal advances) and
  watches every derived width move — which retires 0.8.8's own admission that no host test could tell
  *"asks the font"* from *"divides by the constant"*.
- ⇒ **When the face arrives, crab should need nothing but the load itself.** That is the point of
  having done this half first.

## Related

- [`2026-09-13-dhancha-scalable-text-allocates-per-call.md`](2026-09-13-dhancha-scalable-text-allocates-per-call.md)
  — the **second, independent** blocker on the same roadmap item, owned by dhancha. Both must clear.
  Canonical: `dhancha/docs/development/issues/2026-09-13-scalable-text-allocates-per-call-outside-the-frame-arena.md`.
