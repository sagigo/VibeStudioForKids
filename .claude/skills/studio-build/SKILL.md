---
name: studio-build
description: Vibe Studio for Kids - Phase 1 thin end-to-end slice. Runs the full stub pipeline (Translator -> Intake -> Tech Spec -> Task Planner -> Developer -> QA -> Reviewer -> Delivery) for a single hardcoded toy request, pausing at the Delivery gate for the end user's explicit go-ahead before anything is deployed publicly to GitHub Pages.
---

# Studio build (Phase 1 thin slice)

This is the studio's entry point. Phase 1's job is only to prove the pipe's
mechanics work end to end - every role below is a real, standalone agent
(see `.claude/agents/`), but most of them are intentionally simple stubs.
The one thing that must be real is the deployment at the end: a working
shareable GitHub Pages link, only produced after the end user explicitly
says go.

Do not ask the end user what app to build - Phase 1 has no real Intake yet,
so the request is fixed. Follow these steps in order.

## 0. Set up the run

- Generate a run id: `date +%Y%m%d-%H%M%S`.
- Create `runs/<run-id>/`.
- Write the fixed hardcoded kid request (Hebrew) to
  `runs/<run-id>/00-request.he.txt`:
  ```
  תבנה לי דף שכתוב עליו "שלום" עם כפתור אדום גדול
  ```
- Pick an app slug for this run, e.g. `hello-button`, and an app directory
  `apps/hello-button/`.

## 1. Orchestrator

Invoke the `orchestrator` agent (foreground) with the run directory path.
It writes `runs/<run-id>/plan.json` confirming the stage order and gate
placement. Read it back before continuing.

## 2. Translator (in)

Invoke the `translator` agent: input `00-request.he.txt`, direction
`he->en`, output `runs/<run-id>/01-request.en.txt`.

## 3. Intake

Invoke the `intake` agent: input `01-request.en.txt`, output
`runs/<run-id>/02-requirement.md`.

## 4. Tech Spec

Invoke the `tech-spec` agent: input `02-requirement.md`, output
`runs/<run-id>/03-tech-spec.md`.

## 5. Task Planner

Invoke the `task-planner` agent: input `03-tech-spec.md`, output
`runs/<run-id>/04-tasks.md`.

## 6. Developer

Invoke the `developer` agent: inputs `03-tech-spec.md` and `04-tasks.md`,
target app directory `apps/hello-button/`, and have it write
`runs/<run-id>/05-dev-notes.md`.

## 7. QA

Invoke the `qa` agent: inputs `04-tasks.md` and the app directory, output
`runs/<run-id>/06-qa-report.md`. If it fails, stop and report to the user
rather than continuing (Phase 1 has no dev/QA retry loop yet - that's
Phase 6).

## 8. Reviewer

Invoke the `reviewer` agent: inputs `02-requirement.md`, `06-qa-report.md`,
and the app directory, output `runs/<run-id>/07-review.md`. If it fails,
stop and report to the user.

## 9. Delivery - stage 1 (local)

Invoke the `delivery` agent (stage 1 / local build report): inputs
`07-review.md` and the app directory, output
`runs/<run-id>/08-delivery-local.md`.

## 10. The gate - stop and ask

This is the one step that is not a stub. Summarize for the end user, in
plain terms: what was built, that QA and Review both passed, and where the
local files are. Then use `AskUserQuestion` to ask explicit go/no-go: do
they want this made public on a real shareable GitHub Pages link, yes or
no. Do not proceed past this point without an explicit yes - this is the
Delivery gate the roadmap calls out by name.

## 11. Deploy (only after explicit yes)

This part is deliberately **not** delegated to the Delivery agent - it's a
real push to the remote, so it happens here in the open, in the main
conversation, where the user can see exactly what's being pushed:

- `git add apps/hello-button runs/<run-id>` (plus any pipeline files not
  yet committed, if this is the first run).
- Commit with a clear message.
- Push to the current branch with `git push -u origin <branch>`.
- The push should trigger `.github/workflows/deploy-pages.yml`. Poll the
  Actions run (`mcp__github__actions_list` / `actions_get` /
  `get_job_logs`) until it succeeds or fails. If Pages enablement fails
  (e.g. permissions locked down at the repo/org level), report that
  clearly and fall back to telling the user the exact manual steps
  (Settings -> Pages -> set source) rather than silently giving up.
- Once live, the URL is `https://sagigo.github.io/VibeStudioForKids/hello-button/`.

## 12. Delivery - stage 2 (hand-back message)

Invoke the `delivery` agent again (stage 2 / hand-back message) with the
real live URL, output `runs/<run-id>/09-delivery-message.en.txt`.

## 13. Translator (out)

Invoke the `translator` agent: input `09-delivery-message.en.txt`,
direction `en->he`, output `runs/<run-id>/10-message.he.txt`. Show this
final Hebrew message, and the live link, to the user as the last thing you
do.
