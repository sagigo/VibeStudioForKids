# Vibe Studio for Kids

An agentic development environment, built inside Claude Code, that takes a
plain-language app idea — written by someone with zero technical
background, including a kid — and carries it all the way through to a
working, deployed application.

## The idea

A non-technical person describes an app in plain Hebrew — that might be one
sentence, like *"make a game where a cat jumps over fences"*, or a whole
rambling story about what they want. From that description, the environment
behaves like a small software team and produces a real, working app,
without the person ever needing to see or understand code.

## How it works (planned)

The pipeline mirrors the roles of a small software team, each backed by
its own Claude Code agent:

- **Translator** — a Hebrew ↔ English gate sitting on every exchange with
  the kid, not just the initial idea, so the rest of the pipeline can work
  in English while the kid always reads and writes Hebrew.
- **Orchestrator** — coordinates the pipeline; doesn't build anything
  itself, just decides what runs next.
- **Intake** — turns the description into a clear requirement through an
  interactive conversation: asking clarifying questions as concrete
  multiple-choice-style options (always including a "you decide" choice),
  not open-ended questions a kid would freeze on.
- **Technical specification** — translates the requirement into a concrete
  technical plan: app type, stack, components.
- **Task breakdown** — splits the spec into development and testing tasks.
- **Development + QA** — builds and tests the app, one after the other
  with feedback loops rather than strictly one-shot.
- **Review** — a final check against the original requirement before
  anything is handed back.
- **Delivery** — runs the app locally first; only deploys it to a
  shareable link once the person explicitly confirms it's ready to share.

The pipeline is intentionally generic: the same flow should be able to
produce a 2D game, a small utility app, a tracker, or a form-based tool,
figuring out domain-specific needs (e.g. "this needs a database") from the
request itself rather than hardcoding per app type.

## Status

This project is in early, active design. Nothing is built yet — the
architecture is being worked out step by step, starting from the smallest
possible end-to-end slice of the pipeline.