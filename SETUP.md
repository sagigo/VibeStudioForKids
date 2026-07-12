# Setting up the studio

Two ways to run Vibe Studio for Kids. Option 1 needs no installation at
all; Option 2 puts it on the kid's own PC.

## Option 1 — Claude Code on the web (zero install)

1. On any browser, go to **claude.ai/code** and sign in (parent's Claude
   account — Pro or Max subscription).
2. Connect the session to the `sagigo/VibeStudioForKids` GitHub repo.
3. Type `/studio-build` and hand the keyboard to the kid.

This is the environment the studio was built and tested in - everything
(browser testing, deployment) works out of the box.

## Option 2 — local install (Windows, macOS, or Linux)

### One-time setup (parent, ~20 minutes)

1. **Git** — on Windows install [Git for Windows](https://git-scm.com/download/win)
   (Claude Code runs its shell commands through Git Bash, so this is
   required, not optional). macOS/Linux usually have git already.
2. **Node.js** — install the LTS version from nodejs.org.
3. **Python** — install from python.org (used to serve apps locally
   during testing). On Windows the command is `python`; on macOS/Linux
   it's `python3` — the studio's instructions handle both.
4. **GitHub CLI** — install from cli.github.com, then `gh auth login`
   with an account that has push access to this repo (this is what
   deploys apps after you approve them).
5. **Claude Code** — `npm install -g @anthropic-ai/claude-code`, then run
   `claude` once anywhere and log in to your Claude account.
6. **Clone the studio:**
   ```
   git clone https://github.com/sagigo/VibeStudioForKids
   cd VibeStudioForKids
   ```
7. **Playwright browsers** (used by QA to actually test the apps):
   ```
   npm install playwright
   npx playwright install chromium
   ```

### Every time the kid wants to build

Open a terminal in the `VibeStudioForKids` folder, run `claude`, and type:

- `/studio-build` — build a brand-new app
- `/studio-update` — change an app that already exists

The kid chats in Hebrew (or any language - the studio detects it), answers
a few multiple-choice questions, sees a screenshot of the finished app,
and nothing goes public until an explicit "yes" at the final question.

### Notes for parents

- **Windows specifics:** Claude Code uses Git Bash under the hood, so the
  Unix-style commands in the studio's playbooks work as-is. If something
  Python-related fails, check `python --version` works in Git Bash.
- **Permissions:** the repo ships a `.claude/settings.json` that
  pre-approves the studio's routine safe operations (running the local
  test server, git commits, triggering the deploy workflow) so the kid
  isn't interrupted by technical English prompts mid-run. Anything
  outside that list still asks.
- **Publishing is double-gated:** a git push alone never publishes
  anything; deployment only happens via an explicit workflow trigger
  after the go/no-go question. Still, the kid is pushing to your GitHub
  repo under your credentials - supervision at the final gate is
  recommended.
- **Cost:** each run makes many model calls (the Developer and QA roles
  use the most). A Claude Pro plan works; Max is more comfortable for
  frequent use.
