# Cursor Skills — Quick Reference

Skills are task-specific playbooks the agent reads when your request matches their description. Rules in `.cursor/rules/` apply automatically in the background; skills provide deeper guidance for specific work.

## Skills Cheat Sheet

| Your work | Skill | Example prompts |
|---|---|---|
| CI failures, PRs, workflow runs | **github-ops** | "Why did CI Smoke Test fail on mdots-distro?", "Check PR #99 status", "List recent workflow runs", "Is this branch ready to merge?" |
| Debugging build / smoke test issues | **systematic-debugging** | "Debug why Zeek smoke test fails", "Reproduce the Build All phase 2 error", "Isolate whether this is Redis or GitHub API" |
| Agent keeps retrying the same failed fix | **no-retry-loop** | "We're going in circles — don't retry old approaches", "CI still failing after 3 fixes, try a new angle", "Avoid repeating solutions that already failed" |
| Code review before merge | **reviewing-code** | "Review my changes to app.py", "Review this Flask route for bugs", "Check this PR diff before I merge" |
| New features / plans / specs | **planning-new-features** | "Plan CI smoke test gate for Build All", "Write a spec before implementing X in app.py", "Design refactor plan for app.py" |
| Writing tests (pytest, coverage) | **tdd-workflow** | "Add pytest tests for Build All logic", "Write tests first for this Flask endpoint", "Fix this bug with TDD" |
| Docker builds / containers | **docker-patterns** | "Review Dockerfile.zeek-plugin-builder", "Fix docker_build_zeek.sh networking issue", "Improve Zeek plugin build container setup" |

## By Repo Area

| Area | Skill(s) to lean on |
|---|---|
| `app.py` — Flask routes, Redis, GitHub API | **systematic-debugging**, **reviewing-code**, **tdd-workflow** |
| `templates/index.html` — UI / Build All JS | **reviewing-code**, **planning-new-features** |
| `mdots-distro/.github/workflows/` — CI YAML | **github-ops**, **systematic-debugging** |
| `scripts/debug-network-sensor-zeekctl.sh` | **systematic-debugging**, **docker-patterns** |
| `docker_build_zeek.sh`, `Dockerfile.zeek-plugin-builder` | **docker-patterns**, **systematic-debugging** |
| New feature in `docs/plans/` | **planning-new-features** |
| Git commit / PR / release | **github-ops** (+ `common-git-workflow` rule) |

## Example Session Openers

```text
# github-ops
Check the latest CI Smoke Test run on opswat-eng/mdots-distro development branch

# systematic-debugging
Build All phase 2 fails after smoke test — help me reproduce and isolate the root cause

# no-retry-loop
The agent already tried restarting Redis and re-running the workflow — don't repeat those, find a new approach

# reviewing-code
Review the changes around line 2969 in app.py before I push

# planning-new-features
Plan how to split app.py into smaller modules — spec first, no code yet

# tdd-workflow
Add pytest tests for the zeek inventory helpers in app.py — tests first

# docker-patterns
Why does the Zeek plugin Docker build fail? Review docker_build_zeek.sh and the Dockerfile
```

## Notes

- **Rules vs skills:** `.cursor/rules/` (security, testing, coding-style) apply automatically. Skills are loaded on demand when the task matches.
- **User-invocable skills:** `planning-new-features` and `systematic-debugging` can be invoked directly if your Cursor build supports `/skill-name`.
- **no-retry-loop:** Has `disable-model-invocation: true` — mention "don't retry old solutions" or "avoid going in circles" in your prompt to activate it.

## Skill Index

| Skill | Path |
|---|---|
| docker-patterns | [docker-patterns/SKILL.md](docker-patterns/SKILL.md) |
| github-ops | [github-ops/SKILL.md](github-ops/SKILL.md) |
| no-retry-loop | [no-retry-loop/SKILL.md](no-retry-loop/SKILL.md) |
| planning-new-features | [planning-new-features/SKILL.md](planning-new-features/SKILL.md) |
| reviewing-code | [reviewing-code/SKILL.md](reviewing-code/SKILL.md) |
| systematic-debugging | [systematic-debugging/SKILL.md](systematic-debugging/SKILL.md) |
| tdd-workflow | [tdd-workflow/SKILL.md](tdd-workflow/SKILL.md) |
