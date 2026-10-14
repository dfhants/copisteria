# Repro pages for kobimusic/copisteria#1

Five page images of a short original piece for two voices and piano (released under CC0, use them as fixtures if
useful). Each is read with `python -m copisteria.pipeline <page>.png --out out`, copisteria 1.3.4 at `fbc5ab6`,
copista-28m. Coordinates below are pixels in the page image, x from the left, y from the top.

The piece has five systems: a piano-only introduction (the voice staves are hidden), then four systems of
soprano and alto staves over a braced piano. Page 1 holds the introduction and two full systems; page 2 holds
two full systems.

| Page | Expected | 1.3.4 |
|---|---|---|
| `1-faint-opening-line/page-1.png` | 3 systems | 5 systems |
| `1-faint-opening-line/page-2.png` | 2 systems | 4 systems |
| `2-brace-merged-with-staff/page-1.png` | 3 systems | 4 systems |
| `2-brace-merged-with-staff/page-2.png` | 2 systems | 4 systems |
| `3-hidden-staves/page-1.png` | bars 1-4 in the piano part, the voices empty | bars 1-4 in the soprano and alto parts, the piano empty |

`annotated/` has a close-up of each case with the features below marked.

## 1. A faint opening line

`annotated/1-faint-opening-line.png`

A small page (529 x 749, staff space about 4.4 px). In every full system the line that opens the system runs
from the soprano down through the piano at **x = 37, one pixel wide and grey 204**: the line is drawn
narrower than a pixel and anti-aliased to grey.

In the gap between the alto and the piano (on page 2, y 152-215 and y 493-558) `_ink_links` searches x 16-38,
which takes in the line, but `ink_mask` counts as ink only what is darker than **200 on page 2 (188 on
page 1)**. The line is paler than that, so `left` comes out at 0.07 to 0.11, below `INK_SPLIT`, and no bar line
crosses the gap: each system is split into a choir system and a piano system.

The soprano-to-alto gap is not affected, because the choir's bracket (x 34-35, black) crosses it.

## 2. A brace merged with the piano staff

`annotated/2-brace-merged-with-staff.png`

A page at 150 dpi (1240 x 1755) carrying faint JPEG noise: next to the ink, where the paper should be 255, some
pixels are 247-253. For example, column **x = 72, beside the piano's brace in the third system (y 1442-1484)**,
holds 249-253 all the way down.

The opening line is solid black at **x = 89**, in every system, from the soprano to the bottom of the piano.

What goes wrong:

1. The staff-line finder's Otsu threshold on this near-white page is **254**, so the 247-253 noise counts as
   ink. With its 2-px gap closing, the brace and the noise beside it join the ends of the piano's staff lines.
2. The piano staves are taken to start on the brace: **x = 72-73 in the second system and x = 75-76 in the
   third**, against x = 88 for the soprano and alto in the third system.
3. `_ink_links` searches only as far as `min(a.x0, b.x0) + sp`, plus 2 px either side of each column it
   tries. For the alto-to-piano gap of the third system (y 1353-1440) that is **x = 88**: one pixel short of the
   line at 89. `left` is 0.10 (the bracket's foot) and `bars` 0.25 (one of four bar lines runs on).
4. That is between `INK_SPLIT` and `INK_LINK`, so the ink leaves the pair open and the links model decides. On page 1 it joins the second system and splits
   the third (4 systems for 3). On page 2 it splits both (4 for 2).

The line is drawn and found on the soprano-to-alto pairs. It is missed only beside the piano, where the brace
has moved the staff's start.

## 3. A system with hidden staves

`annotated/3-hidden-staves.png`

The first system (y 264-414) has **two staves**, the piano alone: the soprano and alto staves are hidden
because they rest. Every other system has **four**.

The systems are found correctly (3 on the page). `write` then fills the parts by position in the fullest
system: `sy.staves[pos]`. So the first system's two staves go to positions 0 and 1, the soprano and alto
parts. The piano introduction (bars 1-4) comes out in the voice parts and the piano part is empty for those
bars.
