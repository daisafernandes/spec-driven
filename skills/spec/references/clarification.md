# Question Loop and Closure Gate

No spec is finalized with open questions. Every ambiguity must be either answered by the human or recorded as a signed assumption with default, rationale, and explicit confirmation.

[process.md](process.md) defines **what** to ask. This file defines **how** to ask and when the spec is ready to generate.

## Question Modes

| Mode | Use when | How |
| ---- | -------- | --- |
| **Batch** | questions are independent | ask 4-6 focused questions with concrete options when useful |
| **Interview** | decisions depend on previous answers | ask one question at a time with a recommendation |
| **Synthesis** | card/conversation/code already provides most context | summarize understanding, list only gaps, then close doubts |

Rules:

- Do not repeat answered questions.
- Challenge vague answers: "good" how, "users" who, "simple" how much.
- Confirm understanding after each block or answer.
- Switch modes when needed.
- Asking clarifies how to implement the current scope; it does not expand scope.

## Loop

Repeat until closed:

1. Collect doubts from code gaps, user contradictions, unresolved Phase 5 dimensions, and API/rule mismatches.
2. Choose batch/interview/synthesis and ask.
3. Record answers and assumptions.
4. Reassess whether new doubts appeared.
5. Before generating, present a **Doubt Balance**:
   - Resolved: short summary.
   - Signed assumptions: default + reason; ask for explicit confirmation.
   - Still open: do not generate while any item remains.

## Closure Gate

All checks must pass before generating:

1. **Precision:** every `CA##` has one interpretation and a defined expected outcome.
2. **Doubt closure:** every decision is answered or recorded in **Premissas e Duvidas em Aberto** with confirmation.
3. **Contract grounding:** endpoints, payloads, and rules are confirmed in code or marked `(proposto)` / `(a confirmar)`.

Large scope requires the full gate. Medium can record lower-risk assumptions. Small stays lean, but never silent.

## Scope Guardrail

If the user suggests something outside scope, record it under **Fora de escopo** or **Ideias adiadas** and return to the current point. Do not silently add new capabilities.

## Document Section

Use the template's pt-BR section:

```markdown
## Premissas e Duvidas em Aberto

| Premissa / Decisao | Default escolhido | Justificativa | Confirmada? |
| ------------------ | ----------------- | ------------- | ----------- |
| [ambiguidade] | [o que faremos] | [por que] | Sim/Nao |

**Duvidas em aberto:** nenhuma - todas resolvidas ou registradas acima.
```
