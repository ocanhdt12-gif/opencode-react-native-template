# Rollback Agent — Git Checkpoint Strategy

> ⚠️ **Maintenance mode override:** state dùng `features[]`/`bugs[]`; **KHÔNG** ghi/đọc `currentLayer` khi ở maintenance mode; **cấm push thẳng `forbidden_branch`** (mặc định `main`); branch/push model theo `.agent/FEATURE_WORKFLOW.md` §6 (default staging-direct). Workflow hiện hành: `.agent/FEATURE_WORKFLOW.md` + `AGENTS.md` (ưu tiên). Phần greenfield dưới đây chỉ dùng khi build từ đầu.

> ⚠️ **Maintenance push override:** Maintenance mode overrides all literal push examples below.
> Do NOT push `main`/`develop` from these examples. Auto-push after PASS review must use
> branch model in `.agent/FEATURE_WORKFLOW.md` §6 (default: `git push origin <target_branch>` only from `target_branch`).
> Bare `git push` is denied by `opencode.jsonc`. Production/main promotion requires explicit
> human approval outside normal maintenance workflow.

## Role
Manage git checkpoints để có thể rollback khi task/layer fail.

## Trigger
- Automated: sau mỗi task PASS (commit) và mỗi layer PASS (tag)
- Manual: khi cần revert về state trước

---

## Checkpoint Strategy

### After Each Task PASS
```bash
git add {task-related files}
git commit -m "feat(layer-{N}): task-{NN} {description}"
```

### After Each Layer PASS (all tasks + review)
> ⚠️ **Maintenance mode override:** KHÔNG push `main`/`forbidden_branch` (xem `AGENTS.md`).
```bash
git tag "layer-{N}-done" -m "Layer {N} complete: {layer description}"
git push origin main --tags   # ❌ LEGACY greenfield — bị override ở maintenance mode
```

### Before Risky Operations
```bash
# Create safety checkpoint before experimental changes
git stash push -m "safety-checkpoint-before-{description}"
```

---

## Rollback Procedures

### Rollback Single Task (last commit)
```bash
# Undo last commit but keep changes
git reset --soft HEAD~1

# Or discard changes entirely
git reset --hard HEAD~1
```

### Rollback Entire Layer
```bash
# Find the tag for previous layer
git log --oneline --tags

# Reset to previous layer checkpoint
git reset --hard layer-{N-1}-done

# Force push (careful!) — ❌ bị CHẶN ở maintenance mode (permission.bash deny --force)
git push origin main --force
```

### Rollback to Any Checkpoint
```bash
# List all tags
git tag -l "layer-*"

# Reset to specific checkpoint
git reset --hard {tag-name}
```

---

## When to Rollback

| Situation | Action |
|-----------|--------|
| Task fails tests after 3 retries | Reset that task's commit |
| Layer review fails fundamentally | Consider reset to prev layer tag |
| Wrong architecture decision | Reset to decision point + re-plan |
| Corrupted state | Reset to last known good tag |

---

## Safety Rules

1. **Never force push without human approval**
2. **Always tag before destructive operations**
3. **Prefer `git revert` over `git reset` on shared branches**
4. **Keep local backup branch before reset:**
   ```bash
   git branch backup-before-rollback
   git reset --hard {target}
   ```
5. **Update progress.json after rollback** — state must match code
6. **Notify human after any rollback** — explain what was rolled back and why

---

## Tag Convention

```
layer-0-done     # Infrastructure complete
layer-1-done     # Core backend complete
layer-2-done     # Core frontend complete
layer-N-done     # Layer N complete
v1.0.0-mvp       # MVP release
v1.0.0           # Production release
```
