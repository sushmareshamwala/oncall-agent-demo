# On-call agent demo · let Claude watch your site while you're on leave 🪔

A single GitHub Actions workflow ([`examples/oncall.yml`](examples/oncall.yml)) that:

1. **checks your site every 15 minutes** (`curl`, with retries)
2. if it's down, **opens one GitHub issue** labelled `incident` (no duplicates while one is open)
3. **runs the Claude Code GitHub Action** on a new branch: it reads the recent commits and code, makes the smallest fix, **opens a pull request**, and **comments on the issue** with the cause and evidence
4. **you** get the GitHub notification on your phone, read the fix, and decide. **Claude never merges.**

Made for the "Diwali leave" Reels on [@sushma_reshamwala](https://instagram.com/sushma_reshamwala). The YAML is checked against the official docs; run the test below in your own repo before you rely on it.

## Setup (about 10 minutes)
1. **Copy** [`examples/oncall.yml`](examples/oncall.yml) into **your** repo as `.github/workflows/oncall.yml`. (It sits in `examples/` here so it never runs in this demo repo.)
2. **Install the Claude GitHub App** on that repo: https://github.com/apps/claude (or run `/install-github-app` inside Claude Code, which also adds the secret).
3. **Add the secret** `ANTHROPIC_API_KEY` (Settings → Secrets and variables → Actions). Using a Claude subscription instead? Run `claude setup-token`, save it as `CLAUDE_CODE_OAUTH_TOKEN`, and change the action's `anthropic_api_key:` line to `claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}`.
4. **Add the variable** `SITE_URL` (same page, Variables tab), e.g. `https://your-app.com/health`.
5. **Make "never merges" a hard rule:** Settings → Rules → Rulesets → new ruleset on your default branch → *Require a pull request before merging* with **1 approval**. Claude can't approve pull requests, so only you can merge.
6. **Phone alerts:** turn on notifications in the GitHub mobile app for this repo.

## Test it
Set `SITE_URL` to a page that doesn't exist (e.g. `https://your-app.com/this-should-404`), then **Actions → oncall → Run workflow**. You should get an `incident` issue and, a few minutes later, Claude's comment (and a PR if it finds a code cause; for a missing page it should explain there's no code fix). Set `SITE_URL` back afterwards and close the issue.

## How it stays safe
- Claude may only use the tools in `--allowedTools`: read/edit files, `git log/show/diff/status/add/commit`, push **only** with `git push -u origin HEAD` (the branch the workflow made), `gh pr create` and `gh issue comment`.
- `--disallowedTools` blocks `gh pr merge`, any edit under `.github/`, and reading `.env*` files. Deny rules win over allow rules.
- `--max-turns 15` and `timeout-minutes: 20` cap each run.
- The site's response body is never passed to Claude, so a hacked page can't inject instructions.
- The workflow runs with `permissions: {}` by default; the agent job only gets `issues: write` (to open the issue) and `id-token: write` (for the Claude GitHub App login).

## Good to know
- **Costs:** each run uses GitHub Actions minutes and Claude API tokens (or your Claude subscription with an OAuth token). It only runs Claude when the site is down.
- **GitHub schedules:** the shortest interval is 5 minutes, runs can start late at busy times, scheduled runs use the default branch, and in a **public** repo the schedule turns off after **60 days** with no activity: push a commit before a long trip.
- Just want the alert, no AI? Use [`examples/uptime-only.yml`](examples/uptime-only.yml).

## Docs
- Claude Code GitHub Actions: https://code.claude.com/docs/en/github-actions
- claude-code-action: https://github.com/anthropics/claude-code-action
- GitHub schedule trigger: https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule

MIT licence.
