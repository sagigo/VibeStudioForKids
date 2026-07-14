# QA report

App: `apps/space-math-racer/index.html` (single self-contained file)
Method: served with `python -m http.server` on 127.0.0.1:8123 and driven headless
with Playwright/Chromium. The canvas game state lives in an IIFE closure, so I
instrumented the page (via `addInitScript`, without altering the app source) by
hooking `CanvasRenderingContext2D.clearRect`/`fillText` to read back the actually
rendered car target number and tile expressions (with their screen positions),
then drove real keyboard input and read the live DOM HUD. No console errors and
no page errors occurred during the entire session.

## Testing 1 — Page loads directly into gameplay
PASS - On load the game-over overlay is hidden (no start screen), the HUD shows
score `0` and 3 full hearts, and the canvas is already rendering a live scene: a
car target number (`23`) is drawn, 3 math-expression tiles are drawn, and tiles
scroll downward at ~150 px/s (measured). A pixel sample of the canvas returned
190 distinct colors (nebula gradient, starfield, neon road/lane dividers, car,
tiles), confirming the background/dividers/car/tiles are all rendered. The
gameplay loop is running immediately with no user action.

## Testing 2 — ArrowLeft/ArrowRight move one lane, clamped
PASS - Car starts centered at x=500. One ArrowLeft settles it to x≈320 (leftmost
lane); two further ArrowLefts leave it at x≈320 (clamped, no wrap). ArrowRight
then steps it to x≈500 (center) then x≈680 (rightmost); two further ArrowRights
leave it at x≈680 (clamped). Exactly three distinct lane centers (~320 / 500 /
680), one lane changed per key press.

## Testing 3 — Correct collision increments score and shows a new target
PASS - Steering the car into a tile whose value equals the current target drove
the displayed score from 1 to 2, and the car's target number changed from 85 to
86 immediately afterward (verified from the rendered car number). New target is
always distinct (`pickNewTarget` uses a do/while `t === target` guard).

## Testing 4 — Wrong collision removes tile, decrements lives, score unchanged
PASS - With score=2 / lives=2, steering into a tile whose value does NOT equal
the target dropped lives from 2 to 1 (a full heart became empty: empty-heart
count 1→2) while the score stayed at 2. The struck tile is flagged and removed.

## Testing 5 — HUD stays in sync and is independent of canvas scaling
PASS - `#score-display` is a real DOM `<span>` (not drawn on the canvas;
`el.closest('canvas')` is null) and it tracked events live through tasks 3–4.
Resizing the viewport from 1000x800 to 640x900 (which rescales the canvas) left
the HUD intact and correct (score 2, lives 2). Because the HUD is an HTML/CSS
overlay above the canvas, canvas scaling cannot affect its readouts.

## Testing 6 — Game-over overlay on losing all lives; gameplay stops
PASS - After losing all 3 lives, the overlay became visible, lives showed 0, and
the final-score element read `12`, matching the in-play score of 12. Sampling the
scene 500 ms apart, the car moved 0 px and all 9 matched tiles moved 0 px, so
tile/car updates are frozen while the overlay is shown (the update loop returns
early when `state !== 'playing'`).

## Testing 7 — Restart resets state and resumes without reload
PASS - Clicking "עוד סיבוב!" (restart) hid the overlay, reset score to 0 and
lives to 3, and tiles were present and moving again (~150 px/s). No reload
occurred: a `window` sentinel set before the click survived it, and 0 main-frame
navigations fired during the click. Static inspection confirms no
`location.reload`/`location.href`/`location.assign` anywhere in the file.

## Testing 8 — Difficulty ramp (speed/spawn increases with score)
PASS - Measured tile scroll speed was 150.5 px/s at score 0 and 295.8 px/s at
score 16 (predicted 294 from `speed = clamp(BASE_SPEED + score*SPEED_PER_SCORE,
…)` with BASE_SPEED=150, SPEED_PER_SCORE=9) — nearly double. Spawn rate also
scales, since rows spawn at fixed world spacing while `spawnAccumulator +=
speed*dt`, so faster scroll = more frequent spawns.

## Testing 9 — Solvability guarantee holds during live play
PASS - Across ~90+ frames sampled during both early play (score ~0–2) and
high-difficulty play (score ~12–16), every sample had at least one currently
visible tile whose evaluated value equals the current target (0 failures). This
is upheld by `reassignTilesForTarget()` on each collect plus the per-frame safety
net that spawns a fresh row when no correct tile is ahead.

## Testing 10 — Self-contained, no network, no storage (static)
PASS - Static inspection of `index.html` found no external references: no
`http(s)://`, no `src=`/`href=`/`<link>`, no `fetch`/`XMLHttpRequest`/`WebSocket`/
`import`/`EventSource`/`sendBeacon`, and no CSS `url()`/`@import`/`@font-face`/
`<img>`/`<audio>` asset loads. There is exactly one inline `<style>` and one
inline `<script>`. No `localStorage`/`sessionStorage`/`document.cookie`/
`indexedDB`/`caches` usage. The app directory contains only `index.html`.

## Overall
PASS
