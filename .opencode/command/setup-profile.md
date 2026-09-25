---
description: Onboarding repo thật — auto-detect stack, sửa profile THEO NHÓM (chọn nhóm → sửa cả nhóm → chọn tiếp/dừng), sync quyền verify command.
---

Chạy **Project Profile Setup** cho repo hiện tại. Đây là bước chạy **một lần** khi clone template
vào project thật (maintenance mode). Mục tiêu: biến placeholder trong `.agent/PROJECT_PROFILE.md`
thành giá trị thật để `/bug`, `/feature`, builder, reviewer biết chạy lệnh gì và commit/push ở đâu.

`$ARGUMENTS`

## Nguyên tắc tương tác — sửa theo NHÓM (bắt buộc)

- **KHÔNG hỏi từng field rời rạc.** Toàn bộ thay đổi diễn ra trong một vòng lặp nhóm.
- Mỗi vòng: dùng `question` tool hiển thị **danh sách nhóm** kèm trạng thái hiện tại, cho chọn
  **nhiều nhóm** một lượt (`multiple: true`). Luôn kèm 2 lựa chọn điều khiển:
  - **`Xong — ghi file & kết thúc`** → thoát vòng lặp, sang Bước cuối.
  - **`Dừng — không ghi`** → thoát ngay, không ghi file (in nháp đã chốt, xem cuối file).
- User chọn (các) nhóm → hỏi **toàn bộ field của (các) nhóm đó trong cùng một lượt**, gom theo nhóm,
  rồi cập nhật giá trị nháp trong bộ nhớ. Không hỏi field thuộc nhóm user không chọn.
- Sửa xong các nhóm đã chọn → **quay lại hiển thị danh sách nhóm** (đánh dấu
  `✅ đã chốt` / `⬜ chưa chốt` / `🔎 auto-detect, cần confirm`) → để user chọn tiếp hoặc dừng.
- **Chưa ghi file** giữa các vòng. Chỉ ghi ở Bước cuối khi user chọn `Xong`.

## Bước 0 — Auto-detect (không hỏi)

1. Chạy `node scripts/detect-profile.mjs` và đọc JSON trả về.
2. Nếu script không tồn tại, đọc repo thủ công: lockfile, `package.json` scripts, `source_roots`,
   `prisma/schema.prisma` hoặc `drizzle.config.*`.
3. Hiển thị gọn giá trị detect được + `warnings` (nếu có), và ghi rõ nhóm nào đã có auto-detect
   để user biết chỉ cần confirm.

## Danh mục nhóm

| # | Nhóm | Field | Nguồn |
|---|------|-------|-------|
| 1 | **Git & branch** | `target_branch`, `forbidden_branch`, `auto_push_after_pass` | hỏi |
| 2 | **Stack & source** | `package_manager`, `source_roots` | auto-detect + confirm |
| 3 | **Verify commands** | `install`/`lint`/`typecheck`/`test`/`build` (mobile: không có split web/api, dùng alias generic) | auto-detect + confirm |
| 4 | **Database & migration** | `db_tool`, `migration_required`, `staging_db`, `prod_db`, `migration_command` | auto-detect + confirm |
| 5 | **Models per role** | `models.builder`, `models.builder_strong`, `models.reviewer`, `models.spec_validator` | hỏi |

## Chi tiết nhóm (chỉ hỏi field của nhóm user chọn)

### Nhóm 1 — Git & branch
- **`target_branch`** — branch đích để commit/push (vd `staging`, `develop`). Không được là `main`
  nếu `forbidden_branch: main`.
- **`forbidden_branch`** — branch cấm push. Mặc định `main`, chỉ cần confirm.
- **`auto_push_after_pass`** — `true|false`. ⚠️ **KHÔNG phải "tự commit"**: commit sau PASS review là
  **bắt buộc luôn** trong mọi trường hợp. Flag này **chỉ** quyết định có tự động
  `git push origin <target_branch>` sau PASS hay không:
  - `false` (mặc định) → chỉ commit local, **chờ bạn yêu cầu rõ** mới push.
  - `true` → tự push `target_branch` sau PASS (vẫn cấm push `forbidden_branch`).
  (Tên cũ `auto_commit_after_pass` vẫn được chấp nhận khi đọc file.)

### Nhóm 2 — Stack & source
- Confirm `package_manager` (`none|pnpm|npm|yarn|bun`).
- Confirm `source_roots` (vd `[src]`, `[apps, packages]`, hoặc `[]` nếu chưa có app code).

### Nhóm 3 — Verify commands
- Confirm: `install`, `lint`, `typecheck`, `test`, `build`.
- Mobile: dùng alias generic (`lint_command`, `typecheck_command`, `test_command`, `build_command`);
  KHÔNG cần split web/api. Với Expo: typecheck = `tsc --noEmit`, lint = `expo lint`, test = `jest`/`vitest`.
  `install_command`/`lint_command`/`typecheck_command`/`build_command`/`test_command`.
- Repo không phải Node hoặc `package.json` không có script → để `null`
  (workflow sẽ ghi `skip, no app configured`).

### Nhóm 4 — Database & migration
- Confirm `db_tool` (`none|prisma|drizzle|other`) và `migration_required`.
- **Chỉ khi `db_tool != none`**: hỏi `staging_db`, `prod_db` — ghi **tên env var**, KHÔNG ghi
  secret/connection string. Bắt buộc `staging_db != prod_db`; nếu trùng → dừng nhóm, báo user
  (workflow cấm dùng chung/sync data staging↔prod).
- Hỏi `migration_command` nếu project có lệnh migrate riêng (vd `pnpm prisma migrate deploy`).

### Nhóm 5 — Models per role
- Hỏi 4 vai: **`builder`** (model code chính), **`builder_strong`** (model mạnh hơn — **chỉ dùng khi
  user yêu cầu rõ**), **`reviewer`**, **`spec_validator`**.
- ⚠️ **Ràng buộc: `builder` phải KHÁC HỌ PROVIDER với `reviewer`** (vd Anthropic vs OpenAI) để lộ
  blind spot khác nhau; `spec_validator` nên là họ thứ 3 nếu có. Nếu user chọn trùng họ → cảnh báo,
  hỏi lại.
- Không biết model nào → để placeholder, ghi rõ chưa chốt (không bịa).
- Sau khi chốt nhóm này, nhắc user: bỏ comment dòng `model:` trong `.opencode/agent/*.md`
  (`builder`, `builder-strong`, `reviewer`, `spec-validator`) cho khớp `models:` và **restart opencode**.

## Bước cuối — khi user chọn `Xong`

1. **Ghi `.agent/PROJECT_PROFILE.md`**
   - Fill đúng các field đã chốt, **giữ nguyên cấu trúc + comment** của file.
   - Ghi `auto_push_after_pass`; nếu file còn key cũ `auto_commit_after_pass` thì thay bằng tên mới.
   - KHÔNG ghi secret/token/connection string; secret chỉ đọc từ env.
   - Field không xác định để `null` / giữ placeholder, không bịa.

2. **Sync quyền verify command (dry-run trước, ghi sau)**
   1. Chạy `node scripts/apply-verify-permissions.mjs` (**mặc định dry-run**, chưa ghi).
      Script đọc command trong profile, tự bỏ qua command không an toàn (quote/backslash, wildcard toàn bộ,
      DB-destructive, git-mutating, `--force`) và in danh sách `Bỏ qua` kèm lý do.
   2. Xem output:
      - Command đủ an toàn → sẽ được auto-allow.
      - Command bị bỏ qua → **hỏi user**: có muốn tự thêm allow rule không? Nếu có, hướng dẫn sửa tay
        block `# verify-commands:start` … `# verify-commands:end` trong `.opencode/agent/reviewer.md`
        và `.opencode/agent/spec-validator.md`.
   3. Chỉ khi user đồng ý với danh sách auto-allow, chạy lại `node scripts/apply-verify-permissions.mjs --write`
      để ghi file.
   4. Nếu script exit code khác 0 (thiếu marker `# verify-commands:start/end`) → **dừng**, báo chưa hoàn tất,
      không tự thêm marker vào file agent.
   5. Không auto-allow `migration_command` (lệnh migrate) — để user tự thêm nếu thật sự cần.
   6. Pattern sinh ra là **exact** (không nối `*`) để tránh `cmd && lệnh phá hoại` đi kèm.

3. **Tóm tắt** — in ra:
   - `target_branch`, `forbidden_branch`, `auto_push_after_pass` (nói rõ: commit luôn bắt buộc; push tự động nếu `true`)
   - package manager + verify commands đã ghi
   - `db_tool`/`migration_required` + `staging_db`/`prod_db`
   - `models` theo vai (và cảnh báo nếu builder trùng họ reviewer)
   - allow rules đã sync
   - nhắc: **quit và restart opencode** vì `.opencode/*` và config không hot-reload.

## Khi user chọn `Dừng — không ghi`

- KHÔNG ghi `PROJECT_PROFILE.md`, KHÔNG sync quyền.
- In **nháp** các giá trị đã chốt theo nhóm để user thấy đã tới đâu, kèm nhắc có thể chạy lại
  `/setup-profile` để làm tiếp.

Không commit/push trong các bước này. Nếu `PROJECT_PROFILE.md` còn field quan trọng chưa chốt, ghi rõ
là chưa hoàn tất thay vì báo done.
