# Blackboard Agent — State Management

## Role
Quản lý shared state cho toàn bộ hệ thống agents. File `.context/progress.json` là source of truth cho trạng thái project.

## Trigger
- Mỗi khi bắt đầu session mới (resume)
- Mỗi khi agent cần đọc/ghi state

---

## State Schema

### `.context/progress.json`

```json
{
  "projectName": "string",
  "currentLayer": 0,
  "totalLayers": 0,
  "completedTasks": ["layer-0/task-01", "layer-0/task-02"],
  "inProgressTask": "layer-1/task-01" | null,
  "blockedTasks": [
    {
      "task": "layer-1/task-03",
      "reason": "External API not available",
      "since": "ISO timestamp"
    }
  ],
  "decisions": [
    {
      "date": "ISO timestamp",
      "decision": "Use JWT instead of session auth",
      "reason": "Stateless, better for scaling"
    }
  ],
  "lastUpdated": "ISO timestamp"
}
```

---

## Resume Instructions

Khi agent bắt đầu session mới:

1. **Read** `.context/progress.json`
2. **Determine state:**
   - `inProgressTask != null` → Task đang dở, tiếp tục từ loop.md
   - `inProgressTask == null` → Check next task trong current layer
   - All tasks in current layer done → Check review status → Unlock next layer
   - All layers done → Project complete
3. **Read context:**
   - Current task file (nếu có)
   - `.context/error-memory.md` (recent errors)
   - `.context/decisions.md` (past decisions for consistency)
4. **Continue execution** theo trạng thái

---

## State Transitions

```
[START] → brainstorm → graph → loop ←→ review
                                 ↓
                          [LAYER COMPLETE]
                                 ↓
                          next layer / [DONE]
```

### Valid State Updates

| Event | Update |
|-------|--------|
| Task started | `inProgressTask = "layer-X/task-YY"` |
| Task PASS | Move to `completedTasks`, clear `inProgressTask` |
| Task BLOCKED | Move to `blockedTasks` with reason |
| Layer complete | `currentLayer++` |
| Decision made | Append to `decisions[]` |

---

## Rules

1. **Atomic updates** — đọc file → modify → ghi lại toàn bộ
2. **Always update `lastUpdated`** khi modify progress.json
3. **Never delete completedTasks** — chỉ append
4. **Decisions are permanent** — ghi lại lý do để future agents hiểu context
5. **blockedTasks vẫn count** — layer không complete nếu còn blocked tasks (trừ khi human approve skip)
