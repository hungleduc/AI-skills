---
name: analyze-jira-requirements
description: >-
  Loads Jira issues via connected Atlassian MCP when possible, then converts
  natural-language ticket text into structured, testable engineering specs
  (goals, scope, acceptance criteria, questions). Use when a Jira requirement
  is unclear; supply an issue URL/key or paste text; prefers live Jira reads
  over paste when MCP is available.
---

# Analyze requirements (NL → engineering)

## Goal

Interpret **messy natural language** into **precise specs** your AI/agent or team can execute: clear entities, verbs, boundaries, acceptance checks, and what is still unknown.

## Source of truth: Jira + Atlassian MCP (preferred)

**Default workflow when Atlassian MCP (Jira) is connected:**

1. **Resolve the issue identifier** — from the user’s Jira URL, issue key (`PROJ-123`), or “current ticket”; ask once if ambiguous.
2. **Read the requirement from Jira** — use **Atlassian/Jira MCP** tools (or bundled Atlassian Cursor integration) to load the issue: summary, description, type, priority, labels/components, epic/parent links, status, assignee/reporter fields the API exposes, **and relevant comments** (especially acceptance notes, QA, pasted logs).
3. **If MCP call fails or returns empty** — say so plainly: ask the user to **confirm MCP connection and Jira credentials/scopes**, or to paste summary + description + last comments.

**Paste-only fallback:** If MCP is not configured or the user insists on offline text, paste **summary + description + key comments**. Do **not** treat paste as canonical if MCP can retrieve the ticket—prefer re-fetch before finalizing edits that might change Jira wording.

Treat attachments as opaque unless MCP exposes summaries or text—do **not** invent filenames or attachment contents.

## Procedure

1. **Acquire source text** — Jira MCP fetch (preferred above) → else paste → else stop and instruct connect/paste.
2. **Restate plainly** — 2–4 sentences from the reporter’s viewpoint (no jargon yet). If contradictory, quote the clash.
3. **Extract signals**
   - **Who** uses or is affected?
   - **What** observable outcome is wanted (not “better” unless quantified)?
   - **Where** in the system does it attach (surface, API, infra, HA, build, CLI, docs)?
   - **When/triggers** — on-demand, cron, failover, upgrade, install?
   - **Constraints** — versions, SLA, rollback, downtime, compat, locales?
4. **Translate to technical anchors** — name likely components/files/services *as hypotheses*: “probably touches X because Y”; mark **uncertain** where needed.
5. **Produce acceptance criteria** — **given / when / then** bullets; tie to observable outputs (logs, exit codes, UI state, metric), not vibes.
6. **Gap analysis** — list **blocking questions** and **cheap validation** steps (one command or one spike) per gap.

Treat this as falsifiable: refine when new facts arrive (Jira comment updates via MCP refresh, reproduction, codebase).

## Output template

Copy and fill; omit sections when N/A:

```markdown
## Intent (human language)
-

## Interpretation (engineering)
- Probable subsystem(s):
- Data / control flow (short):
-

## Functional requirements (numbered)
1.

## Non-functional / constraints
-

## Acceptance criteria (testable)
- [ ]

## Dependencies & risks
-

## Assumptions (explicit — may be wrong)
-

## Questions for reporter / analyst
1.

## Next technical steps (suggested order)
1.
```

## Phrase → typical technical reading (hypotheses, not doctrine)

Use only as a **guess** checklist; validate against ticket + code:

| NL pattern | Might mean (verify) |
|------------|---------------------|
| “Make it robust” | Retries/idempotency, timeouts, backoff, draining, quorum |
| “Faster” | Latency budgets, profiling target, caching, bulk vs single |
| “Same as prod” | Parity matrix (config, versions, feature flags), drift |
| “Should just work after upgrade” | Migrations order, rollback, state compatibility |
| “Only sometimes fails” | Races, resource limits, network partitions, flaky I/O |

## Guardrails

- **No hallucinated priorities** unless ticket states them—call out inferred priority separately.
- **No secrets** in restatements; scrub tokens, IPs, passwords.
- Prefer **narrow scope**: if epic-sized, propose split into backlog items with refs.
- When **MCP read succeeded**, cite concrete fields you used (“from Jira summary/description/comment …”)—do **not** paraphrase as if from memory alone.

### If MCP is not connected

Briefly instruct: **Cursor → MCP / Atlassian** (or workspace Atlassian integration) → connect → retry with issue key or URL; offer paste-only summary as interim if the user cannot connect now.
