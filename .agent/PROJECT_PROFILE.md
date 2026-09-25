# PROJECT_PROFILE.md — Parameterize the workflow

> **Điền file này khi clone template.** Mọi workflow/subagent đọc giá trị ở đây,
> KHÔNG hardcode branch / package manager / DB / lệnh check trong generic docs.
> Nếu file còn placeholder (`<...>`) → coi như chưa cấu hình, phải hỏi user.
>
> 💡 Cách nhanh: chạy `/setup-profile` — auto-detect stack rồi sửa file này **theo nhóm**
> (Git & branch / Stack & source / Verify commands / Database & migration), chọn nhóm để sửa,
> sửa xong có thể chọn nhóm tiếp hoặc dừng; ghi file + sync quyền verify command cho
> reviewer/spec-validator khi chọn "Xong".

## Profile

```yaml
project: <tên dự án>
output_language: vi            # vi | en — ngôn ngữ cho docs/summary

# ── Git ──
target_branch: <target_branch> # default staging-direct: current branch phải là branch này khi commit/push PASS; chưa điền = hỏi user
forbidden_branch: main         # cấm push trực tiếp (opencode.jsonc hard-deny main/ref main)
branch_pattern: "<target_branch>" # default staging-direct; feature/<slug>|bug/<slug> chỉ khi user yêu cầu feature branch
auto_push_after_pass: false    # ⚠️ KHÔNG phải "tự commit": commit sau PASS luôn BẮT BUỘC. Flag này chỉ = AUTO-PUSH: true = tự `git push origin <target_branch>` sau PASS; false = chỉ commit local, chờ user yêu cầu rõ. (tên cũ `auto_commit_after_pass` vẫn được chấp nhận khi đọc)

# ── Package / source ──
package_manager: <none|pnpm|npm|yarn|bun>  # none/chưa điền = không hardcode lệnh
source_roots: []               # ví dụ: [src] hoặc [apps, packages]; [] nếu chưa có app code

# ── Verify commands (null/placeholder = skip, no app configured) ──
# Mobile: KHÔNG có split web/api — dùng alias generic bên dưới (lint/typecheck/test/build).
# Các field web_*/api_* giữ lại chỉ vì apply-verify-permissions.mjs đọc chung, để null.
web_typecheck_command: null
web_lint_command: null
api_typecheck_command: null
api_lint_command: null
test_command: null          # mobile: vd `jest` / `vitest run`
install_command: null       # mobile: vd `npm install` / `pnpm install`
lint_command: null          # mobile: vd `expo lint` hoặc `eslint .`
typecheck_command: null     # mobile: vd `tsc --noEmit`
build_command: null
migration_command: null     # only used when db_tool != none and migration_required: true

# ── Database ──
db_tool: none                  # none | prisma | drizzle | other
migration_required: false      # true nếu cần migration versioned (chỉ khi db_tool != none)
staging_db: <env var staging, vd DATABASE_URL_STAGING>   # TÊN ENV VAR, không phải connection string/secret
prod_db: <env var production, vd DATABASE_URL_PROD>       # TÊN ENV VAR; phải khác staging_db
destructive_migration_policy: HIGH_RISK_MIGRATION
```

### DB / migration

- **`db_tool: none`** → project không dùng DB/ORM: **bỏ qua toàn bộ migration safety rules**
  (Phase 1 schema, migration gate ở `.agent/FEATURE_WORKFLOW.md` §6, ERD/migration checks).
- `db_tool != none` **và** `migration_required: true` → áp dụng migration gate:
  migration phải versioned + committed, không sửa migration đã apply.
- Không hardcode Prisma/Drizzle: dùng đúng `db_tool` đã khai.
- Trước commit phải inspect migration artifact theo `db_tool`/`migration_command`. Nếu có `DROP TABLE/COLUMN`,
  đổi type, `SET NOT NULL`, `UNIQUE/FK` trên data cũ, enum phá hoại, bulk transform/backfill
  → gắn `HIGH_RISK_MIGRATION`, không promote production, báo destructive op, table/column ảnh hưởng,
  tương thích data, backfill, rollback, kết quả verify staging.
- Cấm `db push`, `migrate reset`, seed/reset, clone data giữa staging/prod. Flow: dev → migration versioned
  → staging deploy bằng command đã cấu hình → verify → promote đúng migration đã test lên prod.
- `staging_db` phải khác `prod_db`; data độc lập; không sync data staging→prod.

## Check commands (chạy trước khi báo xong)

> Mọi người (builder/reviewer) PHẢI dùng đúng lệnh đã cấu hình ở profile, filter theo package bị đụng nếu monorepo.
> Nếu command là `null`/placeholder hoặc repo chưa có app code (`source_roots: []`) → ghi `skip, no app configured`.
> KHÔNG tự suy ra `npm test`, `pnpm lint`, Prisma, package name, hay path `apps/` khi chưa cấu hình.

```yaml
check_commands:
  install: null             # alias of install_command
  web_typecheck: null       # hoặc = web_typecheck_command
  web_lint: null            # hoặc = web_lint_command
  api_typecheck: null       # hoặc = api_typecheck_command
  api_lint: null            # hoặc = api_lint_command
  test: null                # hoặc = test_command
  build: null               # hoặc = build_command
  migration: null           # hoặc = migration_command khi migration_required=true
  docs_inventory: null
```

## Models per role

> Model KHÔNG còn đọc từ `.env.local` (biến đó không có tác dụng). Khai ở đây rồi
> **bỏ comment + copy sang frontmatter** của từng file `.opencode/agent/*.md`,
> rồi **restart opencode** (config không hot-reload).
> Quy tắc: **builder ≠ reviewer** (khác họ provider) để lộ blind spot khác nhau.

```yaml
models:
  builder:        <provider>/<model-code-chinh>
  builder_strong: <provider>/<model-manh-hon>     # chỉ dùng khi user yêu cầu rõ (§7 gate)
  reviewer:       <provider>/<model-khac-ho>
  spec_validator: <provider>/<model-ho-thu-3>     # họ thứ 3 nếu có
```

## UI rules (nếu project có UI)

```yaml
ui:
  responsive_breakpoints: [375, 768, 1280]
  max_file_lines: 300
  max_function_lines: 50
```

## Secrets

```yaml
secrets:
  source: env                    # chỉ đọc từ env; KHÔNG hardcode/commit
  required: [DATABASE_URL, JWT_SECRET]
```
