# FEATURE_WORKFLOW.md — Maintenance entry point (bug · feature · update)

> Entry point cho **mọi request sau khi project đã tồn tại** (maintenance mode).
> Greenfield build (từ BRIEF/spec tới deploy lần đầu) → `AGENT.md`.
> Luật cứng/route nhanh → `AGENTS.md`. Giá trị project → `.agent/PROJECT_PROFILE.md`.

## 0. Precedence

1. `AGENTS.md` — luôn thắng.
2. `.agent/FEATURE_WORKFLOW.md` (file này).
3. `.agent/PROJECT_PROFILE.md` — giá trị cụ thể (branch, package manager, lệnh check, model).
4. Các file `.agent/*.md` khác (greenfield) — chỉ đọc phần không bị override.
   Mọi file legacy có dòng `Maintenance mode override:` ở đầu → phần bị override KHÔNG áp dụng.

> ⚠️ Nếu file legacy ghi `git push origin main --tags` hoặc ghi `currentLayer` vào state
> → **bỏ qua**. Ở maintenance mode: cấm push thẳng `forbidden_branch`; state dùng `features[]`/`bugs[]`.

---

## 1. Router

| Intent | Route |
|---|---|
| "fix bug", "lỗi", "broken", regression, crash (bug đã biết) | **§2 Bug workflow** → `/bug` |
| "soi/kiểm tra màn", "cảm giác nhiều lỗi nhưng không rõ" | **§2b Bug discovery (sweep)** → `/bug-check` — read-only |
| "thêm/sửa/bỏ/xóa tính năng", đổi behavior | **§3 Change Request workflow** → `/feature` |
| "implement feature" (đã có spec/task) | gọi subagent `builder` theo task file |
| "review", "check", "soát" | gọi subagent `reviewer` — không tự sửa |
| hỏi / điều tra | research-only — không edit tới khi user yêu cầu fix |

Không rõ intent → hỏi 1 câu ngắn. **Không tự phân loại thành "chắc là bug nhỏ, sửa luôn".**

---

## 2. Bug workflow

```
Triage → Reproduce → Root cause → Task → Builder → Reviewer PASS
   → Doc Impact/Reconcile → progress.json → commit current branch (= target_branch mặc định)
   → (push current branch nếu được phép)
```

### 2.1 Triage (bắt buộc trước khi sửa)
- Xác định: **màn hình/module**, **bước tái hiện**, **expected vs actual**, **role/vai trò**, **môi trường**.
- Thiếu bất kỳ mục nào → **hỏi ngắn 1 lần** (gom câu hỏi), KHÔNG tự giả định.
- Phân loại mức độ: `blocker` / `high` / `medium` / `low`.

### 2.2 Reproduce
- Dựng lại đúng điều kiện. Nếu không reproduce được → ghi nhận, hỏi thêm, **không sửa mò**.
- Chạy verify command thật từ `.agent/PROJECT_PROFILE.md` (`lint_command`, `typecheck_command`,
  `test_command`, `build_command` — mobile dùng alias generic, không có split web/api).

### 2.3 Root cause (Iron Law)
- **KHÔNG fix khi chưa có root cause.** Đọc `skills/superpowers/systematic-debugging.md`.
- Ghi root cause + evidence (log/stack/trace) vào bug task.
- ≥3 lần fix fail → set status `architecture_review_needed`, nghi ngờ **kiến trúc**, dừng lại, báo human. Không thử fix #4.

### 2.4 Task
- Bug **1 dòng, rõ ràng, không risk** → có thể sửa trực tiếp (vẫn phải update progress nếu đổi trạng thái bug và ghi `Repro-Verification` trong commit body).
- Còn lại → tạo `tasks/bug-<slug>/phase-<N>-task-<NN>.md` (format §5).
- Task bug bắt buộc ghi `Classification / Risk`: severity (`blocker|high|medium|low`), scope,
  root cause category, expected review level, blast radius, doc impact, decision impact.

### 2.5 Builder → Reviewer
- Gọi subagent `builder` (hoặc `builder-strong` — xem §7) implement + test.
- Gọi subagent `reviewer` kiểm tra **độc lập** (không sửa source; chỉ ghi report scoped). FAIL → trả lại builder, **không** đóng bug.
- Regression: **phải có test tái hiện bug fail trước fix**, pass sau fix (test-first).
- **Chỉ khi reviewer PASS** mới đi tiếp bước 2.7–2.9. FAIL → không update progress là "done", không commit/push.

### 2.6 Nhánh "đầu vào là danh sách bug"
Áp dụng cả khi input đến từ `/bug-check` hoặc user nói "fix tất cả defect".

1. **KHÔNG gọi Builder ngay**.
2. **Tách mỗi bug thành task riêng** `tasks/bug-<slug>/`.
3. Tóm tắt số lượng defect, severity, root cause nghi ngờ, file cần sửa.
4. Đề xuất thứ tự xử lý (blocker/high trước), xử lý **tuần tự**.
5. Nêu rõ bug nào gộp vì **cùng root cause**; ngoài trường hợp đó không gộp nhiều bug vào 1 diff.
6. **DỪNG hỏi user xác nhận** trước khi gọi Builder.

Chỉ bỏ checkpoint nếu prompt có đúng một trong các cụm: `auto proceed`, `khỏi hỏi lại`,
`tự xử lý hết không cần hỏi`.

### 2.7 Progress (bắt buộc)
- Cập nhật `.context/progress.json` ngay khi bug đổi trạng thái:
  thêm/cập nhật entry trong `bugs[]` (`status: triaged → reproducing → root_caused → fixing → review → done|blocked|architecture_review_needed`),
  set `activeWorkItem`.
- Sau Reviewer PASS, xác định Doc Impact & Reconcile (§6) trước khi đóng bug; không impact → ghi `no doc impact`.
- Set `done` sau khi reviewer PASS, Doc Impact/Reconcile đã xong hoặc ghi `no doc impact`, và report đúng tên tồn tại trong `.context/review-reports/` (§5). Trạng thái `done`/progress/doc-impact này phải nằm trong close-out commit; progress/task file không cần biết SHA của commit đang được tạo.

### 2.8 Commit / push (commit-first)
- Sau khi task/bug/phase PASS review + close-out + cập nhật `.context/progress.json`, phải commit lên branch hiện tại theo convention bên dưới.
- **1 task = 1 commit**, trừ khi có lý do rõ ràng như shared file interleave nhiều scope; ghi lý do trong commit body hoặc report.
- Commit là source of truth cho: changed files, timestamp, SHA, rollback point.
- Task file là source of truth cho: root cause, repro/evidence, residual risk, doc impact/reconcile, verification summary.
- `.context/progress.json` là source of truth cho: current status, active/completed phase/task, reviewer result/report path.
- Trước commit: chạy `git status` + `git diff` để chắc không lẫn file ngoài scope task.
- Chỉ stage file thuộc task hiện tại; không stage dirty cũ ngoài scope.
- Working tree còn việc khác đang dở → không gộp vào commit task hiện tại.
- Nếu file shared bị interleave nhiều scope (schema/service/docs) khiến tách commit không an toàn
  → cho phép 1 combined batch commit cho đúng epic đó, ghi rõ lý do; không cố partial-staging gây hỏng build.
- Commit subject convention:
  - `feat(feature-<slug>): phase-<N>-task-<NN> <summary>`
  - `fix(bug-<slug>): phase-<N>-task-<NN> <summary>`
  - Workflow/template/config: `chore(workflow): <summary>` hoặc `docs(workflow): <summary>`
- Commit body nên có trailer:
  ```
  Task: tasks/<feature-or-bug-slug>/phase-<N>-task-<NN>.md
  Review: <FAST|NORMAL|STRICT> PASS
  Review-Report: .context/review-reports/<report>.md
  Tests: <command> = PASS
  Migration: <none|migration-name SAFE_ADDITIVE|migration-name HIGH_RISK>
  Doc-Impact: <none|API_SPEC|ERD|DESIGN|GAPS>
  ```
- Với one-line obvious bug không tạo task file, commit body phải có:
  ```
  Repro-Verification: <short evidence of root cause + expected/actual>
  ```
- Branch model: default **staging-direct** nghĩa là commit trên current branch khi current branch = `target_branch` và push bằng `git push origin <target_branch>`; nếu user yêu cầu feature branch thì commit/push chính current feature branch bằng `git push origin <current-branch>` và chỉ mở PR khi user yêu cầu rõ.
- Tuyệt đối không push `forbidden_branch`; không `--force`/`-f` (đã chặn ở `opencode.jsonc`).
- Push chỉ khi user yêu cầu rõ hoặc `auto_push_after_pass: true` trong `.agent/PROJECT_PROFILE.md`; Reviewer FAIL / progress chưa xong → **không** commit/push.
- Không hardcode tên branch — luôn đọc từ profile.

### 2.9 Nhánh "bug đã biết" = 1 bug
- `/bug` chỉ xử lý **một bug đã biết**. Nếu input là khu vực mơ hồ / danh sách nghi vấn
  → chạy `/bug-check` (§2b) trước, dừng chờ user chọn.

## 2b. Bug discovery (sweep) — `/bug-check`

Chế độ **READ-ONLY** để soi một màn/khu vực mơ hồ, KHÔNG sửa gì.

```
Xác định phạm vi → Đọc code/docs → Liệt kê defect (file:line)
   → ghi tasks/bug-<slug>/scan.md → DỪNG chờ user chọn defect → (user chạy /bug)
```

Quy tắc bắt buộc:
- **KHÔNG** sửa code, **KHÔNG** gọi `builder`/`builder-strong`, **KHÔNG** update `progress.json`,
  **KHÔNG** commit/push.
- Chỉ được tạo/ghi **một file**: `tasks/bug-<slug>/scan.md`.
- Phân loại trước khi soi: **SINGLE-SURFACE** (1 màn/luồng) hoặc **CROSS-CUTTING** (theme/dark mode,
  permission, i18n, tenant/campus, responsive, format tiền/ngày, a11y, loading/empty state).
- CROSS-CUTTING: bắt buộc enumerate toàn bộ surface ứng viên theo `source_roots` trong
  `.agent/PROJECT_PROFILE.md`; **CẤM sampling**. Surface RN gồm screen `**/screens/**/*.tsx`,
  component dùng chung `**/components/**/*.tsx`, navigation `**/navigation/**/*.tsx`,
  theme/constants `**/theme/**`, `**/constants/**`.
- CROSS-CUTTING dùng query count-based: đếm match theo từng file; `count=0` vẫn ghi coverage là đã soi;
  `count>0` đọc đúng vùng match để xác nhận và loại trừ variant hợp lệ như `dark:` hoặc token đúng.
- Chia batch 15–25 file/batch; append `scan.md` sau **mỗi batch**.
- Không kết luận khi coverage chưa đủ. Chỉ dừng khi 100% surface đã enumerate hoặc `scan.md` có
  `## Chưa soi` nêu lý do + `% đã soi`.
- `scan.md` bắt buộc có `## Coverage` với enumerate / đã soi / % / mỗi file `soi | count | kết luận`,
  và bắt buộc có `## Chưa soi`.
- CRUD/capability: không đánh giá cấp module. Mỗi API collection/mutation
  (`GET/POST/PATCH/DELETE /module/resource`) là một dòng capability riêng với cột:
  `Module | Sub-resource/API | FE section/table | List | Create UI | Edit UI | Delete UI | Empty CTA | Evidence`.
- `Create UI = Có` chỉ khi đúng resource đó có nút/form; không suy từ resource khác cùng module.
- API có POST nhưng FE chỉ list, không có nút/form/empty CTA → DEFECT hoặc `[cần xác nhận]`.
- Empty state không chỉ cách tạo data nguồn → DATA_SETUP/UX_DEFECT.
- Report là bảng defect: `# | Mô tả | Tái hiện | Expected | Actual | Root cause (file:line) | Severity | File cần sửa | Ước lượng`.
- Kết thúc: chạy `git status --short`; nếu có file nào khác `scan.md` biến động → cảnh báo vi phạm read-only.
- Output xong → **dừng**, chờ user chọn defect (mỗi defect xử lý bằng `/bug`).

---

## 3. Change Request workflow (feature / update)

```
Classify → Spec delta → Spec Validator → Phase/Task → Human duyệt plan
   → Loop(builder/reviewer) → Phase Review → Doc Impact/Reconcile → Progress
   → commit current branch (= target_branch mặc định) → (push current branch nếu được phép)
```

### 3.1 Classify
- **ADDITIVE** (thêm mới) / **MODIFY** (đổi behavior) / **REMOVE** (bỏ).
- Requirement mơ hồ → hỏi lại. Không tự chọn giả định lớn.

### 3.2 Spec delta
- Ghi rõ thay đổi so với `SPECIFICATIONS.md` (thêm/sửa/xóa mục nào, API/DB/UI bị ảnh hưởng).
- Nếu thay đổi behavior/scope → **cập nhật spec** hoặc ghi rõ lý do không cần.
- Liệt kê ảnh hưởng tới phase/task đã có (regression risk).

### 3.3 Spec Validator
- Gọi subagent `spec-validator` (không sửa source; chỉ được ghi report scoped) cross-check delta vs spec & docs.
- Ghi report pre-plan: `.context/review-reports/feature-<slug>-spec-validation.md`.
- FAIL → quay lại làm rõ. PASS → chia phase/task.

### 3.4 Phase / Task
- Chia theo **Phase model** (§4), mỗi task: scope, inputs, outputs, acceptance criteria, deps.
- File: `tasks/feature-<slug>/phase-<N>-task-<NN>.md`.
- Task feature/update bắt buộc ghi `Classification / Risk`: change type, scope, expected review level,
  blast radius, doc impact, decision impact.

### 3.5 Human duyệt plan
- Trình danh sách phase + task + thứ tự. **Chờ user duyệt** mới code.

### 3.6 Loop
- Mỗi task: builder implement+test → reviewer độc lập. FAIL → trả lại builder (max 2 vòng).
- Test/check fail, acceptance criteria chưa đạt, hoặc behavior sau update chưa khớp spec delta → **chưa xong**;
  quay lại builder cập nhật trong cùng task. Không được báo done chỉ vì đã edit code.
- Retry / Escalation Policy:
  - Attempt 1 fail: áp dụng quy trình trong `.agent/error-analyzer.md` (phần không bị maintenance override), xác định lại root cause, fix tối thiểu.
  - Attempt 2 fail: dừng patch triệu chứng; so với pattern code đang hoạt động và kiểm tra lại assumption.
  - Attempt 3 fail: **KHÔNG thử fix #4**. Set status `architecture_review_needed`, ghi rõ
    blocker/residual risk/verify evidence, hỏi human thay vì tự đóng.
  - Structural Review bắt buộc gồm: data flow, ownership/scope boundary, API contract,
    permission/tenant/school filters, state/cache layer, mock/real data boundary, schema/domain mismatch.
- Mỗi failed attempt phải append `.context/error-memory.md` hoặc ghi rõ vì sao không có entry.
- Nếu fix/update đổi kiến trúc, ownership/scope boundary, API contract, hoặc mock/real data boundary
  → append `.context/decisions.md`.
- Task report bắt buộc có verification summary:
  `Acceptance criteria: PASS|FAIL|BLOCKED`, `Verify commands + result`, `Reviewer verdict`.
- **Không tự chạy task/phase tiếp theo** khi chưa qua checkpoint (§8).

### 3.7 Phase Review
- Sau khi cả phase PASS: `spec-validator` cross-check "đã build đúng & đủ so với spec delta".
- PASS → xác định Doc Impact & Reconcile (§6) trước khi phase done/checkpoint; không impact → ghi `no doc impact`.
  GAP/FAIL → quay lại bổ sung trong task/phase liên quan; không set phase `done`.

### 3.8 Nhánh "đầu vào là danh sách feature"
- Tách **mỗi feature thành task/feature riêng**, chốt ưu tiên, xử lý **tuần tự**.
- Gộp chỉ khi cùng mục tiêu/scope (1 feature nhiều phase).

### 3.9 Progress (bắt buộc)
- Update `.context/progress.json`: thêm/cập nhật entry trong `features[]`, set `activeWorkItem`.
- Trước khi set feature/phase/task `done`, phải có acceptance PASS, verify commands PASS/skip có lý do,
  Reviewer PASS, Spec Validator PASS khi hết phase, và hoàn tất Doc Impact & Reconcile (§6) hoặc ghi `no doc impact`.
  Trạng thái `done`/progress/doc-impact này phải nằm trong close-out commit; progress/task file không cần biết SHA của commit đang được tạo.
- Nếu test/check/review/spec status là `FAIL`, `BLOCKED`, hoặc unknown → không set `done`.
- Commit/push: xem §2.8 (commit-first sau PASS; default staging-direct push `target_branch`, feature branch chỉ khi user yêu cầu).

---

## 4. Phase model

| Phase | Nội dung | Reviewer tập trung |
|---|---|---|
| 1 | Schema / domain (migration, model, types) | migration versioned, không phá dữ liệu cũ |
| 2 | Backend / API (service, route, validation, auth) | contract, OWASP, BOLA/IDOR, lỗi |
| 3 | UI (component, screen, states, responsive) | a11y, responsive nhiều kích thước (mobile-first, tablet), craft-floor |
| 4 | Integration (FE↔BE, auth flow, error/loading) | contract thật, không hardcode shape |
| 5 | Test / UAT (unit, integration, e2e, edge) | coverage path xấu, không false-confidence |

> Không nhảy cóc: schema xong mới API, API xong mới UI, integration xong mới UAT.
> Task có thể gộp phase nếu thật nhỏ — nhưng phải ghi rõ.

### 4b. Skill gates (bổ trợ, risk-based)

Áp dụng khi diff/phase có code phù hợp; reviewer ghi kết quả từng gate vào report.
Repo chưa cài tool / không có app code / không áp dụng → ghi `N/A` hoặc `skip <lý do>`, **không** fail workflow oan.

| Skill | Ai chạy | Gate |
|---|---|---|
| `skills/anti-slop/SKILL.md` | Builder + Reviewer, task đụng TS/JS | `npx oxlint` (khi repo có oxlint config); error mới → FAIL. Bổ trợ `aislop` (mùi nội dung) bằng mùi kiểu dáng code |
| `skills/open-code-review/SKILL.md` | Reviewer, task code change | `ocr review`/`ocr delegate`; **CRITICAL** (XSS/SQLi/NPE/thread-safety) → FAIL, ≥3 MAJOR → FAIL. Chưa cài `ocr` → ghi chú, không chặn PASS |
| `skills/ai-readable-codebase/SKILL.md` | Reviewer mọi phase + Builder khi viết mới | AI-chaos indicators **≥3 → FAIL**; code mới đổi luồng chính phải cập nhật `README.md`/`ARCHITECTURE.md` |
| `skills/blitzstrike/SKILL.md` | Reviewer Phase 2 STRICT (optional) | pentest live trên môi trường được phép; chỉ finding **STRIKE-validated** mới tính FAIL; chưa cài/không môi trường → bỏ qua |

> Trong bảng trên, gate nào trỏ skill không tồn tại trong repo (vd `ai-friendly-web`, `m3e-canvas`
> — web-only, không port sang mobile) → ghi `N/A, skill không có trong template mobile`.
> Gate `ai-readable-codebase` áp dụng được nếu skill đã được thêm vào repo.

> Thứ tự ưu tiên khi mâu thuẫn: `impeccable` (craft-floor) > `taste-skill-v2`;
> `aislop` (nội dung) + `anti-slop` (kiểu dáng) + `ocr` (bug thật) là 3 lớp bổ trợ, không thay thế nhau.

---

## 5. State & paths

### State — `.context/progress.json` (schema maintenance)
Tối thiểu file phải là:

```json
{
  "mode": "maintenance",
  "activeWorkItem": null,
  "features": [],
  "bugs": []
}
```

Khi có work item, có thể mở rộng trong `features[]` / `bugs[]`:

```json
{
  "mode": "maintenance",
  "activeWorkItem": null,
  "features": [
    { "slug": "", "title": "", "type": "ADDITIVE|MODIFY|REMOVE",
      "status": "planned|in_progress|blocked|architecture_review_needed|done", "currentPhase": 0, "tasks": [] }
  ],
  "bugs": [
    { "slug": "", "title": "", "severity": "blocker|high|medium|low",
      "status": "triaged|reproducing|root_caused|fixing|review|done|blocked|architecture_review_needed",
      "task": "tasks/bug-<slug>/..." }
  ]
}
```
- Tối thiểu: `mode`, `activeWorkItem`, `features`, `bugs`. Có thể thêm `lastUpdated` nếu muốn.
- Status semantics: `blocked` = chặn chung như thiếu info/môi trường; `architecture_review_needed` = đã fail ≥3 attempt, cần review kiến trúc/refactor trước khi sửa tiếp.
- **KHÔNG** dùng field greenfield (`currentLayer`, `totalLayers`, `completedTasks`, `inProgressTask`, …)
  trong maintenance mode.
- `activeWorkItem` = `{ type, slug }` của bug/feature đang làm, hoặc `null`.
- Cập nhật progress.json là bước **bắt buộc** (§2.7, §3.9).

### Task
- Feature: `tasks/feature-<slug>/phase-<N>-task-<NN>.md`
- Bug: `tasks/bug-<slug>/phase-<N>-task-<NN>.md`
- Trước khi Builder chạy, task phải có các block: `Classification / Risk`, `Acceptance Criteria`,
  `Verification Plan`, `Retry / Error Memory`, `Doc / Decision Impact`.
- Bug task phải có `Repro Verification`; feature/update task phải có `Feature Verification`.

### Review report
- Task reviewer report phải đúng tên: `.context/review-reports/<feature|bug>-<slug>-phase-<N>-task-<NN>-round-<R>-review.md`.
- Phase-level/spec-validator report khi hết phase: `.context/review-reports/<feature|bug>-<slug>-phase-<N>-round-<R>-review.md` (kèm hậu tố `-spec` nếu là spec-validator).
- Pre-plan spec validation (trước khi chia phase/task): `.context/review-reports/feature-<slug>-spec-validation.md`.
- Quy tắc `round-<R>`: luôn ghi rõ vòng (1, 2, …); không gộp nhiều vòng vào một file; rerun cùng round sau khi bị cancel → **ghi đè** (không tạo file trùng).
- Trước khi set status `done`: kiểm tra report tồn tại bằng slug/path. Không có report → **KHÔNG đóng việc**.

## 6. Cổng chặn (gates)

- **Branch**: default **staging-direct**: làm việc trực tiếp trên `target_branch`; current branch phải là `target_branch` trước khi commit/push. Nếu user yêu cầu feature branch, làm việc trên branch hiện tại (`feature/<slug>` hoặc `bug/<slug>`), push chính branch đó, và chỉ mở PR khi user yêu cầu rõ.
  **Cấm push `forbidden_branch`**, cấm `--force`/`-f` (gate ở `opencode.jsonc` → `permission.bash`).
- **Commit-first close-out**: sau task/bug/phase PASS review + Doc Impact/Reconcile + progress xong,
  commit lên branch hiện tại theo §2.8. FAIL → không commit/push.
- **Push**: mặc định không. Chỉ khi user yêu cầu rõ hoặc `auto_push_after_pass: true`
  (`.agent/PROJECT_PROFILE.md`), theo branch model ở §2.8/§6.
- **Commit hygiene**: trước commit phải `git status` + `git diff`; chỉ stage file thuộc task;
  không stage dirty cũ ngoài scope. Nếu shared file interleave nhiều scope khiến tách commit không an toàn,
  chỉ combined batch commit cho đúng epic đó và ghi rõ lý do.
- **Migration**: chỉ áp dụng khi `db_tool != none` **và** `migration_required: true`;
  versioned + committed; không sửa migration đã apply, tạo migration mới. `db_tool: none` → bỏ qua gate này.
- **Migration high-risk**: trước commit phải inspect `migration.sql` hoặc migration artifact theo `db_tool`.
  Nếu có `DROP TABLE/COLUMN`, đổi type, `SET NOT NULL`, `UNIQUE/FK` trên data cũ, enum phá hoại,
  bulk transform/backfill → gắn `HIGH_RISK_MIGRATION`, **KHÔNG promote production**, báo destructive op,
  table/column ảnh hưởng, tương thích data, backfill, rollback, kết quả verify staging.
- **Migration forbidden ops**: cấm `db push`, `migrate reset`, seed/reset, clone data giữa môi trường cho staging/prod.
  Flow chuẩn: dev → migration versioned → staging deploy theo `migration_command`/`db_tool` đã cấu hình
  → verify → promote đúng migration đã test lên prod.
- **Migration env isolation**: `staging_db` phải khác `prod_db`; data độc lập; không sync data staging→prod.
- **Secrets**: chỉ từ env; không hardcode/commit; không log.
- **UI**: responsive mobile-first (điện thoại → tablet); a11y; đo contrast; không slop (skills UI).
- **Progress bắt buộc**: mọi thay đổi trạng thái bug/feature → update `.context/progress.json`.
- **Close-out report gate**: trước status `done`, grep/check `.context/review-reports/` theo slug và đúng tên
  `<feature|bug>-<slug>-phase-<N>-task-<NN>-round-<R>-review.md` cho task review; phase/spec-validator review dùng
  `<feature|bug>-<slug>-phase-<N>-round-<R>-review.md`. Với feature, kiểm tra thêm report pre-plan
  `feature-<slug>-spec-validation.md`. Không có report → status `blocked`, không commit/push.
- **Commit gate**: sau Reviewer PASS + Doc Impact/Reconcile + report gate, set progress/task status `done`
  như một phần của close-out commit. Trước final response, HEAD commit phải tồn tại và bao gồm trạng thái
  `done`/progress/doc-impact close-out đó. Progress/task file không cần biết SHA của commit đang được tạo.
  Nếu không tạo được commit sau PASS close-out, không báo complete; giữ/set status `blocked` và ghi residual risk/lý do.
- **Doc reconcile**: sau task/bug/phase PASS và trước status `done`, xác định doc impact và reconcile as-built docs:
  API contract/endpoint/response shape → `docs/API_SPEC.md`; schema/model/enum → `docs/ERD.md` + regen
  `docs/generated/*` nếu có; kiến trúc/flow/current behavior → `docs/DESIGN.md` current-state; gap đã giải quyết
  → đổi status gap register. Không đổi contract/schema/behavior tài liệu hoá → ghi rõ `no doc impact`.
- **As-built vs intent**: as-built docs (`docs/API_SPEC.md`, `docs/ERD.md`, `docs/DESIGN.md` current-state,
  generated inventory, gap register status) được reconcile khi code đổi, kèm evidence code. Intent docs
  (`docs/BRD.md`, `docs/PRD.md`, `docs/USER_FLOW.md` target, business rules trong `SPECIFICATIONS.md` nếu có)
  chỉ đổi qua Change Request + user duyệt.
- **Code ≠ intent**: ghi gap vào gap register nếu có (vd `docs/changes/TECHNICAL_REQUIREMENT_GAPS.md`),
  **KHÔNG** hạ cấp intent cho khớp code. Fix code sai rồi sửa doc cho khớp = hợp pháp hoá bug, coi là vi phạm.
- **Docs không nhúng code tay**: API_SPEC/ERD là overview + pointer tới source of truth/generated.

### Tool Loop Guard

- Không chạy lặp cùng 1 shell/search/read command y hệt quá 1 lần.
- Không thử cùng 1 giả thuyết quá 2 lần bằng biến thể gần giống.
- Command/search trả empty hoặc non-zero → ghi nhận kết quả và chuyển hướng; không retry vô hạn.
- Bash bị permission deny → **DỪNG NGAY**: không retry, không đổi biến thể, không vòng qua pipeline;
  chuyển Grep/Read hoặc ghi `Blocked`.
- Không xác minh được → ghi `Residual risk`/`Blocked`, không lặp tool.

### Session handoff & resume (Run Journal)

- **Artifact:** `.context/runs/<type>-<slug>-<phaseTask>.md` (template `.context/runs/_TEMPLATE.md`).
  Primary ghi; subagent không ghi. WIP/checkpoint **KHÔNG** ghi vào task file.
- **Luật đầy đủ:** `AGENTS.md` § Session Handoff (write-ahead checkpoint, banner, Session Start Protocol).
- **Resume matrix** (journal `step`/`status` → việc session mới làm):

  | Cancel ở | journal | Session mới làm |
  |---|---|---|
  | giữa builder | `builder` / `running` | **redo builder** (read-before-write, không revert) |
  | sau builder, trước reviewer | `builder` / `awaiting` | **bỏ builder → chạy reviewer** |
  | giữa reviewer | `reviewer` / `running` | **rerun reviewer** (dọn report dở) |
  | sau reviewer | `reviewer` / `awaiting` | bước kế: `fix` \| `spec_validator` \| `closeout` \| phase sau |
  | giữa `fix` | `fix` / `running` | **redo builder ở chế độ fix** (đọc finding trong report trước) |
  | giữa `spec_validator` | `spec_validator` / `running` | **rerun spec_validator** |
  | giữa `closeout` | `closeout` / `running` | **idempotent close-out** (xem dưới) |

- **Guardrail redo** (bắt buộc khi redo bất kỳ bước):
  - Scope theo `filesTouched`/`filesNew` trong journal — **không revert toàn cục**
    (`checkout --`/`reset --hard` bị deny).
  - Reviewer: **dọn/ghi đè report dở** trước khi rerun.
  - Builder: check side-effect đã lỡ chạy — migration file đã tạo (không tạo lại), generate client,
    test DB local, process/port còn treo.
  - `interrupted ≠ failed attempt` — redo do cancel **không** tăng `attempt`.

- **Close-out idempotent** (tránh double-commit khi cancel giữa close-out):
  1. journal `closeout` / `running`.
  2. Cập nhật `.context/progress.json` (status/phase/task/verdict) — **TRƯỚC** commit,
     để pointer không stale nếu bị cancel giữa chừng.
  3. `git log --oneline` kiểm task đã có commit chưa:
     - chưa có → commit (stage **đúng** file thuộc task) → push theo branch model (§2.8: chỉ khi
       user yêu cầu rõ hoặc `auto_push_after_pass: true`) → 4.
     - đã có commit nhưng chưa push → `push` nếu được phép (§2.8) (**không** commit lại) → 4.
  4. journal `done`.

- **Nguyên tắc:** đĩa là sự thật, pointer (`progress.json`) chỉ là hint; `interrupted ≠ failed attempt`.
- **Report path:** journal lưu `evidence.reportPath` **chính xác** — resume không parse tên file report.
- **Usage gate:** `.context/session-policy.json` (`usageGate`); ở mỗi checkpoint gọi tool `usage()`,
  vượt ngưỡng thì hỏi user End/Làm tiếp (xem `AGENTS.md` § Session Handoff).

### Check commands
Lấy từ `.agent/PROJECT_PROFILE.md` → các field `lint_command`, `typecheck_command`,
`test_command`, `build_command` (mobile dùng alias generic; field `web_*`/`api_*` để null). `check_commands` chỉ là alias tổng hợp
từ các field trên nếu project đã điền.
Ví dụ placeholder (thay bằng lệnh thật khi repo có app code):
```
web typecheck → <configured command or skip, no app configured>
web lint      → <configured command or skip, no app configured>
api typecheck → <configured command or skip, no app configured>
api lint      → <configured command or skip, no app configured>
test          → <configured command or skip, no app configured>
```
Monorepo: filter theo package bị đụng bằng package manager đã cấu hình (vd `<pm> --filter <pkg> ...`).
Nếu chưa cấu hình package manager/test command hoặc chưa có `apps/`, API/web/test → **skip, no app configured**,
không fail workflow và không tự hardcode lệnh.

### Reviewer level (risk-based)
- Reviewer tự chọn `FAST` / `NORMAL` / `STRICT`; mặc định `NORMAL`.
- `FAST` chỉ dùng khi scope rất hẹp, không shared/API/auth/tenant/schema, Builder đã test PASS.
- Bắt buộc `STRICT` nếu có risk đỏ: auth/RBAC/permission; tenant/school/org isolation;
  schema/migration/database; data loss/destructive/bulk update; API contract/DTO/response shape dùng nhiều màn/client;
  shared service/hook/component/API client/cache key/navigation; payment/subscription/entitlement;
  import/export/report; cron/background job/webhook; security/token/session/password/upload/file access;
  root cause chưa rõ; logic quan trọng thiếu test.
- Report bắt buộc có: `Review level`, `Reason`, `Blast radius`, `Verify commands + result`,
  `Findings`, `Verdict PASS/FAIL`.

---

## 7. Model mapping + luật opt-in

### Cách bật model mapping (bắt buộc khi clone template)
1. Khai model từng vai ở `.agent/PROJECT_PROFILE.md` → block `models:`
   (`builder`, `builder_strong`, `reviewer`, `spec_validator`).
2. **Bỏ comment** dòng `model:` trong frontmatter `.opencode/agent/*.md` (hiện đang comment
   `<provider>/<...>` để kế thừa).
3. **Restart opencode** — agent/config **không hot-reload**; chưa restart thì model mới chưa có hiệu lực.
4. Kiểm: `builder ≠ reviewer` (khác họ provider) để lộ blind spot khác nhau; `spec-validator` họ thứ 3 nếu có.

- Nếu **chưa** cấu hình, frontmatter để comment → subagent **kế thừa model chính**
  (builder == reviewer, mất tác dụng tránh bias). Khi cần, chạy lại `0.5.C` rồi restart.
- `reviewer` / `spec-validator` không được sửa source; chỉ được ghi report scoped khi đang review.
- Chạy dạng **subagent** → context sạch, không thừa hưởng completion report của builder.
- **`explore` (built-in)**: subagent kế thừa model/variant của session cha. Nếu cha chạy variant `high`,
  `explore` cũng tốn variant `high` → set override `agent.explore` trong `opencode.jsonc`
  (`model` rẻ hơn + `variant: low`) để tránh đốt reasoning token. Đi kèm rule chống spawn `explore`
  thừa ở `AGENTS.md` § Tool Loop Guard.

### Luật opt-in `builder-strong`
- **CHỈ dùng khi user yêu cầu rõ.** Không tự chọn theo phán đoán "bài này khó".
- Bị chặn cứng ở `opencode.jsonc` → `permission.task."builder-strong": "ask"`.
- ⚠️ Auto-mode (`--auto` / auto-approve) sẽ tự duyệt `ask` → mất gate. Muốn giữ gate, không bật auto.

---

## 8. Human checkpoints

1. **Trước khi code** feature lớn → duyệt phase/task plan.
2. **Hết mỗi phase** → tóm tắt (task done, test, review) → chờ user.
3. **Trước deploy** → chờ user approve production.
4. **Khi bị block** (≥3 retry / không reproduce / nghi ngờ kiến trúc) → dừng, báo user.
