# Purpose of this repository

This repository *is* Vibe Studio for Kids. It does not contain a single
app — it contains the definition of an agentic environment, built entirely
out of Claude Code primitives (agents, sub-agents, skills, and whatever
else Claude Code offers), that turns a non-technical person's plain-
language app idea into a working, deployed application.

If you are an agent (or a human) working in this repository, your job is
almost never "build the app the end user described." Your job is to build
and maintain the **studio** — the pipeline of agents/skills that will do
that job on behalf of the end user, generically, for any app idea.

## The mental model

The studio behaves like a small software team. Each role is its own
standalone Claude Code agent:

- **Translator** — a Hebrew ↔ English gate sitting on every exchange with
  the end user (not just the initial idea), so the rest of the pipeline
  can work in English while the person only ever reads/writes Hebrew.
- **Orchestrator** — coordinates the pipeline and enforces gates between
  phases; builds nothing itself.
- **Intake** — turns a plain-language (possibly vague, possibly a whole
  rambling story) description into a clear product requirement, through an
  interactive, multiple-choice-style conversation rather than a one-shot
  guess.
- **Technical specification** — translate the requirement into a technical
  plan: app type, stack, components.
- **Task breakdown** — split the spec into concrete development and
  testing tasks.
- **Development + QA** — build the app, with testing following in a
  feedback loop rather than strictly a one-shot pass.
- **Review** — a final check against the original requirement.
- **Delivery** — run the app locally first; only deploy it to a shareable
  link once the end user explicitly confirms it's ready to share.

The exact agent/skill boundaries for these roles are still being designed
and are not final. See `docs/DECISIONS.md` for the reasoning behind each
of the above and for open items we've deliberately parked.

## Design constraints agreed so far

- **Generic, not domain-specific.** The pipeline must not be hardcoded for
  "games" or any other single app type. Domain judgment (does this need a
  database, is this a game loop, is this a form) is something the pipeline
  figures out per request, not something special-cased per app type.
- **Delivery target (first milestone):** static web pages (HTML/CSS/JS),
  deployed via GitHub Pages, giving the end user a shareable link with no
  server to run or maintain.
- **Entry point (first milestone):** the end user interacts with the
  studio directly inside Claude Code, via a Skill (a custom slash command)
  that takes their plain-language idea and drives the rest of the pipeline.

## Status

Early, active design. No agents/skills implementing the pipeline exist
yet. Architecture decisions are being made incrementally and
collaboratively, one component at a time, in direct conversation with the
project owner — see commit history and conversation record for the
reasoning behind each decision, not just the outcome.

## Audience note

`README.md` describes the project for a human reader (what it is, why it
exists). This file (`AGENT.md`) is for any agent — including a future
instance of Claude Code — that opens this repository to do work: it exists
so that work continues from the actual current state of the design instead
of re-deriving it from scratch.
