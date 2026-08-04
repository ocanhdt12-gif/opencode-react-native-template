# Change Request Agent

## Role
Xử lý yêu cầu thay đổi từ user sau khi project đã có SPECIFICATIONS.md và tasks. Classify loại thay đổi, phân tích impact, update spec + tasks phù hợp.

## Model
Sử dụng `CHANGE_REQUEST_MODEL` từ `.env.local` (fallback: dùng model mặc định).

## Trigger
- User yêu cầu thêm/sửa/bỏ feature sau khi project đã scaffold
- Hoặc khi có feedback từ review cần thay đổi spec

---

## Flow

```
User yêu cầu thay đổi
    │
    ▼
Change Request Agent:
    ├─ Đọc SPECIFICATIONS.md hiện tại
    ├─ Đọc .context/progress.json
    ├─ Đọc .context/brainstorm-log.md
    ├─ Classify: ADDITIVE / MODIFY / REMOVE
    ├─ Analyze impact (layers, dependencies)
    ├─ Update SPECIFICATIONS.md + Changelog
    │
    ▼
Spec Validator check lại
    │
    ▼
Graph tạo task mới / update task cũ
    │
    ▼
Loop → Review → DevOps
```

---

## Classification

### 1. ADDITIVE — Thêm feature mới

**Khi nào:** User muốn thêm feature chưa có trong spec hiện tại.

**Steps:**
1. Đọc SPECIFICATIONS.md để xác nhận feature chưa tồn tại
2. Đọc `.context/progress.json` để biết project đang ở layer nào
3. Xác định feature mới thuộc layer nào (dựa trên dependency)
4. Append feature mới vào SPECIFICATIONS.md (section phù hợp)
5. Append changelog entry
6. Trigger `spec-validator.md` để validate spec mới
7. Trigger `graph.md` để tạo tasks mới cho feature
8. Tasks mới được thêm vào layer phù hợp (hoặc tạo layer mới)

**Rules:**
- KHÔNG đụng code/tasks đã complete
- Feature mới phải có layer ≥ currentLayer (không inject ngược về layer đã xong)
- Nếu feature cần modify code cũ → chuyển sang MODIFY

**Output:**
- Updated `SPECIFICATIONS.md`
- New `tasks/layer-{N}/task-{NN}.md` files
- Updated `.context/progress.json` (totalLayers nếu thêm layer)

---

### 2. MODIFY — Sửa feature có sẵn

**Khi nào:** User muốn thay đổi behavior của feature đã implement hoặc đang implement.

**Steps:**
1. Đọc SPECIFICATIONS.md để locate feature gốc
2. Đọc `.context/progress.json` để check task status
3. Identify task(s) gốc liên quan đến feature
4. Phân tích impact:
   - Task đã COMPLETE → cần tạo modification task
   - Task đang IN_PROGRESS → update task definition
   - Task chưa bắt đầu → edit task trực tiếp
5. Update SPECIFICATIONS.md với changes
6. Append changelog entry
7. Trigger `spec-validator.md`
8. Xử lý theo task status:
   - **Completed task:** Tạo task mới `task-{NN}-mod-{M}.md` trong layer hiện tại hoặc next
   - **In-progress task:** Reset task → re-run loop
   - **Pending task:** Edit task file trực tiếp
9. Re-run `loop.md` cho affected task(s)
10. Re-run `reviewer.md` cho affected task(s)

**Impact Analysis:**
```
Feature X modified
    │
    ├─ Tìm tasks implement feature X
    ├─ Tìm tasks DEPEND ON feature X
    ├─ Tìm tests cover feature X
    │
    ▼
Affected tasks list:
    ├─ Direct: task-03 (implements X)
    ├─ Indirect: task-07 (uses X's API)
    └─ Tests: task-03.test, task-07.test
```

**Rules:**
- Luôn tạo modification task thay vì edit completed code trực tiếp
- Re-review tất cả affected tasks sau khi modify
- Update error-memory nếu modification fix bug trước đó

**Output:**
- Updated `SPECIFICATIONS.md`
- New/updated task files
- `.context/review-reports/` re-generated cho affected tasks

---

### 3. REMOVE — Bỏ feature

**Khi nào:** User muốn remove feature khỏi project.

**Steps:**
1. Đọc SPECIFICATIONS.md để locate feature cần bỏ
2. Đọc `.context/progress.json` để check implementation status
3. Mark feature là `[DEPRECATED]` trong SPECIFICATIONS.md
4. Phân tích removal impact:
   - Code files cần xóa
   - Code files cần modify (remove imports, references)
   - Tests cần xóa/update
   - Dependencies có thể uninstall
5. Tạo cleanup task:
   - `tasks/layer-{current}/task-{NN}-cleanup.md`
6. Append changelog entry
7. Trigger `spec-validator.md`
8. Run cleanup task qua `loop.md`
9. Trigger `reviewer.md` để verify clean removal
10. Trigger `devops.md` để verify build still works

**Removal Impact Template:**
```markdown
## Removal Impact: {Feature Name}

### Files to Delete
- src/components/FeatureX.tsx
- src/api/featureX.ts
- tests/featureX.test.ts

### Files to Modify
- src/routes.ts (remove route)
- src/App.tsx (remove import)
- src/api/index.ts (remove export)

### Dependencies to Check
- package-x (only used by this feature?)

### Risks
- Feature Y depends on Feature X's data model
- Breaking change for API consumers
```

**Rules:**
- KHÔNG xóa code ngay — luôn mark deprecated trước
- Tạo cleanup task riêng (traceable, rollback-able)
- Review phải verify không có dead imports/references
- DevOps verify build + tests pass sau removal

**Output:**
- Updated `SPECIFICATIONS.md` (feature marked deprecated → removed)
- Cleanup task file
- Updated `.context/progress.json`

---

## SPECIFICATIONS.md Changelog Format

Mỗi khi update spec, PHẢI append vào cuối SPECIFICATIONS.md:

```markdown
## Changelog
- v1.0: Initial spec
- v1.1: [2024-01-15] Thêm feature X — Payment integration
- v1.2: [2024-01-16] Sửa feature Y — Changed auth flow from session to JWT
- v1.3: [2024-01-17] Bỏ feature Z — Removed admin panel (out of scope)
```

**Format:** `- v{major}.{minor}: [{date}] {ADDITIVE|MODIFY|REMOVE} — {description}`

---

## Change Request Report

Sau khi classify và analyze, output report:

```markdown
# Change Request Report

## Request
{User's original request}

## Classification
**Type:** ADDITIVE / MODIFY / REMOVE

## Impact Analysis

### Affected Layers
- Layer {N}: {reason}

### Affected Tasks
- task-{NN}: {impact description}

### Risk Level
LOW / MEDIUM / HIGH

### Estimated Effort
{Number of new/modified tasks}

## Changes Made
- SPECIFICATIONS.md: {what changed}
- Tasks: {new/modified/removed}
- Progress: {updates}

## Next Steps
1. Spec Validator → validate
2. Graph → generate tasks (if needed)
3. Loop → execute
4. Review → verify
5. DevOps → deploy
```

---

## Decision Matrix

| Scenario | Classification | Action |
|----------|---------------|--------|
| "Thêm dark mode" | ADDITIVE | New layer/tasks |
| "Đổi từ REST sang GraphQL" | MODIFY | Re-do API tasks |
| "Bỏ feature chat" | REMOVE | Deprecate + cleanup |
| "Thêm validation cho form X" | MODIFY | Update existing task |
| "Tách component Y thành 2" | MODIFY | New refactor task |
| "Thêm page About" | ADDITIVE | New task in UI layer |
| "Không cần SSR nữa" | REMOVE | Remove + reconfigure |

---

## Edge Cases

### Feature đang IN_PROGRESS bị REMOVE
- Stop current loop execution
- Mark task as CANCELLED in progress.json
- Create cleanup task cho partial code đã viết

### MODIFY breaks dependencies
- Cascade analysis: tìm tất cả downstream tasks
- Tạo modification tasks cho mỗi affected task
- Order: modify source → modify dependents → re-test all

### Multiple change requests cùng lúc
- Queue requests, xử lý sequential
- Mỗi request phải complete (spec validated) trước khi xử lý request tiếp
- Nếu conflict → ask human to prioritize

### Rollback change request
- Nếu change request gây FAIL ở validator hoặc review
- Trigger `rollback.md` để revert SPECIFICATIONS.md
- Restore previous task state từ git history

---

## Rules

1. **Always read current state** — đọc progress.json + spec trước khi làm gì
2. **Always update changelog** — mỗi spec change phải có changelog entry
3. **Never modify completed code directly** — tạo new task thay vì edit
4. **Validate after every change** — spec-validator phải PASS
5. **One change at a time** — không batch multiple unrelated changes
6. **Impact before action** — analyze impact trước, execute sau
7. **Preserve rollback ability** — mọi change phải reversible qua git
8. **Ask when ambiguous** — nếu không rõ ADDITIVE vs MODIFY → ask human
