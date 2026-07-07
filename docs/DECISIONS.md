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
- **Safety check** — cross-cutting, sits in front of Translator on
  kid-authored input only. A light, conservative screen for content that
  needs adult attention (see Phase 2 lesson below for why it's a separate
  agent rather than folded into Translator).
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

## Lessons from Phase 1 (thin end-to-end slice)

- **Task scoping across gated stages:** the Task Planner stub scheduled a
  testing task ("verify the deployed URL") that could not possibly pass at
  QA time, because deployment is intentionally gated later in the
  pipeline. QA correctly flagged it as not-yet-checkable rather than a
  real failure; Review correctly excluded it from its verdict. Lesson for
  Phase 5/6: task lists need to be aware of which pipeline stage they run
  in, not just what "should eventually be true" — a task that can only be
  verified after a later gate belongs to the stage after that gate.
- **GitHub Pages via Actions needs two one-time, human-only steps** on a
  fresh repo: (1) enabling Pages itself (Settings → Pages → Source →
  GitHub Actions — `actions/configure-pages`'s `enablement: true` cannot
  create the Pages site itself if the default `GITHUB_TOKEN` lacks
  permission), and (2) allow-listing any non-default branch in the
  auto-created `github-pages` environment's deployment branch policy
  (Settings → Environments → github-pages), or deploys from a feature
  branch are silently rejected before a runner even starts. Both are
  one-time per repo, not per-deploy, but worth documenting since Phase 8
  (full Delivery) will hit the same thing on any new repo.

## Lessons from Phase 2 (Translator, real)

- **A safety check cannot share a call with the job it's guarding.** The
  first attempt put a "screen this input, then translate it" instruction
  in one Translator call. Even with an explicit mandatory checklist, real
  PII (a phone number + home address) was translated straight through,
  silently, twice in a row - the "translate faithfully" instruction won
  every time. Splitting it into two *mode sections within the same agent
  file* wasn't enough either: with both sections still in context on every
  call, a "nothing flagged" check call sometimes went ahead and wrote a
  full translation instead of the expected `{"flagged": false}`. What
  actually worked: two entirely separate agent files (`safety-check.md`
  and `translator.md`), each with only its own single-purpose instructions
  and no knowledge of the other job existing at all. Lesson for every
  future gate/check that sits in front of a "do the task" role (QA before
  Developer, Reviewer before Delivery, etc.): a check and the thing it's
  checking should not be able to see each other's instructions in the same
  context, not just be dispatched via a mode flag inside one prompt.

## Lessons from Phase 3 (Intake, interactive)

- **`AskUserQuestion` is not available inside subagents at all.** Confirmed
  by direct test: a subagent couldn't even load the tool's schema. This is
  almost certainly intentional (subagents are designed to run to
  completion and return, not block mid-run for human interaction), not a
  configuration gap. Any role that needs a real back-and-forth with the
  end user (not just "translate this" or "decide X") cannot own that
  back-and-forth itself - it has to be a stateless per-call decision-maker
  that the driving Skill (running in the main thread, where
  `AskUserQuestion` *is* available) calls repeatedly. This is now the
  pattern for Intake and should be assumed for any future interactive
  role rather than re-discovered each time.
- **A tool omitted from an agent's `tools:` frontmatter can fail silently
  instead of erroring.** `reviewer.md` never had `Write` listed, since
  Phase 1 - yet its Phase 1 run produced a review file anyway, so the gap
  went unnoticed for two phases. In this phase's live run, the same agent
  answered in chat instead of writing the file, and nothing in the tool
  result signaled a failure - it just silently didn't produce the
  artifact the rest of the pipeline expected. Lesson: don't infer an
  agent's tool access is correct from one successful run; check the
  `tools:` line matches what the agent's own instructions ask it to do,
  and treat "wrote confirmation in chat but the file isn't on disk" as a
  distinct failure mode to check for, not just "did the call error."
- **A subagent can mistake this session's own ambient context for file
  content.** The `safety-check` agent flagged a completely benign kid
  request as prompt injection, because the `<system-reminder>` blocks this
  environment appends to tool outputs (MCP server instructions, task list
  reminders, etc.) got attributed to the file it had just read, not to the
  surrounding session. Verified directly: the file itself, read back
  independently, contained only the kid's plain Hebrew text. This is a
  false positive in the safety mechanism added in Phase 2, not a defect in
  Phase 3's design, but it's the first time it actually fired on real
  input, so it's recorded here. Not yet fixed - worth revisiting if it
  recurs, since a safety check with a meaningful false-positive rate on
  ordinary input undermines trust in it faster than an occasional missed
  true positive would.

## Parked (not decided, not urgent)

Left as notes for a future round, not active open questions:

- Numeric quality gates (e.g. claude-sub-agent's 95%/85% thresholds) vs.
  simple pass/fail with a reason.
- Whether pipeline stage outputs are persisted as artifact files in a
  predictable per-run directory convention, or handled some other way.
