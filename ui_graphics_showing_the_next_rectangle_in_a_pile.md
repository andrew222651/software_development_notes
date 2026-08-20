Browser extensions like Surfingkeys allow you to click links using the keyboard. To do this you first turn on "link hints", which are sequences of letters shown in a box beside each link on the page. Type the sequence of letters and you go to the link. But what if links are packed tightly together and hints cover each other up?

![Overlapping link hints](https://user-images.githubusercontent.com/889657/233379197-9de0de55-d1dc-4567-9ad2-4b7620c3b8d5.png)

One option is to have a keyboard shortcut that changes the z-index of overlapping hints so that ones on the bottom come to the top. That way we can flip through a stack of overlapping hints until we see the one we want.
Below we describe one approach to implementing this behavior.

First we need an algorithm that takes a set of axis-aligned rectangles on the screen and returns all rectangles that overlap with a given rectangle.
Doing this by directly checking against every other rectangle is too slow.
We'll use a preprocessing step initially, which takes the set of all rectangles.

Algorithm `PreProcess`:
1. `maxwidth` = maximum width over all rectangles
1. `maxheight` = maximum height over all rectangles
1. define a grid (`grid`) with cells of size `maxwidth` by `maxheight` that covers the screen,
  and for each cell define a set (initially empty) using a hash table from cell indices to sets
1. for each rectangle
    1. for each corner of the rectangle
        1. store the rectangle in the cell of `grid` that contains the corner

Proposition: If one of the input rectangles overlaps a cell in `grid` with positive area, then one of the rectangle's corners is in that cell.

Then, to compute all rectangles that overlap with a given rectangle, we can do the following. If two rectangles overlap, there must be at least one `grid` cell that they both overlap with; by the proposition, every cell overlapped by the input rectangle is one of the cells containing an input corner.

Algorithm `FindOverlappingRectangles`:
1. Create a set `cands` of candidate rectangles, initially empty
1. For each corner of the input rectangle, find the cell in `grid` that contains it and add to `cands` all rectangles stored there
1. Select those rectangles in `cands` that actually overlap with the input rectangle, excluding the input rectangle itself.

This pair of algorithms is reasonably efficient when all rectangles are axis-aligned and roughly the same size, which is the case for link hints. Alternatives for more general scenarios are R-trees and quadtrees.

Now that we can find overlapping rectangles, we need a way to cycle through stacks of overlapping rectangles.
The following algorithm takes a list `rectangles` where if two rectangles overlap, the one later in the list is shown on top.
In the context of link hints, we'd pass all hints on the screen sorted by z-index.

Algorithm `Cycle`:
1. Run `PreProcess` on all rectangles
1. Define an ordered list `to_move`, initially empty
1. Define a set `excluded`, initially empty
1. For each `rectangle` in `rectangles` in order,
    1. If `rectangle` is in `excluded`, continue
    1. Run `FindOverlappingRectangles` on `rectangle` and remove any rectangles already in `excluded`
    1. If there are any rectangles remaining, add them to `excluded` and append `rectangle` to `to_move`
1. Move all rectangles in `to_move` to the end of `rectangles`, preserving their relative order, and return it

The algorithm `Cycle` finds bottommost rectangles that still have an unprocessed overlapping rectangle above them, and sends those bottommost rectangles to the top.
For a finite pile of mutually overlapping rectangles, repeated calls move the current bottom rectangle to the top, so every rectangle reaches the top after finitely many calls.

E.g. if we have `rectangles = [a, b, c, d]` where each rectangle overlaps with its neighbors in the list, one call of `Cycle` could return `[b, d, a, c]`.

Compare to the algorithms [in VimFx](https://github.com/akhodakivskiy/VimFx/blob/a2f1c749168ef67f1122f60af1eef5369e16ed44/extension/lib/marker-container.coffee#L401) and [in Surfingkeys](https://github.com/brookhong/Surfingkeys/blob/1299b0fef7d448d8502c5af0b9b8d9b2690096ad/src/content_scripts/common/hints.js#L278).
VimFx does slow overlap detection and would just do one promotion: `[b, c, d, a]`.
Assuming the links have the same z-index, Surfingkeys just reverses the order of all hints and would give `[d, c, b, a]` so e.g. `b` would never reach the top.

To cycle in the reverse direction, simply reverse `rectangles`, run `Cycle`, then reverse the output.
