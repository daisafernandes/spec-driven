# Questionnaire by Phase

Use this file for **what** to ask. Use [clarification.md](clarification.md) for **how** to ask and how to close doubts.

Act as a reasoning partner, not an interrogator. Adapt phases to scope and project type. Do not ask UI questions for backend-only work or migration questions for frontend-only work.

## Conduct Rules

- Do not repeat answered questions.
- Challenge vague answers.
- Ask only what is needed for the selected scope.
- Extract information from screenshots/Figma/images before asking.
- Confirm understanding after each block or answer.

## Phases

### Phase 1: Problem Understanding

Always ask or synthesize:

1. What problem are we solving?
2. Who is the user/persona?
3. What action do they need to perform?
4. Why does it matter?
5. What reference exists: card, Figma, prototype, benchmark, doc?

Final spec writes the user story in pt-BR.

### Phase 2: Scope and Structure

6. Delivery type: feature, bugfix, change, refactor, integration, spike?
7. Overall structure: backend flow, UI surface, CLI command, job, event, etc.
8. Main areas or steps.
9. Edit/update flow beyond creation.
10. Dependencies on other features or systems.

### Phase 3: Functional Detail

11. Required fields/data and types.
12. Required versus optional data.
13. Conditional behavior.
14. Validations: format, range, uniqueness, limits.
15. States: loading, empty, error, success, disabled, selected.

### Phase 4: Integrations and APIs

Use when backend/API/integration-heavy or when frontend consumes/contracts APIs.

16. Endpoints/events consumed or created: method/path/topic.
17. Request/response payloads.
18. Error handling: codes, messages, retries.
19. Dependent calls or ordering.
20. Cache, polling, retry, idempotency.

Cross-check with investigated code. Mark unconfirmed contracts `(a confirmar)` or proposed contracts `(proposto)`.

### Phase 5: Gray Areas and Business Rules

Always cover these dimensions. Each becomes a requirement or `N/A porque [motivo]`.

| Dimension | Cover |
| --------- | ----- |
| Input validation and limits | formats, ranges, sanitization |
| Failure / partial failure | timeouts, rollback, partial save |
| Idempotency / duplication | safe retries, dedup keys |
| Auth / RBAC / limits | who can do what |
| Concurrency / ordering | races, locks, ordering |
| Data lifecycle | TTL, archival, deletion |
| Observability | logs, metrics, traces |
| External dependency failure | fallback, circuit breaker |
| State transitions | valid transitions and guards |

Closing questions:

21. Which business rules apply? Assign `RN##`.
22. Known edge cases.
23. Quantity, size, or time limits.
24. Feature flag or access control.
25. What happens on failure?

### Phase 6: Definition of Done

26. Acceptance criteria: assign `CA##`, one observable outcome per criterion, pt-BR format `QUANDO ... ENTAO ...`.
27. Required tests: unit, integration, e2e.
28. Accessibility requirements for frontend.
29. Performance requirements.

See [examples.md](../examples.md) for good/bad `CA##` patterns.

## Traceability

Assign IDs during the questionnaire:

- `RF##`: functional requirements
- `RN##`: business rules
- `CA##`: acceptance criteria

The template must cross `CA##`, `RF##`/`RN##`, and test expectations.

## Screenshot/Figma Analysis

For visual inputs, extract fields, input types, buttons, actions, flows, navigation, validations, and UI states. Confirm the extraction before treating it as requirement truth.
