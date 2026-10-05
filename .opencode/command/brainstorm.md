---
description: Brainstorm — đọc spec/code rồi hỏi user nhập thông tin dự án + cấu hình vận hành (git, package, verify commands, DB, models, deploy, monitoring), ghi .context/project-config.md. Chạy sau /spec-init hoặc chạy lại để update.
---

Chạy **Brainstorm** cho repo hiện tại: đọc spec/code đã có → hỏi user **từng câu** để chốt
thông tin dự án + cấu hình vận hành → ghi `.context/project-config.md` cho mọi workflow/agent đọc.

`$ARGUMENTS` (tùy chọn: `<nhóm>` để chỉ sửa 1 nhóm — vd `git`, `models`; `--dry-run` chỉ in nháp, không ghi)

## Flow
Thực thi đầy đủ `.agent/brainstorm.md`:
1. **Phase 0 — Scan** (không hỏi): đọc `SPECIFICATIONS.md`, `docs/**`, `spec/**`; chạy `node scripts/detect-profile.mjs`.
2. **Phase 0.5 — Project config**: hỏi theo **7 nhóm** (Git & branch / Stack & source / Verify commands / Database & migration / Models per role / Deploy & CI-CD / Monitoring & UI). Auto-detect được → chỉ confirm.
3. **Phase 1–3 — Requirements**: chỉ khi dựng spec mới (skip câu đã có trong spec/docs); clarification + summary confirm.
4. **Ghi**: `.context/project-config.md` (config, KHÔNG secret) + `.env.local` (secret, git-ignored) + `.context/brainstorm-log.md`.
5. **Sync quyền verify command** (`node scripts/apply-verify-permissions.mjs`, dry-run trước) + nhắc **restart opencode**.

## Chạy lại / update
`/brainstorm` đọc `.context/project-config.md` hiện tại → hiển thị giá trị đang có → chỉ hỏi lại nhóm user chọn → merge (giữ field không đổi). `/brainstorm <nhóm>` chỉ sửa nhóm đó.

## Không làm
- KHÔNG hỏi nhiều câu một lúc; KHÔNG hỏi lại thứ đã có trong spec/code/config.
- KHÔNG ghi secret vào `.context/project-config.md`; KHÔNG commit/push trong bước brainstorm.
- Field không chắc để `null`/`<...>`, không bịa.
