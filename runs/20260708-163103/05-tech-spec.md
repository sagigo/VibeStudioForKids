# Technical specification

## App type
Static single-page HTML app, no backend. All geometry math and rendering happen client-side in the browser.

## Components
- **Input form** - lets Moshiko choose which combination of 3 values he knows (3 sides / 2 sides + included angle / 2 angles + a side / 3 angles) and enter those values. A mode selector keeps the form clear about which fields are sides vs. angles for each case, rather than asking him to guess which combos are valid.
- **Triangle solver** - a small geometry module that takes the 3 known values and computes the remaining sides/angles using the Law of Cosines, Law of Sines, and the angle-sum rule, and validates that the inputs actually form a real triangle (side lengths satisfy the triangle inequality, angles sum to 180°, etc.).
- **Canvas renderer** - takes the solved triangle (3 side lengths, 3 angles) and draws it to scale on an HTML `<canvas>`, auto-scaling/centering so triangles of any size fit the view, drawn in blue, with all three side lengths and all three angles labeled on the figure.
- **Error message area** - a friendly inline message shown in place of the drawing when inputs are invalid or don't form a real triangle (instead of crashing or drawing nonsense).

## Stack
Plain HTML/CSS/vanilla JS with the Canvas API for drawing. No framework or drawing library needed - the geometry is straightforward trig and the visual is a single labeled shape, well within what vanilla JS + canvas handles cleanly.

## Storage
None needed - this is a stateless calculate-and-draw tool with no history, accounts, or saved triangles required by the spec.

## Deployment target
Static file(s) served via GitHub Pages.

## Notes on edge cases
- **3 angles (AAA) case**: three angles alone fix the triangle's *shape* but not its *size* (infinite similar triangles satisfy them). The solver will pick a fixed default side length (e.g. one side = 200px-equivalent unit) so a concrete, labeled triangle can still be drawn - this is a reasonable, expected interpretation of "draw a triangle from 3 angles," not a shortcut around the requirement.
