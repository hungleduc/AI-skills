---
name: planning-new-features
description: Plans new features before coding — writes planning docs, prepares code sketches in markdown, validates syntax, and produces an implementation checklist. Use when the user wants to plan a feature, write a spec, design before implementation, or asks for a feature plan.
user-invocable: true
---

# Planning New Features

Plan before you code. Produce two markdown artifacts, validate proposed code, then hand off for implementation.

## When to Use

- User asks to plan, spec, or design a new feature
- Feature touches multiple files or has unclear scope
- User wants code prepared in markdown before editing source files
- User says "plan first", "write a spec", or "design doc"

## Workflow Overview

```
Task Progress:
- [ ] 1. Gather context
- [ ] 2. Write planning doc
- [ ] 3. Write implementation spec (code in markdown)
- [ ] 4. Validate syntax
- [ ] 5. Review and hand off
```

---

## Step 1: Gather Context

Before writing anything:

1. Read the user's request and clarify ambiguities (ask only what blocks planning)
2. Explore relevant code — routes, models, templates, tests, config
3. Identify constraints: existing patterns, auth, DB schema, API contracts
4. Decide scope: in-scope, out-of-scope, follow-up work

Output a one-paragraph summary of what you understood before proceeding.

---

## Step 2: Write Planning Doc

Create a markdown file in `docs/plans/` (create the directory if needed):

```
docs/plans/<feature-slug>.md
```

Use kebab-case for `<feature-slug>` (e.g. `user-export-csv`, `webhook-retry`).

Copy structure from [templates/plan-template.md](templates/plan-template.md). Required sections:

| Section | Purpose |
|---------|---------|
| Problem | What pain exists today |
| Goal | What success looks like |
| Scope | What's included |
| Non-goals | What's explicitly excluded |
| Design | Approach, data flow, API/UI changes |
| Files affected | High-level list of paths |
| Risks | Edge cases, migrations, breaking changes |
| Test plan | How to verify the feature works |

Keep the planning doc at the **design level** — no full code blocks here. Pseudocode and diagrams are fine.

---

## Step 3: Write Implementation Spec (Code in Markdown)

Create a companion file:

```
docs/plans/<feature-slug>-implementation.md
```

Use [templates/implementation-spec-template.md](templates/implementation-spec-template.md).

For each file to change:

1. State the file path
2. Describe the change (add / modify / delete)
3. Include the **proposed code** in fenced blocks with the correct language tag
4. Mark insertion points when modifying existing files (`// ... existing code ...`)

Rules for code sketches:

- Match project conventions (naming, imports, error handling)
- Keep diffs minimal — only what the feature needs
- Include new tests in the spec when behavior changes
- Do **not** edit source files yet unless the user explicitly asks to implement

---

## Step 4: Validate Syntax

Extract code from the implementation spec and verify it compiles/parses before handoff.

### Python (this project)

```bash
# Extract each python block to a temp file and compile
python -m py_compile /tmp/extracted_snippet.py
```

Or run project linters if available:

```bash
python -m compileall -q app.py   # sanity check existing app
# ruff check / mypy — if configured in the project
```

### JavaScript / TypeScript

```bash
node --check file.js
npx tsc --noEmit
```

### HTML / Jinja templates

- Verify tags and blocks are balanced
- Check that referenced variables/routes exist in the codebase

### SQL

- Validate against the project's migration style
- Flag destructive operations (DROP, DELETE without WHERE)

### Validation checklist

- [ ] Every code block has a language tag
- [ ] Imports and symbols referenced in snippets exist or are defined in the same spec
- [ ] No syntax errors in extracted snippets
- [ ] Proposed API shapes match existing patterns in the codebase
- [ ] Test snippets follow the project's test framework

If validation fails, fix the implementation spec and re-run. Do not proceed to implementation with broken snippets.

---

## Step 5: Review and Hand Off

Present to the user:

1. Link/path to the planning doc
2. Link/path to the implementation spec
3. Syntax validation result (pass/fail + what was checked)
4. Open questions or decisions needed
5. Estimated implementation order (which files, in what sequence)

Ask whether to proceed with implementation. Only edit source files after explicit approval.

---

## Output Locations

| Artifact | Path |
|----------|------|
| Planning doc | `docs/plans/<feature-slug>.md` |
| Implementation spec | `docs/plans/<feature-slug>-implementation.md` |

If the user prefers a different location (e.g. `.cursor/plans/`, issue comment, PR description), use that instead and note it in the doc header.

---

## Rules

- **Plan first, code second** — don't jump to editing `app.py` or templates during planning
- **Minimal scope** — prefer the smallest change that meets the goal
- **Reuse existing patterns** — read surrounding code before proposing new abstractions
- **No secrets in docs** — never put API keys, tokens, or credentials in planning files
- **Keep docs current** — if implementation diverges from the spec, update the doc or note the delta

## Additional Resources

- Planning doc template: [templates/plan-template.md](templates/plan-template.md)
- Implementation spec template: [templates/implementation-spec-template.md](templates/implementation-spec-template.md)
