# Tasks

## Development
1. Build the `index.html` skeleton with the canvas element, HTML/CSS HUD
   layer, and the in-memory game state (target number, active tiles,
   score, lives, speed) that the rest of the game reads and mutates —
   nothing is persisted, so this is just plain JS variables/objects set up
   at load and reset on restart.
2. Implement the game loop & difficulty ramp: a `requestAnimationFrame`
   loop driving tile movement/spawn timing, with scroll speed and spawn
   rate increasing as score climbs.
3. Implement the target number & tile generator: picks a new target
   (0-100), produces a mix of addition/subtraction tiles (operands/results
   0-100), and guarantees at least one currently-visible tile matches the
   current target at all times; regenerates the target and refreshes
   tiles whenever a correct tile is collected.
4. Implement input handling: `keydown` listener for ArrowLeft/ArrowRight
   that moves the car one lane at a time, clamped to the lane count.
5. Implement collision & scoring/lives logic: per-frame check of car lane
   vs. tile lane/position; correct match increments score, clears the
   tile, and triggers target regeneration; wrong match removes the tile
   and decrements lives.
6. Implement the game view (canvas rendering): starfield/planet
   background, glowing neon lane dividers, the car sprite with its target
   number displayed, and the scrolling math tiles, driven by the game
   state and loop from tasks 1-5.
7. Implement the HUD overlay (HTML/CSS layer above the canvas) showing
   score and lives (3 icons/hearts), staying in sync with game state.
8. Implement the game-over overlay: shown when lives hit 0, displaying
   final score and a restart button that resets game state and resumes
   the loop without a page reload.

## Testing
1. Verify the page loads directly into gameplay (no start screen): canvas
   shows the scrolling starfield/background, lane dividers, a car sprite
   with a visible target number, and math tiles scrolling toward the
   player.
2. Verify ArrowLeft/ArrowRight move the car one lane at a time, and that
   the car cannot move past the leftmost or rightmost lane (clamped).
3. Verify colliding the car with a tile matching the current target
   increments the displayed score and that a new target number is shown
   afterward.
4. Verify colliding the car with a tile that does NOT match the current
   target removes that tile and decrements the displayed lives (one
   heart/icon disappears), without changing the score.
5. Verify the HUD score and lives readouts update live during play and
   stay in sync with in-game events (collisions), independent of any
   canvas scaling.
6. Verify losing all 3 lives shows the game-over overlay with the final
   score and a restart button, and that gameplay stops (tiles/car stop
   updating) while it's shown.
7. Verify clicking restart clears the game-over overlay, resets score to
   0 and lives to 3, and resumes tile spawning/movement — and that this
   happens without a full page navigation/reload (e.g., no
   `location.reload`/navigation call is triggered).
8. Verify the difficulty ramp: tile scroll speed and/or spawn rate is
   measurably higher after the score has climbed for a while compared to
   the start of the session.
9. Verify the solvability guarantee holds during live play: at any point
   sampled during a session, at least one currently-visible tile's value
   matches the current target number.
10. Statically verify `index.html` is fully self-contained (no external
    script/style/asset requests, no network calls) and contains no
    `localStorage`/`sessionStorage`/cookie usage, matching the spec's
    no-persistence, single-file design.
