---
name: code-review
description: >
  Reviews PRs, branches, WIP diffs, and CA/spec validations on three separate axes: Spec (CA/RF), Standards (AGENTS.md and repo conventions), and Quality (bugs, security, performance, correctness, maintainability). Use when the user asks for a code review, PR review, branch review, validation against a spec/CA, or bug/security/quality review of changed code.
---

# Code Review

Review changed code only. Report findings; do not implement fixes unless the user explicitly asks.

## Core Rules

- Keep **Spec**, **Standards**, and **Quality** separate. Do not merge severities across axes.
- Every finding needs concrete evidence from the diff or referenced spec/standard. If you cannot cite it, do not report it.
- Review changed hunks only. Flag untouched legacy only when the change exposes or worsens the risk.
- Spec is source of truth. Cite CA/RF/spec lines; do not invent missing requirements.
- PR mode stops after the report and artifact. No remediation loop, no handoff, no `gh pr review` unless asked.
- WIP mode may write a gaps file only for blocking/Critical findings, then offer remediation.

## Context Detection

| Context | Signal | Axes | After report |
| ------- | ------ | ---- | ------------ |
| WIP Spec | Spec path, post-`implement`, "validate CA" | Spec + Standards + Quality | gaps + optional remediation |
| WIP Branch | Local branch or diff from base/SHA | Standards + Quality (+ Spec if found) | gaps + optional remediation |
| PR | URL, `#123`, "review the PR" | Standards + Quality (+ Spec if found) | artifact + stop |
| Ambiguous | "review" without PR, branch, spec, or base | ask one clarifying question | - |

## Workflow

### 1. Pin Scope

Establish a stable review boundary before reviewing.

**WIP:** use the user-provided base when present; otherwise infer the default branch (`main`, `master`, or upstream default). Capture:

- fixed-point ref and resolved SHA
- diff from fixed-point to `HEAD`
- commits included in the review

Use a three-dot diff for branch work against its merge base. Reuse the same fixed-point on revalidation.

**PR:** resolve the PR number/URL/current branch using GitHub CLI, provider context, or available metadata. Capture:

- PR number, URL, base, head
- PR diff from the hosting provider
- commits included in the PR

For PR reviews, prefer provider/CLI PR diff over local git diff unless the user asks otherwise.

If the diff is empty, stop and report that there is nothing to review.

### 2. Discover Inputs

- **Spec:** prefer user path; otherwise scan commit messages, PR body, branch/title, then `docs/specs/` by id/title. If none exists: WIP asks whether to continue without Spec; PR reports `Spec: no spec available`.
- **Standards:** read repo guidance (`AGENTS.md`, README pointers, documented test conventions) when available. If none exists, review Standards only for explicit repo-local patterns visible in the changed area.
- **Quality:** use the diff plus minimal surrounding code needed to validate whether each suspected issue is real.

### 3. Review Axes

Use subagents in parallel when available and useful; otherwise review the axes sequentially. Subagents must be read-only and receive the same pinned diff, commits, fixed-point/PR metadata, and the relevant checklist below.

#### Spec

Report only mismatches against explicit CA/RF/spec text.

| Finding | Severity |
| ------- | -------- |
| missing CA/RF behavior | blocking |
| wrong behavior versus spec | blocking |
| incomplete required edge/assertion | blocking |
| incomplete non-required detail | important |
| scope creep outside spec | important, or blocking if API/contract changed without RF |

Test without a CA/outcome assertion counts as partial. When the spec is ambiguous, ask or record an assumption; do not invent expected behavior.

#### Standards

Report mandatory pattern violations and repo-specific conventions.

- `blocking`: mandatory `AGENTS.md` / project standard / required test convention violation.
- `important`: should fix, but not mandatory or not delivery-blocking.
- `nit`: small cleanup with clear repo/style basis.

Use Fowler smell names only when they explain a changed-code problem: Mysterious Name, Duplicated Code, Feature Envy, Data Clumps, Primitive Obsession, Repeated Switches, Shotgun Surgery, Divergent Change, Speculative Generality, Message Chains, Middle Man, Refused Bequest.

No preference-only style comments without a repo standard or smell basis.

#### Quality

Find concrete bugs and risks introduced or exposed by the change.

| Severity | When |
| -------- | ---- |
| Critical | security vulnerability, data loss, crash, corrupt state |
| Warning | bug, performance problem, risky anti-pattern |
| Info | minor maintainability suggestion |

Categories: `[Security]`, `[Performance]`, `[Correctness]`, `[Maintainability]`.

Scan for:

- **Security:** SQL/command injection, XSS, CSRF, auth/authz flaws, secrets, insecure deserialization, path traversal, SSRF.
- **Performance:** N+1 queries, unbounded loops/queries, resource leaks, avoidable hot-path allocations, missing DB index when query shape changes.
- **Correctness:** null/empty/overflow edge cases, races, error handling, off-by-one, type/contract mismatch.
- **Maintainability:** unclear names, needless duplication introduced by the diff, single-responsibility breaks, missing tests for changed behavior.

Critical and Warning findings must include a concise suggested fix or verification step. Info fixes are optional.

## Verdict

| Verdict | Criteria |
| ------- | -------- |
| `FAIL` | any Spec/Standards `blocking` or Quality `Critical` |
| `PASS_WITH_NITS` | only `important`/`nit` or `Warning`/`Info` findings |
| `PASS` | no findings |

## Report Format

```markdown
# Code Review

**Context:** WIP | PR
**Fixed-point:** `<fp>` (`<sha>`) or `PR #<n> (base: <base>, head: <head>)`
**Commits:** N
**Artifact:** `<path>` _(if saved)_

## Spec

- **blocking** - CA03 missing required empty-state behavior (`src/.../foo.ts`; spec L120)

_(or: no spec available)_

## Standards

- **important** - Mysterious Name: `data2` obscures the changed payment payload (`src/.../bar.ts` L44)

## Quality

### Critical

- **[Security] A03** - SQL query concatenates user input (`src/.../repo.ts` L42)
  ```text
  Use a parameterized query / prepared statement for the user-controlled value.
  ```

### Warning

- **[Performance]** - new loop loads related rows one-by-one (`src/.../service.ts` L88)

### Info

- **[Maintainability]** - rename `data2` to a domain name such as `orderPayload` (`src/.../bar.ts` L44)

## Verdict

`PASS` | `PASS_WITH_NITS` | `FAIL`

- Spec: X findings; worst: blocking | important | nit | none
- Standards: X findings; worst: blocking | important | nit | none
- Quality: X findings; worst: Critical | Warning | Info | none
```

Keep findings compact. Each bullet must include severity, what, where, and why it matters. Prefer `path:line` anchors when available.

## Artifacts

Create review artifacts only when required by context.

| Context | When | File |
| ------- | ---- | ---- |
| WIP | `FAIL` | `<id> - review-gaps.md` |
| PR | always | `pr-<n> - code-review.md` |

Path resolution:

- Source spec in `docs/specs/<file>` -> `docs/specs/review-gaps/`
- Source ADR in `docs/adr/<file>` -> `docs/adr/review-gaps/`
- PR without source doc -> `docs/review-gaps/pr-<n> - code-review.md`
- WIP without source doc -> ask; default `docs/review-gaps/<id-or-branch> - review-gaps.md`

### WIP Gaps Template

```markdown
# Review Gaps - <id> <short title>

| Field | Value |
| ----- | ----- |
| Source | `../<source-file>.md` |
| Fixed-point | `<fp>` (`<sha>`) |
| Branch | `<branch>` |
| Round | N/3 |
| Verdict | FAIL |
| Date | YYYY-MM-DD |

## Gaps

| id | axis | severity | CA/RF | evidence | expected outcome |
| -- | ---- | -------- | ----- | -------- | ---------------- |
| G1 | Spec | blocking | CA03 | `src/.../foo.ts` | implement required empty-state behavior |
| G2 | Quality | Critical | - | `src/.../repo.ts` L42 | parameterized query; no injection path |

## Remediation Plan

1. [Step] -> files: [...] -> CA/RF: CA0X -> verify: [cmd]

## Residual After Revalidation

| id | status | notes |
| -- | ------ | ----- |
| G1 | fixed / open | ... |
```

Only include blocking/Critical gaps unless the user asks to include lower severities. Keep gap ids stable across revalidation rounds.

## WIP Remediation Loop

For WIP `FAIL`, ask once per round, max 3 rounds:

```markdown
### Remediation (round N/3)

There are N blocking/critical findings. File: <path>
Implement now with `implement`? [yes / no]
```

If yes, hand off to `.agents/skills/implement/SKILL.md` with the gaps file, source spec if any, fixed-point, and round scope. After implementation, revalidate using the same fixed-point and overwrite the gaps file with residual findings.

After round 3, stop with the final verdict. Do not offer `pr-ship` while blocking/Critical findings remain.

## Prohibited

- Implement fixes unless explicitly asked.
- Post `gh pr review` unless explicitly asked.
- Run mutation testing or external audit scripts.
- Report speculative findings without diff/spec evidence.
- Mix axis rankings into one severity scale.
- For PR: create gaps, start remediation loops, or hand off to implementation.
