---
name: studio-build
description: Vibe Studio for Kids - main entry point. Asks the kid what they want to build, in whatever language they use, runs a real interactive Intake conversation to clarify it, then the rest of the pipeline (Tech Spec -> Task Planner -> Developer -> QA -> Reviewer -> Delivery), pausing at the Delivery gate for the end user's explicit go-ahead before anything is deployed publicly to GitHub Pages.
---

# Studio build

This is the studio's entry point. Every role below is a real, standalone
agent (see `.claude/agents/`); Reviewer and Delivery are still Phase
1-level stubs (deepened in later phases). Translator, the safety screen,
Intake, Tech Spec, Task Planner, and Development + QA's bounded retry loop
are real. Follow these steps in order.

The studio doesn't assume the kid's language - Translator detects it from
their first message and that detected language is used for everything
kid-facing for the rest of the run, including the deployed app's own UI
text.

## 0. Set up the run

- Generate a run id: `date +%Y%m%d-%H%M%S`.
- Create `runs/<run-id>/`.

## 1. Orchestrator

Invoke the `orchestrator` agent (foreground) with the run directory path.
It writes `runs/<run-id>/plan.json` confirming the stage order. Read it
back before continuing.

## 2. Ask the kid what they want to build

The studio doesn't know the kid's language yet at this point, so greet in
Hebrew (today's primary audience) as a reasonable starting default, but
don't assume the reply will match - step 3 detects the real language from
whatever they actually send back and that's what's used from then on.
Compose a short, warm English prompt (e.g. "Hi! What would you like to
build today? Tell me a bit about your idea."), invoke the `translator`
agent (direction `en->Hebrew`) to translate it, then show that text to the
user as a normal message and wait for their reply - this is an ordinary
conversation turn, not `AskUserQuestion` (the idea itself is open-ended, so
it can't be multiple-choice). Save their raw reply to
`runs/<run-id>/00-request.txt`.

## 3. Safety check, then Translator (in) - detects the kid's language

Invoke the `safety-check` agent: input `00-request.txt`, output
`runs/<run-id>/01-check.json`. If `flagged` is `true`, stop the pipeline
immediately, do not continue to Intake, and surface the flag (and its
reason) to the end user rather than the kid.

Only if `flagged` is `false`, invoke the `translator` agent: input
`00-request.txt`, direction `detect->en`, output
`runs/<run-id>/02-request.en.txt`. Read the sidecar
`02-request.en.txt.lang.json` it writes (`language_code`, `language_name`,
`rtl`) - this is the kid's language for the rest of the run. Keep it
handy; you'll pass it to `translator` (as the `en-><lang>` target) and to
`developer` for the rest of this run.

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
   options into the kid's language (one `translator` call, direction
   `en-><kid's language>`, covering the whole question + options together
   so phrasing stays coherent), then call `AskUserQuestion` with that
   translated question and those translated options.
   - If the kid picks one of Intake's given options: reuse the English
     label you already have for that option - no need to re-translate
     something you already know in English.
   - If the kid types a free-text "Other" answer: run the `safety-check`
     agent on it first, exactly like step 3's initial check. If flagged,
     stop the pipeline the same way. If clear, invoke `translator`
     (direction `detect->en` - reuse the earlier detected language rather
     than trusting a fresh detection on a short answer, unless the kid
     clearly switched languages) on it.
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

Derive a short, readable, lowercase-hyphenated slug (2-4 words, in
English regardless of the kid's language - it's a URL path, not
kid-facing text) from the requirement's Summary line, e.g.
`pet-feeding-tracker`. Check `apps/` for a collision and adjust if needed.
Target app directory is `apps/<slug>/`.

## 8-9. Development + QA - bounded retry loop

Up to 4 total Developer attempts (1 build + up to 3 fix cycles). Track the
attempt number as you go.

1. **Attempt 1 (build mode):** invoke the `developer` agent, mode `build`:
   inputs `05-tech-spec.md` and `06-tasks.md`, target app directory
   `apps/<slug>/`, the kid's language and its `rtl` flag (from step 3's
   sidecar), and have it write `runs/<run-id>/07-dev-notes.md`. All UI
   text the kid will see must be in the kid's language, not English - this
   is Developer's job, not a separate localization pass.
2. Invoke the `qa` agent: inputs `06-tasks.md` and the app directory,
   output `runs/<run-id>/08-qa-report-<attempt>.md` (one file per attempt,
   so the history isn't overwritten - e.g. `08-qa-report-1.md`).
3. If QA's overall verdict is PASS, continue to step 10 (Reviewer), using
   this attempt's QA report as `08-qa-report.md` (copy or reference the
   winning attempt's file under that name, since later steps expect it).
4. If FAIL and attempts remain (fewer than 4 total so far): invoke
   `developer` again, mode `fix` - same inputs as attempt 1, plus this
   attempt's QA report path. Go back to step 2 for the next attempt.
5. If FAIL on the 4th attempt: stop the pipeline here. Do not continue to
   Reviewer or Delivery. Report to the end user what was tried, what's
   still failing (from the last QA report), and that it needs human
   attention - this is a real, bounded give-up, not a silent failure.

## 10. Reviewer

Invoke the `reviewer` agent: inputs `04-requirement.md`, `05-tech-spec.md`
(so it can see any `Scope adjustments` and judge against the adjusted
scope, not blindly the original ask), `08-qa-report.md`, and the app
directory, output `runs/<run-id>/09-review.md`. If it fails, stop and
report to the user.

## 11. Delivery - stage 1 (local)

Invoke the `delivery` agent (stage 1 / local build report): inputs
`09-review.md` and the app directory, output
`runs/<run-id>/10-delivery-local.md`. It actually runs the app locally and
saves a screenshot at `runs/<run-id>/preview.png` - use `SendUserFile` to
show that screenshot to the end user before the gate below, so "what was
built" isn't just a text description.

## 12. The gate - stop and ask

This is the one step that is not a stub. Show the screenshot from step 11,
and summarize for the end user in plain terms: what was built, that QA and
Review both passed, and that the local run was clean. Then use
`AskUserQuestion` to ask explicit go/no-go: do they want this made public
on a real shareable GitHub Pages link, yes or no. Do not proceed past this
point without an explicit yes - this is the Delivery gate the roadmap
calls out by name.

## 13. Deploy (only after explicit yes)

This part is deliberately **not** delegated to the Delivery agent - it's a
real push to the remote, so it happens here in the open, in the main
conversation, where the user can see exactly what's being pushed:

- `git add apps/<slug> runs/<run-id>` (plus any pipeline files not yet
  committed).
- Commit with a clear message.
- Push to the current branch with `git push -u origin <branch>`. This step
  (making it public) doesn't depend on the GitHub MCP tools at all - it's
  plain git over the already-authenticated remote, so it still works even
  if those tools are disconnected.
- The push should trigger `.github/workflows/deploy-pages.yml` (it
  watches `apps/**`). Poll the Actions run (`mcp__github__actions_list` /
  `actions_get` / `get_job_logs`) until it succeeds or fails. If it fails,
  report clearly rather than silently giving up - see
  `docs/DECISIONS.md` for the one-time repo/environment setup this can
  depend on.
- **If the GitHub MCP tools are unavailable** (disconnected/need
  re-auth): the push above still happened and the app is genuinely on its
  way to being public - don't block on this or retry the tool call in a
  loop. Say plainly that the push succeeded but you can't confirm the
  Actions run finished from here, and point the user at
  `https://github.com/sagigo/VibeStudioForKids/actions` to check
  themselves. Don't claim it's live when you haven't verified it, and
  don't claim it failed when you simply couldn't check.
- Once confirmed live, the URL is
  `https://sagigo.github.io/VibeStudioForKids/<slug>/`.

## 14. Delivery - stage 2 (hand-back message)

Invoke the `delivery` agent again (stage 2 / hand-back message) with the
real live URL if confirmed, or a note that it's pending human verification
if the Actions run couldn't be checked, output
`runs/<run-id>/11-delivery-message.en.txt`.

## 15. Translator (out)

Invoke the `translator` agent: input `11-delivery-message.en.txt`,
direction `en-><kid's language>`, output `runs/<run-id>/12-message.txt`.
Show this final message, and the live link, to the user as the last thing
you do.
