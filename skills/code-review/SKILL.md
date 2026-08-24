---
name: code-review
description: >-
  Reviews diffs on three axes — Spec (CA/RF), Standards (AGENTS.md, tests,
  smells), and Quality (bugs, security, performance, correctness). WIP: local
  branch or post-implement → report, review-gaps on FAIL, handoff to
  implement/pr-ship. PR (URL/#/gh): artifact only, no
  handoff. Use for CA validation, branch review,
  bug/security/quality review, or existing PR review. Does not implement code.
---

# Code Review

## Mindset

- **Three axes, never mixed or ranked** — Spec · Standards · Quality.
- **Spec is source of truth** — cite spec lines; do not invent CA/RF.
- **Quality focuses on changed code** — diff hunks only; do not flag untouched legacy unless the change exposes or worsens it.
- **WIP:** report only; gaps file + remediation via `implement`.
- **PR:** artifact + chat report, then **stop** — author fixes; no gaps/loop/handoff.

## Context

| Context | Signal | Mode | Axes | After report |
| -------- | ------ | ---- | ---- | ------------ |
| WIP | Spec path / post-`implement` / "validate CA" | Spec | Spec + Standards + Quality | gaps + loop + handoff |
| WIP | Local branch, diff from main/SHA | Branch | Standards + Quality (+ Spec if found) | same |
| PR | URL, `#123`, "review the PR" | — | Standards + Quality (+ Spec if in PR) | **artifact only** |
| Ambiguous | "review" without PR or branch | — | — | **ask** |

## Process

**Goal:** pin diff → three axes (parallel) → report → (**WIP only**) gaps + remediation loop.

### 1. Pin

**WIP:** Fixed-point = user input (`main`, SHA, `HEAD~N`, tag); default `main`/`master`.

1. `git rev-parse <fp>` — invalid → stop and ask.
2. `git diff <fp>...HEAD` (three-dot) — empty → stop.
3. `git log <fp>..HEAD --oneline`.
4. Store `fixed_point`, SHA, diff command, commits. **Reuse same fp** on revalidation.

**PR:**

1. Resolve `#123`, URL, or `gh pr list --head <branch>`.
2. `gh pr view <n> --json number,title,url,baseRefName,headRefName,commits,body`.
3. `gh pr diff <n>` — empty → stop. Do **not** use local `git diff` unless asked.
4. Store `pr_number`, `pr_url`, `base`, `head`, diff, commits.

PR fixed-point in report: `PR #<n> (base: <base>…head: <head>)`.

### 2. Spec discovery

1. User path → 2. commits/PR body refs → 3. `docs/specs/` match by id/branch/title.
4. Nothing found → **WIP:** ask · **PR:** Spec = `"no spec available"` (Standards + Quality only).

### 3. Spawn axes (parallel)

One message, up to **three** `Task` / `generalPurpose` subagents, `readonly: true`. Include in each: diff, commits, fixed-point. **Paste checklist from [Axes](#axes)** in each prompt.

| Sub-agent | When | Extra input |
| --------- | ---- | ----------- |
| **Spec** | Spec found | Spec content/path |
| **Standards** | Always | `AGENTS.md` |
| **Quality** | Always | — |

No spec → skip Spec subagent; run Standards + Quality only.

### 4. Report

Findings under `## Spec`, `## Standards`, and `## Quality` — separate rankings. Format: [Report](#report).

**Spec / Standards severity:** `blocking` | `important` | `nit`

**Quality severity:** `Critical` | `Warning` | `Info`

**Verdict mapping (for FAIL / gaps):**

| Axis | Triggers FAIL / gap |
| ---- | ------------------- |
| Spec / Standards | `blocking` |
| Quality | `Critical` |

- Verdict: any FAIL trigger → `FAIL`; only important/nit / Warning/Info → `PASS_WITH_NITS`; zero findings → `PASS`

| Outcome | Action |
| ------- | ------ |
| **PR** (any verdict) | Save artifact → deliver in chat → **stop** |
| **WIP** PASS / PASS_WITH_NITS | Report in chat; offer `pr-ship` (+ optional `qa-test-plan`, `security-code-analysis`) |
| **WIP** FAIL | Save gaps → §5 |

### 5. Loop 2C (WIP + FAIL only)

Ask **once per round** (max **3**):

```markdown
### Remediation (round N/3)

There are N blocking/critical findings. File: <path>
Implement now with spec-implement? [yes / no]
```

| Response | Action |
| -------- | ------ |
| **no** | Stop; return gaps path + verdict |
| **yes** | Handoff `.agents/skills/implement/SKILL.md` with gaps file; spec = CA source; gaps = round scope |

After implement: revalidate (same fp) → overwrite gaps with residual → blocking/critical & round < 3 → ask again. Round 3 still blocking/critical → final verdict + residual; **no** `pr-ship`. Do not restart without explicit request.

## Axes

Paste in subagent prompts (sub-agent does not read this skill). Max ~400 words each.

### Spec

| Outcome | When | Severity |
| ------- | ---- | -------- |
| **missing** | CA/RF absent from diff | blocking |
| **partial** | Incomplete; missing edge/assertion | blocking if CA requires; else important |
| **wrong** | Diverges from spec | blocking |
| **scope creep** | Outside spec scope | important (blocking if API change without RF) |

Test without CA assertion = partial. Unsigned assumption → ask.

### Standards

**Hard violations:** `AGENTS.md`, Unit Test Standards, spec conventions → blocking when mandatory.

**Smells** (Fowler ch.3; repo override if documented): name smell + cite hunk.

Mysterious Name · Duplicated Code · Feature Envy · Data Clumps · Primitive Obsession · Repeated Switches · Shotgun Surgery · Divergent Change · Speculative Generality · Message Chains · Middle Man · Refused Bequest

**Lenses:** Architecture (layer, repo atomicity, coupling) · test quality (useless tests, missing cases covered by CA).

Debatable preference → important/nit. No style without `AGENTS.md` or smell basis.

### Quality

Find bugs, security issues, and quality risks in **changed code**. Group every finding by severity and tag with category `[Security|Performance|Correctness|Maintainability]`.

| Severity | Criterion |
| -------- | --------- |
| **Critical** | Security vulnerabilities, data loss risks, crashes |
| **Warning** | Bugs, performance issues, anti-patterns |
| **Info** | Suggestions, minor improvements |

**Review dimensions** — scan changed hunks against each row; tag OWASP id on Security when clear (e.g. A01, A03).

| Dimension | Check |
| --------- | ----- |
| **Security** | SQL injection · XSS · CSRF · authentication/authorization flaws · secrets/credentials in code · insecure deserialization · path traversal · SSRF |
| **Performance** | N+1 queries · unnecessary memory allocations · algorithmic complexity (O(n²) in hot paths) · missing DB indexes (when query added/changed) · unbounded queries/loops · resource leaks |
| **Correctness** | edge cases (empty input, null, overflow) · race conditions/concurrency · error handling/propagation · off-by-one · type safety |
| **Maintainability** | naming clarity · single responsibility · duplication **introduced by this change** · test coverage for new/changed behavior · docs for non-obvious logic |

**Axis boundaries** — Fowler smells, `AGENTS.md`, mandatory test patterns, CA-covered cases → **Standards** or **Spec** (not Quality). Quality Maintainability = gaps visible in the diff without re-auditing the whole module.

**Actionable output:** `Critical` and `Warning` findings **must** include a short suggested fix as a fenced code block (minimal diff or pseudocode). `Info` — suggestion encouraged.

## Report

```markdown
# Code Review

**Context:** WIP | PR · **Mode:** Spec | Branch · **Fixed-point:** `<fp>` (`<sha>`) · **Commits:** N · **Round:** N/3 _(WIP)_
**PR:** `#<n>` — `<title>` — `<url>` _(PR)_
**Artifact:** `<path>` _(when saved)_

## Spec

- **blocking** — CA03 missing: … (`…/file`, spec L120)

_(or: no spec available)_

## Standards

- **blocking** — unit test uses a concrete repository implementation instead of the interface/abstraction — AGENTS Unit Test Standards
- **nit** — Mysterious Name: `data2` in `…/bar`

## Quality

### Critical

- **[Security] A03** — SQL concatenation with user input (`…/repo` L42)
  ```text
  // suggested: use a parameterized query / prepared statement
  await db.query('SELECT … WHERE id = $1', [id]);
  ```

### Warning

- **[Performance]** — N+1: loop loads related entity per row (`…/service` L88)

### Info

- **[Maintainability]** — rename `data2` → `orderPayload` for clarity (`…/bar`)

## Verdict

`PASS` | `PASS_WITH_NITS` | `FAIL`

- Spec: X (B/I/N) · worst: …
- Standards: Y (B/I/N) · worst: …
- Quality: Z (C/W/I) · worst: …
```

Each bullet: severity + what + where (one anchor). `Critical`/`Warning` include fix snippet. No paragraphs.

| Severity (Spec/Standards) | Criterion |
| --------------------------- | --------- |
| blocking | Missing/wrong CA/RF; mandatory pattern violation |
| important | Should fix; non-blocking partial |
| nit | Optional |

| Severity (Quality) | Criterion |
| ------------------ | --------- |
| Critical | Security vuln, data loss, crash — maps to FAIL |
| Warning | Bug, perf issue, anti-pattern |
| Info | Minor suggestion, maintainability nit |

## Artifact paths

Subfolder `review-gaps/` next to source doc. Create if missing.

| Context | When | File |
| -------- | ------ | ---- |
| WIP | FAIL | `<id> - review-gaps.md` |
| PR | Always | `pr-<n> - code-review.md` |

**Resolution (first match):** `docs/specs/<file>` → `docs/specs/review-gaps/` · `docs/adr/<file>` → `docs/adr/review-gaps/` · PR only → `docs/review-gaps/pr-<n> - code-review.md` · WIP no doc → ask (default `docs/review-gaps/<id-or-branch> - review-gaps.md`).

## review-gaps.md template

```markdown
# Review Gaps — <id> <short title>

| Field | Value |
| ----- | ----- |
| Spec / ADR | `../<source-file>.md` |
| Fixed-point | `<fp>` (`<sha>`) |
| Branch | `<branch>` |
| Round | N/3 |
| Verdict | FAIL |
| Date | YYYY-MM-DD |

## Gaps

| id | axis | severity | CA/RF | evidence | expected outcome |
| -- | ---- | -------- | ----- | -------- | ---------------- |
| G1 | Spec | blocking | CA03 | `src/…/foo` | … |
| G2 | Quality | Critical | — | `src/…/repo` L42 | parameterized query; no injection |

## Execution Plan (remediation)

1. [Step] → files: […] → CA: CA0X → blocked-by: none → verify: [cmd] → commit: <type>(scope): …

Only blocking / Critical (and human-requested important/Warning). Stable ids G1, G2… for residual. Overwrite each round.

## Assumptions / questions

- … (spec doubt → ask; do not invent)

## Residual (after revalidation)

| id | status | notes |
| -- | ------ | ----- |
| G1 | fixed / open | … |
```

## Prohibited

- Implement fixes · post `gh pr review` (unless asked) · external scripts · mutation testing
- **PR:** gaps, loop, or handoff · mix axis rankings · `pr-ship` with residual blocking/critical after 3 rounds (WIP)