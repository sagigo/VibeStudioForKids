# Requirement

## Original request
Hello I'm Moshiko. I need to practice geometry. I want a program to draw triangles by length and angle.

## Summary
A drawing tool for Moshiko to practice geometry: he enters known measurements of a triangle (any mix of side lengths and angles), the program uses geometry rules to figure out the rest, and draws the triangle with all the numbers (side lengths and angles) labeled on the picture.

## Clarified details
- Input flexibility: accepts any combination the user provides - 3 sides, 2 sides + included angle, 2 angles + a side, or 3 angles - and fills in the remaining values itself using geometry rules.
- Output: the drawn triangle shows the side lengths and angles written on the picture (not just a plain shape).
- Scope: a drawing tool only - no quiz/practice mode.
- Color: kid asked to be surprised, so the builder picks a friendly, clear color for the triangle (blue) - no further input needed.

## Acceptance criteria
- User can input a triangle by any supported combination of 3 known values (sides and/or angles).
- The program computes the missing sides/angles using standard geometry/trigonometry rules.
- The program renders the triangle to scale (or a clear proportional approximation) on screen.
- All three side lengths and all three angles are labeled directly on the drawn triangle.
- The triangle is drawn in blue, no quiz or scoring features included.
- Invalid triangle inputs (e.g., values that can't form a real triangle) are handled gracefully with a friendly message rather than crashing.
