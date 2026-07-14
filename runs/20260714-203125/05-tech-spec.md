# Technical specification

## App type
Static single-page HTML game, no backend. Real-time arcade gameplay
(continuous animation, keyboard input, collision detection) runs entirely
client-side in the browser.

## Components
- **Game view (canvas)** — the space highway itself: starfield/planet
  background, glowing neon lane dividers, the car sprite with its target
  number, and the math tiles scrolling toward the player. One continuously
  rendered scene, not separate screens, since gameplay never navigates away
  from it.
- **HUD overlay** — score and lives (3 icons/hearts), drawn as an HTML/CSS
  layer above the canvas so text stays crisp regardless of canvas scaling.
- **Game loop & difficulty ramp** — a `requestAnimationFrame` loop driving
  tile movement/spawn timing; scroll speed and spawn rate increase as score
  climbs, per the requirement's difficulty curve.
- **Target number & tile generator** — picks a new target (0-100) and
  produces a mix of addition/subtraction tiles (operands and results
  0-100), always guaranteeing at least one currently-visible tile matches
  the current target so the level stays solvable (acceptance criterion).
  Regenerates the target (and refreshes the tile set) each time a correct
  tile is collected.
- **Input handling** — `keydown` listener for ArrowLeft/ArrowRight, moving
  the car one lane at a time, clamped to the lane count so it can't drive
  off the road.
- **Collision & scoring/lives logic** — per-frame check of car lane vs.
  tile lane/position; correct match increments score and clears/regenerates
  the target; wrong match removes the tile and decrements lives.
- **Game over overlay** — shown when lives hit 0: final score plus a
  restart button that resets game state and resumes the loop, without a
  page reload.

No separate start screen is required by the acceptance criteria; play can
begin as soon as the page loads, with Game Over/restart as the only other
state the UI needs.

## Stack
Plain HTML/CSS/vanilla JS, single self-contained `index.html`. Canvas 2D
for the scrolling highway/tiles/car (smooth per-frame animation and simple
lane-based collision math are both a natural fit for canvas), with an
HTML/CSS HUD layer on top for crisp score/lives text — the same split used
in this studio's other arcade game (Coin Jump Runner). No framework or
build step needed: the entity count and game rules are small enough that
vanilla JS keeps this simple and dependency-free for static hosting.

## Storage
None needed. All game state (target number, active tiles, score, lives,
speed) lives in memory for the duration of a play session and resets on
restart or reload. The requirement doesn't ask for scores to persist
between visits, so no `localStorage` is introduced.

## Deployment target
Static file(s) served via GitHub Pages.
