# Implementation Spec: [Feature Name]

> Planning doc: [link to plan-template output]
> Status: draft | syntax-validated | implemented
> Date: [YYYY-MM-DD]

## Summary

[One paragraph: what this spec contains and how many files change.]

## Prerequisites

- [ ] Planning doc reviewed and approved
- [ ] Dependencies or migrations identified
- [ ] No blocking open questions

---

## File 1: `path/to/file.py`

**Change type:** add | modify | delete

**Description:** [What this change does and why.]

```python
# Full proposed code or diff-style snippet.
# Use "# ... existing code ..." for unchanged sections.

def new_handler():
    pass
```

**Notes:**
- [Any caveats, e.g. "register this route in the blueprint section around line 120"]

---

## File 2: `templates/example.html`

**Change type:** modify

**Description:** [What UI behavior changes.]

```html
<!-- Proposed template snippet -->
<div class="feature-block">
  {{ variable }}
</div>
```

---

## File 3: `tests/test_feature.py`

**Change type:** add

**Description:** [What behavior is covered.]

```python
def test_feature_happy_path(client):
    response = client.get("/api/example")
    assert response.status_code == 200
```

---

## Syntax Validation Log

| Snippet | Language | Check | Result |
|---------|----------|-------|--------|
| `file.py` block | python | `python -m py_compile` | pass / fail |
| `example.html` block | html | manual tag balance | pass / fail |

## Implementation Checklist

- [ ] Apply changes in the order listed above
- [ ] Run project test suite
- [ ] Manual smoke test per planning doc test plan
- [ ] Update planning doc status to `implemented`

## Deviations

[Leave empty during planning. Fill in during implementation if code diverges from spec.]
