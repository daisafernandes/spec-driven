---
name: implement
description: >-
  Implements a refined technical spec surgically: discovers project conventions, builds or reuses an execution plan, asks once about branch/commits, works step by step with TDD anchored in acceptance criteria, runs gates before commits, preserves scope, and offers delivery handoff. Use when the user wants to implement a spec, execute tasks from a spec, remediate review gaps, or turn accepted requirements into code.
---

# Implement

Turn a **refined spec** or review gaps file into code, one independently verifiable step at a time.

## Core Rules

- Read `AGENTS.md` first and build a runtime **Project Profile**: stack, structure, build/test/lint commands, test patterns, spec directory, commit rules, and repo conventions.
- Read the full spec and cited ADRs/gaps before coding.
- Stop and ask if the spec status is not **Refined** or equivalent, or if open questions remain. Suggest `spec` when requirements are incomplete.
- The spec is source of truth. Do not invent FR, CA, contracts, behaviors, or expected outcomes.
- Uncertainty never becomes code. Ask or record a human-approved assumption.
- Keep changes surgical: only files needed for the current step; no "while I'm here" work.
- Verification decides success. Do not advance or commit without passing the relevant gate.
- Do not push or open PR here; offer `pr-ship` after implementation.

## Workflow

1. **Discover profile:** read `AGENTS.md` and linked project guidance.
2. **Read inputs:** spec, ADRs, review gaps, existing task breakdown if present.
3. **Build/reuse Execution Plan:** one atomic, verifiable, committable step per delivery slice.
4. **Setup:** ask once before coding about branch creation and atomic commits.
5. **Execute sequentially by default:** offer parallel only for independent, non-overlapping steps and wait for approval.
6. **Per step:** pre-plan -> tests from CA -> implementation -> gate -> commit if enabled -> progress update.
7. **Final gates:** build/test/lint/typecheck for touched apps/modules.
8. **Adequacy check:** confirm CA coverage, no shallow tests, no scope creep, no silent spec divergence.
9. **Handoff:** offer `code-review`, `pr-ship`, `test`, `docs-writer`, and `security-code-analysis` when relevant.

## Execution Plan

Reuse a task breakdown from the spec when it has dependencies and verify commands. Otherwise derive one from FR/CA/edge cases/architecture/impacted files.

```markdown
## Execution Plan

1. [Step] -> files: [...] -> CA/RF: CA0X -> blocked-by: none -> verify: [cmd] -> commit: <type>(scope): ...
2. [Step] -> files: [...] -> CA/RF: CA0Y -> blocked-by: 1 -> verify: [cmd] -> commit: <type>(scope): ...
```

Each step must be one deliverable, independently verifiable, independently committable, and anchored to at least one `CA##`/RF or explicit `N/A` for pure wiring/config.

Persist `tasks.md` next to the spec only when the plan is long or dependency-heavy. Announce the plan before coding. If the user asks for one step, do only that step and its required dependencies.

## Setup

After the plan and before coding, ask once:

```markdown
### Setup

1. **Branch** - create `<type>/<id-or-slug>` from base? [yes / no - stay on current branch]
2. **Commits** - atomic commits after each passing step gate? [yes (default) / no - accumulate and report diff]
```

Branch creation requires explicit approval. Derive branch prefix from delivery type: `feat/`, `fix/`, `refactor/`, `docs/`, `test/`, `chore/`, `perf/`, or `hotfix/`.

Commits default to yes after Setup unless the user opts out. Do not ask again each step.

## Per-Step Cycle

### 1. Pre-Plan

Declare before editing:

```markdown
### Pre-Implementation

- Assumptions: [...]
- Files to touch: [...]
- Success criteria: [CA/RF + gate command]
```

### 2. Tests

Write or update tests before implementation when the step has testable behavior.

- Assertions must target the outcome defined by the spec, not the implementation.
- Every new test maps to `CA##`, RF, edge case, or the step's "Done when".
- Do not weaken assertions, skip/delete tests, or change a "wrong" test without asking.
- If the spec lacks a precise expected outcome, stop with a spec-precision gap and ask.

If the step is not meaningfully unit-testable (wiring/config only), use build/typecheck/lint as the gate.

### 3. Implement

Before writing code, read the real flow touched by the step. Use the lowest rung that satisfies the spec:

1. already covered by current behavior -> do no work
2. existing project pattern/helper -> reuse it
3. standard library/runtime/platform feature -> use it
4. installed dependency -> use it
5. small local change at the point of truth
6. only then add the minimum new structure needed

Do not add a dependency, abstraction, factory, wrapper, or broad refactor without FR/ADR or explicit approval. For deliberate shortcuts with a known ceiling, leave a short code comment with the ceiling and upgrade path.

### 4. Gate

Run the step's verify command. Exit code not zero means fix and rerun. Confirm tests did not silently disappear or get skipped.

### 5. Commit

When commits are enabled: one passing step = one commit.

Use Conventional Commits:

```text
<type>(<scope>): <description>
```

Rules: imperative lowercase description, no trailing period, concise header, only step files, no `--no-verify`, no force push. If a hook rejects the commit, fix forward with an allowed commit workflow; do not bypass hooks.

### 6. Scope Guardrail

| Finding | Action |
| ------- | ------ |
| Bug blocking this CA/RF | fix if needed for the step |
| Bug not blocking this step | report as deferred |
| Improvement/refactor outside step | defer |
| Behavior not specified | ask; do not invent |

## Gates

Per-step gate comes from the Execution Plan and Project Profile:

- with tests: related/unit test command for the touched behavior
- without tests: typecheck, build, lint, or module/app gate
- parallel wave: each worker runs its step gate; orchestrator runs final gates

Final gates, run by the orchestrator after all steps:

1. build touched apps/modules
2. related tests for all modified feature sources
3. lint/typecheck when defined by Project Profile

If a final gate fails, open fix steps before declaring done.

Automatic failure patterns: committing before gates pass, skipping/deleting tests, weakening assertions, claiming CA coverage without business-value assertions, running only a convenient partial gate.

## Optional Parallel

Default is sequential. Offer parallel only when all are true:

- at least two steps are independent in the same dependency wave
- no file overlap
- the user explicitly approves

Workers receive only the relevant spec excerpt, step definition, Project Profile gates, and hard constraints. Prefer worker patches; the orchestrator validates, integrates, runs gates, and commits. Pause the wave if any worker reports a blocker or spec-precision gap.

## Adequacy Check

Before final handoff, confirm:

- each in-scope `CA##`/RF has at least one business-value assertion or explicit non-testable rationale
- no shallow tests (`should be defined`, tautologies, call counts without outcome)
- project standards and test patterns were followed
- final gates passed
- no scope creep was committed
- any spec divergence was explicitly approved or recorded

## Handoff

Offer as appropriate:

- `code-review` before `pr-ship`
- `pr-ship` to push/open PR
- `test` for Gherkin in the spec's test-plan section
- `docs-writer` when docs need updating
- `security-code-analysis` for auth, external input, sensitive data, or trust-boundary changes
- spec status update to **Implemented** only with user confirmation

## Prohibited

- Parallel work without explicit approval.
- Branch creation without approval.
- Inventing FR/CA/contracts/outcomes.
- Weakening, skipping, or deleting tests to pass.
- Committing without a passing gate.
- Bypassing hooks with `--no-verify`.
- Scope creep.
- Push/open PR here; use `pr-ship`.
