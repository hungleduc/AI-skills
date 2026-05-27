---
name: planning-new-features
description: >-
  Plans new features before coding (planning + implementation markdown specs),
  then prescribes implementing with TDD; embeds RED–GREEN–REFACTOR, coverage, and
  test-type expectations aligned with sibling tdd-workflow. Use when planning or
  speccing a feature, designing before coding, or when the user expects tests first.
user-invocable: true
---

# Planning New Features

Plan before you code. Produce two markdown artifacts, validate proposed code, then **implement behavior with test-driven development** (norms below align with sibling [../tdd-workflow/SKILL.md](../tdd-workflow/SKILL.md) for deeper examples and patterns).

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
- [ ] 3. Write implementation spec (tests-before-code order in markdown; see Step 3)
- [ ] 4. Validate syntax
- [ ] 5. Review and hand off (explicit approval before editing source files)
- [ ] 6. Implement with TDD: RED → GREEN → REFACTOR → coverage check (below)
```

If the user only wants a design/spec with **no implementation in this session**, complete steps 1–5 and attach the embedded TDD section as the **mandatory playbook** for whoever implements later.

---

## TDD norms (included from `tdd-workflow`)

Use whenever **net-new behavior** from the plan is implemented (same session or handoff).

### Principles

1. **Tests before production code** — write or extend failing tests first, then minimal code to pass.
2. **Coverage targets** — aim for **≥80%** on touched code **where the project tooling supports it** (unit **+** integration **+** E2E **as appropriate**; not every feature needs full E2E).
3. **Test layers**
   - **Unit:** pure logic, helpers, isolated components/functions.
   - **Integration:** DB, HTTP handlers, subprocesses, file I/O, real wiring with narrow scope.
   - **E2E:** browser or full-stack journeys for **critical** user paths only—keep small and stable.

Adapt commands to this repo (`pytest`, `npm test`, `go test`, etc.); examples in [../tdd-workflow/SKILL.md](../tdd-workflow/SKILL.md) are illustrative.

### RED → GREEN → REFACTOR loop

**RED**

- Add or update tests that express the **acceptance criteria** from the planning doc before changing production logic.
- **Run** the relevant test target; confirm **meaningful RED**:
  - **Runtime RED:** exercise the new assertion / missing behavior.
  - **Compile-time RED** is acceptable **only when** compilation failure is intentionally the RED signal for the new API or fixture.
  - Ignore RED from broken setup, unrelated syntax, or flaky infrastructure—fix setup first.

**GREEN**

- Write the **smallest** production change that makes the RED tests pass.

**REFACTOR**

- Improve structure/readability **while tests stay green**; no behavior change mixed in unless covered by tests.

Repeat per slice until the implementation spec checklist is satisfied.

### Git checkpoints (when using Git)

- Prefer **explicit commits** marking stages when the workflow is substantive (optional for tiny edits only if RED/GREEN are still observable in CI or logs).
- Compact baseline: failing test (**RED**) → minimal implementation (**GREEN**) → optional refactor.
- Checkpoint messages should name the stage and observable evidence (`test:` / `fix:` / `refactor:` as per team convention).

### After features land

- Run the project coverage command (**e.g.** `pytest --cov`, `npm run test:coverage`) on affected packages and close obvious gaps reflected in the plan’s test plan section.

Full deep-dive patterns (TS/Jest mocks, Playwright layout, pitfalls): **[../tdd-workflow/SKILL.md](../tdd-workflow/SKILL.md)**.

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
| Test plan | How to verify—including **behavior-level** bullets that will drive unit/integration/E2E cases |

Keep the planning doc at the **design level** — no full code blocks here. Pseudocode and diagrams are fine. The **Test plan** rows should be concrete enough that someone can derive **exact test descriptions** later (given/when/then mentally).

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
- **Test-first ordering in the doc:** present **new/updated tests** (or test file excerpts) **before** the production code that should satisfy them, unless the spike is exploratory—then revert to documented TDD after the spike
- Specify which tests are unit vs integration vs E2E-minded (labels in the markdown are enough)
- Do **not** edit source files yet unless the user explicitly asks to implement—but when implementing, immediately follow **TDD norms** above and [tdd-workflow/SKILL.md](tdd-workflow/SKILL.md)

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
- [ ] Behavioral slices show **tests / test outline before** paired production snippets (TDD order)

If validation fails, fix the implementation spec and re-run. Do not proceed to implementation with broken snippets.

---

## Step 5: Review and Hand Off

Present to the user:

1. Link/path to the planning doc
2. Link/path to the implementation spec
3. Syntax validation result (pass/fail + what was checked)
4. Open questions or decisions needed
5. Estimated implementation order (which files, in what sequence)—**first slice should name the RED test(s)** you’ll add before production edits

Ask whether to proceed with implementation. Only edit source files after explicit approval. **When proceeding, run the RED → GREEN → REFACTOR workflow** described above; do **not** add production-only changes without failing tests guiding them.

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
- **TDD implementation** — when shipping behavior, follow RED → GREEN → REFACTOR and project test commands; deepen with [../tdd-workflow/SKILL.md](../tdd-workflow/SKILL.md)
- **Minimal scope** — prefer the smallest change that meets the goal
- **Reuse existing patterns** — read surrounding code before proposing new abstractions
- **No secrets in docs** — never put API keys, tokens, or credentials in planning files
- **Keep docs current** — if implementation diverges from the spec, update the doc or note the delta

## Additional Resources

- Planning doc template: [templates/plan-template.md](templates/plan-template.md)
- Implementation spec template: [templates/implementation-spec-template.md](templates/implementation-spec-template.md)
- Expanded TDD workflows, checkpoints, and examples: [../tdd-workflow/SKILL.md](../tdd-workflow/SKILL.md)
