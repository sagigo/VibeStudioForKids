---
name: studio-build
description: Vibe Studio for Kids - main entry point. Asks the kid what they want to build (in Hebrew), runs a real interactive Intake conversation to clarify it, then the rest of the pipeline (Tech Spec -> Task Planner -> Developer -> QA -> Reviewer -> Delivery), pausing at the Delivery gate for the end user's explicit go-ahead before anything is deployed publicly to GitHub Pages.
---

# Studio build

This is the studio's entry point. Every role below is a real, standalone
agent (see `.claude/agents/`); Tech Spec, Task Planner, Developer, QA,
Reviewer and Delivery are still Phase 1-level stubs (deepened in later
phases), but Translator, the safety screen, and Intake are real as of
Phase 2/3. Follow these steps in order.

## 0. Set up the run

- Generate a run id: `date +%Y%m%d-%H%M%S`.
- Create `runs/<run-id>/`.

## 1. Orchestrator

Invoke the `orchestrator` agent (foreground) with the run directory path.
It writes `runs/<run-id>/plan.json` confirming the stage order. Read it
back before continuing.

## 2. Ask the kid what they want to build

Compose a short, warm English prompt (e.g. "Hi! What would you like to
build today? Tell me a bit about your idea."). Invoke the `translator`
agent (en->he) to translate it, then show the Hebrew text to the user as
a normal message and wait for their reply - this is an ordinary
conversation turn, not `AskUserQuestion` (the idea itself is open-ended,
so it can't be multiple-choice). Save their raw reply to
`runs/<run-id>/00-request.he.txt`.

## 3. Safety check, then Translator (in)

Invoke the `safety-check` agent: input `00-request.he.txt`, output
`runs/<run-id>/01-check.json`. If `flagged` is `true`, stop the pipeline
immediately, do not continue to Intake, and surface the flag (and its
reason) to the end user rather than the kid.

Only if `flagged` is `false`, invoke the `translator` agent: input
`00-request.he.txt`, direction `he->en`, output
`runs/<run-id>/02-request.en.txt`.

## 4. Interactive Intake loop

`AskUserQuestion` is not available inside subagents, so this loop is
driven by you, not by the Intake agent itself - Intake only ever decides
what to ask or when it has enough; you own every actual exchange with the
kid.

1. Initialize `runs/<run-id>/03-transcript.json`:
   ```json
   {"original_idea_en": "<contents of 02-request.en.txt>", "qa_history": []}
   ```
2. Invoke the `intake` agent: input `03-transcript.json`, output
   `runs/<run-id>/03-decision.json` (Intake's decision each round always
   goes to this one path, overwritten each time). You separately own
   `03-transcript.json` - Intake never writes to it, only reads it; you
   update it yourself in step 3 below after each answer.
3. If the decision is `"done": false`: translate the question and its
   options to Hebrew (one `translator` call covering the whole question +
   options together, so phrasing stays coherent), then call
   `AskUserQuestion` with that Hebrew question and those Hebrew options.
   - If the kid picks one of Intake's given options: translate their
     chosen option's label back to English (or just reuse the English
     label you already have for that option - no need to re-translate
     something you already know in English).
   - If the kid types a free-text "Other" answer: run the `safety-check`
     agent on it first, exactly like step 3's initial check. If flagged,
     stop the pipeline the same way. If clear, invoke `translator` (he->en)
     on it.
   - Append `{"question": "<English question>", "answer": "<English
     answer>"}` to `qa_history` in `03-transcript.json`, and go back to
     step 2.
4. If the decision is `"done": true`: write `requirement_markdown` to
   `runs/<run-id>/04-requirement.md` and exit the loop.

## 5. Tech Spec

Invoke the `tech-spec` agent: input `04-requirement.md`, output
`runs/<run-id>/05-tech-spec.md`.

## 6. Task Planner

Invoke the `task-planner` agent: input `05-tech-spec.md`, output
`runs/<run-id>/06-tasks.md`.

## 7. Pick an app slug

Derive a short, readable, lowercase-hyphenated slug (2-4 words) from the
requirement's Summary line, e.g. `pet-feeding-tracker`. Check `apps/` for
a collision and adjust if needed. Target app directory is
`apps/<slug>/`.

## 8. Developer

Invoke the `developer` agent: inputs `05-tech-spec.md` and `06-tasks.md`,
target app directory `apps/<slug>/`, and have it write
`runs/<run-id>/07-dev-notes.md`.

## 9. QA

Invoke the `qa` agent: inputs `06-tasks.md` and the app directory, output
`runs/<run-id>/08-qa-report.md`. If it fails, stop and report to the user
rather than continuing (no dev/QA retry loop yet - that's Phase 6).

## 10. Reviewer

Invoke the `reviewer` agent: inputs `04-requirement.md`, `08-qa-report.md`,
and the app directory, output `runs/<run-id>/09-review.md`. If it fails,
stop and report to the user.

## 11. Delivery - stage 1 (local)

Invoke the `delivery` agent (stage 1 / local build report): inputs
`09-review.md` and the app directory, output
`runs/<run-id>/10-delivery-local.md`.

## 12. The gate - stop and ask

This is the one step that is not a stub. Summarize for the end user, in
plain terms: what was built, that QA and Review both passed, and where the
local files are. Then use `AskUserQuestion` to ask explicit go/no-go: do
they want this made public on a real shareable GitHub Pages link, yes or
no. Do not proceed past this point without an explicit yes - this is the
Delivery gate the roadmap calls out by name.

## 13. Deploy (only after explicit yes)

This part is deliberately **not** delegated to the Delivery agent - it's a
real push to the remote, so it happens here in the open, in the main
conversation, where the user can see exactly what's being pushed:

- `git add apps/<slug> runs/<run-id>` (plus any pipeline files not yet
  committed).
- Commit with a clear message.
- Push to the current branch with `git push -u origin <branch>`.
- The push should trigger `.github/workflows/deploy-pages.yml` (it
  watches `apps/**`). Poll the Actions run (`mcp__github__actions_list` /
  `actions_get` / `get_job_logs`) until it succeeds or fails. If it fails,
  report clearly rather than silently giving up - see
  `docs/DECISIONS.md` for the one-time repo/environment setup this can
  depend on.
- Once live, the URL is
  `https://sagigo.github.io/VibeStudioForKids/<slug>/`.

## 14. Delivery - stage 2 (hand-back message)

Invoke the `delivery` agent again (stage 2 / hand-back message) with the
real live URL, output `runs/<run-id>/11-delivery-message.en.txt`.

## 15. Translator (out)

Invoke the `translator` agent: input `11-delivery-message.en.txt`,
direction `en->he`, output `runs/<run-id>/12-message.he.txt`. Show this
final Hebrew message, and the live link, to the user as the last thing you
do.
