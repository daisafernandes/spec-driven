---
name: docs-writer
description: >
  Write, review, and update documentation files with consistent structure, tone, and technical accuracy. Use when: creating new docs, reviewing existing markdown files in /docs or README.md, updating docs after code changes in /apps, /libs or /src (or the default folder). Keep documentation accurate, clear, consistent, up-to-date, and easy for users to understand — developers, SREs, and product managers. Do NOT use for: inline code comments, docstrings, API annotation/decorator documentation, or auto-generated documentation tooling.
---

# Docs Writer

## Absolute Rules

1. **PT-BR in all docs.** All documentation uses Brazilian Portuguese. Technical terms without a consolidated translation (e.g., "branch", "commit", "deploy") stay in English.
2. **No duplication.** Each type of information (e.g., environment variables, integrations, glossary terms) has a single source-of-truth doc, as defined in the project's `.agents.md`. Never copy that content into other docs — link to the source instead.
3. **Link, don't copy.** If information already exists in another doc, add a link. Don't repeat it.
4. **Verify against code.** Before documenting any behavior, read the relevant code to confirm what you're writing is true.
5. **Integration status is mandatory.** Every documented integration must have a clear status: Active in Production / Active in HML (or any other environment) / Inactive / Awaiting activation.

---

## Project Documentation Map

This skill doesn't assume a fixed docs structure or a fixed code → documentation mapping — each project defines its own. Before writing or updating docs:

1. Read the project's `.agents.md` (or equivalent root instructions file) to learn which docs exist, what each one covers, which doc is the single source of truth for each type of information, and the rules that map each type of code change to the docs that must be updated.
2. If the project has no such mapping documented yet, infer it by inspecting the `docs/` folder structure and confirm with the user before proceeding.

Typical categories found across projects (exact names and files vary): system overview, architecture/flows, local setup, deployment/CI-CD, external integrations, environment variables, monitoring/alerts, runbooks, troubleshooting/FAQ, domain glossary, tech stack, ADRs.

---

## Writing/Update Process

### Step 1: Understand the goal

1. Read the relevant code — don't document what you haven't verified
2. If updating: read the current version of the doc before editing
3. Consult the project's `.agents.md` to identify which docs are affected by this type of change
4. Check the project's glossary doc (per `.agents.md`) for correct terminology

### Step 2: Check impact on other docs

- If adding an integration: check whether the architecture/flow doc and the environment-variables doc (per `.agents.md`) need updating
- If removing an enum or constant: check whether other docs reference it
- If renaming a component, module, or service: search all occurrences across docs

### Step 3: Write

**For new documents** — use the structure: Overview → Technical details → Examples/Commands → Troubleshooting (if applicable)

**For updates** — change only what changed. Don't rewrite intact sections.

**Quality checks:**

- No content duplicated from another doc (link to the single source of truth instead)
- Integration status clearly indicated, if applicable
- Commands are testable and correct
- Internal links working (use relative paths)
- Terminology consistent with the project's glossary doc

### Step 4: Verify

- Re-read the doc after editing
- Confirm all internal links are valid (relative, not absolute)
- Confirm content is factually correct against the code
- For large changes, explicitly list what was changed

---

## Style Standards

- **Headings:** H1 for the doc title, H2 for main sections, H3 for subsections
- **Tables:** For lists of variables, enums, integrations — prefer tables over lists
- **Code:** Always in code blocks with the language specified (` ```bash `, ` ```<language> `)
- **Important notices:** Use `> **Important:** text` or `> **Warning:** text`
- **Status:** Use consistent text badges: **Active in Production** / **Active in HML** / **Inactive** / **Awaiting activation**
- **Command lists:** Include a comment explaining what the command does before each block
- **Docs language:** All documentation content must be in **PT-BR**

---

## What NOT to document here

- Inline code comments/docstrings — go in the code
- API documentation annotations/decorators — go in the code
- Implementation details that the code already explains clearly
- Temporary decisions — if it's temporary, don't document it; if permanent, create an ADR in the project's ADR folder

---

## Example: updating docs after a PR

**Docs to update (per the project's `.agents.md` mapping):**

1. The integrations doc → add new messaging integration with status, topic, and flow
2. The architecture/flows doc → add new flow in the relevant section
3. The environment-variables doc → if there's a new variable for the topic, add it

**What NOT to do:** copy code or configurations — link to the code in the repository, or describe the behavior in prose.
