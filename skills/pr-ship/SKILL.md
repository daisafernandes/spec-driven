---
name: pr-ship
description: >-
  Prepares and publishes a Git delivery: inspects branch and working tree, handles uncommitted changes safely, creates a branch only with approval, pushes, opens a pull request using the repository template. Use when the user wants to open a PR, create a pull request, push and PR, ship a branch, or publish completed work.
---

# PR Ship

Publish completed work. Do not implement features here. Prefer a recent `code-review` before shipping, but do not block if the user chooses to proceed.

## Inspect

Collect enough Git context to know:

- current branch and upstream
- clean/dirty working tree, including staged and unstaged changes
- commits ahead of base
- diff from base to `HEAD`
- default/base branch (`main`, `master`, or remote default)
- whether remote tracking is outdated

If there are no commits ahead of base, stop and report that there is nothing to open.

If there are uncommitted changes, stop and ask what to do. Do not commit, stash, or discard without approval.

| Situation | Action |
| --------- | ------ |
| Clean working tree | proceed |
| Irrelevant local files/reports | ask whether to ignore/exclude |
| Feature changes without commit | ask: commit now / stash / abort |
| Secrets or credentials | do not stage; warn |

If the user approves a commit, use Conventional Commits:

```text
<type>(<scope>): <description>
```

Allowed types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`, `style`, `build`, `ci`, `revert`.

Use an imperative lowercase description, no trailing period, and no `--no-verify`. If commitlint/hooks reject, fix the message/workflow; do not bypass hooks.

## Branch

Create a branch only when the user approves and one is needed: current branch is protected (`main`/`master`), user requested a new branch, or current branch does not represent this delivery.

Derive prefix from spec/commits or ask:

| Type | Prefix |
| ---- | ------ |
| New feature | `feat/` |
| Bug fix | `fix/` |
| Refactor | `refactor/` |
| Docs | `docs/` |
| Tests | `test/` |
| Chore/tooling | `chore/` |
| Performance | `perf/` |
| Hotfix | `hotfix/` |

Name: `<type>/<id>-<slug>` when a task/spec id exists, otherwise `<type>/<slug>`. Do not invent task ids.

If already on a suitable branch, proceed after confirming it is the intended delivery branch.

## Push

Push the current branch and set upstream when needed. Never force-push to `main`/`master`. If push fails because of auth, non-fast-forward, or protection rules, report and stop unless the user explicitly chooses a safe next action.

## Pull Request

Read `.github/pull_request_template.md` at repo root. Fill it using spec CA/RF, recent commits, and diff. Keep the PR body in the same language and structure as the template.

PR title should align with the delivery type and repo convention, for example:

```text
feat(prices): markup by state
```

Use the repo default base unless the user or branch metadata indicates another base. Return the PR URL.

## Input Handoff

Reuse context when available:

| Source | Reuse |
| ------ | ----- |
| `implement` | spec path, task id, summary, covered CAs |
| `code-review` | PASS/PASS_WITH_NITS verdict, spec path |
| ad-hoc work | git log/diff; ask for type and task id if unclear |

If there is no recent `code-review` for the same branch, offer it before push.

## Prohibited

- Implementing code.
- Force-pushing to protected branches.
- Using `--no-verify`.
- Inventing a task id.
- Committing secrets.
- Opening a PR with no commits ahead of base.
- Writing the PR body in a different language or shape than the template.
