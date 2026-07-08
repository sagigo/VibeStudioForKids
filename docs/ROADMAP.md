# Studio roadmap

Long-term, high-level phase plan for building Vibe Studio for Kids itself
(the studio, not any app it will eventually produce). Phases are ordered
by priority — top first. This doc is meant to be **updated in place**: as
a phase lands, flip its status; don't renumber finished phases, and append
newly discovered phases at the end rather than inserting them.

See `docs/DECISIONS.md` for the reasoning behind specific choices, and
`AGENT.md` for the stable overview.

**Status legend:** Not started · In progress · Done · Parked

---

## Phase 0 — Foundation
**Goal:** shared understanding and repo conventions in place before any
agent/skill code exists.
**Scope:** README, AGENT.md, role list, decisions log.
**Status:** Done (living — revisited whenever design changes)

## Phase 1 — Thin end-to-end slice
**Goal:** prove the pipe's mechanics work at all, before deepening any
single role's intelligence.
**Scope:** one Skill entry point; stub/hardcoded versions of the
Orchestrator and every role; a single trivial toy request (e.g. "a page
that says hello with a big red button"); real deployment to GitHub Pages;
a working shareable link handed back. Also proves the Delivery gate
mechanic — build locally, wait for explicit confirmation, then deploy —
even though the "app" itself is trivial.
**Status:** Done. `/studio-build` Skill + one standalone agent per role
(`.claude/agents/`) built and run end to end for the fixed toy request;
QA/Review passed; human confirmed at the Delivery gate; deployed for real
via a GitHub Actions workflow to GitHub Pages —
https://sagigo.github.io/VibeStudioForKids/hello-button/. Lessons that
surfaced (see `docs/DECISIONS.md`): a Task Planner task tried to verify
the deployed URL before deployment was gated to happen, and the
auto-created `github-pages` environment's deployment-branch policy
blocked a non-default branch until manually allow-listed.

## Phase 2 — Translator
**Goal:** a real Hebrew ↔ English gate on every kid-facing exchange.
**Why now:** every later role that talks to the kid (starting with
Intake) depends on it; cheaper to build once and reuse than bolt on after
Intake exists.
**Status:** Done. `.claude/agents/translator.md` hardened for messy real
kid input (typos, rambling, mixed Hebrew/English) rather than just the one
fixed Phase 1 phrase, validated against several varied cases. Added a
light content-safety screen on kid-authored input as its own dedicated
`safety-check` agent run before Translator (see `docs/DECISIONS.md` for
why it's a separate agent, not a mode of Translator). `studio-build`
updated to call `safety-check` before `translator` on the way in, and halt
the pipeline rather than proceed to Intake if flagged.

## Phase 3 — Intake (interactive)
**Goal:** the real requirements-gathering conversation — clarifying
questions posed as concrete, multiple-choice-style options, proactive
surfacing of choices the kid may not have considered, and a "you decide"
fallback that resolves to a reasonable default.
**Status:** Done. `intake.md` rewritten as a stateless per-call decision
agent (ask next question, or finalize) after discovering `AskUserQuestion`
is unavailable inside subagents - `studio-build` drives the actual
back-and-forth (translate question → ask → translate answer → call Intake
again). Unit-tested against 5 synthetic transcripts covering vague/clear
ideas, "surprise me" vs. "let's decide together", and the finalize path.
Validated with one real live session: a kid-typed idea for a
charge-and-release-jump coin-collecting runner game, including the kid
explicitly saying "don't ask me any more questions" mid-idea, which Intake
correctly honored. Ran the resulting requirement through the full existing
pipeline (a real, non-trivial app, not Phase 1's static page) and deployed
it for real: https://sagigo.github.io/VibeStudioForKids/coin-jump-runner/.
Two more real bugs found and fixed along the way - see `docs/DECISIONS.md`.

## Phase 4 — Technical Specification
**Goal:** turn a requirement into a concrete technical plan (app type,
stack, storage needs) using generic judgment rather than per-app-type
hardcoding.
**Status:** Done. `tech-spec.md` deepened with a `Components` breakdown and
a concrete `Storage` data model (proportional - skipped when trivial), and
explicit handling for requirements that imply something GitHub Pages'
static-only hosting can't do (a server, accounts, real-time cross-device
sync): designs the closest static-only equivalent and states the tradeoff
in a `Scope adjustments` section rather than silently building something
different from what was asked. Tested across 4 varied requirement types
(a storage-backed chore tracker, a game, and two requests implying a
server - a shared leaderboard and real-time chat) - correctly skipped
scope adjustments where none were needed and applied them with sensible,
well-justified static-only redesigns where they were.

## Phase 5 — Task Planner
**Goal:** split a spec into a concrete list of development and testing
tasks.
**Status:** Done. `task-planner.md` now maps development tasks directly to
Tech Spec's `Components`/`Storage` structure (data model before things that
read/write it, foundational mechanics before things that depend on them),
respects `Scope adjustments` when present (builds the static-only version,
not the original ask), and keeps testing tasks scoped to what QA can
actually check today (static code inspection) - explicitly forbidding
tasks that depend on a later pipeline stage (the Phase 1 "verify the
deployed URL" mistake). Tested against 3 varied tech specs (storage-backed
tracker, scope-adjusted leaderboard, multi-mechanic game) - correct
dependency ordering and component mapping in all three, and the
scope-adjusted case even added a task verifying no network code sneaks
in, reinforcing Tech Spec's own scope-down decision.

## Phase 6 — Development + QA
**Goal:** real implementation and testing — sequential, with a bounded
feedback/retry loop between them (see `docs/DECISIONS.md`).
**Status:** Done. QA upgraded from static-inspection-only to real live
browser testing (headless Chromium via Playwright) for interactive/
behavioral claims, keeping static inspection only for structural/absence
claims it's actually better suited to. Developer gained a `fix` mode for
targeted, QA-report-driven fixes instead of full rewrites. `studio-build`
now runs a bounded retry loop (1 build + up to 3 fix cycles, 4 attempts
total - user's explicit choice) between them instead of stopping cold on
the first QA failure. Validated end to end on a deliberately broken toy
app: QA caught a real bug via live interaction (button click not updating
the DOM) with the exact root cause, Developer's fix mode changed exactly
one line and touched nothing else, and QA's re-run genuinely passed via
the same live interaction, not a rubber stamp.

## Phase 7 — Reviewer
**Goal:** a final pass/fail-with-reason check against the original
requirement before anything reaches Delivery.
**Status:** Done. Found and fixed a real integration gap while deepening
this: Reviewer previously only saw the original requirement, with no way
to know Tech Spec (Phase 4) had legitimately scoped down a request - it
would have wrongly failed a correctly-adjusted app for not doing something
Tech Spec already explained why it wasn't building. `reviewer.md` now also
reads the tech spec and judges against the adjusted scope when a `Scope
adjustments` section is present, without re-litigating that decision.
Tested across 3 scenarios: no adjustment + genuine gap -> correctly
FAILed; adjustment present + incomplete build -> correctly FAILed, for
real reasons unrelated to the adjustment (didn't let the adjustment become
a blanket excuse); adjustment present + complete faithful build ->
correctly PASSed. Also independently caught a QA-report/actual-app
mismatch in one test fixture that wasn't even the thing being tested for.

## Phase 8 — Delivery (full)
**Goal:** replace Phase 1's stub with the real thing — local build/run,
then remote deployment to GitHub Pages, gated on the end user's explicit
go-ahead.
**Status:** Done. The "remote deployment, gated on go-ahead" half was
already real since Phase 1 and proven repeatedly since; the actual gap
was "local build/run" - stage 1 used to just check files exist. Delivery
now actually serves the app locally and loads it with headless Chromium,
confirms no console/page errors, and captures a real screenshot the end
user sees before the gate - not just a text description. Tested on
coin-jump-runner: produced a genuine, verified screenshot of the live app.
Also hardened the deploy step for GitHub MCP tool unavailability
(happened mid-session during this phase): the push itself doesn't depend
on those tools and still works, only Actions-run verification does - the
studio now says so plainly instead of blocking or silently claiming
success/failure it can't back up.

## Phase 9 — Orchestrator hardening
**Goal:** once every role's real shape is known, harden the coordinator:
state tracking across phases, resumability after interruption, gate
enforcement, retry bounds, error paths.
**Status:** Not started (a stub Orchestrator exists from Phase 1)

## Phase 10 — Cross-domain validation
**Goal:** confirm the pipeline is actually generic by running it
end-to-end against clearly different app types (a game, a tracker, a
form-based tool) and fixing anything that turns out to be secretly
game-specific.
**Status:** Not started

---

Phases 2–8 are ordered by dependency and priority as understood today, but
Phase 1 is expected to surface lessons that reorder or reshape them —
that's the point of building the thin slice first.
