---
name: mdots-feature-delivery
description: >-
  End-to-end MDOTS feature delivery: Jira requirements → planning spec → TDD
  implementation → delivery gates (unit tests, ISO rebuild, VM smoke). Use when
  the user gives a todo list like "analyze ticket, plan feature, implement UI"
  or asks to ship a feature in mdots-distro.
user-invocable: true
---

# MDOTS feature delivery (Jira → plan → ship)

Orchestrates sibling skills with **mandatory gates**. Do not skip gates or mark
the workflow complete until every applicable gate passes.

## Related skills

| Step | Skill | Path |
|------|-------|------|
| 1 | Analyze requirements | [../analyze-jira-requirements/SKILL.md](../analyze-jira-requirements/SKILL.md) |
| 2 | Plan + implementation spec | [../planning-new-features/SKILL.md](../planning-new-features/SKILL.md) |
| 3 | TDD during implement | [../tdd-workflow/SKILL.md](../tdd-workflow/SKILL.md) |
| 4 | Boot / VM bugs | [../systematic-debugging/SKILL.md](../systematic-debugging/SKILL.md) |

## User todo template

When the user sends a delivery todo list, run in order:

1. **Analyze Jira** — fetch ticket; output must include **Runtime context** (see analyze-jira skill).
2. **Plan feature** — attach UI screenshots if provided; test plan must include **VM smoke** when installer/kiosk/ISO involved.
3. **Implement with TDD** — from `docs/plans/<slug>-implementation.md`; mirror sibling files (Gate C).
4. **Run delivery gates A–E** below — report pass/fail for each.

Suggested single user message:

```text
Delivery workflow for OTSECURITY-XXXX:
1. analyze-jira-requirements: fetch PROJ-123
2. planning-new-features: (attach UI screenshots)
3. Implement with TDD — mirror kiosk patterns in bootstrap.py
4. Gates: unittest + py_compile + rebuild ISO if needed + VM smoke
5. Do not mark complete until .py source exists (not only __pycache__)
```

---

## Gate A — After Jira analysis

Before planning, confirm the requirements doc includes:

- [ ] **Runtime context**: USB live boot / first disk boot / subsequent boot
- [ ] **UI type**: PyQt kiosk / CLI / web / none
- [ ] **Build artifact**: ISO / OVA / AMI / none (exact name pattern, e.g. `mdots-ha-*.iso`)
- [ ] **Entrypoint**: script that launches the feature (e.g. `start-installer-gui.sh` → `ha-ui.py`)
- [ ] Acceptance criteria include **first-boot** and **VM-observable** outcomes (not only unit-testable helpers)

**Block planning** if installer/kiosk feature lacks runtime context — infer from ticket + codebase or ask once.

---

## Gate B — After planning doc + implementation spec

Before editing source:

- [ ] Plan doc at `docs/plans/<feature-slug>.md`
- [ ] Implementation spec at `docs/plans/<feature-slug>-implementation.md`
- [ ] **Platform parity** table lists sibling files to mirror (grep same stack first)
- [ ] Test plan has three layers where applicable: unit + integration + **VM/boot smoke**
- [ ] Implementation spec lists **exact `.py` paths** that must exist when done
- [ ] Syntax validation run on spec snippets (`python3 -m py_compile`, `bash -n`, etc.)
- [ ] User approved plan **or** user explicitly said "implement now" in the same thread

---

## Gate C — Before first production edit

- [ ] `git status` — no target module exists **only** under `__pycache__/` without matching `.py`
- [ ] Read **entrypoint** (e.g. `apps/startup/start-installer-gui.sh`) and **one sibling UI** (`bootstrap.py`, `installer-gui.py`, `default-ip.py`)
- [ ] For PyQt kiosk: read repo rule `.cursor/rules/mdots-kiosk-ui.mdc`
- [ ] Confirm disk-image copy path (e.g. `apps/mdots/ha/` → `/scripts/ha/` via `build-disk-image.sh`)

---

## Gate D — During implementation (TDD slices)

Per slice: **RED** test → **GREEN** code → **REFACTOR** → run tests.

### Python (HA / mdots modules)

```bash
cd apps/mdots/ha   # or affected package
python3 -m unittest discover -s . -p 'test_*.py' -v
python3 -m py_compile <touched-files>.py
```

### Required contract tests for new PyQt kiosk apps

- [ ] stdout/stderr suppressed before PyQt imports (source-level test or grep contract)
- [ ] No `print()` on hot paths — log to `/var/log/*.log` only
- [ ] Widget init order: no `refresh_*` touching widgets before they are created in `init_ui`
- [ ] Network helpers: unit tests for no-default-route / fresh-boot interface pick
- [ ] Netplan: MAC-based match when static IP is configured

Do **not** mark implementation complete when only unit tests pass if Gate E applies.

---

## Gate E — Before "done"

- [ ] All modules from implementation spec exist as **source files** in the workspace
- [ ] Full unit test suite for touched packages passes
- [ ] `bash -n` on touched shell scripts

### If feature ships on ISO/OVA

- [ ] Rebuild artifact (example HA path):

```bash
cd build/
sudo ./docker-build-disk-image.sh --ha
sudo ./docker-build-parallel.sh --ha-only
```

- [ ] Note rebuilt ISO path and that it is **newer** than source edits

### VM smoke (installer / kiosk features)

- [ ] Run or document:

```bash
cd build/
./test-with-virtualbox.sh --ha 30GB   # or --iso variant without --ha
```

- [ ] Confirm: GUI visible, **no console scroll** behind UI, primary user path works
- [ ] Logs to verify on VM:
  - `/var/log/start-installer-gui.log`
  - Feature log (e.g. `/var/log/ha-operation.log`)

Use [systematic-debugging](../systematic-debugging/SKILL.md) if VM fails but unit tests pass.

---

## PyQt kiosk invariants (summary)

Full detail: `mdots-distro/.cursor/rules/mdots-kiosk-ui.mdc`

1. Suppress stdout/stderr at top of script (before PyQt imports)
2. Log to file only — never `print()` to TTY in production paths
3. Network: fallback interface when no default route (fresh boot)
4. Netplan: match by MAC, not interface name
5. `init_ui`: create all widgets before any handler that references them
6. Know runtime path on target system (`/scripts/...` vs repo path)

---

## Session handoff checklist

If stopping before Gate E:

- [ ] List which gates passed / failed
- [ ] List uncommitted files and whether ISO was rebuilt
- [ ] Warn if only `__pycache__` or docs exist without `.py` sources

---

## Rules

- **Gates are blocking** — do not skip because unit tests are green
- **Prefer incremental commits** when user asks to commit: `test:` → `feat:` → `refactor:`
- **No secrets** in plans, logs, or skill examples
- Update plan docs if implementation diverges from spec
