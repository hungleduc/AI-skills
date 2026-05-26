---
name: session-handoff
description: >-
  Compacts long agent sessions into a structured handoff document so work can
  continue in a fresh chat without re-explaining context. Use when the user asks
  to save context, hand off, compact the session, start a new session, resume
  from handoff, or when conversation is long and accuracy is degrading (~140k
  context).
user-invocable: true
---

# Session Handoff

Preserve task continuity across chats by exporting a **compact handoff** before context degrades, then importing it in a new session.

**Target size:** 1,500–4,000 tokens. Dense facts, not conversation replay.

---

## When to Use

| Mode | Triggers |
|------|----------|
| **Export** | "save context", "hand off", "compact session", "start new chat", context feels stale, before closing a long task |
| **Import** | "continue from handoff", "resume", user attaches/pastes a handoff file, new session with `@.cursor/handoff/latest.md` |

---

## Export Workflow

```
Task Progress:
- [ ] 1. Gather current state
- [ ] 2. Write handoff document
- [ ] 3. Verify completeness
- [ ] 4. Tell user how to resume
```

### Step 1: Gather Current State

Run in parallel where possible:

```bash
git status --short
git branch --show-current
git log -5 --oneline
git diff --stat
```

Also capture from conversation (do not paste long code):

- Primary goal and definition of done
- Decisions made **with rationale** (rejections matter too)
- Files touched and **why** (path + one line each)
- Test commands run and pass/fail status
- Blockers, open questions, user preferences stated this session
- What was explicitly **out of scope**

**Drop from handoff:** exploratory dead ends, verbose tool output, full diffs, repeated explanations.

### Step 2: Write Handoff Document

Write to **both** locations:

1. **Project (default resume point):** `.cursor/handoff/latest.md`
2. **Personal archive (optional, cross-machine):** `~/.cursor/handoff/<repo-basename>-latest.md`

Create directories if missing. Use the template at [templates/handoff-template.md](templates/handoff-template.md).

Replace `<placeholders>`. Keep sections; use `—` or `None` for empty fields.

### Step 3: Verify Completeness

Before finishing export, confirm the handoff answers:

- [ ] What is the user trying to accomplish?
- [ ] What was already done?
- [ ] What is the very next action?
- [ ] What must **not** be changed or re-decided?
- [ ] How does the next agent verify state? (commands + expected outcome)

If any answer is missing, update the handoff.

### Step 4: Tell User How to Resume

Give the user this exact starter prompt for the new session:

```markdown
Continue this task. Read @.cursor/handoff/latest.md and follow the Next Actions.
Do not re-explore solved areas unless the handoff says verification failed.
```

If the project has no `.cursor/` yet, tell them to paste the handoff body directly instead.

---

## Import Workflow

When resuming from a handoff:

1. **Read** `.cursor/handoff/latest.md` (or the pasted/archived file) first — treat it as source of truth over chat history.
2. **Verify** live state matches the handoff:
   ```bash
   git status --short
   git branch --show-current
   git diff --stat
   ```
3. **Reconcile** mismatches — if files/commits differ, note drift in one short paragraph, then proceed from **verified** state.
4. **Execute** the first item under **Next Actions** without re-asking settled decisions.
5. **Update** the handoff after meaningful progress (export-lite: refresh Status, Decisions, Next Actions only).

Do **not** re-read entire codebase unless the handoff flags uncertainty or verification failed.

---

## Compaction Rules

When the session is long, prioritize retention in this order:

1. **Goal + definition of done**
2. **Next 1–3 concrete actions** (with file paths)
3. **Decisions + constraints** (including user "do not" rules)
4. **Current git state** (branch, dirty files summary)
5. **Test/verify commands** and last known result
6. **Key file map** (path → role, max ~10 entries)
7. **Blockers / open questions**

Omit: intermediate debugging, failed approaches unless the failure reason prevents repeat mistakes, full code blocks (use `path:line` references instead).

---

## Handoff Freshness

- One `latest.md` per project — overwrite on each export.
- Add `exported_at` ISO date in the handoff header.
- If resuming and handoff is >7 days old, verify git state before trusting file lists.

---

## Additional Resources

- Template: [templates/handoff-template.md](templates/handoff-template.md)
- Example: [examples/example-handoff.md](examples/example-handoff.md)
