# Studio design decisions

Append-only log of decisions made about Vibe Studio for Kids' own
architecture — the studio itself, not any app it will eventually build.
Each entry is what was decided and why, so nobody has to re-derive it from
chat history. See `AGENT.md` for the stable, high-level overview.

## Roles

Each role is its own standalone Claude Code agent. Two roles get merged
into one agent only when there's a specific, demonstrated case that
splitting them is pure overhead — that's the exception to justify, not the
default.

- **Translator** — cross-cutting, not a pipeline phase. Sits on every
  kid-facing exchange (not just the initial idea): Hebrew → English for
  the rest of the pipeline to work with, English → Hebrew for anything
  shown back to the kid, including Intake's clarifying questions.
- **Orchestrator** — coordinates phase transitions and gate decisions;
  builds nothing itself.
- **Intake** — plain description → clear requirement, interactively.
- **Technical Specification** — requirement → app type, stack, components.
- **Task Planner** — spec → concrete development and testing tasks.
- **Developer** — implementation.
- **QA / Tester** — testing.
- **Reviewer** — final check against the original requirement.
- **Delivery** — local build/run, then remote deployment once confirmed.

## Decisions

### Pipeline roles are one agent each
Every role above gets its own agent by default.

### Dev/QA relationship: sequential with a retry loop
Considered true concurrency (e.g. via git-worktree-isolated agents, loosely
inspired by claude-sub-agent's quality gates) but it's not required. Default
is Developer then QA/Tester, with feedback able to send work back, rather
than both working the same code at the same time.

### Orchestrator is a standalone agent
It routes work and enforces gates between phases; it does not do any of
the phases' work itself.

### Translator is a cross-cutting gate, not a pipeline phase
The end user writes in Hebrew, possibly as a whole story rather than a
single clean sentence. A dedicated Translator handles every round-trip
between the kid and the rest of the pipeline, so every other role can
assume it's working in English, and the kid never has to read or write
anything but Hebrew.

### Intake is interactive, not one-shot
Intake may ask clarifying questions and proactively surface options the
kid may not have thought of on their own. Questions are posed as concrete,
multiple-choice-style prompts — not open-ended ones a kid would freeze on
— and always include a "you decide" option that falls through to a
reasonable default when picked.

### Delivery is a two-stage, human-gated process
Build and run the app locally first. Remote deployment (e.g. GitHub Pages)
only happens after the person explicitly confirms the app is ready to
share — nothing goes public without that sign-off.

## Parked (not decided, not urgent)

Left as notes for a future round, not active open questions:

- Numeric quality gates (e.g. claude-sub-agent's 95%/85% thresholds) vs.
  simple pass/fail with a reason.
- Whether pipeline stage outputs are persisted as artifact files in a
  predictable per-run directory convention, or handled some other way.
