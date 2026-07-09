# Requirement

## Original request
Make me a game of a kid, exactly, running and collecting coins. The kid runs from left to right and the coins are scattered at different heights. The longer you hold down the space key, the higher the jump will be. The jump happens the moment you let go of the space key. That is, by holding it down it's like you're charging up the jump. Don't ask me any more questions. Just do it.

## Summary
A 2D side-scrolling runner game. A kid character automatically runs from left to right across the screen, collecting coins that are scattered at different heights. The only control is the spacebar: holding it down charges up the jump, and the jump happens the instant the key is released, with jump height increasing the longer the key was held.

## Clarified details
The kid asked to skip further questions and have the game just be built, so no questions were asked. Sensible defaults were applied for everything not explicitly specified:
- Game style: continuous side-scrolling runner - the kid character auto-runs left to right; the world scrolls to keep the action on screen.
- Controls: spacebar only. Holding it charges the jump (with simple visual feedback, e.g. a charge meter or character crouching), and releasing it triggers the jump - the longer the hold, the higher the jump, up to a reasonable maximum charge.
- Coins: scattered at a mix of heights along the path so the charge-jump mechanic is needed to reach some of them; touching a coin collects it and increases an on-screen score counter.
- Obstacles/hazards: kept simple - gaps or low platforms that require jumping over, so the jump mechanic matters for survival, not just coin collection.
- Difficulty/progression: the game gradually speeds up over time for replay value; there's no fixed end level.
- Losing/ending a run: falling into a gap or missing a jump ends the run, shows the final coin score, and offers a quick restart.
- Visual style: simple, colorful, kid-friendly cartoon look using basic shapes/sprites (no realistic graphics needed).
- Platform: a browser-based HTML5/JavaScript game controlled with the keyboard, suitable for playing directly in the browser and deploying on GitHub Pages.

## Acceptance criteria
- The kid character continuously runs from left to right without needing left/right movement controls.
- Coins are visibly scattered at multiple different heights across the play area.
- Holding the spacebar visibly charges up the jump (some on-screen indicator of charge level).
- Releasing the spacebar triggers the jump immediately, and jump height increases with how long the spacebar was held, up to a capped maximum.
- Touching a coin removes it from the screen and increases a visible score counter in real time.
- The character can fall or fail a jump, ending the current run.
- When a run ends, the final score is shown and the player can easily restart.
- The game runs in a standard web browser using only the keyboard (spacebar).
