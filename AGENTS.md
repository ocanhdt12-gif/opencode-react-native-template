# AGENTS.md — AI Workflow Router (entry point)

> This is the **always-loaded entry point**. Read it before acting on any request.
> Detailed workflow: greenfield build → `AGENT.md`; maintenance (bug/feature/update) → `.agent/FEATURE_WORKFLOW.md`.
> Project-specific values (branch, package manager, check commands) → `.agent/PROJECT_PROFILE.md`.

## Precedence

`AGENTS.md` **always wins** over every file in `.agent/`. If a legacy workflow file
(`.agent/blackboard.md`, `.agent/rollback.md`, `.agent/graph.md`, …) conflicts with this
file or `.agent/FEATURE_WORKFLOW.md`, follow **this file**. Legacy files carry a
`Maintenance mode override` note at the top — honor it.

## Router — classify intent BEFORE coding

| Intent (user says…) | Route (mandatory) |
|---|---|
| "fix bug", "lỗi", "broken", regression, crash (đã biết rõ bug nào) | **Bug workflow** (§Bug) → `/bug` |
| "soi/kiểm tra màn", "cảm giác nhiều lỗi nhưng không rõ" | **Bug discovery / sweep** → `/bug-check` — READ-ONLY, KHÔNG fix |
| "thêm/sửa/bỏ/xóa tính năng", "change/update feature" | **Change Request workflow** → `.agent/FEATURE_WORKFLOW.md` → `/feature` |
| "implement feature" (spec/task đã có sẵn) | **Builder theo task** → `.opencode/agent/builder` |
| "review", "check", "soát" (một diff/task cụ thể) | **Reviewer** → `.opencode/agent/reviewer` — KHÔNG tự sửa code |
| "thêm skill", "add skill", "tạo skill", "register skill" | **Customize opencode** — tạo/cập nhật runtime skill đúng format (§Local skills) |
| hỏi / điều tra / "tại sao", "how does X work" | **Research-only** — KHÔNG edit nếu user chưa yêu cầu fix |

Không rõ intent → hỏi 1 câu ngắn để phân loại, đừng đoán.

### Phân biệt command

| Command | Dùng khi | Tính chất |
|---|---|---|
| `/bug-check` | Khu vực/màn mơ hồ, "cảm giác nhiều lỗi" | **READ-ONLY** — soi, liệt kê defect vào `tasks/bug-<slug>/scan.md`, **dừng chờ user chọn**. Không sửa, không commit. |
| `/bug` | **Một bug đã biết** hoặc list bug đã xác nhận | Diagnose root cause → task → builder → reviewer → progress → commit-first → push theo branch model nếu được phép |
| `/feature` | Thêm/sửa/bỏ tính năng | Classify ADDITIVE/MODIFY/REMOVE → spec delta → phase/task → builder/reviewer/spec-validator → progress |
| `/resume` | Mở session mới **làm tiếp** việc đang dở | Đọc Run Journal → reconcile đĩa → thực hiện `next`. **KHÔNG** classify/phase-plan lại (§ Session Handoff) |

---

## Bug rules (bắt buộc)

1. **Không sửa triệu chứng trước khi có root cause.** (Iron Law — `skills/superpowers/systematic-debugging.md`)
2. **Thiếu info** (màn hình / bước tái hiện / expected-actual / role/vai trò) → **hỏi ngắn trước**, không tự giả định.
3. Bug **không phải sửa 1 dòng** → tạo task: `tasks/bug-<slug>/phase-<N>-task-<NN>.md`.
4. Fix-loop + `Repro Verification` theo `.agent/FEATURE_WORKFLOW.md` §2 và `/bug`: chỉ `done` khi repro PASS + Reviewer PASS.
5. Retry/Escalation: sau 3 attempt fail → status `architecture_review_needed`, dừng chờ review kiến trúc/refactor.
6. **Builder** code + test; **Reviewer** kiểm tra độc lập (không sửa source; chỉ ghi report scoped).
7. **Cập nhật `.context/progress.json`** (schema maintenance) sau mỗi bước đổi trạng thái bug.
8. Sau Reviewer PASS + close-out + progress cập nhật, **commit-first** theo `.agent/FEATURE_WORKFLOW.md` §2.8.
   Reviewer FAIL → không commit/push.
9. **Danh sách bug** hoặc kết quả `/bug-check`, kể cả "fix tất cả defect" → tách từng bug/task,
   tóm tắt số lượng defect, đề xuất thứ tự, nêu bug nào gộp vì cùng root cause, rồi **DỪNG hỏi xác nhận**
   trước khi gọi Builder. Chỉ bỏ checkpoint nếu user ghi rõ `auto proceed`, `khỏi hỏi lại`,
   hoặc `tự xử lý hết không cần hỏi`.

## Feature rules (bắt buộc)

1. **Classify ADDITIVE / MODIFY / REMOVE** trước khi code.
2. Requirement mơ hồ → **hỏi lại**, không tự chọn giả định lớn.
3. Đổi behavior/scope → cập nhật **spec delta** hoặc ghi rõ lý do không cần.
4. Tạo `tasks/feature-<slug>/phase-<N>-task-<NN>.md` khi nhiều bước hoặc có risk.
5. Task phải có `Classification / Risk`, verification summary, và Retry/Escalation theo `/feature` + `.agent/FEATURE_WORKFLOW.md` §3.
6. Sau 3 attempt fail → status `architecture_review_needed`, dừng chờ review kiến trúc/refactor.
7. **Builder** code + test; **Reviewer** độc lập; **Spec Validator** cross-check gap so với spec.
8. **Cập nhật `.context/progress.json`** (schema maintenance).
9. **Danh sách feature** → tách **mỗi feature thành task riêng**, chốt ưu tiên, xử lý **tuần tự**.
   Gộp chỉ khi cùng mục tiêu/scope (1 feature nhiều phase).

## Commit-First Tracking

- Sau khi task/bug/phase PASS review + close-out + cập nhật `.context/progress.json`, phải commit lên branch hiện tại.
- Branch model mặc định là **staging-direct**: current branch phải là `target_branch`, commit ở đó; **push không tự động** — chỉ `git push origin <target_branch>` khi user yêu cầu rõ hoặc `auto_push_after_pass: true` (xem § Non-negotiables). Nếu user yêu cầu feature branch thì push chính current branch (`git push origin <current-branch>`) và chỉ mở PR khi user yêu cầu rõ.
- **1 task = 1 commit**, trừ khi có lý do rõ ràng.
- Commit là source of truth cho changed files, timestamp, SHA, rollback point.
- Task file là source of truth cho root cause, repro/evidence, residual risk, doc impact/reconcile, verification summary.
- `.context/progress.json` là source of truth cho current status, active/completed phase/task, reviewer result/report path.
- Commit message convention và body trailer: xem `.agent/FEATURE_WORKFLOW.md` §2.8.
- Commit-first tracking là source of truth; không còn `docs/history/YYYY-MM.md` (đã xoá).

### Doc Impact & Reconcile Rules

Sau khi task/bug/phase PASS, xác định doc impact và reconcile **TRƯỚC** khi đóng việc:

| Thay đổi | Doc cập nhật |
|---|---|
| API contract/endpoint/response shape | `docs/API_SPEC.md` |
| Schema/model/enum | `docs/ERD.md` + regen `docs/generated/*` nếu có |
| Kiến trúc/flow/current behavior | `docs/DESIGN.md` (current-state) |
| Giải quyết gap đã ghi | đổi status gap register |
| Không đổi contract/schema/behavior tài liệu hoá | ghi rõ `no doc impact` |

Phân biệt 2 loại doc — **CẤM tự sửa intent docs cho khớp code**:
- **As-built** (`docs/API_SPEC.md`, `docs/ERD.md`, `docs/DESIGN.md` current-state, generated inventory,
  gap register status): reconcile khi code đổi, kèm evidence code.
- **Intent** (`docs/BRD.md`, `docs/PRD.md`, `docs/USER_FLOW.md` target, business rules trong
  `SPECIFICATIONS.md` — nếu file tồn tại): chỉ đổi qua Change Request + user duyệt.

Code ≠ intent → ghi gap vào gap register (nếu có, vd `docs/changes/TECHNICAL_REQUIREMENT_GAPS.md`),
**KHÔNG** hạ cấp intent cho khớp code. Fix code sai rồi sửa doc cho khớp = hợp pháp hoá bug, coi là vi phạm.

---

## Non-negotiables (mọi route)

- **Commit** sau PASS theo Commit-First Tracking. `auto_push_after_pass: true` chỉ cho phép auto-push
  `target_branch` sau PASS; **deploy / mở PR luôn cần user yêu cầu rõ**.
- **Default staging-direct**: chỉ auto-push `target_branch` khi current branch = `target_branch`; nếu user yêu cầu feature branch thì push current branch và PR chỉ khi user yêu cầu rõ. **Cấm push `forbidden_branch`**,
  cấm `--force` / `-f`. Gate cứng ở `opencode.jsonc` (`permission.bash`).
- **KHÔNG commit/push khi Reviewer FAIL** hoặc khi progress chưa cập nhật.
- **KHÔNG tự sửa source khi đang review** — reviewer/spec-validator chỉ được ghi report scoped.
- **Check commands lấy từ `.agent/PROJECT_PROFILE.md`** — không hardcode `npm`.
- Nếu repo chưa có app code/API/web/test hoặc command chưa cấu hình → verify ghi `skip, no app configured`,
  không hardcode package manager/test command và không fail workflow vì thiếu app.
- **Migration safety** chỉ áp dụng khi `db_tool != none` / `migration_required: true`
  (`.agent/PROJECT_PROFILE.md`); `db_tool: none` → bỏ qua gate migration.
- Khi migration gate áp dụng: migration phải versioned + committed; không sửa migration đã apply.
  Trước commit inspect migration artifact; destructive/high-risk ops (`DROP`, đổi type, `SET NOT NULL`,
  `UNIQUE/FK` trên data cũ, enum phá hoại, bulk transform/backfill) → gắn `HIGH_RISK_MIGRATION`,
  không promote production, báo rõ destructive op/table/column/data/backfill/rollback/verify staging.
- Cấm `db push`, `migrate reset`, seed/reset, clone data giữa môi trường cho staging/prod.
  Flow: dev → migration versioned → staging deploy → verify → promote đúng migration đã test lên prod.
  `staging_db` phải khác `prod_db`; không sync data staging→prod.
- **Model mạnh (`builder-strong`) chỉ dùng khi user yêu cầu rõ** — không tự chọn theo độ khó.
- Xong việc → không tự chạy phase/task tiếp theo khi chưa qua **human checkpoint**.
- **Session handoff**: dừng ở ranh giới step → ghi Run Journal (write-ahead); session mới resume qua
  `/resume` (xem § Session Handoff) — **đĩa là sự thật**, pointer là hint.

## Tool Loop Guard

- Không chạy lặp cùng 1 shell/search/read command y hệt quá 1 lần.
- Không thử cùng 1 giả thuyết quá 2 lần bằng biến thể gần giống.
- Command/search trả empty hoặc non-zero → ghi nhận và chuyển hướng, không retry vô hạn.
- Bash bị permission deny → **DỪNG NGAY**: không retry, không đổi biến thể, không vòng qua pipeline;
  chuyển Grep/Read hoặc ghi `Blocked`.
- Không xác minh được → ghi `Residual risk`/`Blocked`, không lặp tool.
- **Chi phí subagent `explore`:** chỉ spawn `explore` khi root cause **chưa xác định**. Đã có
  `file:line`/root cause chứng minh (từ `/bug-check`, journal, repro, hoặc check đơn giản) → **cấm spawn
  `explore`**; tự `Read` đúng vị trí và truyền thẳng `file:line` cho Builder. Chưa chắc root cause → tối đa
  **1 lần** cho mỗi điều tra. Lý do: mỗi subagent là session riêng, tự đọc lại context từ đầu, không share cache.

## Session Handoff (Run Journal)

Mục tiêu: **dừng ở bất kỳ ranh giới step, mở session mới làm tiếp** — không mất code, không lạc step,
worst case **redo đúng 1 step**. Resume là **tái dựng** từ artifact, **không** phải nối tiếp lossless.

**Artifact:** `.context/runs/<type>-<slug>-<phaseTask>.md` (template: `.context/runs/_TEMPLATE.md`).
Primary ghi journal; **subagent không ghi**.

**Write-ahead checkpoint (bắt buộc):**
1. TRƯỚC khi gọi subagent: ghi `step=<bước>, status=running`, snapshot `filesTouched`/`filesNew`
   từ `git status --short`, in `▶ START <bước> <phaseTask>`.
2. SAU khi subagent trả về: ghi `status=awaiting`, `evidence`, `next`, cập nhật manifest,
   **rồi mới** in `✅ DONE <bước> <phaseTask>`. Ghi journal **TRƯỚC** khi in `✅ DONE`.
3. `✅ DONE` là **điểm dừng an toàn**. `▶ START ... running` mà cancel → session sau **redo bước đó**.

**Banner:** mỗi checkpoint in `▶ START` / `✅ DONE` kèm `phaseTask` + `next`; khi `status=running`
ghi rõ "cancel sẽ redo bước này".

**Session Start Protocol (đầu mỗi session):**
1. Đọc journal của `activeWorkItem` trong `.context/progress.json` (không có journal → coi pointer là hint).
2. Reconcile với đĩa: `git status --short` (so manifest → file lạ = nhiễm chéo), `git log --oneline`
   (task đã commit chưa), `evidence.reportPath` (reviewer đã chạy chưa, round mấy).
3. In **Resume Briefing**: workItem, step, dirty files vs manifest, evidence, `next` → rồi mới làm.
4. **Đĩa là sự thật, pointer là hint.** Lệch → theo đĩa; bước mơ hồ → redo bước đó.

**Redo policy:** cắt ngang `builder`/`reviewer` → **redo nguyên bước**. Redo là chạy lại **trên đĩa**
(đọc file + `git diff` trước khi sửa), **không revert** (`git checkout --` / `reset --hard` bị deny).
Dọn report dở trước khi rerun reviewer.
**`interrupted ≠ failed attempt`** — redo do cancel **KHÔNG** tăng `attempt`.

**Loop signal:** thấy cùng lỗi lặp ≥ 2 lần → ghi `loopSignal` vào journal + escalate
`architecture_review_needed`, không tự redo vô hạn.

**Sau auto-compact:** phải **re-read journal** từ đĩa, không tin trí nhớ tóm tắt.

**Usage gate (safe-point theo ngưỡng):** policy ở `.context/session-policy.json`
(`usageGate: {enabled, threshold, minStepsLeft, hardThreshold}`). Ở mỗi checkpoint, gọi tool `usage()`
(plugin `loop-guard`); nếu `percent ≥ threshold` **và** còn ≥ `minStepsLeft` bước (hoặc `percent ≥ hardThreshold`)
→ hỏi user bằng `question` tool: `[End — mở session mới] / [Làm tiếp] / [Tiếp, đừng hỏi tới hardThreshold]`.
Chọn End → in Resume Briefing + dòng `/resume <type>/<slug>` để copy sang session mới, rồi dừng turn.

## Local skills

- Runtime skills của template nằm trong `skills/<skill-name>/SKILL.md` và được đăng ký qua `opencode.jsonc` → `skills.paths: ["./skills"]`.
- Khi user yêu cầu **thêm skill**, phải tạo folder `skills/<lowercase-hyphen-name>/SKILL.md` với frontmatter `name` + `description`; `description` phải nêu rõ khi nào auto-trigger bằng keyword cụ thể.
- Nếu skill có nhiều tài liệu chi tiết, giữ chúng trong cùng folder và để `SKILL.md` làm wrapper trỏ tới các file đó.
- Sau mọi thay đổi skill/config/agent/command, nhắc user **restart opencode** vì config không hot-reload.

## Reviewer rules (risk-based)

- Reviewer tự chọn `FAST` / `NORMAL` / `STRICT`; mặc định `NORMAL`.
- `FAST` chỉ khi scope rất hẹp, không shared/API/auth/tenant/schema, Builder đã test PASS.
- `STRICT` bắt buộc khi có risk đỏ: auth/RBAC/permission; tenant/school/org isolation; DB/schema/migration;
  data loss/bulk update; API contract/DTO/response shape dùng nhiều client; shared service/hook/component/API client/cache key/navigation;
  payment/subscription; import/export/report; cron/webhook; security/token/session/password/upload/file access;
  root cause chưa rõ; logic quan trọng thiếu test.
- Report phải có `Review level`, `Reason`, `Blast radius`, `Verify commands + result`, `Findings`, `Verdict PASS/FAIL`.
