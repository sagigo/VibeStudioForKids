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

### Dev/QA retry loop is bounded at 4 total attempts
1 build + up to 3 fix cycles (user's explicit choice over tighter/looser
options). If QA still fails on the 4th attempt, the pipeline stops and
reports to the end user rather than retrying indefinitely or quietly
shipping something QA never actually passed.

### QA fix cycles are targeted edits, not rewrites
When Developer is invoked in `fix` mode (after a QA failure), it edits only
what QA flagged as broken and leaves everything else untouched, rather than
regenerating the app from scratch each retry. A full rewrite each cycle
risks reintroducing bugs in parts that already passed QA.

### Reviewer judges against Tech Spec's adjusted scope, not blindly the original ask
Found while deepening Phase 7: Reviewer only ever saw the original
requirement, with no visibility into Tech Spec's `Scope adjustments` (see
the Phase 4 decision above). It would have wrongly failed a correctly
scoped-down app for not doing something Tech Spec already explained it
wasn't building. Reviewer now also reads the tech spec and judges against
the adjusted scope when present - confirmed by testing that this doesn't
turn Reviewer into a rubber stamp: an incomplete build with a legitimate
scope adjustment still correctly failed, for real gaps unrelated to the
adjustment itself.

### "Local build/run" means actually running it, not checking files exist
Delivery's stage 1 used to just confirm the app's files were present and
summarize QA's report. It now actually serves the app and loads it with a
real headless browser, capturing a screenshot the end user sees before the
Delivery gate - the human deciding whether to make something public should
see it, not just read a description of it.

### GitHub MCP tool unavailability doesn't block a push, only the deploy trigger and its verification
**Superseded by the Phase 10 finding below** - at the time this was written,
a push touching `apps/**` auto-triggered the deploy, so "the push doesn't
depend on GitHub MCP tools" also meant "the deploy doesn't either." That's
no longer true: deploying now requires an explicit `workflow_dispatch`
call, which *does* need the GitHub MCP tools. A push with those tools
disconnected still saves the work, but nothing goes public until the
tools are back and the dispatch call succeeds.

### Deploy must be an explicit trigger, never implied by a push
Found live, in Phase 10: the deploy workflow auto-triggered on any push
touching `apps/**`, which meant an ordinary progress-commit (Developer's
build, made *during* the run, before QA had even started) silently
deployed the app to the public URL - hours before Review or the human
gate. Nothing inappropriate went out (the app happened to pass QA/Review
and the human happened to say yes anyway), but the mechanism was broken:
had QA or Review failed, or had the human said no, the app would already
have been public with no way to un-ship it before that decision was even
made. Root cause: `state.json`'s gate check (Phase 9) governs whether the
driver *chooses* to push and deploy, but a workflow that deploys on any
matching push doesn't care why the push happened - a backup commit and an
authorized release commit look identical to it. Fixed by removing the
`push` trigger from `deploy-pages.yml` entirely - it now only deploys on
`workflow_dispatch`, called explicitly by `studio-build` after the gate,
so committing progress during a run can never publish anything by
accident again. Lesson for any future automation that reacts to
repository state: a side effect as consequential as "make this public"
should never be implied by an action (like committing) that also happens
for unrelated, harmless reasons.

### Orchestrator owns state.json, the driver maintains it
Orchestrator isn't a continuously-running process - it's consulted at the
start of a run (to initialize `state.json`) and can be re-consulted mid-run
to interpret an existing one (status report, resumability), but the actual
moment-to-moment updates as stages complete are the driving Skill's job,
since it's the one actually executing each stage. This replaced Phase 1's
static `plan.json`, which was written once and never updated - useless for
knowing where an interrupted run actually stood.

### Gate enforcement means a real check, not just an instruction to ask
The Delivery gate isn't just "ask AskUserQuestion and hope the prose is
followed" - `studio-build` now checks `state.json`'s
`delivery-deploy-gate.passed` immediately before the real push, and
refuses to push if it isn't `true`. This makes the gate an actual
technical check with a persisted record, not only a convention that
depends on the driver remembering to honor it in the moment.

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

### Requests needing a server get a static-only equivalent, not a dead end
Static GitHub Pages hosting can't do a server, a database, accounts, or
real-time sync between different people's devices. When a kid's idea
implies one of these (a live shared leaderboard, real-time chat with a
friend), Tech Spec doesn't stop and refuse, and doesn't silently build
something different from what was asked either - it designs the closest
static-only equivalent (e.g. a local-device leaderboard, a shared-screen
pass-the-device chat) and states the substitution explicitly in a "Scope
adjustments" section, so it's visible to Review and the end user rather
than a quiet downgrade. Revisit if/when a real backend ever becomes a
supported delivery target.

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
  Phase 3's design. Addressed (not just noted) in the language-generalization
  pass right after: `safety-check.md` now explicitly distinguishes the
  kid's actual message from this session's own ambient tool-output
  scaffolding, and says to re-read the file in isolation when unsure. Not
  re-tested against the exact case that triggered it - worth confirming it
  actually holds next time real input goes through it.

## Post-Phase-3 fix: the deployed app's own UI was never localized

The pipeline translated every *conversational* kid-facing exchange
(questions, the final hand-back message) but never the app it actually
built - Developer worked purely from English requirement/tech-spec
artifacts and had no reason to write anything but English into the app
itself. The first real live run (Phase 3's coin-collecting game) shipped
with an English-language UI to a kid who'd typed their idea in Hebrew.
Caught by inspection after the fact, not by any QA/Review check - neither
role had "is the UI in the kid's language" on its list, because nothing
upstream told them what language to check against.

Decided (user's explicit call, not the simpler option): rather than
hardcoding the UI language to Hebrew to match the studio's current
single-language assumption, generalize Translator to detect whatever
language the kid actually uses. `detect->en` replaces the fixed `he->en`
direction (writing a `.lang.json` sidecar with the detected language and
whether it's RTL), `en-><lang>` replaces the fixed `en->he` direction, and
that detected language is threaded through the whole run - including into
Developer, which now must write all kid-visible UI text (not code/comments)
in that language, with correct `dir="rtl"` layout when applicable, not just
mirrored text. Retrofitted the already-deployed coin-jump-runner app to
prove the fix works on a real, previously-English UI, not just new builds.

## Parked (not decided, not urgent)

Left as notes for a future round, not active open questions:

- Numeric quality gates (e.g. claude-sub-agent's 95%/85% thresholds) vs.
  simple pass/fail with a reason.
- Whether pipeline stage outputs are persisted as artifact files in a
  predictable per-run directory convention, or handled some other way.
