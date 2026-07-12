# Vibe Studio for Kids — start here

This repo IS the studio (a multi-agent pipeline that turns a kid's
plain-language app idea into a deployed app), not any single app.

**Read `AGENT.md` before doing any work here** — it holds the mental
model, the agreed design constraints, and hard-won operational facts
(silent tool failures, subagent limitations, deploy-gate rules) that will
save you from re-discovering known problems.

Quick map:
- `.claude/agents/` — one agent per studio role
- `.claude/skills/studio-build/` — the entry-point Skill that drives a run
- `docs/ROADMAP.md` — phase plan + status (update in place as phases land)
- `docs/DECISIONS.md` — why things are the way they are; append, don't
  rewrite history
- `apps/<slug>/` — the apps the studio has built (deployed to GitHub Pages)
- `runs/<run-id>/` — per-run artifacts; `state.json` is the live run state

Hard rule: nothing under `apps/` goes public without the end user's
explicit go-ahead at the Delivery gate. Deployment only happens via
explicit `workflow_dispatch` — never as a side effect of a push.
