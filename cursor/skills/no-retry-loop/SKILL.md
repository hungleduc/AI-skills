---
name: no-retry-loop
disable-model-invocation: true
description: >-
  Prevents debugging and problem-solving loops where earlier failed approaches are
  re-tried later without new evidence (e.g. solution1 fails, tries 2–5, then
  returns to solution1). Use when iterating on fixes, stubborn bugs, CI failures,
  or any multi-step troubleshooting; also when the user asks to avoid "going in
  circles" or repeating old solutions.
---

# Anti–solution-loop / no stale retries

## Goal

Avoid **circular retries**: proposing an approach **after other approaches failed**, without anything that invalidated the earlier reason it failed. That wastes time and looks like forgetting prior context.

## Mandatory: attempt ledger

When troubleshooting spans **more than one** fix attempt, keep an explicit ledger (in the assistant reply or internal notes). Before each new suggestion, update it:

| # | Approach (short) | Outcome | Why it failed or was insufficient (fact, not vibes) |
|---|------------------|---------|--------------------------------------------------------|

**Rule:** The next proposal must either be **novel** versus this table, or a **explicitly gated rerun** (see below).

## Post-attempt checkpoint ("hook")

**After each failed attempt and before proposing the next one**, briefly answer:

1. **Signal:** Exact error/output/behavior observed (quoted or pasted when possible).
2. **Hypothesis tested:** What we believed would happen if this approach worked.
3. **Verdict:** Hypothesis falsified? Test inconclusive? Wrong layer (config vs code vs env)?
4. **Ruled-out list:** Bullet list of approaches that are **off the table** until a gate clears.

Treat this checkpoint as blocking: no new “try X” until the ledger + checkpoint exist for the last attempt.

## Gated rerun (only way to revisit an old approach)

Returning to something **similar to a previously failed approach** is allowed **only if** all are true:

- You state **what was wrong last time** (from the ledger).
- You state **what changed**: new fact, corrected assumption, narrower scope, different version, reproduction step fixed, etc.
- The new attempt is **not identical** unless the environment or premises truly changed.

Otherwise: **pick a different hypothesis axis** (different subsystem, prerequisite, tooling version, isolation test).

## Pivot rules

- **Third failed distinct hypothesis** (or still stuck after probing the same symptom with materially different tactics): stop guessing. Deliver a **short recap** (symptom, what was ruled out), list **specific information** needed (logs, minimal repro, one command output), or offer **narrow binary checks** rather than repeating broad ideas.
- **Do not** cycle A → B → C → D → A without the gated-rerun checklist.

## Red flags (forbidden patterns)

- Suggesting a fix that **already appears in the ledger as failed**, with no "what changed".
- Replacing wording but keeping the **same underlying action** (e.g. "use env var X" vs "export X before run" counts as one approach unless the mechanism of failure changed).
- **Breadth wandering:** many shallow tries without updating ruled-out hypotheses.

## Optional tie-in with Cursor hooks

Cursor **hooks** can remind you between steps; they cannot replace reasoning. Prefer this skill's **ledger + post-attempt checkpoint** as the source of truth. If the user configures a hook, use it only to trigger the checkpoint text, not to skip the ledger.
