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
standalone Claude Code agent (in `.claude/agents/`), driven in sequence by
the `/studio-build` Skill (in `.claude/skills/studio-build/`):

- **safety-check** — cross-cutting; screens every piece of kid-authored
  input before anything else touches it. Deliberately a separate agent
  from Translator (see `docs/DECISIONS.md` for why that separation is
  load-bearing).
- **translator** — cross-cutting; detects the kid's language from their
  first message (`detect->en`, writing a `.lang.json` sidecar with
  language + RTL flag) and translates every exchange in both directions.
  The rest of the pipeline works in English; the kid — and the finished
  app's UI — stay in the kid's language.
- **orchestrator** — initializes `runs/<run-id>/state.json` on a new run
  and interprets it on a resumed one; enforces nothing at runtime itself
  (the driving Skill maintains state as it executes stages).
- **intake** — interactive requirements-gathering. Stateless per call:
  reads the conversation transcript, returns either "ask this
  multiple-choice question next" or "done, here's the requirement." The
  Skill owns the actual asking (AskUserQuestion doesn't exist inside
  subagents — confirmed by test, see DECISIONS).
- **tech-spec** — requirement → app type, components, stack, data model.
  Flags `Scope adjustments` when an idea needs something static hosting
  can't do, designing the closest static-only equivalent openly.
- **task-planner** — spec → ordered dev/testing tasks, mapped to the
  spec's own component breakdown; never schedules tasks that depend on
  later pipeline stages.
- **developer** — builds the app for real (`build` mode) or applies
  targeted fixes from a QA report (`fix` mode). Writes all kid-visible UI
  text in the kid's language, with correct RTL layout when applicable.
- **qa** — tests for real: live headless-browser interaction (Playwright +
  bundled Chromium) for behavioral claims, static inspection for
  structural ones. Dev+QA form a bounded retry loop (4 attempts max).
- **reviewer** — final pass/fail against the requirement *as adjusted* by
  any Scope adjustments — checks the spirit of the ask, not just QA's
  checklist.
- **delivery** — stage 1 actually runs the app locally and captures a
  screenshot for the human gate; stage 2 drafts the kid-facing hand-back
  message after explicit confirmation. Never deploys anything itself.

Deployment is an explicit `workflow_dispatch` trigger invoked by the Skill
after the human gate — never a side effect of a push (that exact accident
happened once; see DECISIONS).

## Design constraints agreed so far

- **Generic, not domain-specific.** The pipeline must not be hardcoded for
  "games" or any other single app type. Domain judgment (does this need a
  database, is this a game loop, is this a form) is something the pipeline
  figures out per request, not something special-cased per app type.
  Validated in Phase 10 against a game, a static page, and a math tool.
- **Delivery target:** static web pages (HTML/CSS/JS), deployed via GitHub
  Pages under `apps/<slug>/`, giving the end user a shareable link with no
  server to run or maintain. Ideas needing more than static hosting get an
  explicit, visible scope adjustment, not a silent substitute or a dead
  end.
- **Entry point:** the `/studio-build` Skill drives the whole pipeline,
  including every interactive exchange with the kid.
- **Nothing goes public without explicit human sign-off**, enforced three
  ways: the Skill asks via AskUserQuestion, `state.json`'s
  `delivery-deploy-gate.passed` is checked before the push, and the deploy
  workflow only fires on explicit dispatch.

## Operational facts worth knowing before you work here

- Agent-definition and skill changes only register on a session reload;
  mid-session you can test changed definitions by giving a
  `general-purpose` agent the file's contents as its brief.
- `AskUserQuestion` is unavailable inside subagents. Any interactive role
  must be a stateless per-call decision-maker driven by the main thread.
- A tool missing from an agent's `tools:` frontmatter can fail silently
  (the agent answers in chat instead of writing the file). Check artifacts
  landed on disk; don't trust chat summaries alone.
- Subagents can mistake this environment's ambient `<system-reminder>`
  scaffolding for file content; agent prompts that classify input should
  say how to tell them apart (safety-check.md does).
- Run artifacts live in `runs/<run-id>/`, numbered in pipeline order, with
  `state.json` as the live record of progress, gates, and halt reasons.

## Status

All 10 roadmap phases are implemented and validated — see
`docs/ROADMAP.md` (per-phase outcomes) and `docs/DECISIONS.md` (decisions,
plus the real bugs found and what they taught). Three apps are live; see
`README.md` for links.

## References

- [zhsama/claude-sub-agent](https://github.com/zhsama/claude-sub-agent) —
  a strongly recommended reference for this project's job. It implements a
  similar multi-agent spec→build pipeline on top of Claude Code sub-agents
  (orchestrator, quality gates, artifact handoff between phases). We
  borrow patterns from it deliberately (see `docs/DECISIONS.md`) rather
  than importing its structure wholesale — it's built for a professional
  dev team, ours is built for a kid.
- Beyond that one repo: actively search the web for other publicly shared
  Claude Code agents/skills when designing a role. Multi-agent pipelines,
  orchestration patterns, and individual agent/skill definitions are being
  shared publicly and evolving quickly; checking what already exists
  before designing a role from scratch is encouraged, not just tolerated.

## Audience note

`README.md` describes the project for a human reader (what it is, why it
exists). This file (`AGENT.md`) is for any agent — including a future
instance of Claude Code — that opens this repository to do work: it exists
so that work continues from the actual current state of the design instead
of re-deriving it from scratch.
