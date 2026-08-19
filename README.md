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
| [`te_ep-1320_medieval/`](te_ep-1320_medieval/) | Teenage Engineering EP–1320 Medieval | US Letter landscape |

```
te_ep-1320_medieval/
├── ep_1320_medieval_zine.html   the zine — open it, print it
└── medieval.jpg                 cover art
```

The HTML has no dependencies and no build step. Open it in a browser and print.

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

---

## Contributing

New gear is welcome. One folder per unit, named `<maker>_<model>_<variant>`, self-contained HTML, no build step, no external assets. Say which paper size you laid it out for, and confirm it prints on that paper before opening a PR.

Corrections to existing zines are just as welcome — if a combo is wrong, it is wrong in someone's gig bag.

---

## Notes

These are unofficial fan-made references. Not affiliated with, endorsed by, or supported by any manufacturer. Product names and trademarks belong to their respective owners. Every combo is transcribed from the manufacturer's published documentation and checked against the hardware where possible, but verify anything destructive before trusting it on stage.
