# Delivery - Local Build Report

## Status: Ready for review (not yet public)

## What it is

A "Coin Jump Runner" browser game: a kid character auto-runs left to right,
coins are scattered at different heights, and the spacebar is a
charge-and-release jump control (hold spacebar to charge, release to jump,
longer hold = higher jump, capped). Coins boost the score, falling into gaps
or hitting obstacles ends the run, and the game-over screen shows the final
score with an easy restart.

Reviewer verdict on the build: **PASS** (see
`runs/20260707-144317/10-review.md`). Every acceptance criterion from the
original requirement (auto-run, multi-height coins, charge-jump mechanic,
live scoring, fail/game-over/restart, keyboard-only play in a standard
browser) was independently checked against the code and confirmed working.

## Where the files are

- App directory: `apps/coin-jump-runner/`
- Single self-contained file: `apps/coin-jump-runner/index.html`
  - No backend, no external dependencies, no data storage - pure HTML/CSS/JS.
  - Can be opened directly in any standard browser to play right now.

## What's been done so far

- The game has been built and passed QA and independent review.
- Everything currently lives only in this local project directory and the
  project's git history on this machine - **nothing has been made public or
  deployed anywhere yet**. There is no live URL.

## What's next

This build is ready for the end user (the kid) to try locally. Making it
public (e.g., publishing to a live URL) only happens after the end user
explicitly confirms they want that - that confirmation step has not
happened yet.
