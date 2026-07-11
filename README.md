# Vibe Studio for Kids

An agentic development environment, built inside Claude Code, that takes a
plain-language app idea — written by someone with zero technical
background, including a kid — and carries it all the way through to a
working, deployed application.

## The idea

A non-technical person describes an app in their own language (Hebrew for
our first users, but the studio detects whatever language they actually
use) — one sentence like *"make a game where a cat jumps over fences"*, or
a whole rambling story. From that description, the environment behaves
like a small software team and produces a real, working app, without the
person ever needing to see or understand code.

## How it works

The pipeline mirrors the roles of a small software team, each backed by
its own Claude Code agent:

- **Safety check** — a light screen on everything the kid writes, run
  before anything else; flags content that needs adult attention.
- **Translator** — detects the kid's language and sits on every exchange,
  so the rest of the pipeline works in English while the kid only ever
  reads and writes their own language — including the finished app's UI.
- **Orchestrator** — initializes and interprets per-run state
  (`runs/<run-id>/state.json`); doesn't build anything itself.
- **Intake** — turns the description into a clear requirement through an
  interactive conversation: concrete multiple-choice-style questions
  (always including a "you decide" option), never open-ended ones a kid
  would freeze on. Respects "stop asking me questions" when the kid says
  so.
- **Technical specification** — requirement → app type, stack, components,
  data model; when an idea needs something static hosting can't do (a
  server, real-time sync), designs the closest static-only equivalent and
  says so openly.
- **Task breakdown** — splits the spec into development and testing tasks,
  ordered by dependency.
- **Development + QA** — builds the app, then tests it for real (live
  headless-browser interaction, not just reading the code), in a bounded
  retry loop (up to 4 attempts) before giving up and asking a human.
- **Review** — a final check against the original requirement before
  anything is handed back.
- **Delivery** — runs the app locally, shows the human a real screenshot,
  and only deploys to a public shareable link after explicit confirmation.
  Deployment is an explicit trigger, never a side effect of a commit.

The pipeline is intentionally generic: the same flow has produced a 2D
game, a math-education tool, and a static page, figuring out
domain-specific needs from the request itself rather than hardcoding per
app type.

## Apps built so far

| App | What it is | Live |
|---|---|---|
| Hello Button | Phase 1's trivial proof-of-pipe page | [link](https://sagigo.github.io/VibeStudioForKids/hello-button/) |
| Coin Jump Runner | Side-scrolling coin-collecting game with a charge-and-release jump (Hebrew UI) | [link](https://sagigo.github.io/VibeStudioForKids/coin-jump-runner/) |
| Triangle Drawer | Geometry practice tool - enter any 3 known sides/angles, it solves and draws the rest (Hebrew UI) | [link](https://sagigo.github.io/VibeStudioForKids/triangle-drawer/) |

## Getting started

See `SETUP.md` — the studio runs either in Claude Code on the web (zero
install) or locally on the kid's own PC (Windows/macOS/Linux). Entry
points: `/studio-build` for a new app, `/studio-update` to change an
existing one.

## Status

All 10 phases of the original roadmap are implemented and validated, plus
a post-review improvement pass (Phase 11) and local-portability support —
see `docs/ROADMAP.md` for what each phase delivered and
`docs/DECISIONS.md` for the design decisions and the real bugs found and
fixed along the way.
