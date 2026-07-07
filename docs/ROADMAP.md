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
**Status:** Not started

## Phase 6 — Development + QA
**Goal:** real implementation and testing — sequential, with a bounded
feedback/retry loop between them (see `docs/DECISIONS.md`).
**Status:** Not started

## Phase 7 — Reviewer
**Goal:** a final pass/fail-with-reason check against the original
requirement before anything reaches Delivery.
**Status:** Not started

## Phase 8 — Delivery (full)
**Goal:** replace Phase 1's stub with the real thing — local build/run,
then remote deployment to GitHub Pages, gated on the end user's explicit
go-ahead.
**Status:** Not started

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
