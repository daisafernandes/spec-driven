# Code Investigation and Anti-Fabrication

Base the spec on real project code and documented conventions. Clearly mark what already exists, what is proposed, and what still needs confirmation.

Prerequisite: read `AGENTS.md` and build the Project Profile. Investigation follows that profile, not a fixed folder model.

## Verification Chain

Use sources in order:

1. **Codebase:** code, conventions, flows, patterns.
2. **Project docs:** `AGENTS.md`, architecture docs, `/docs`, prior specs/ADRs.
3. **Library docs:** current official docs for dependencies pinned in the project.
4. **Web:** official/trusted sources only when local sources are insufficient.
5. **Uncertainty:** if still not found, say you did not find it and mark `(a confirmar)`.

Never present uncertain information as confirmed project behavior.

## What To Investigate

Targets come from the Project Profile and the requested change. Investigate the equivalents in this repo:

- structure, layers, modules, and ownership boundaries
- entry points: routes, screens, CLI commands, jobs, events
- business logic to mirror or change
- data models, schemas, migrations, persistence contracts
- API/integration contracts: request, response, errors, auth
- test patterns and verification commands
- architecture and coding conventions

If the repo is undocumented, infer from folder structure and manifests, then record confidence gaps as questions.

## Anti-Fabrication Rules

- Existing behavior must cite file paths.
- New behavior must be marked `(proposto)`.
- Unconfirmed behavior must be marked `(a confirmar)` and sent to the question loop.
- API contracts must reflect real code or be marked proposed/unconfirmed.
- Do not invent APIs, helpers, fields, errors, conventions, or test commands.

## Output

Return two lists for the spec workflow:

- real files/patterns found, with paths
- gaps/doubts to clarify

After investigation, return to the SKILL workflow and reassess scope before asking questions.
