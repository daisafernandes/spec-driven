---
name: spec
description: >-
  Creates or refines technical specs, spikes, and ADRs through code investigation, project convention discovery, structured clarification, acceptance criteria, architecture/API/component planning, and implementation handoff. Use when the user wants to create a specification, document or refine a feature/task, validate requirements, plan implementation, produce an ADR, or turn an idea/card into an implementable spec.
---

# Spec

Create project-grounded specs. Investigate the real code and conventions before writing. Generated specs, ADRs, spikes, and user-facing confirmation prompts must be in **pt-BR**; skill instructions and internal reasoning stay in English. Keep technical terms in English when the team has no stable pt-BR term.

## Core Rules

- Read `AGENTS.md` first and build a runtime **Project Profile**: stack, layers, spec/ADR directories, naming convention, test patterns, error contracts, and linked standards.
- If `AGENTS.md` is missing, ask where conventions live or offer to create a minimal one.
- Never use defaults from another repo.
- Never invent FR, CA, contracts, IDs, behaviors, or decisions. Open questions must be answered by the human or recorded as explicit signed assumptions.
- Use the project template and include only relevant modules. Do not create project-type-specific templates.
- Default document status is **Draft**. Set **Refined** only after the closure gate passes and the user explicitly confirms.
- Do not set **Implemented** while writing the first draft; humans or `implement` do that after delivery.

## Routing

| Request | Artifact | Template |
| ------- | -------- | -------- |
| Feature, bugfix, refactor with delivery | Spec in Project Profile specs directory | [templates/TEMPLATE.md](templates/TEMPLATE.md) |
| Spike / discovery without a hard decision | Spike in specs directory | [templates/TEMPLATE_SPIKE.md](templates/TEMPLATE_SPIKE.md) |
| Hard-to-reverse architecture decision | ADR in ADR directory | [templates/TEMPLATE_ADR.md](templates/TEMPLATE_ADR.md) |
| Existing spec path or refine/update request | Update existing artifact | [references/refine.md](references/refine.md) |

Create an ADR only when all three are true:

- **Hard to reverse:** changing later has meaningful cost.
- **Surprising without context:** future readers would ask why this path was chosen.
- **Real trade-off:** genuine alternatives existed and one was chosen for specific reasons.

If any criterion is missing, use a spike or the Architecture/Components section of the feature spec instead.

## File Naming

Use the naming pattern from the Project Profile. In this repo, use:

```text
<task number> - <task name>.md
```

No special accents in filenames.

Resolve `<task number>` in order:

1. explicit user/card number
2. numeric segment from branch name, e.g. `feat/<id>-<slug>`
3. existing spec filename prefix

If no number is available, ask once in pt-BR: `"Qual o numero da tarefa/card para o nome do arquivo?"` If declined or unknown, use `<slug-descritivo>.md` and set **ID da Estoria** to `N/A`. Do not invent IDs.

## Workflow

### Refine Mode

When the user provides an existing spec path or asks to refine/update/complete, follow [references/refine.md](references/refine.md). Preserve existing IDs, ground changes against code, and ask only for missing information.

### Create Mode

Follow this order:

1. **Discover conventions:** read `AGENTS.md` and linked references; build the Project Profile.
2. **Pre-size:** classify preliminary scope from the request/card. Announce in pt-BR: `"Porte preliminar: ..."`
3. **Investigate code:** use [references/grounding.md](references/grounding.md). Verify real flows before asking or proposing.
4. **Re-size:** after investigation, confirm or adjust scope and phases. Announce changes before asking.
5. **Clarify:** ask only the questions needed for the scope using [references/process.md](references/process.md) and [references/clarification.md](references/clarification.md).
6. **Close doubts:** continue until every blocker is answered or signed as an assumption.
7. **Quality gate:** run [references/quality-gate.md](references/quality-gate.md) before delivery.
8. **Generate artifact:** save the spec/spike/ADR in the Project Profile directory using the routed template. Keep **Status: Draft** until explicit refinement confirmation.
9. **Confirm Refined:** if the closure gate passed, ask: `"Posso marcar o Status como **Refined**?"`
10. **Handoff:** offer relevant next skills: `implement`, `test`, `docs-writer`, `security-code-analysis`.

## Auto-Sizing

Use preliminary scope before investigation and final scope after investigation.

| Scope | Signals | Investigation | Questions | Phases |
| ----- | ------- | ------------- | --------- | ------ |
| **Small** | Point bugfix/refactor, up to 3 files, 1 layer | affected files | few, focused | 1, 5, 6 |
| **Medium** | Clear feature in existing module | module + neighbors | targeted questionnaire | 1-3, 5, 6; add 4 for APIs/integrations |
| **Large** | Multi-component or multi-layer feature | layers + integrations | full questionnaire | 1-6 + mandatory atomic task breakdown |
| **Spike** | Investigation without direct delivery | current code state and alternatives | findings/risks focused | objective, grounding, findings |

For Medium scope, include Phase 4 when the feature creates/consumes endpoints, events, integrations, or backend/API contracts. Frontend-only work with stable existing contracts may skip it unless contracts are unclear.

Large specs must include an atomic implementation breakdown in the same pass or a sibling `tasks.md`, as required by the quality gate.

## Feature Specs

Use [templates/TEMPLATE.md](templates/TEMPLATE.md) as one modular template. Include only modules that apply:

- **Backend/API:** APIs, Architecture, Data, Events, Technical Considerations as needed.
- **Frontend:** UI, consumed APIs, component architecture, states, validation, accessibility as needed.
- **CLI/lib/other:** modules that match the discovered stack and delivery surface.

Adapt layer and folder names to the discovered project. If project type or ownership is unclear, ask.

## Security-Sensitive Features

Flag and specify controls when the feature touches:

- authentication, authorization, RBAC, session/token handling
- PII, financial data, credentials, or secrets
- file upload, user-controlled URLs, external calls, or SSRF-prone integrations
- new public endpoints or changed trust boundaries

Document threats and controls in **Consideracoes Tecnicas > Seguranca** or the relevant auth/security phase. After **Refined**, offer `security-code-analysis`; do not block spec delivery on it unless the user asks.

## Multi-Repo Features

When the feature spans repositories:

- Create one spec per repo; each artifact describes only that repo's slice.
- Treat the current repo as source of truth for the local slice.
- Add a pt-BR **Dependencias cross-repo** section with repo/system, spec/contract, responsibility, and suggested order.
- The repo exposing the API/event owns the contract. Consumers link it and mark uncertain items as `(a confirmar)`.
- Offer to open/refine sibling specs as separate steps. Do not generate files in other repos without approval.

## References

Read these only when needed by the workflow:

- Investigation and anti-fabrication: [references/grounding.md](references/grounding.md)
- Refine existing spec: [references/refine.md](references/refine.md)
- Questionnaire by phase: [references/process.md](references/process.md)
- Clarification modes and closure gate: [references/clarification.md](references/clarification.md)
- Quality gate and handoff payload: [references/quality-gate.md](references/quality-gate.md)
- Examples for CA, assumptions, and spikes: [examples.md](examples.md)

## Templates

- Spec: [templates/TEMPLATE.md](templates/TEMPLATE.md)
- Spike: [templates/TEMPLATE_SPIKE.md](templates/TEMPLATE_SPIKE.md)
- ADR: [templates/TEMPLATE_ADR.md](templates/TEMPLATE_ADR.md)
