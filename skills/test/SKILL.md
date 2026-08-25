---
name: test
description: >-
  Generates Gherkin BDD test scenarios (pt-BR) from feature specs or CA##.
  Default: append a Plano de Teste section inside the existing spec.
  Use when: creating a test plan, Gherkin scenarios, BDD coverage from a
  spec, or after spec handoff for test.
---

# Test Plan

This document contains technical implementation details for generating Gherkin BDD test scenarios.

## Output destination (mandatory)

| Mode | When | Where to write |
| ---- | ---- | -------------- |
| **Default — inline in spec** | Handoff from `spec-writer`, or user provides/points to a feature spec | Append/replace section **`## 10. Plano de Teste (Gherkin)`** in the **same** spec file (`docs/specs/<id> - <nome>.md`). Do **not** create a sibling `… - Test Plan.md` unless the user explicitly asks. |
| **Standalone file** | User explicitly asks for a separate file, or there is no feature spec | `docs/specs/<id> - <nome> - Test Plan.md` (or chat-only if the user prefers copy-paste) |

**Rules for inline mode:**

1. Resolve `spec_path` from the handoff payload or ask once for the spec path.
2. Read the existing spec; generate scenarios from `CA##` / RF / RN (and code context when useful).
3. Insert or replace **`## 10. Plano de Teste (Gherkin)`** **after** section 9 (Próximos Passos) and **before** `## Histórico de Revisões` (or after Histórico if section 10 already exists elsewhere — keep a single section 10).
4. In **§6.1 Testes Necessários**, add a one-line pointer: `Plano Gherkin detalhado → [§10](#10-plano-de-teste-gherkin)`.
5. Bump **Histórico de Revisões** (e.g. “Plano de teste Gherkin adicionado/atualizado”).
6. Tag each scenario with the related `CA##` in a trailing comment (`# CA01`) when mapping is clear.
7. After writing, summarize in chat: path of the spec + scenario count by category — do not dump the full Gherkin again unless the user asks.

## 🏗️ Gherkin Scenario Structure

All scenarios **MUST** strictly follow the format below:

```gherkin
### Cenário N – [Descriptive name of the scenario]
Dado que [precondition / initial context]
  E [additional precondition, if needed]
Quando [action performed by the actor]
  E [additional action, if needed]
Então [expected observable outcome]
  E [additional outcome, if needed]
```

### Golden Rules

- **Dado que** → sets the initial state / context (never an action)
- **Quando** → describes a single user/system action
- **Então** → describes the observable, verifiable result
- **E** → extends the immediately preceding keyword's chain
- One behaviour per scenario — never combine two distinct actions in one `Quando`
- Outcomes in **Então** must be verifiable (visible, measurable, or logged)
- Language must be clear enough for a non-technical stakeholder to understand

---

## 📂 Scenario Categories

Always generate scenarios across **all applicable categories**. Mark each scenario with its category tag:

| Tag | Category | Description |
|-----|----------|-------------|
| `@fluxo-feliz` | Happy Path | The main expected flow with valid data |
| `@fluxo-alternativo` | Alternative Flow | Valid secondary flows or optional branches |
| `@validacao` | Validation | Input validation, required fields, format checks |
| `@erro` | Error Handling | System errors, API failures, timeouts |
| `@borda` | Edge Case | Boundary values, empty states, extreme inputs |
| `@acesso` | Access Control | Permissions, roles, authenticated vs. unauthenticated |
| `@integracao` | Integration | Third-party services, APIs, message brokers |
| `@performance` | Performance | Pagination, large datasets, concurrent requests |

---

## 📋 Response Format

When generating test plans, always use this structured format.

**Inline in spec (default):** write the block below as section `## 10. Plano de Teste (Gherkin)` inside the feature spec (pt-BR headings). Keep Gherkin scenario bodies as specified.

**Standalone / chat:** the same structure may use the emoji headings below.

---

## 10. Plano de Teste (Gherkin)

> Preenchido pela skill `test`. Fonte: `CA##` da spec. Não criar arquivo `… - Test Plan.md` por padrão.

### Summary

| Field | Detail |
|-------|---------|
| **Feature** | [Name] |
| **Actors** | [User roles involved] |
| **Total Scenarios** | X |
| **Coverage** | Happy path, alternative, validation, error, edge case, access |

---

### ✅ Happy Path

```gherkin
### Cenário 1 – [Title]
Dado que [context]
Quando [action]
Então [outcome]
```

---

#### 🔀 Alternative Flows

```gherkin
### Cenário N – [Title]
...
```

---

#### ⚠️ Input Validations

```gherkin
### Cenário N – [Title]
...
```

---

#### ❌ Error Scenarios

```gherkin
### Cenário N – [Title]
...
```

---

#### 🔲 Edge Cases

```gherkin
### Cenário N – [Title]
...
```

---

#### 🔐 Access Control and Permissions

```gherkin
### Cenário N – [Title]
...
```

---

#### 🔗 External Service Integration *(if applicable)*

```gherkin
### Cenário N – [Title]
...
```

---

#### 📊 Scenario Summary

| # | Scenario | Category | Priority |
|---|---------|-----------|------------|
| 1 | [Title] | Happy Path | 🔴 High |
| 2 | [Title] | Validation | 🟡 Medium |
| … | … | … | … |

---

#### 💡 Automation Suggestions

- **Automation candidates**: [Scenarios suitable for automation]
- **Priority manual tests**: [Scenarios that need human judgment]
- **Suggested tools**: the E2E/API/BDD tools already adopted by the project (e.g., Cypress/Playwright for E2E, the stack's own test runner for API tests, Cucumber/Behave for pure BDD)

---

## 🎯 Priority Levels for Scenarios

Assign priority based on business risk and user impact:

| Priority | Criteria |
|------------|----------|
| 🔴 **High** | Core business flow, data integrity, authentication |
| 🟡 **Medium** | Alternative flows, common validations |
| 🟢 **Low** | Edge cases, cosmetic feedback, non-blocking errors |

---

## 🧠 Context-Aware Generation

When the user provides file paths or code context, use `codebase` and `search` tools to:

1. Identify input fields, required validations, and data types
2. Read controller/service logic to understand business rules
3. Map API contracts (DTOs, request/response shapes)
4. Detect existing error handling to generate error scenarios
5. Identify roles and guards to generate access control scenarios

**Example context lookups:**

```text
// From a controller/handler, extract:
// - Route method + path → Dado que o usuário acessa POST /prices
// - Auth guards/middleware → Cenário de acesso sem autenticação
// - Validation rules (required fields, type checks) → Cenários de validação
// - Response status codes → Cenários Então a API deve retornar 201 / 400 / 403
```

---

## ✍️ Language & Style Rules

- **All Gherkin scenarios**: Written in **Brazilian Portuguese**
- **All instructions and configuration**: Written in **English** (this file, frontmatter, comments)
- **Scenario titles**: Descriptive, action-oriented, starting with a verb or context noun
- **Avoid jargon**: Use business language, not technical terms (e.g., "o sistema exibe uma mensagem de erro" not "retorna HTTP 400")
- **Actor-centric language**: Write from the user's perspective, not the system's implementation

---

## 📚 Real-World Examples from This Project

### Example 1 – Price Listing Search (B2B)

```gherkin
### Cenário 1 – Busca acionada pela tecla Enter com SKU válido
Dado que o usuário está na listagem de preços (canal B2B ou B2P)
  E o campo de busca está visível e habilitado
Quando o usuário digita um SKU válido no campo de busca
  E pressiona a tecla Enter
Então a consulta de preços deve ser disparada com o filtro de busca igual ao SKU informado
  E a tabela deve exibir apenas os itens correspondentes ao SKU


### Cenário 2 – Busca acionada pelo botão de lupa com SKU válido
Dado que o usuário está na listagem de preços (canal B2B ou B2P)
  E o campo de busca está visível e habilitado
Quando o usuário digita um SKU válido no campo de busca
  E clica no ícone de lupa
Então a consulta de preços deve ser disparada com o filtro de busca igual ao SKU informado
  E a tabela deve exibir apenas os itens correspondentes ao SKU


### Cenário 3 – Busca com SKU inexistente
Dado que o usuário está na listagem de preços
  E o campo de busca está visível e habilitado
Quando o usuário digita um SKU que não existe na base de dados
  E pressiona a tecla Enter
Então a tabela não deve exibir nenhum resultado
  E o sistema deve exibir a mensagem "Nenhum item encontrado para o SKU informado"


### Cenário 4 – Campo de busca enviado vazio
Dado que o usuário está na listagem de preços
  E o campo de busca está visível, habilitado e vazio
Quando o usuário pressiona a tecla Enter sem digitar nenhum valor
Então nenhuma consulta deve ser disparada
  E a tabela deve permanecer com os dados já carregados


### Cenário 5 – Acesso à busca por usuário não autenticado
Dado que o usuário não está autenticado no sistema
Quando tenta acessar a listagem de preços diretamente pela URL
Então o sistema deve redirecionar para a tela de login
  E não deve exibir nenhum dado de preço
```

---

## 🔄 Workflow

1. **Understand the request**: Read the feature description / handoff (`spec_path`, `CA##`). Prefer an existing feature spec.
2. **Choose destination**: **Default = inline §10 in the spec**. Only use a sibling `… - Test Plan.md` (or chat-only) if the user explicitly asks or there is no spec.
3. **Gather context**: Read the spec (`CA##`, RF, RN, UI). If file paths are given, read the code to extract business rules and validations.
4. **Plan the categories**: Decide which scenario categories apply (happy path, errors, edge cases, access).
5. **Write scenarios**: Generate Gherkin in Brazilian Portuguese using the strict structure; map to `CA##` when possible.
6. **Persist**: Edit the spec file — insert/replace `## 10. Plano de Teste (Gherkin)`, link from §6.1, bump revision history. Wrap scenario groups in labeled code blocks (still copy-paste friendly for TestRail/Cucumber).
7. **Review for completeness**: Ensure all user paths, error states, and boundaries are covered.
8. **Suggest automation**: Recommend which scenarios are best suited for automated vs. manual testing.
9. **Chat summary**: Report spec path, total scenarios, and coverage — avoid re-pasting the entire plan.
