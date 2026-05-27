---
name: mdots-feature-delivery
description: >-
  End-to-end MDOTS feature delivery: Jira requirements → planning spec → TDD
  implementation → delivery gates (unit tests, ISO rebuild, VM smoke). Use for
  any mdots-distro feature (HA, installer UI, shell tooling, nginx, etc.), not
  only HA. Use when the user gives a delivery todo list or asks to ship a feature.
user-invocable: true
---

# MDOTS feature delivery (Jira → plan → ship)

**Scope:** Any feature in `mdots-distro` — HA UI, standard installer, Python helpers,
shell scripts, nginx configs, boot routing, etc. HA appears only as one example below.

Orchestrates sibling skills with **mandatory gates**. Skip gates that do not apply
(see **Gate applicability**). Do not mark complete until every **applicable** gate passes.

## Related skills

| Step | Skill | Path |
|------|-------|------|
| 1 | Analyze requirements | [../analyze-jira-requirements/SKILL.md](../analyze-jira-requirements/SKILL.md) |
| 2 | Plan + implementation spec | [../planning-new-features/SKILL.md](../planning-new-features/SKILL.md) |
| 3 | TDD during implement | [../tdd-workflow/SKILL.md](../tdd-workflow/SKILL.md) |
| 4 | Boot / VM bugs | [../systematic-debugging/SKILL.md](../systematic-debugging/SKILL.md) |

---

## Delivery profile (derive once, reuse in Gates C–F)

After Gate A/B, record this in the implementation spec (`docs/plans/<slug>-implementation.md`).
**Do not assume HA paths** — fill from ticket + codebase grep.

| Field | How to discover | Examples |
|-------|-----------------|----------|
| `PKG` | Repo directory with production + test code | `apps/mdots/ha`, `apps/mdots`, `apps/startup` |
| `ENTRY` | Boot/script that runs the feature (if any) | `apps/mdots/ha/ha-ui.py`, `apps/startup/installer-gui.py` |
| `RUNTIME_ENTRY` | Path on target appliance | `/scripts/ha/ha-ui.py`, `/scripts/installer-gui.py` |
| `UI_TYPE` | From requirements | `pyqt-kiosk`, `cli`, `shell-only`, `none` |
| `BUILD_VARIANT` | ISO/OVA variant flags | `standard`, `ha` |
| `BUILD_CMD` | From `build/docker-build-*.sh` help | see Gate E table |
| `VM_SMOKE_CMD` | From `build/test-with-virtualbox.sh` | see Gate E table |
| `LOG_PATHS` | Feature + boot logs on VM | `/var/log/ha-operation.log`, `/var/log/start-installer-gui.log` |

### Repo → runtime copy paths (all features)

Read `build/build-disk-image.sh` (installer copy step). Default mapping:

| Repo | Target disk |
|------|-------------|
| `apps/startup/*` | `/scripts/*` |
| `apps/mdots/*` | `/scripts/*` (subdir names preserved: `apps/mdots/ha/x` → `/scripts/ha/x`) |

Gate C checklist: **confirm each touched repo path** has the expected runtime path
under `/scripts/...` after copy — do not hardcode `ha/` unless the feature lives there.

### Example profiles

| Feature | PKG | ENTRY | BUILD_VARIANT | VM smoke |
|---------|-----|-------|---------------|----------|
| EM HA UI | `apps/mdots/ha` | `apps/mdots/ha/ha-ui.py` | `ha` | `./test-with-virtualbox.sh --ha 30GB` |
| Standard installer GUI | `apps/startup` | `apps/startup/installer-gui.py` | `standard` | `./test-with-virtualbox.sh 30GB` |
| Network helper only | `apps/mdots` | `apps/mdots/default-ip.py` | `standard` | optional if no boot UI change |
| Shell-only (install script) | `apps/mdots/ha` | — (no Python entry) | `ha` | boot + run script on VM |

---

## Gate applicability

| Gate | Always? | When skipped |
|------|---------|--------------|
| A — Jira runtime context | Yes | — |
| B — Plan + spec | Yes | — |
| C — Pre-edit safety | Yes | — |
| D — TDD | Yes | — |
| F — Source verification | If shipping on ISO/OVA **or** Python in `PKG` | Pure docs-only change |
| F2 — Kiosk entry checks | Only if `UI_TYPE=pyqt-kiosk` | CLI / shell-only |
| E — ISO rebuild + VM smoke | If feature affects boot, installer, or appliance runtime | Host-only / docs-only |

---

## User todo template

When the user sends a delivery todo list, run in order:

1. **Analyze Jira** — output must include **Runtime context** (see analyze-jira skill).
2. **Plan feature** — write **delivery profile** into implementation spec.
3. **Implement with TDD** — from `docs/plans/<slug>-implementation.md`; mirror sibling files (Gate C).
4. **Run applicable gates A–F** — report pass/fail / N/A for each in your summary.

Suggested single user message:

```text
Delivery workflow for OTSECURITY-XXXX:
1. analyze-jira-requirements: fetch PROJ-123
2. planning-new-features: (attach UI screenshots if any)
3. Implement with TDD — mirror sibling patterns from delivery profile
4. Run Gate F (PKG/ENTRY from spec) + unittest + rebuild ISO + VM smoke if applicable
5. Real .py source only — never pyc shims
```

---

## Gate A — After Jira analysis

Before planning, confirm the requirements doc includes:

- [ ] **Runtime context**: USB live boot / first disk boot / subsequent boot
- [ ] **UI type**: PyQt kiosk / CLI / web / shell-only / none
- [ ] **Build artifact**: ISO / OVA / AMI / none (name pattern, e.g. `mdots-installer-*.iso`, `mdots-ha-*.iso`)
- [ ] **Entrypoint**: script that launches the feature (grep `apps/startup/`, variant routing in `start-installer-gui.sh`)
- [ ] Acceptance criteria include **first-boot** and **VM-observable** outcomes when runtime is involved

**Block planning** if installer/kiosk feature lacks runtime context — infer from ticket + codebase or ask once.

---

## Gate B — After planning doc + implementation spec

Before editing source:

- [ ] Plan doc at `docs/plans/<feature-slug>.md`
- [ ] Implementation spec at `docs/plans/<feature-slug>-implementation.md`
- [ ] **Delivery profile** table filled (PKG, ENTRY, RUNTIME_ENTRY, UI_TYPE, BUILD_VARIANT, BUILD_CMD, VM_SMOKE_CMD, LOG_PATHS)
- [ ] **Platform parity** table lists sibling files to mirror (grep same stack first)
- [ ] Test plan: unit + integration + **VM/boot smoke** when ISO/boot involved
- [ ] Implementation spec lists **exact paths** (repo + runtime) that must exist when done
- [ ] Syntax validation on spec snippets (`python3 -m py_compile`, `bash -n`, etc.)
- [ ] User approved plan **or** user explicitly said "implement now" in the same thread

---

## Gate C — Before first production edit

- [ ] `git status` — no target module exists **only** under `__pycache__/` without matching `.py`
- [ ] **No bytecode shims** — no `.py` that loads `__pycache__/orig/*.pyc` via `importlib` (ISO deletes `__pycache__`; see `build/build-disk-image.sh`)
- [ ] Read **entrypoint** from delivery profile (and `apps/startup/start-installer-gui.sh` if boot-routed)
- [ ] Read **one sibling** in the same UI/stack (e.g. `bootstrap.py`, `installer-gui.py`, `default-ip.py`, or existing feature under `apps/mdots/`)
- [ ] If `UI_TYPE=pyqt-kiosk`: read `.cursor/rules/mdots-kiosk-ui.mdc`
- [ ] Confirm **repo → runtime copy** for every touched path (see delivery profile; grep `build/build-disk-image.sh`)

**Forbidden pattern (will break on ISO):**

```python
_pyc = _py.parent / "__pycache__" / "orig" / "<module>.cpython-312.pyc"
spec = importlib.util.spec_from_file_location(..., _pyc)
```

**Never** replace missing source with pyc loaders — restore real `.py` from git history instead.

---

## Gate D — During implementation (TDD slices)

Per slice: **RED** test → **GREEN** code → **REFACTOR** → run tests.

Use `PKG` from the delivery profile:

```bash
cd "$PKG"
python3 -m unittest discover -s . -p 'test_*.py' -v
python3 -m py_compile *.py    # or list touched modules
bash -n *.sh                  # if shell scripts in PKG
```

### Contract tests when `UI_TYPE=pyqt-kiosk`

- [ ] stdout/stderr suppressed before PyQt imports
- [ ] No `print()` on hot paths — log to `/var/log/*.log` only
- [ ] Widget init order: no `refresh_*` before widgets exist in `init_ui`
- [ ] Network helpers: tests for no-default-route / fresh-boot interface pick (if feature touches network)
- [ ] Netplan: MAC-based match when static IP is configured (if applicable)
- [ ] Tests are **real** `unittest` modules — not shims loading `test_*.cpython-*.pyc`

Do **not** mark implementation complete when only unit tests pass if Gates E–F apply.

---

## Gate F — Source verification (agent runs before ISO rebuild)

Agent **must run** applicable checks and report pass/fail. No repo support scripts.

Use `PKG` and `ENTRY` from the **delivery profile** (not hardcoded HA paths).

### F1 — No bytecode shims in production code

```bash
rg '__pycache__/orig|spec_from_file_location.*\.pyc' "$PKG" --glob '*.py' --glob '!test_*'
# Must return no matches
```

### F2 — Kiosk entrypoint is real source (only if `UI_TYPE=pyqt-kiosk`)

```bash
wc -l "$ENTRY"                    # expect substantial size vs thin shim (~50 lines)
rg 'sys.stdout = open\(os.devnull' "$ENTRY"
rg 'from PyQt5' "$ENTRY"
# devnull line must appear BEFORE PyQt5 import in file order
```

Skip F2 with reason if no PyQt entry (shell-only, nginx-only, etc.).

### F3 — Tests are real Python

```bash
rg '_pyc = _py.parent / "__pycache__" / "orig"' "$PKG"/test_*.py
# Must return no matches (or no test files yet)
```

### F4 — Compile and unit tests

```bash
cd "$PKG" && python3 -m py_compile *.py && python3 -m unittest discover -s . -p 'test_*.py' -v
```

### F5 — Build strips pycache (read-only confirm)

Read `build/build-disk-image.sh` — confirm it removes `__pycache__`. Shims cannot work on ISO.

**Do not rebuild ISO until F1–F4 pass (or are N/A with documented reason).**

---

## Gate E — Before "done"

- [ ] **Gate F (F1–F4) all passed or N/A** — agent reported results
- [ ] All modules from implementation spec exist as **real source files** (not shims)
- [ ] `bash -n` on touched shell scripts

### If feature ships on ISO/OVA (`BUILD_VARIANT` from profile)

| BUILD_VARIANT | Disk image | Parallel ISO | VM smoke |
|---------------|------------|--------------|----------|
| `standard` | `sudo ./docker-build-disk-image.sh` | `sudo ./docker-build-parallel.sh --iso-only` | `./test-with-virtualbox.sh 30GB` |
| `ha` | `sudo ./docker-build-disk-image.sh --ha` | `sudo ./docker-build-parallel.sh --ha-only` | `./test-with-virtualbox.sh --ha 30GB` |

All commands run from `build/`. Note rebuilt artifact path and that it is **newer** than source edits.

### VM smoke (boot / installer / kiosk features)

- [ ] Run `VM_SMOKE_CMD` from delivery profile
- [ ] Confirm observable outcomes from acceptance criteria (GUI visible, no console bleed, script runs, etc.)
- [ ] Check `LOG_PATHS` from profile on the VM

Use [systematic-debugging](../systematic-debugging/SKILL.md) if VM fails but unit tests pass.

---

## PyQt kiosk invariants (when `UI_TYPE=pyqt-kiosk` only)

Full detail: `.cursor/rules/mdots-kiosk-ui.mdc`

1. Suppress stdout/stderr before PyQt imports
2. Log to file only — never `print()` to TTY in production paths
3. Network: fallback interface when no default route (fresh boot)
4. Netplan: match by MAC, not interface name
5. `init_ui`: create all widgets before handlers that reference them
6. Know runtime path on target (`/scripts/...` vs repo path — from delivery profile)

---

## Session handoff checklist

If stopping before Gate E:

- [ ] List which gates passed / failed / N/A (include F1–F4)
- [ ] Paste delivery profile (PKG, ENTRY, BUILD_VARIANT)
- [ ] List uncommitted files and whether ISO was rebuilt
- [ ] Warn if only `__pycache__` or docs exist without `.py` sources

---

## Rules

- **Gates are blocking** for applicable scope — do not skip because unit tests are green
- **Derive paths from profile + codebase** — never assume `apps/mdots/ha/` unless the feature is HA
- **Never ship pyc shims** for ISO/appliance code — `build-disk-image.sh` strips `__pycache__`
- **Never create bytecode recovery shims** — restore from `git log` / `git show <commit>:path`
- **Prefer incremental commits** when user asks to commit: `test:` → `feat:` → `refactor:`
- **No secrets** in plans, logs, or skill examples
- Update plan docs if implementation diverges from spec
