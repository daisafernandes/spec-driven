# Quality Gate and Handoff

Apply routing from [SKILL.md](../SKILL.md#routing): feature/spec, spike, ADR, or refine. Use the matching template and Project Profile output directory.

## Feature Spec Gate

Only finalize when all relevant checks pass:

- Every `RF##` has at least one `CA##`.
- Every `CA##` is verifiable/testable and uses `QUANDO ... ENTAO ...`.
- Traceability crosses `CA##`, `RF##`/`RN##`, and test expectations.
- API contracts are confirmed in code or marked `(proposto)` / `(a confirmar)`.
- Phase 5 gray areas are answered or marked `N/A porque [motivo]`.
- **Fora de escopo** lists explicit non-goals or `N/A` with reason.
- Cross-repo dependencies are documented when applicable.
- **Premissas e Duvidas em Aberto** has no unresolved doubts.
- Backend specs identify affected layers/use cases/repositories with real paths.
- Frontend specs identify screens/components/states/API calls.
- File is saved in the Project Profile specs directory and body is pt-BR.
- Large only: atomic breakdown exists before **Refined**.

## Spike Gate

Only finalize when:

- objective and questions are clear
- findings cite real paths or are marked `(proposto)` / `(a confirmar)`
- risks and recommendations are documented
- RF/CA are included only for verifiable deliverables; otherwise deferred to a future delivery spec
- assumptions/doubts section has no unresolved doubts
- hard decisions route to ADR instead of bloating the spike
- file is saved in the specs directory and body is pt-BR

## ADR Gate

Only finalize when:

- the 3-axis ADR criterion from SKILL.md was confirmed
- Context and Decision are clear
- Alternatives and Consequences exist when there is a real trade-off
- feature RF/CA are not forced into the ADR
- cited paths/code are grounded or marked `(a confirmar)`
- file is saved in the ADR directory and body is pt-BR

If any gate fails, return to investigation or the question loop.

## Document Generation

1. Choose the template from SKILL routing.
2. For feature specs, include only applicable optional modules.
3. For spikes, use only [TEMPLATE_SPIKE.md](../templates/TEMPLATE_SPIKE.md).
4. Save to the Project Profile directory.
5. Name the file per SKILL filename rules.
6. First write uses **Status: Draft**.
7. Refine mode edits the existing file and bumps revision history.

## Status: Draft To Refined

After the relevant gate passes:

1. Summarize what was delivered.
2. Large: ensure breakdown is already present.
3. Ask: `"Posso marcar o Status como **Refined**?"`
4. Set **Refined** only after explicit user confirmation.
5. If the user wants changes, keep draft status and return to clarification.

Spike specs may be **Refined** when findings are complete even without RF/CA for a future delivery.

## Breakdown

| Scope | Requirement |
| ----- | ----------- |
| Large | mandatory before Refined |
| Spike | recommended when findings imply multi-step delivery |
| Medium | optional; include when more than about 6 steps or non-trivial dependencies |
| Small | optional; usually skip |

Each task must include:

- id and short title
- `CA##`/RF satisfied
- suggested files
- verify command
- dependencies: `blocked-by` / `blocks`

Use inline section 9 or `docs/specs/<id> - tasks.md` when long or dependency-heavy.

## Handoff Offers

After generation and mandatory Large breakdown, offer relevant next steps:

1. `implement` to execute the spec.
2. `qa-test-plan` to append/replace `## 10. Plano de Teste (Gherkin)` in the same spec by default.
3. `docs-writer` if user or feature requires docs.
4. `security-code-analysis` if security-sensitive scope was flagged.

## Handoff Payload

Pass at minimum:

```yaml
spec_path: docs/specs/<id> - <nome>.md
card_id: <task id> | null
status: Refined
scope: Small | Medium | Large | Spike
project_type: backend | frontend | CLI | lib | other

test_plan_destination: inline_section_10

breakdown:
  - id: T1
    title: <short step>
    satisfies: [CA01]
    files: [path/to/file.ts]
    verify: <cmd>
    blocked-by: []
    blocks: [T2]
```
