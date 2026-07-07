# Technical specification

## App type
Static single-page HTML5 game, no backend.
Everything (rendering, game loop, input, scoring) can run entirely client-side in the browser - no server logic is needed.

## Stack
Plain HTML/CSS/vanilla JavaScript with the Canvas API - no framework or game engine needed.
The game logic (auto-run, charge-and-release jump, scrolling, coin collision, obstacles, difficulty ramp) is simple enough to hand-roll with `requestAnimationFrame` and basic shape/sprite drawing; a framework like Phaser would add weight without real benefit here.

## Storage
None needed.
The score only needs to persist for the current run (shown at game over, reset on restart); there's no requirement for high scores, accounts, or saved progress, so no localStorage or backend storage is required.

## Deployment target
Static file(s) served via GitHub Pages.
