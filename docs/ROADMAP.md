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
**Status:** Not started — next up.

## Phase 2 — Translator
**Goal:** a real Hebrew ↔ English gate on every kid-facing exchange.
**Why now:** every later role that talks to the kid (starting with
Intake) depends on it; cheaper to build once and reuse than bolt on after
Intake exists.
**Status:** Not started

## Phase 3 — Intake (interactive)
**Goal:** the real requirements-gathering conversation — clarifying
questions posed as concrete, multiple-choice-style options, proactive
surfacing of choices the kid may not have considered, and a "you decide"
fallback that resolves to a reasonable default.
**Status:** Not started

## Phase 4 — Technical Specification
**Goal:** turn a requirement into a concrete technical plan (app type,
stack, storage needs) using generic judgment rather than per-app-type
hardcoding.
**Status:** Not started

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
