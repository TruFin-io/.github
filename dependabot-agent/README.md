# Dependabot agent

Reusable workflow that checks Dependabot PRs, asks Claude to fix them when needed, and reports back on the PR with a comment and a label. Tracking issue: TruFin-io/research#194.

## How it works

When Dependabot opens or updates a PR, the repo's small caller workflow runs [`.github/workflows/dependabot-agent.yml`](../.github/workflows/dependabot-agent.yml) from this repo:

1. Checks out the PR head and this repo's `dependabot-agent/` tooling.
2. Runs the checks for every directory whose `package.json` or lockfile changed (`config.yml`, plus the repo override).
3. All green and a patch/minor bump: label `dep-agent:verified`, no Claude call.
4. Anything failing, or a major bump: runs `anthropics/claude-code-action` with `SKILL.md`. Claude only edits files. It cannot run git.
5. Re-runs the checks and runs the guardrail diff check (no `eslint-disable`, `@ts-ignore`, skipped or deleted tests, workflow/env files, or paths outside `allowed_paths`).
6. If green and clean: commits and pushes to the Dependabot branch with the Claude App token. The push triggers the repo's normal CI.
7. Posts or updates one comment, and sets `dep-agent:verified`, `dep-agent:fixed` or `dep-agent:needs-human`.

Claude's own push re-triggers the caller with actor `claude[bot]`, which the caller skips, so there is no loop.

## Adding it to a repo

1. Add `.github/workflows/dependabot-agent.yml`:

   ```yaml
   name: Dependabot agent

   on:
     pull_request:
       types: [opened, reopened, synchronize]

   jobs:
     dependabot-agent:
       if: github.actor == 'dependabot[bot]'
       permissions:
         contents: write
         pull-requests: write
         issues: write
         id-token: write
         actions: read
       uses: TruFin-io/.github/.github/workflows/dependabot-agent.yml@main
       secrets: inherit
   ```

2. Make sure the repo has a `.github/dependabot.yml`.
3. Make sure the Claude GitHub App is installed on the repo.

### Secrets

Workflows triggered by Dependabot only see **Dependabot secrets**, not Actions secrets. Set `CLAUDE_CODE_OAUTH_TOKEN` (or `ANTHROPIC_API_KEY`) as an **org-level Dependabot secret**: Org settings → Secrets and variables → Dependabot.

### Per-repo override (optional)

`.github/dependabot-agent.yml` in the repo:

```yaml
enabled: false          # opt out
checks:                 # replace the commands for an ecosystem
  bun:
    - { name: install, run: bun install --frozen-lockfile }
    - { name: test, run: bunx vitest --run }
allowed_paths:          # extra paths Claude may change
  - "scripts/**"
```

A repo can also add `.claude/skills/dependabot-triage/SKILL.md`, which is appended to the shared skill.

## Security

- This repo is public. Keep secrets out of it. The prompt and guardrail rules are visible to anyone.
- The checks run install scripts from the new package versions in the same job that holds the Claude token. Use a token or key you can rotate.
- The PR body goes to Claude inside a delimited untrusted block.

## Known gaps

- Claude gets one session per run. It can re-run checks itself within its turn cap.
- Checks only cover npm, pnpm, yarn and bun, with the lockfile next to the changed `package.json`.
- The comment shows local check results. Target CI results show on the PR as usual.
- After the agent pushes, Dependabot stops rebasing the PR, and `@dependabot recreate` wipes the agent commit.
