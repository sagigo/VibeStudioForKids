# Dev notes — space-math-racer (build)

## What was built
A complete, self-contained space-race math game at
`C:\Users\sagig\source\repos\VibeStudioForKids\apps\space-math-racer\index.html`
(single file: inline CSS + vanilla JS + Canvas 2D, no build step, no
external requests, no storage APIs — verified statically). All kid-facing
text is in Hebrew; the page is `dir="rtl"`.

The kid flies a glowing space car down a 3-lane neon highway. The car
carries a target number (gold, the biggest/boldest element per the style
spec). Rows of add/subtract exercise tiles (operands and results within
0–100) scroll from the top toward the car near the bottom. ArrowLeft /
ArrowRight steer the car one lane at a time (clamped to the road). When a
row reaches the car: a tile in the car's lane whose answer equals the
target = correct collection (+1 score, gold pulse + green/gold particle
burst, new target chosen); a wrong-answer tile in the car's lane costs one
life (red flash + "אופס, ניסיון הבא!"); an empty lane passes safely.
Three lives; at zero, a Game Over overlay shows the final score as the
hero number with an "עוד סיבוב!" restart button (resets state in-memory,
no page reload). Speed ramps with score (150→430 px/s, capped), and the
share of hazard lanes grows with score, so it gets harder over time.

## How the acceptance-critical bits are handled
- **Solvability guarantee:** every spawned row contains exactly one tile
  whose answer equals the current target; when the target changes on a
  correct collect, `reassignTilesForTarget()` re-rolls all still-approaching
  rows so each keeps exactly one correct tile for the NEW target (and no
  previously-correct tile can unfairly become a hazard). Collision resolves
  against the LIVE target (`tile.answer === target`), not a stale flag. A
  per-frame safety net force-spawns a fresh row if nothing correct is
  currently heading for the car.
- **RTL split:** HUD (score right / lives left), the fading how-to hint,
  and the Game Over panel are RTL. The canvas/game world is `dir="ltr"`
  and direction-neutral: lane order, car position, tile spawn lanes and the
  ArrowLeft=physical-left / ArrowRight=physical-right mapping are never
  mirrored. Exercise text and the car number always render as Western
  digits LTR.
- **No start screen:** play begins on load; a non-blocking hint fades out
  after ~6s. Touch steering buttons (physical L/R) are added as a bonus but
  keyboard is the primary control.

## Palette / tone
Implements the style spec: deep-space `#0a0e2a` + faint `#241b4d` nebula,
white starfield, neon-cyan `#00e5ff` lane glow + engine trail + score,
gold `#ffd400` car number, green `#3dff8a` correct burst, red-pink
`#ff4d6d` hazard/lives. Microcopy is playful race-announcer Hebrew
("מעולה!", "יש!", "טיסה טובה בחלל!").

## Files
- `apps/space-math-racer/index.html` — the entire game (created).

## Notes for QA
- Difficulty ramp is score-driven (`speed = 150 + score*9`, cap 430) plus a
  rising wrong-tile density; both are measurable after the score climbs.
- Reference pattern followed: `apps/coin-jump-runner/index.html`
  (canvas + HTML/CSS HUD, DPR handling, requestAnimationFrame loop).
