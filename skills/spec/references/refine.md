# Refine Mode

Use when the user wants to update, refine, or complete an existing spec/ADR instead of creating a new artifact.

Signals: spec path, "refine", "atualizar spec", "completar spec", or an existing matching file in `docs/specs/` / `docs/adr/`.

## Entry

1. Read the full existing document.
2. Note status, existing `RF##` / `RN##` / `CA##`, signed assumptions, and revision history.
3. Ground against current code via [grounding.md](grounding.md); the spec may be stale.
4. Announce in pt-BR: `"Modo refine - vou preservar IDs confirmados e focar so em lacunas ou mudancas."`

## Preserve

- Existing IDs. Do not renumber unless merging/splitting; document changes in revision history.
- Answers already confirmed in the spec or conversation.
- Signed assumptions marked `Confirmada? Sim`, unless code or user input contradicts them.
- Valid existing sections; do not delete them without user approval.

## Update Rules

| Trigger | Action |
| ------- | ------ |
| Code diverged from spec | update affected sections and cite path evidence |
| New scope from user | add new RF/CA/RN IDs; move unrelated ideas to **Fora de escopo** |
| Open doubts | run [clarification.md](clarification.md) only for those items |
| Draft status | after closure gate, ask to mark **Refined** |
| Spike becomes delivery | prefer a new feature spec linked from the spike |

## Questionnaire

Default to synthesis: summarize current spec + code delta and ask only gaps. Use batch or interview only for unresolved or changed decisions. Do not rerun Phases 1-6 if still valid.

## Output

1. Edit the same file unless the user asks for a new file or spike-to-feature split.
2. Bump revision history with date, version, change summary, and author.
3. If scope grew, reassess auto-sizing and add missing modules/sections.

## Handoff

Follow [quality-gate.md](quality-gate.md). Large specs need a breakdown before **Refined**; other scopes may offer one. Then offer `implement`, `qa-test-plan`, `docs-writer`, or `security-code-analysis` as relevant, passing the updated spec path and task/card id if known.
