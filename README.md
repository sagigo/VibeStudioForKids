# Vibe Studio for Kids

An agentic development environment, built inside Claude Code, that takes a
plain-language app idea — written by someone with zero technical
background, including a kid — and carries it all the way through to a
working, deployed application.

## The idea

A non-technical person types something like *"make a game where a cat
jumps over fences"* or *"make an app that tracks my allowance"*. From that
one sentence, the environment behaves like a small software team and
produces a real, working app, without the person ever needing to see or
understand code.

## How it works (planned)

The pipeline mirrors the roles of a small software team, each backed by
Claude Code agents and/or skills:

1. **Requirements** — turn the plain-language description into a clear
   product requirement.
2. **Technical specification** — translate requirements into a concrete
   technical plan: app type, stack, components.
3. **Task breakdown** — split the spec into development and testing tasks.
4. **Development + QA** — build and test the app in parallel.
5. **Review and delivery** — a final check, then hand the person their app
   (running locally and/or deployed to a shareable link).

The pipeline is intentionally generic: the same flow should be able to
produce a 2D game, a small utility app, a tracker, or a form-based tool,
figuring out domain-specific needs (e.g. "this needs a database") from the
request itself rather than hardcoding per app type.

## Status

This project is in early, active design. Nothing is built yet — the
architecture is being worked out step by step, starting from the smallest
possible end-to-end slice of the pipeline.