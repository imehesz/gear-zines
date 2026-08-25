# Gear Zines

Print-at-home reference zines for electronic music gear.

Every zine is a **single self-contained HTML file** that prints onto **one sheet of paper**. Fold it three times, make one cut, and you get an **8-page booklet** small enough to sit next to the machine it documents.

The target is gear whose manual you can never find when you need it — samplers, grooveboxes, synths, drum machines, pedals, Eurorack modules. The stuff with twelve buttons and four hundred key combos.

---

## Why

Manufacturer manuals are PDFs on a laptop that is closed, in another room, while your hands are on the machine. A zine is paper. It lives in the gig bag, it does not need a battery, and it costs one sheet to replace when you spill something on it.

The goal is not to reproduce the manual. It is to distill the parts you actually reach for — the combos, the knob assignments, the things that are impossible to guess.

---

## What's in here

| Zine | Gear | Sheet |
|---|---|---|
| [`akai_mpc-one/`](akai_mpc-one/) | Akai MPC One / MPC One+ | US Letter landscape |
| [`dirtywave_m8/`](dirtywave_m8/) | Dirtywave M8 (Model:01 / Model:02) | US Letter landscape |
| [`polyend_tracker/`](polyend_tracker/) | Polyend Tracker | US Letter landscape |
| [`te_ep-1320_medieval/`](te_ep-1320_medieval/) | Teenage Engineering EP–1320 Medieval | US Letter landscape |
| [`te_po-14_sub/`](te_po-14_sub/) | Teenage Engineering PO-14 sub | US Letter landscape |
| [`te_po-32_tonic/`](te_po-32_tonic/) | Teenage Engineering PO-32 tonic | US Letter landscape |
| [`te_po-33_ko/`](te_po-33_ko/) | Teenage Engineering PO-33 K.O! | US Letter landscape |
| [`te_po-35_speak/`](te_po-35_speak/) | Teenage Engineering PO-35 speak | US Letter landscape |

Every folder has the same two files — one HTML file, one cover image:

```
te_po-33_ko/
├── po_33_ko_zine.html   the zine — open it, print it
└── ko.jpg               cover art
```

The HTML has no dependencies and no build step. Open it in a browser and print.

### On the Pocket Operators

The four PO zines share a chassis — 16 keys, two knobs, 16-step patterns, and TE's
SY0–SY5 sync table — so they are laid out the same way and the sync page is common
to all of them.

What is *not* common is the button that does the housekeeping, and it is the thing
that catches people moving between units. Clearing a pattern is `record + pattern`
on the PO-33 and PO-35, `acc + pattern` on the PO-32, and `key + pattern` on the
PO-14. Each zine states its own unit's combos; do not carry them across.

Covers are 1-bit line art on purpose. The whole point is a machine you can feed one
sheet of paper — these print on a black-and-white laser with no grey to smear.

### On the MPC One, the M8 and the Tracker

These three do not share a chassis with anything else here, and all are moving targets,
so each zine states the documentation it was written against.

The **MPC One** zine follows **MPC Standalone OS 3.x**. The silkscreen on the panel is
from the 2.x era and no longer matches in one place that matters: `Prog Edit` now opens
**Track Edit**, because what 2.x called a *program* is a *track* in 3. Everything else
on the zine's button page is a hardware combo and is the same on both. The MPC One+ is
the same machine for these purposes except for its power supply, its storage, and the
Link port — which it does not have.

The **M8** zine follows **Operation Manual v6.0.0** and covers Model:01 and Model:02.
The only split between the two is the microphone: Model:02 has one, Model:01 does not,
so the `SRC` list in the sample recorder is one entry shorter on an 01.

The **Polyend Tracker** zine is written for **OS 1.9.2**, the current firmware, using the
official manual v1.9.2a *plus Polyend's own release notes* — because on this machine the
two disagree.

**Where the manual and the firmware conflict, the zine follows the firmware.** The v1.9.2a
manual still says "8 tracks" in 35 places and never mentions that OS 1.9.0 added four
MIDI-only tracks, giving 12 in total. They sit after track 8, show up in both pattern and
song mode, and cannot be hidden. Two other 1.9.x changes the manual does not carry: Render
Selection now cuts hard at both ends (Export Song/Pattern still keeps tails), and Shift
with the encoder makes *bigger* value jumps everywhere except tempo, where it gives 0.1
BPM. All three are on the zine.

Be careful which machine you are holding: *Tracker*, *Tracker+* and *Tracker Mini* are
three different products on three separate firmware lines, each with its own manual
(1.9.2a, 1.2.0a and 2.2.1b respectively at the time of writing). This zine is the original
Tracker. Do not assume a combo carries across to the other two — check Polyend's downloads
page for the manual that matches your unit.

---

## Printing

1. Open the `.html` file in Chrome.
2. **Ctrl+P**.
3. Set these three, then print:

| Setting | Value |
|---|---|
| **Paper size** | Must match the paper **physically in the tray** |
| **Margins** | None |
| **Scale** | Default (100%) |

> **The paper size is the one that bites.** The zine is laid out at the exact sheet dimensions, edge to edge. If the file is built for A4 (297mm wide) and there is Letter (279.4mm) in the tray, the driver silently clips ~18mm off one side — and no margin setting will save you, because the mismatch is bigger than any margin. If a whole column comes out cut off, this is almost always why.

"Background graphics" does **not** need to be ticked. The stylesheet forces `print-color-adjust: exact` so the black label bars and key chips survive regardless.

### Switching paper size

Two edits at the top of the stylesheet, and the whole grid recomputes:

```css
:root {
  --sheet-w: 279.4mm;   /* US Letter landscape */
  --sheet-h: 215.9mm;
}
```

```css
@page { size: 11in 8.5in; }   /* or: a4 landscape */
```

For A4 landscape use `297mm` / `210mm` and `a4 landscape`. Re-check that nothing overflows afterwards — see [Panels are `overflow: hidden`](#panels-are-overflow-hidden).

---

## Folding

The printed sheet carries its own instructions on page 8, but for reference:

1. Fold in half lengthwise (hotdog), unfold.
2. Fold in half widthwise (hamburger).
3. Fold the outer flaps back, accordion style.
4. **Cut** along the thick dashed centre line — it only spans the middle two columns.
5. Open out and push the ends together to form a cross.
6. Wrap the pages around into a booklet.

Dotted lines are folds. The single thick dashed line is the only cut.

---

## Building a new zine

Copy an existing folder and replace the content. The layout machinery is the part worth understanding.

### The grid

Eight panels, four columns by two rows, derived from the sheet so the panel edges always land on the creases:

```css
--col: calc(var(--sheet-w) / 4);
--row: calc(var(--sheet-h) / 2);
```

Page order on the sheet is fixed by the fold and is **not** left-to-right:

```
┌─────────┬─────────┬─────────┬─────────┐
│  5 ↓    │  4 ↓    │  3 ↓    │  2 ↓    │   top row, rotated 180°
├─────────┼════ cut ══════════┼─────────┤
│  6      │  7      │  8      │  1      │   bottom row, upright
└─────────┴─────────┴─────────┴─────────┘
```

Page 1 (the cover) is bottom-right. Page 8 is the back cover and carries the folding instructions.

### Margins: the two-value system

Panels use asymmetric padding, because the two kinds of edge have completely different requirements:

```css
--pad-in:  3mm;   /* sides facing a FOLD — ink can go close */
--pad-out: 10mm;  /* sides facing the PAPER EDGE */
```

`--pad-out` has to clear the printer's unprintable border (~4mm on a consumer laser) **plus** a couple of millimetres of sheet-feed registration drift. 10mm is comfortable. If text still clips at the edges after the paper size is confirmed correct, raise this one value.

**The gotcha:** the top row is rotated 180°, so in content coordinates its "bottom" is the sheet's *top* edge, and its "left" is the sheet's *right*. This means **every panel's content-bottom faces a paper edge**, and the outer columns need `--pad-out` on the side that looks wrong:

```css
.p5 { padding-right: var(--pad-out); }  /* col 1, rotated → clears the LEFT edge  */
.p2 { padding-left:  var(--pad-out); }  /* col 4, rotated → clears the RIGHT edge */
.p6 { padding-left:  var(--pad-out); }  /* col 1, upright                          */
.p1 { padding-right: var(--pad-out); }  /* col 4, upright                          */
```

### Panels are `overflow: hidden`

Overflowing content is **silently clipped** — it looks fine in the browser and loses its last two bullets on paper. Do not eyeball this. Measure it.

Inject a script that compares each panel's content height against its available height, and dump the DOM headlessly:

```js
document.querySelectorAll(".panel").forEach(p => {
  const st = getComputedStyle(p);
  const pt = parseFloat(st.paddingTop);
  const avail = p.clientHeight - pt - parseFloat(st.paddingBottom);
  let last = 0;
  [...p.children].forEach(ch => {
    if (getComputedStyle(ch).position === "absolute") return;
    last = Math.max(last, ch.offsetTop + ch.offsetHeight);
  });
  console.log(p.className, (last - pt) / avail);
});
```

```sh
chromium --headless --disable-gpu --dump-dom "file://$PWD/zine.html"
```

Aim for **≤95% fill**. Print drivers hint fonts slightly differently than headless Chromium, and the last line is the one that vanishes.

Worth checking the same way: the generated PDF's `MediaBox` should exactly equal the target sheet, and the job should be **one page**.

```sh
chromium --headless --disable-gpu --no-pdf-header-footer \
  --print-to-pdf=proof.pdf "file://$PWD/zine.html"
```

### Content

Roughly 1,500 characters per panel at 6pt. What earns its place:

- **Button combos.** The single highest-value thing. Unguessable and constantly forgotten.
- **Knob assignments per mode.** X does one thing in trim and another in envelope.
- **A glossary**, if the hardware uses non-obvious labels.
- **Recipes** — sync to a Pocket Operator, drive external MIDI.

What does not: marketing copy, spec-sheet trivia you will never act on, anything you would only read once.

**Check the manufacturer's own documentation.** Gear is often a re-skin of another unit with every label renamed, and third-party write-ups happily use the wrong ones.

Where a unit's OS is still being revised, say which version the zine was transcribed from — see the MPC One note above for what happens when the printed panel and the current OS disagree.

---

## Contributing

New gear is welcome. One folder per unit, named `<maker>_<model>_<variant>`, self-contained HTML, no build step, no external assets. Say which paper size you laid it out for, and confirm it prints on that paper before opening a PR.

Corrections to existing zines are just as welcome — if a combo is wrong, it is wrong in someone's gig bag.

---

## Notes

These are unofficial fan-made references. Not affiliated with, endorsed by, or supported by any manufacturer. Product names and trademarks belong to their respective owners. Every combo is transcribed from the manufacturer's published documentation and checked against the hardware where possible, but verify anything destructive before trusting it on stage.
