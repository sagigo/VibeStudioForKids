---
name: studio-update
description: Vibe Studio for Kids - change an app that already exists (e.g. "make the coins gold in my runner game"). Same safety, translation, clarification, QA, Review, and human-gated deployment as studio-build, but the Developer performs targeted surgery on the existing app instead of building a new one. For brand-new app ideas, use studio-build instead.
---

# Studio update

Change an already-shipped app in `apps/<slug>/` at a kid's request. Reuses
the same role agents as `studio-build`; the differences are called out
below. Maintain `runs/<run-id>/state.json` throughout exactly as
`studio-build` does (same halt-reason discipline, same gate enforcement).

## 0. Set up the run

- Generate a run id: `date +%Y%m%d-%H%M%S`, create `runs/<run-id>/`.

## 1. Identify the app and the change

If the request already names the app and the change, proceed. If not,
list the apps in `apps/` (with one-line descriptions from their runs'
requirement docs, if findable) and ask the kid which one to change - via
`AskUserQuestion`, translated into the kid's language once known (for the
first exchange, Hebrew is the default greeting language, as in
studio-build). Save the kid's raw change request to
`runs/<run-id>/00-request.txt`.

## 2. Orchestrator

Invoke the `orchestrator` agent with the run directory path, telling it
this is an **update to existing app `<slug>`**. It initializes
`state.json` with `update_of` set and no `design` stage (updates preserve
the app's existing look unless the change is about the look).

## 3. Safety check, then Translator (in)

Identical to studio-build step 3: `safety-check` on `00-request.txt` →
`01-check.json` (halt on flag); then `translator` `detect->en` →
`02-request.en.txt` + `.lang.json` sidecar. Record `kid_language` in
`state.json`.

## 4. Interactive Intake loop

Same driver-orchestrated loop as studio-build step 4, with one change to
the transcript's first line so Intake knows the context:

```json
{"original_idea_en": "UPDATE REQUEST for existing app '<slug>' (<one-line description of what the app currently is>): <contents of 02-request.en.txt>", "qa_history": []}
```

Intake's judgment applies as usual - a trivial, unambiguous change ("make
the coins gold") should finalize with zero questions; only genuinely
ambiguous changes earn one. Finalized output goes to
`runs/<run-id>/04-requirement.md` (this is the *change* requirement).

## 5. Tech Spec (delta)

Invoke the `tech-spec` agent, telling it this is an update to the existing
app and giving it the app directory alongside `04-requirement.md`. Per its
update-run instructions it specs only the delta. Output:
`runs/<run-id>/05-tech-spec.md`.

## 6. Task Planner

Invoke the `task-planner` agent: input `05-tech-spec.md`, output
`runs/<run-id>/06-tasks.md`. For a small change this should be a very
short list - that's correct, not lazy.

## 7-8. Development + QA - bounded retry loop

Same loop and bounds as studio-build (4 attempts max, per-attempt QA
reports `09-qa-report-<n>.md`, same halt discipline), with two
differences:
- First attempt uses `developer` mode **`update`** (inputs:
  `04-requirement.md`, the existing app directory, the kid's language;
  dev notes to `runs/<run-id>/08-dev-notes.md`). Retries use mode `fix`
  as usual.
- **Before the first Developer call**, snapshot the current app state:
  `git diff` should later show only intended changes. If the app has
  uncommitted local changes before you start, stop and sort that out
  first rather than mixing them into the kid's change.

QA tests the *change* (from `06-tasks.md`) - but its testing tasks should
include at least one regression check that the app's core existing
behavior still works, since update-mode surgery can break what was
already shipped. Ask task-planner for that explicitly if the spec didn't
imply it.

## 9. Reviewer

Same as studio-build: inputs `04-requirement.md` (the change requirement),
`05-tech-spec.md`, `09-qa-report.md`, app directory; output
`runs/<run-id>/10-review.md`. The bar: the change the kid asked for is
faithfully made AND the app is still the app it was - an update that
satisfies the change but degrades the rest should FAIL.

## 10-12. Delivery stage 1, the gate, deploy

Identical to studio-build steps 12-14: local run + screenshot
(`11-delivery-local.md`, `preview.png`), show the screenshot, explicit
`AskUserQuestion` go/no-go, mark `delivery-deploy-gate.passed`, verify it
before pushing, push, then explicitly dispatch `deploy-pages.yml` (the
push alone deploys nothing, by design). The app redeploys to its existing
URL - no new slug.

## 13-15. Close-out, hand-back, translate out

Identical to studio-build steps 15-17: orchestrator close-out
(`closeout.json` - note the style-spec artifact is legitimately absent on
update runs), delivery stage 2 message (`12-delivery-message.en.txt` -
this is an update, so the message says what changed, not "your new app"),
translator out (`13-message.txt`), show it with the link, set state
`completed`.
