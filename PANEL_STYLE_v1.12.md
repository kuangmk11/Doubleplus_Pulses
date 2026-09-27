# Front panel style

**Version 1.12.**

House style for my module front panels — what marks go on them, what each one means, and
where it sits. It is a drawing spec, not a program: a panel is in the style if it satisfies
the rules below, whether it was generated or drawn by hand.

The look is borrowed from RYO's *Altered States* (`reference.jpg`) and is deliberately
PCB-native: single-weight plotter lettering, connections routed the way a trace is routed,
and dots where a silkscreen would have vias. Printed as white silkscreen on black
soldermask.

The rules were fitted to a set of eleven generated Kassutronics panels. That is where the
numbers come from, and why some are stated to the hundredth of a millimetre — they were
measured off production artwork, not chosen.

## Two ways to draw one

**Generated.** One `<stem>.layout.json` per panel says what its controls are and how they
relate; every mark comes from a single style library, so changing a number there changes
every panel at once.

```sh
tools/build_styled_panels.sh            # every finished layout
tools/build_styled_panels.sh Slope      # just the ones whose path matches
```

Outputs land beside each source as `<stem>.styled.{panel.json,kicad_pcb,svg}`, leaving the
PDF-derived conversions untouched.

**Drawn.** A hand-written SVG at 1:1, one user unit = 1 mm — `docs/v3-panel.svg` in this
repo is one. Nothing below needs a generator; the tokens are just numbers.

Where a rule is easier to state as tool behaviour ("the composer refuses to write if…"), it
is stated here as the rule being enforced. A generator should enforce it. A hand drawing has
to be checked.

## The one rule that is not negotiable

**Control hole positions are inherited, never computed.**

A panel serving a board that already exists — a port of someone else's module, or a revision
like this adder's v3 — takes its hole positions from that board and may not move them. A
panel whose holes move stops fitting its main board. v3 reassigns hole *diameters* between
6 mm jack and 5 mm toggle and keeps all sixteen positions from the built v2.2 panel.

On a genuinely new board the positions are fitted once, when the board is laid out, and are
inherited from then on.

Either way, **every hole is accounted for**, so a jack cannot silently vanish because a
layout forgot it.

Mounting holes are the exception: they meet the rails, not the module's own board, so they
are regenerated (below).

## Tokens

Every value is one number in one place. A generator holds them together; a hand drawing
copies them from here.

| | value | |
|---|---|---|
| `LINE_W` / `RULE_W` | 0.25 / 0.20 mm | wires, brackets, rings / centre divider |
| `LABEL_SIZE` | 2.0 mm | control labels |
| `SMALL_SIZE` | 1.6 mm | crowded rows, notes, bracket labels |
| `TITLE_SIZE` | 3.2 mm | module name |
| `LOGO_SIZE` | 1.3 mm | wordmark |
| `TRACK_LOGO` / `TRACK_TIGHT` | 0.34 / 0.22 mm | tracking at `LOGO_SIZE`; 0.26 / 0.17 of cap |
| `TRACK_MONO` | 0.06 mm | the `MMM` monogram, set tight |
| `TRAVEL_DOT_D` / `TRAVEL_END_D` | 0.9 / 1.4 mm | knob travel marks |
| `DOT_D` | 0.6 mm | star-field, when used at all |
| `RING_R` / `RING_W` | 4.75 / 0.5 mm | the output ring: fixed radius, drawn heavier than `LINE_W` |
| `LABEL_GAP` | 1.2 mm | drawn extent → nearest edge of the label cell |
| `LABEL_GAP_HW` | 2.2 mm | the same gap at a jack or a switch, where a nut takes the extra |
| `EDGE_MARGIN` | 0.6 mm | no silkscreen closer than this to the cut |
| `DASH_LEN` / `DASH_GAP` | 0.8 / 0.6 mm | a normal: the only dashed line on a panel |

**The face is Routed Gothic.** Every letter, on the panel and off it — silkscreen, the
wordmark, web artwork, banners, favicons. It is the technical-drawing lettering the style
was always reaching for: uniform stroke, rounded terminals, no optical corrections, and
correct by construction rather than by eye.

Use the **stroke source**, not the compiled font: `src/basefont/routed-gothic-stroke-source.sfd`
in [dse/routed-gothic](https://github.com/dse/routed-gothic) holds skeletons before the
stroke is expanded, so a glyph is a centreline that can be drawn at `LINE_W` like any other
mark. Its cap height spans 656 units on the baseline at y = 40; scale that to `LOGO_SIZE`
(or `LABEL_SIZE`, or whatever the row calls for) and the letters land at the house weight.
One wrinkle: the source contours are thin closed racetracks about 0.032 mm across, so
stroke at `LINE_W − 0.032` to finish at a true 0.25.

Routed Gothic is **SIL Open Font License 1.1** — free for commercial use, which is the
reason it is the house face and not Gorton. GortonDigital is the closer revival of the
original engraving type, but it is licensed for non-commercial use only and so cannot go on
anything the shop sells. Where a shape is in doubt, Gorton is a legitimate reference to
*look* at; only Routed Gothic outlines get shipped.

**Tracking.** The reference's lettering is monospaced and widely tracked, but Routed Gothic
is proportional, and fixed pitch strands its narrow glyphs: `I` is 0.032 mm of ink, which
floats in the middle of a 1.45 mm cell with 1.418 mm of air either side against the `M`'s
0.340 mm. So text is still set **one character at a time** — KiCad has no letter-spacing
setting, and the generator places each character anyway — but each advances by **its own
width plus a fixed tracking**, not by a shared pitch. Everything is uppercase.

| | value | |
|---|---|---|
| `TRACK_LOGO` | 0.34 mm | the wordmark, widely tracked as the reference is |
| `TRACK_TIGHT` | 0.22 mm | panel labels, and the wordmark where the width is short |
| `TRACK_MONO` | 0.06 mm | the `MMM` short form |

`TRACK_MONO` is tighter than either because the short form is a **monogram, not a word**:
three identical letters that should read as one mark. Tracked at `TRACK_LOGO` they drift
apart and start reading as an abbreviation being spelled out. Judge it at the size the mark is actually reproduced — the site header renders it 44 px
tall — not at the size you draw it. At 0.06 the diagonals close right up without touching;
by about 0.04 they start to merge.

Both are ratios of cap height — 0.26 and 0.17 — so a label at `LABEL_SIZE` tracks 0.52 /
0.34 mm and a title at `TITLE_SIZE` tracks 0.83 / 0.54 mm. At `LOGO_SIZE` they come out at
the two values above. `TRACK_TIGHT` on the full wordmark returns it to 28.85 mm, within a
rounding error of the 29.0 mm the old fixed pitch produced, which is what makes it the
drop-in choice when a panel is short of room.

The `PITCH_*` tokens are retired. They described a monospaced setting the house face does
not have.

## Marks and what they mean

**Knob travel — a dotted arc.** Eleven dots every 30°, at every position except +90°; the
gap at the bottom is where the label goes and the two dots either side of it, at +120° and
+60°, are the ends of travel and are drawn heavier. These positions are not invented: they
are measured off the starbursts on the ASR, both Attenumix panels and the Avalanche VCO,
which all draw the same 300° pot. The radius sits inside the band that artwork occupies —
3.75 to 8.06 mm on a 7.5 mm pot — so the marks stay visible with a knob fitted.

```
        ·  ·  ·               11 dots, r = hole_r + 3.45
      ·         ·
    ·     (O)     ·           hole
      ·         ·
     ●           ●            heavy: ends of travel, +120 / +60
          FOLD                the 30 deg gap at +90
```

- a **small** knob pulls the radius in for tightly spaced rows (Slope's Shape/Sustain sit on
  13.3 mm centres);
- a **big** knob draws **two** concentric rings, for an oversized main knob whose starburst
  runs out to 18.5 mm rather than 8.5 — the 3340 and KS-20 Frequency, the Ladder Filter, the
  Wavefolder's Fold;
- **bipolar** adds a heavier dot at 12 o'clock for the centre detent, and sets `-` / `+`
  outside the travel ends;
- **named ends** do the same with words, for the KS-20's `LP` / `HP` Mode.

**A ring means signal leaves here.** Every jack is an input or an output, and an output
draws the ring. Keeping it the only closed circle on a panel is what lets direction read at
a glance — which is also why nothing else may be a closed circle.

The ring is **`RING_R` = 4.75 mm radius at `RING_W` = 0.5 mm**, both absolute: the radius is
not measured out from the hole edge, and the line is twice the panel weight. A 6 mm jack
leaves 1.75 mm of bare panel between hole and ring, which is the gap a fitted nut needs
before the circle starts reading as part of the hardware. Drawn at `LINE_W` it disappeared
next to the type around it; at `RING_W` it is the heaviest mark on the panel, which is the
right ranking for the one thing that states direction. The retired `RING_GAP` token
described the old hole-relative construction and is gone — a ring is a fixed circle now, the
same size on every panel whatever the jack.

**A bracket groups one direction only.** It says *these belong together*, so enclosing an
input and an output in one misstates the signal flow. A bracket that mixes directions is an
error, not a judgement call. Labels sit in gaps in the bracket's own top edge. An output row
inside a bracket does not also get rings — the bracket already says it.

**An outline groups by function, not direction.** A plain rounded box at the lighter rule
weight, drawn round everything belonging to one section — the Quantizer's two channels, each
of which necessarily has both an input and an output, so the bracket's one-direction rule
does not apply. Where a bracket labels a row in its own top edge and must not mix
directions, an outline just says *this lot is one thing*.

**A span is one shared label** over a bar covering the controls it names, with a tick turning
down at each end — the 3340's two V/Oct inputs get one word, not the same word twice.

**A wire is a connection**, routed with one knee using only 0°, 90° and 45° segments,
stopping short of what it joins and breaking around any label it passes under. Routing does
**not** avoid obstacles: check that a wire does not cross a control, or drop it.

**A dashed line is a normal** — a connection that holds until a cable goes in, and breaks
when one does. Every normalled input is called out, because it is the one behaviour a
player cannot see: an unpatched input that is quietly receiving a signal. A solid wire says
*always connected*; a dash says *connected until you patch here*, which is exactly what a
switched jack does. It is drawn at `LINE_W` in `DASH_LEN` dashes separated by `DASH_GAP`,
starting and ending on a dash, and nothing else on a panel may be dashed.

- **Normalled from another jack** — channel 2's input taking channel 1's signal. The dash
  runs from the source jack to the normalled one, routed and stopped short exactly like a
  wire. Direction needs no arrow: the source is an output or an input further up the chain,
  and the ring already says which.
- **Normalled to a fixed voltage or an internal signal** — the Attenuverter's inputs, each
  normalled to +5 V so that with nothing patched its knob becomes an offset. There is no
  source jack to run to, so the value is set as text at `SMALL_SIZE` on the side of the jack
  **opposite its name**, `LABEL_GAP_HW` clear of the nut, with a single `DASH_LEN` dash
  between the value and the jack's drawn extent. The dash is what separates it from a name:
  `+5V` on its own above a jack reads as that jack being called `+5V`.

Write the value as the circuit supplies it, **with its sign**: `+5V`, `-10V`, not `5V` — on a
±12 V system the sign is part of the value. An internal signal takes its short name, `LFO`,
`NOISE`. Mark every normalled input where it sits, even when a whole column shares one
value: the four Attenuverter channels each carry `+5V`, for the same reason every ++PULSES
toggle carries its own throw marks — the answer belongs at the jack the hand is on.

Normals to ground are **not** marked. An input switched to 0 V when unpatched behaves as if
nothing is there, which is what an unmarked input already says.

## Layout

**A name goes below the thing it names.** Every jack, every LED, every pot, every switch —
one rule, no exceptions, measured from whatever is drawn around the hole (a pot's travel
ring, an output's ring) and not from the hole itself. A panel where some names sit above
and some below makes the reader work out which each time.

**Text clears a jack or a switch by `LABEL_GAP_HW`, everything else by `LABEL_GAP`.** The
gap runs from the drawn extent to the nearest edge of the label cell, and it applies to a
name below a control and to a position mark beside one alike. Jacks and switches get the
extra millimetre because neither is flush with the panel: a nut, and on a toggle its
shoulder, stand proud of the artwork and swallow the clearance a drawing says is there.
Measured on `++PULSES`, the `A` / `B` throw marks at 1.2 mm sat against the toggle nuts and
the `EXT` name tucked under the jack nut — all three arithmetically clear, all three wrong
in the hand. LEDs, pots and buttons keep `LABEL_GAP`: a travel ring is already drawn out
past anything a knob covers, and an LED has no nut.

**A mark that means a position goes where that position is.** Travel dots ring the pot,
`+` and `−` straddle a toggle, selector values stack beside the throws they select. These
are not names and the rule above does not apply to them — a `+` under a switch is wrong
however consistent it looks.

That leaves the case where a switch needs a name *and* position marks. The name takes the
baseline of the topmost position mark, off to one side of it. It must not sit directly
outboard of a `+` on the same baseline, or the two read as one word — `+OCTAVE`.

Three things adjust themselves and should not be fought:

- a row whose labels would collide, or would need shoving back inside the board, drops to the
  small size **as a whole row**;
- every label in a row is snapped to one baseline, so a ringed output does not sit 1.2 mm
  below its plain neighbours;
- the title shrinks to fit the clear span between the mounting slots.

When a row still needs help, in the order to reach for it: the small size, a larger label
gap, the label moved to a named direction, the label placed at an explicit `[x, y]`. An
explicit title height and size exist for panels where a control near the top leaves only
~2 mm of clear band.

## Mounting

Regenerated to one rule, as 6.4 × 3.2 mm obround slots milled on Edge.Cuts — horizontal, so
a module can slide ±1.6 mm to meet its neighbours.

| | |
|---|---|
| vertical | centre 3.0 mm from the top and bottom edges |
| horizontal | centre 7.5 mm in from the edge; the opposite column steps out by N × 5.08 mm |
| **> 6 HP** | both columns, 4 slots — 8 HP at 7.5 / 32.9, 10 HP at 7.5 / 43.06 |
| **≤ 6 HP** | one slot top and bottom on **opposite corners**, top right and bottom left. One screw per rail is enough at that width, and diagonal placement stops the module pivoting. |

The 7.5 mm inset is the datum and the grid steps out from it, so the far margin lands at
7.26–7.44 rather than exactly 7.5. That is the way round that matters: the grid is what the
rail is drilled to.

Where the panel is inherited (above), its existing mounting holes win — a v2.2-era panel with
round Ø3.2 holes keeps them.

## The wordmark

**The mark names who designed the module.**

| the design is | the panel says |
|---|---|
| mine, original | `MISSING MILE MODULAR` — `MMM` where the width does not allow it |
| a port or revision of someone else's | that maker's name — `KASSUTRONICS`, `KT` on 4 HP |
| this precision adder | `508`, which is what the built v2.2 panel carries, kept for continuity |

Whatever the word, it is letterspaced inside a plain two-lead component frame — the same
schematic-fragment idea as RYO's, a different part. It sits at `height − 7.0`, above the
mounting line, because a 6.4 mm slot leaves only 17.8 mm between the bottom pair and a full
wordmark needs 19.0. At 4 HP the bottom margin has only ~7 mm of clear width, which is what
the short forms are for.

## The back of the panel

The front says who designed the module in three letters. The back says it in full, and
says what may be done with the design — it is the one surface with room, and nothing on
it competes with the artwork.

Two things go there, both on `B.SilkS`, uppercase and letterspaced like everything else,
at `SMALL_SIZE` or just under:

| | |
|---|---|
| credit | `DESIGNED BY MISSING MILE MODULAR <year>` — the year the artwork was cut, not the year the circuit was designed |
| licence | the short form **with its version**, e.g. `CC BY-NC-SA 4.0`. No URL: it will not be read off the back of a module, and it costs a line |

The licence is stated exactly, clause letters and all. `CC-BY-SA` and `CC BY-NC-SA 4.0`
are different licences — one permits commercial builds of the module, the other does not —
and the back of the panel is where that is on the record. Carry the version number too: the
CC licences are not identical across versions, and a bare family name dates badly.

Wrap the credit rather than shrink it. Below about 1.2 mm the strokes start to fill in on
black soldermask, so a long name becomes two lines at the small size, never one line at a
size chosen to fit.

**Placement is whatever the holes leave.** Control holes go clean through, so the back has
no artwork of its own to work around and no keepouts but the holes themselves. Find the
clear bands between hole rows and set the type in those; a band needs the line height plus
`EDGE_MARGIN` top and bottom. There is no fixed position, because no two panels leave the
same gaps.

**Back text is emitted right-to-left.** KiCad mirrors each glyph about its own anchor, so
a run of cells laid out left-to-right in board coordinates reads backwards once the board
is turned over. Since the tracking rule already sets every character in its own cell, the
fix is to reverse the cell order — the string is unchanged, the cells are dealt out from
the right. A generator does this; a hand drawing has to be checked against a mirrored
render, not against the editor.

## Screw holes

3.4 mm holes fix the board behind the panel. They are drilled and then ignored: they need no
layout entry, take no label and get no marks. Mounting slots are treated the same way.

## Copper

**Both copper layers are poured solid, edge to edge.** One filled zone on `F.Cu` and one on
`B.Cu`, each drawn past the board outline so KiCad clips it to `Edge.Cuts` — the pour
follows the cut and any change to the outline, with no polygon to keep in step by hand.

| | value | |
|---|---|---|
| net | `GND` | named, even when nothing on the panel connects to it |
| fill | solid | not hatched |
| clearance | 0.5 mm | to holes, slots and anything else the pour meets |
| min thickness | 0.25 mm | = `LINE_W` |
| island removal | always | a floating sliver is fab dirt in copper |
| priority | `B.Cu` 1, `F.Cu` 0 | only matters if the two ever overlap on one layer |

The reason is how the panel looks and handles, not electrical. Black soldermask over copper
reads as an even, slightly raised gloss; over bare FR-4 it goes flat and patchy, and a
panel with copper in some places and not others shows the seams. A full pour on both sides
also keeps the two faces balanced so a thin panel stays flat, and stiffens it under a
tightened jack nut.

The pour is **covered by soldermask everywhere**. Nothing opens it: silkscreen sits on mask
over copper exactly as it would on mask over laminate, and no mark on the panel is ever
drawn in exposed copper. Holes and slots clear the pour by the 0.5 mm clearance, so a
plated barrel or a nut never touches copper through the mask.

For a hand drawing there is nothing to draw — the pour is a board-house setting, and it
goes in when the SVG becomes a board. For a generated panel, the composer writes both zones.
Either way, **refill before DRC** (`B` in the editor, or `kicad-cli pcb drc --refill-zones`)
so the check sees the copper that will actually be made.

## The star-field

Off. The reference's scatter reads as texture because it covers a whole panel edge to edge;
here, once the keepouts around holes, wires, brackets and type have taken their share, there
is nothing left but orphans — measured, 15 of 17 survivors on Slope had no near neighbour,
and the ASR had none at all. An orphan dot reads as fab dirt. The travel rings already carry
the polar-dot language.

It stays available, and a panel that turns out to have real open space can opt back in.

## Per-module cases

Rules that exist for one panel and are recorded so they are not mistaken for house style.

**The Kassutronics Quantizer's glyphs.** Its shift layer is iconographic, and those icons are
what its user manual teaches, so they are redrawn rather than replaced with words — twelve of
them, each built in a local box running -1..1 from the same strokes and arcs as everything
else and scaled on placement:

| button | glyph | function |
|---|---|---|
| 0 | `gate_length` | gate length — a crotchet and a minim, a short note and a long one |
| 1 | `rotate` | rotate menu |
| 2 / 3 | `transpose_both` / `transpose_one` | transpose |
| 4 / 5 | `offset_both` / `offset_one` | offset |
| 6 | `keyboard` | keyboard mode |
| 7 | `legato` | legato — two crotchets under a slur |
| 8 | `gear` | settings |
| 9 | `tau` | gate length |
| 10 / 11 | `cv_a` / `cv_b` | CV A / CV B menus |

The "both channels" glyphs carry two marks where the "one channel" ones carry one — the
distinction the original draws, and the one the manual describes. `tau` is drawn rather than
set, so it cannot depend on the stroke font carrying Greek.

**The keyboard ring inverts.** The original fills the five *black* keys as wedges on a white
panel. Straight onto black soldermask that reading breaks, so the seven **white** keys are
filled instead and the black keys are left as bare panel — same keyboard, same contrast,
opposite ink. Adjacent white keys (E–F, B–C) merge into one shape, as on a real keyboard.
Each wedge subtracts its own scale button's hole with clearance rather than printing over it.
The numbers moved outside the keyboard ring to make room, and **3 and 9 are omitted** — they
sit dead on the panel edges, exactly as on the original.

**Pulses Plus, and what an 8 HP panel of nothing but toggles forces.**

*The title wraps.* `DOUBLEPLUS++PULSES` is eighteen characters, which at `TITLE_SIZE`
wants around 60 mm on a 40 mm panel (61 mm measured under the retired fixed pitch; the
proportional setting is within a millimetre or two of it at this length). Shrinking it to fit puts the title at label size, where it
stops reading as a title at all. It breaks to two lines at the full `TITLE_SIZE` instead,
centred in the band between the top mounting holes and the first LED. Prefer two full-size
lines to one shrunken one whenever the clear band is tall enough.

*The bus indicator LEDs take no label.* Each bus is a column — indicator, mode switch,
output jack, in line — and the jack at the foot of the column is labelled `A` or `B`. A
letter on the LED as well would print the same word twice in 25 mm. This is a real
exception to "every LED gets a name", allowed only because the column reads as one thing
and the name is directly below it.

*`AND` / `MUTE` / `OR` is one stack between two switches.* Both mode switches carry the
same three positions, and the centre position cannot be marked where it is — that is the
hole. So the three words sit in the centre column, on the switches' own baselines, serving
both. Position marks normally go beside the control they belong to; here they go between
two identical controls, which is the only reading that does not double them.

*The throw marks do not.* All eight routing toggles throw on the X axis and are wired
identically, so `A`/`B` could have been said once. It is said on every switch, because it
is a position mark and not a name: the panel has to answer "which way is bus A" at the
switch the hand is on, not at the top of the column.

**LPG-MIX MK3, and adding controls to a panel that is already drawn.**

*The panel's own metric wins over this document.* MK3's panel was drawn before v1.08 retired
the fixed pitch: every label on it is 1.6 mm on a 1.70 mm pitch, one character per cell. The two
controls MK3 adds are set the same way. A label in the current proportional metric would be
correct by this spec and visibly wrong beside `MODE`. Where a panel already exists, match it, and
note the exception here rather than half-converting the artwork.

*`SW7` gets no name.* It carries three position marks — `33N` above, the unmarked centre, `110N`
below, on `SW5`'s own two mark baselines — and no name of its own. There is nowhere to put one:
`RANGE` at `SMALL_SIZE` needs about 9 mm and the only gap is the 5.2 mm between `SW5`'s `LPG` mark
and `SW7`'s hole. The three capacitance values are self-describing, and the switch is paired
vertically with its marks while the neighbouring `MODE` reads horizontally, so the two switches
do not get confused.

*`DECAY` and `SW7` get no outline.* They are one function and the outline rule would fit, but any
box wide enough to hold both also swallows the variation row's `3` mark, which belongs to a
different group. A grouping mark that has to enclose a foreign label is worse than no grouping
mark.

*Clearance on an inherited panel is checked relatively, not absolutely.* Nominal hardware radii
say the built panel collides with itself everywhere — knobs overlap, travel dots sit under the
knob, labels tuck under nuts — because the artwork is drawn tighter than nominal and works. So
the check measures what the existing panel already demonstrates for each hole diameter, and
requires new controls to be no tighter than that. Absolute thresholds only produce noise.

*Render it.* MK3's `SW7` passed every numeric clearance check at its first position and was still
wrong: the hole sat on top of an existing label the arithmetic had not been told about. Rule 4
below is not optional.

## Checking a panel

For a generated panel, start from a stub with every hole classified by diameter and each
label `TODO`; the build skips any layout still containing `TODO`.

```sh
python3 tools/panel_compose.py --stub documentation/Foo/Foo.panel.json
```

Then, in this order, however the panel was made:

1. **every control matched and every hole accounted for** — no unmatched control, no
   unaccounted hole;
2. **hole positions identical to the board's.** Compare against the drill file, not against
   the last drawing;
3. **no collisions.** For a KiCad panel, `kicad-cli pcb drc` reports **zero** violations —
   not "warnings only", since the keepouts are under our control and anything surviving is
   real. For a drawn panel, print at 100%, check the sheet measures what the drawing claims,
   and check clearance on anything unproven by an existing build;
4. **rasterise it and look at it.**

## Versioning

**Any change to this document increments the version by one hundredth** — v1.02 → v1.03 →
v1.04, and so on. There is no distinction between a typo and a new rule; every edit is a
bump.

Two things move together, and a change that touches only one of them is incomplete:

1. the version line at the top of this file;
2. the filename — `PANEL_STYLE_v1.04.md` → `PANEL_STYLE_v1.05.md`. Rename with `git mv` so
   the history follows the file.

Then update anything pointing at the old filename (`README.md`).
