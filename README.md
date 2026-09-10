# TruFin-io `.github`

Org-wide [community health files](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file) for the TruFin-io organisation.

Files placed under `.github/` here are inherited by every repository in the org
that does not define its own.

## Contents

- **`.github/ISSUE_TEMPLATE/vulnerability.yml`** – template for logging security
  vulnerability findings. Aligned with the
  [Vulnerability Management Policy](https://github.com/TruFin-io/docs) SLAs
  (Critical 1d / High 3d / Medium 7d / Low 30d).

- **`.github/workflows/claude-review.yml`** – reusable workflow that reviews a
  pull request with Claude. Repos call it from their own `claude-review.yml`
  (see the header comment in the file). Runs on the org
  `CLAUDE_CODE_OAUTH_TOKEN` secret, so it bills the Claude subscription.
- **`.claude/skills/code-review/SKILL.md`** – the review skill that workflow
  runs. It is copied into the reviewed repo at run time, so a repo's own
  `CLAUDE.md` files drive the compliance checks. Reviews drafts, re-reviews
  on every push, and updates its summary comment in place.
  Based on Anthropic's [`plugins/code-review/commands/code-review.md`](https://github.com/anthropics/claude-code/blob/main/plugins/code-review/commands/code-review.md),
  copied at commit [`db8834b`](https://github.com/anthropics/claude-code/blob/db8834ba1d72e9a26fba30ac85f3bc4316bb0689/plugins/code-review/commands/code-review.md).
  To check for upstream changes:

  ```bash
  diff <(gh api repos/anthropics/claude-code/contents/plugins/code-review/commands/code-review.md --jq .content | base64 -d) .claude/skills/code-review/SKILL.md
  ```

## Adding to this repo

Defaults only take effect where a repo does not already override them. If you
introduce a new organisation-wide template, audit existing repos for conflicts
first.
