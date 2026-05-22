# Feature: [Feature Name]

> Status: draft | approved | implemented
> Author: [name or agent]
> Date: [YYYY-MM-DD]

## Problem

[What problem or gap exists today? Who is affected?]

## Goal

[One or two sentences describing the desired outcome.]

## Scope

### In scope

- [ ] [Concrete deliverable 1]
- [ ] [Concrete deliverable 2]

### Non-goals

- [Explicitly excluded item 1]
- [Explicitly excluded item 2]

## Design

### Overview

[High-level approach — how the feature fits into the existing system.]

### Data flow

```
[User action] → [Handler] → [Storage/API] → [Response/UI]
```

### API changes

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/example` | [purpose] |

### UI changes

[Describe template/page changes, new components, user interactions.]

### Database / storage

[New tables, columns, migrations, or config keys — or "none".]

## Files Affected

| File | Change |
|------|--------|
| `app.py` | [brief description] |
| `templates/index.html` | [brief description] |

## Risks and Edge Cases

| Risk | Mitigation |
|------|------------|
| [e.g. breaking existing route] | [e.g. add new route, keep old one] |

## Test Plan

- [ ] [Manual test step 1]
- [ ] [Manual test step 2]
- [ ] [Automated test: describe what to assert]

## Open Questions

- [ ] [Decision needed from user or team]

## Implementation Order

1. [First change — e.g. backend route]
2. [Second change — e.g. template]
3. [Tests]
