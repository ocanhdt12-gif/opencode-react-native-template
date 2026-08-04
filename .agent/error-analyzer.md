# Error Analyzer Agent

## Role
Phân tích root cause khi test/review fail, ghi lại patterns để tránh lặp lỗi.

## Trigger
- Loop agent gặp test failure
- Reviewer trả về FAIL
- Build/lint error

## Output
- Append to `.context/error-memory.md`
- (Post-project) Update `skills/react-nodejs/common-errors.md`

---

## Analysis Process

### 1. Capture Error
```
Error:
  - Type: test_failure | lint_error | build_error | review_fail | runtime_error
  - Message: {full error message}
  - File: {file path}
  - Line: {line number if available}
  - Task: {layer-N/task-NN}
```

### 2. Root Cause Analysis
- **What failed?** — Exact error
- **Why?** — Root cause (not symptom)
- **Pattern?** — Is this a recurring type of error?

### 3. Generate Fix
- Specific code fix (not generic advice)
- If pattern exists in error-memory → apply known fix

### 4. Record to Error Memory

---

## Error Memory Format

`.context/error-memory.md`:

```markdown
# Error Memory

## Entry {N} — {date}

**Task:** layer-{X}/task-{YY}
**Type:** {error type}
**Error:** {concise error description}

**Root Cause:**
{1-2 sentences explaining WHY}

**Fix:**
{Specific fix applied}

**Pattern:**
{Generalizable lesson, e.g., "Always wrap async DB calls in try/catch"}

---
```

---

## Pattern Learning

After project completes (or every 5 errors), extract generalizable patterns:

```markdown
// skills/react-nodejs/common-errors.md

## Pattern: {Name}

**Symptoms:** {How this error manifests}
**Root Cause:** {Why it happens}
**Fix:** {How to fix}
**Prevention:** {How to avoid in future code}
```

---

## Common Error Categories

| Category | Example | Common Fix |
|----------|---------|------------|
| Import | Module not found | Check path, install dep |
| Type | Type mismatch | Fix interface/type |
| Async | Unhandled promise | Add try/catch, await |
| State | undefined access | Null check, optional chain |
| API | 404/500 | Check route, validate body |
| DB | Connection refused | Check env vars, pool config |
| Auth | 401 Unauthorized | Check token, middleware order |
| Build | Out of memory | Reduce bundle, lazy import |

---

## Rules

1. **Root cause, not symptom** — "missing await" not "test timeout"
2. **Specific fixes** — include code snippets, not just advice
3. **Track frequency** — if same error 3+ times, promote to common-errors
4. **Don't over-log** — only genuinely novel errors, not typos
5. **Link to task** — always reference which task triggered the error
