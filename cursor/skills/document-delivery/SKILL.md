---
name: document-delivery
description: >-
  Produces concise, audience-aware documentation: session summaries, README
  updates, PR descriptions, handoffs, ADRs, runbooks, and changelog-style
  notes. Use when the user asks to document, summarize deliverables, capture
  decisions, write handoff notes, or turn work into shareable information.
---

# Document delivery

## When to use this skill

- Turning work into something others (or future-you) can consume without re-reading the whole thread.
- Explicit asks: “document this,” “summary for the PR,” “README update,” “handoff,” “what changed,” “runbook.”

## Principles

1. **Audience first.** State who it is for (reviewer, ops, end user, teammate) and tune depth and jargon.
2. **Facts over vibes.** Anchor claims in code paths, commands, versions, ticket IDs, dates—only when known; mark unknowns honestly.
3. **Scannable structure.** Headings, short paragraphs, bullets over walls of text.
4. **Actionable tail.** “What to do next,” “how to verify,” or “open questions” whenever the doc should drive follow-up.
5. **No leaks.** No secrets, tokens, internal URLs that should stay private, or PII.

## Default deliverable shape

Adjust sections to fit; drop what does not apply:

```markdown
## Summary
One short paragraph: what was done and why it matters.

## Context (optional)
Problem, constraints, relevant background—only if needed to understand the summary.

## What changed
- Area / file / behavior — brief note per item

## Decisions (optional)
What was chosen, what was rejected briefly, and trade-offs if non-obvious.

## How to verify / test
Commands, UI steps, or checkpoints.

## Risks & follow-ups (optional)
Known limitations, TODOs, rollout notes.

## References (optional)
Issues, PRs, design docs, specs (full links when public and stable).
```

For **PR bodies**, prefer: Summary → Test plan (checklist) → Notes.

For **handoffs**, add: current state, blockers, “if you only read one thing.”

## Diagrams

Use **mermaid** only when the flow or architecture is hard to grasp from text; keep diagrams small and label components the reader already knows.

## Before you finish

- [ ] Title / first line states the purpose.
- [ ] No duplicate content that belongs in code comments only—prefer linking to the right file.
- [ ] Commands and paths are copy-pasteable and match the repo.
- [ ] Assumptions and unknowns are explicit.

## Localization

Match the language the user asks for (e.g. English for team-wide docs unless they request otherwise).
