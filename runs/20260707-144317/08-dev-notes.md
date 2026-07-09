# Dev notes

## What was built

A single self-contained HTML5 canvas game at `apps/coin-jump-runner/index.html` (no other files, no build step, no external dependencies/frameworks, matching the tech spec). A kid character auto-runs left to right across a flat, continuously scrolling ground. The player charges a jump by holding SPACE (keyboard), mouse-down, or touch-hold, and releases to jump - a visible charge meter (top-right) fills over up to ~550ms of hold, and the release jump velocity scales linearly with charge fraction (min "tap" hop up to a capped max-height jump). Two obstacle types spawn ahead of the player and scroll toward them: pits/gaps (no ground under them - missing the jump means falling through and off the bottom of the screen ends the run) and rock blocks (AABB collision with the player ends the run). Coins spawn in varied patterns (low line, arc, high row) at different heights so kids need to time and size jumps to grab the higher ones; collecting a coin increments a persistent-for-the-run score shown top-left. Difficulty ramps over elapsed play time: scroll speed increases from 250 to a capped 620 px/s, and obstacle spawn intervals shrink toward a floor, so the game gets faster and busier the longer a run lasts. A game-over overlay shows the final coin count and a reason ("bumped into a rock" / "fell into a pit"), with a "Play Again" button that fully resets all game state (score, obstacles, coins, timers, player position/velocity) and returns to active play. A start overlay explains the controls (keyboard/mouse/touch) before the first run begins. The canvas resizes responsively to the window/viewport using devicePixelRatio scaling, and visuals (gradient sky, parallax clouds/hills, animated running legs, spinning coins, squash-on-landing) are drawn entirely with canvas primitives for a bright, friendly look with no image assets.

## Files

- `/home/user/VibeStudioForKids/apps/coin-jump-runner/index.html` - the entire app (inline `<style>` and `<script>`, ~733 lines). This is the only file in the target directory and is what should be deployed to GitHub Pages as-is.

## Verification performed

- `node --check` on the extracted inline script confirmed no JavaScript syntax errors.
- Automated headless-browser runs (Playwright/Chromium) exercised the full flow: loaded the page, clicked "Tap to Start", triggered jumps via keyboard (Space down/up) and via mouse hold/release, let the game run long enough for obstacles, coins, and the difficulty ramp to kick in, and confirmed: no `pageerror`/`console.error` output; the coin counter incremented on collection (reached 7 in one run); a rock collision correctly ended the run and displayed the game-over overlay with the matching final score and reason text; clicking "Play Again" hid the overlay and reset the on-screen score back to 0.
- Visual screenshots were captured and reviewed for the start screen and an in-progress/game-over frame, confirming the kid-friendly styling (sky gradient, sun, clouds, hills, running character, spinning coins, pit and rock obstacles, rounded panel UI) renders correctly at a 1000x700 viewport.

## Notes for QA / Review

- All three input modes (keyboard Space, mouse hold, touch hold) map to the same `beginPress`/`endPress` charge-and-release logic, so behavior should be consistent across devices; touch events call `preventDefault()` to avoid page scroll/zoom interference.
- Ground gaps and rock blocks are the two obstacle types called for in the requirement ("gaps/low obstacles to jump over"); there are no other obstacle/hazard types.
- No storage (localStorage or otherwise) is used anywhere, per the tech spec - score exists only in memory for the current run and resets to 0 on restart or page reload.
