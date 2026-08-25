# Examples — Spec Writer

Concrete patterns for quality. Read when drafting `CA##`, assumptions, or choosing spike vs feature template.

## Critérios de aceite (CA##)

### ❌ Ruim — ambíguo, não testável

| ID | Problema |
| -- | -------- |
| CA01 | O sistema deve funcionar bem |
| CA02 | A API deve ser rápida |
| CA03 | O usuário consegue usar a tela sem dificuldade |

### ✅ Bom — QUANDO/ENTÃO, um comportamento por CA

| ID | Critério |
| -- | -------- |
| CA01 | QUANDO o gestor envia CSV com 10.000 linhas válidas ENTÃO o sistema DEVE processar em background e retornar HTTP 202 com `jobId` |
| CA02 | QUANDO o `jobId` é consultado após conclusão ENTÃO o sistema DEVE retornar status `completed` e URL do arquivo em até 5 minutos (p95) |
| CA03 | QUANDO o CSV contém CNPJ inválido na linha 42 ENTÃO o sistema DEVE rejeitar o arquivo com erro 400 e mensagem indicando linha e campo |

> **Dica:** um evento + um resultado observável por `CA##`. Facilita `test` (Gherkin em §10 da spec) e `implement` (TDD).

## Premissa assinada vs dúvida aberta

### ❌ Dúvida silenciosa (nunca)

Spec publicada sem seção de premissas, com contrato inventado:

```markdown
POST /api/v1/policies retorna 201 com body { "id": "uuid" }
```

*(endpoint não confirmado no código, usuário não validou)*

### ✅ Premissa assinada (ok após confirmação)

```markdown
| Premissa / Decisão | Default escolhido | Justificativa | Confirmada? |
| POST /policies response | 201 + `{ "id": string }` | Alinhar com CreatePolicyUseCase existente; payload completo na spec de API | Sim |
```

### ❌ Dúvida ainda aberta (bloqueia geração)

```markdown
**Dúvidas em aberto:**
- Timeout da integração com SAP: 30s ou 60s?
```

→ Voltar ao loop de clarificação. **Não** gerar/atualizar com Status Refined.

## Spike enxuta vs spec completa

| Situação | Template | O que incluir |
| -------- | -------- | ------------- |
| Investigar se Redis suporta o volume | `TEMPLATE_SPIKE.md` | Objetivo, achados, riscos, recomendação |
| Implementar cache Redis na listagem | `TEMPLATE.md` | RF/CA, arquitetura, módulo APIs, zonas cinzentas |
| Decisão irreversível: Redis vs in-memory cluster-wide | `TEMPLATE_ADR.md` | Contexto, decisão, alternativas |

**Spike boa (trecho):**

```markdown
## 3.1 O que descobrimos
| F1 | Listagem atual faz N+1 em PolicyRepository.findAll | `src/.../policy.repository.ts` | Alto |
## 5.1 Recomendação
Cache de 5min em Redis na camada application — spec de entrega separada.
```

**Evitar:** copiar o template completo de feature numa spike com RF01–RF15 vazios.

## Formato de entrada do usuário (resumo)

Ver também [TRIGGER_PHRASES.md — Spec Writer](../../TRIGGER_PHRASES.md#-spec-writer-triggers).

```
✅ "Gere a spec do card 2498685 - Markup por UF"
✅ "Refine docs/specs/2498685 - Markup por UF.md — código mudou desde a última versão"
✅ "Spike: vale migrar upload para S3? Investigue o fluxo atual em apps/api"
❌ "Me ajuda com essa história" (sem pedir spec/refino/spike)
```
