# Tasks

## Development
1. Build the triangle solver module: given any of the 4 supported input combinations (3 sides / 2 sides + included angle / 2 angles + a side / 3 angles), compute all remaining sides and angles using the Law of Cosines, Law of Sines, and the angle-sum rule. For the 3-angles (AAA) case, apply a fixed default side length so a concrete triangle can be drawn. Include validation logic that checks the triangle inequality (for side-based inputs) and that angles sum to 180° (for angle-based inputs), returning a clear valid/invalid result.
2. Build the input form: a mode selector for the 4 known-values combinations, with fields that are clearly labeled as sides vs. angles for whichever mode is selected, wired to pass the entered values into the triangle solver.
3. Build the canvas renderer: takes a solved triangle (3 sides, 3 angles) and draws it to scale on an HTML `<canvas>`, auto-scaling and centering so triangles of any size fit the view, rendered in blue, with all three side lengths and all three angles labeled on the figure.
4. Build the error message area: a friendly inline message shown in place of the drawing whenever the solver reports invalid inputs or a non-real triangle, replacing the canvas output without crashing.

## Testing
1. Verify the triangle solver correctly derives the remaining sides/angles for each of the 4 modes (SSS, SAS, ASA/AAS, AAA) using the Law of Cosines, Law of Sines, and angle-sum rule.
2. Verify the solver's validation logic rejects side sets that violate the triangle inequality and angle sets that don't sum to 180°, rather than passing them through to the renderer.
3. Verify the AAA case uses a fixed default side length to produce a concrete, drawable triangle instead of erroring or leaving size undefined.
4. Verify the input form's mode selector correctly presents/labels fields as sides vs. angles for each of the 4 combinations, so Moshiko can't enter values in the wrong field type.
5. Verify the canvas renderer auto-scales and centers triangles of varying sizes so they fit within the canvas view.
6. Verify the canvas renderer draws the triangle in blue and labels all three side lengths and all three angles on the figure.
7. Verify that when the solver flags invalid input, the error message area is shown in place of the canvas drawing (no canvas rendering attempted, no crash).
