# Tasks

## Development
1. Create the base HTML page with a `<canvas>` element and link the CSS/JS files.
2. Set up the `requestAnimationFrame` game loop and scrolling background/ground.
3. Implement the auto-running player character with charge-and-release jump input (mouse/touch/keyboard hold-and-release).
4. Add scrolling obstacles with collision detection that ends the run.
5. Add collectible coins with collision detection and a running score counter.
6. Implement difficulty ramp (increasing speed/obstacle frequency over time).
7. Implement game-over screen showing final score and a restart action that resets state.
8. Style the game for a simple, kid-friendly look and ensure the canvas scales to the browser window.

## Testing
1. Verify the charge-and-release jump control works correctly across mouse, touch, and keyboard input, with longer charge producing a higher/longer jump.
2. Verify obstacle collisions correctly trigger game over and coin collisions correctly increment the score.
3. Verify the difficulty ramp increases speed/obstacle frequency over time without breaking collision or scrolling.
4. Verify the game-over screen shows the correct score and restart fully resets the game state.
5. Verify the game runs smoothly in a browser with no backend/storage, matching the static GitHub Pages deployment target.
