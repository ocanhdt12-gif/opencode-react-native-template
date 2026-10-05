---
description: Start project — chạy LIÊN TỤC chuỗi khởi tạo (đọc spec → brainstorm → design → graph → loop). Tự động chuyển bước, chỉ dừng ở human checkpoint; user không phải gõ lại lệnh.
---

Chạy **Project Start** — chuỗi khởi tạo **chạy liên tục** cho repo vào pipeline.

`$ARGUMENTS`

## Nguyên tắc: TỰ ĐỘNG CHUYỂN BƯỚC (không bắt user gõ lại lệnh)

- Sau khi bước N xong → **tự động** sang bước N+1, **KHÔNG** bảo user gõ `/brainstorm`, `/design`, `/graph`…
- Chỉ **DỪNG** ở **human checkpoint** (duyệt/token confirm/duyệt plan). Tại checkpoint in rõ:
  ```
  ✅ <bước> xong. 👉 Reply "ok" (hoặc "tiếp") để em chạy tiếp bước kế tiếp.
  ```
- User reply "ok" → **chạy tiếp ngay** bước kế tiếp trong cùng mạch, **không** hỏi lại lệnh.
- Bước nào đã có kết quả rồi (spec/design/config/plan) → **dùng lại**, không chạy lại, không hỏi lại.
- Nếu user muốn chạy lại 1 bước → chạy tay lệnh tương ứng (`/brainstorm`, `/design`, `/graph`) — đây là **manual override**, không phải bước bắt buộc trong chuỗi.

## Chuỗi (chạy tuần tự, tự chuyển bước)

```
1. ĐỌC SPEC
   ├─ Có SPECIFICATIONS.md rồi  → đọc + dùng luôn
   └─ Chưa có                    → chạy /spec-init (reverse-engineer từ code)

2. BRAINSTORM   (tự chạy tiếp)
   → .agent/brainstorm.md: clear yêu cầu (skills/brainstorming) → design doc docs/specs/
                          + chốt config → .context/project-config.md
   → ⏸ CHECKPOINT: chờ user approve design  → reply "ok" → tự chạy tiếp

3. DESIGN       (tự chạy tiếp, nếu project có UI)
   → .agent/design.md: design tokens skills/<stack>/design-tokens.md
                       + screen specs .context/design-spec.md + diagram docs/diagrams/
   → ⏸ CHECKPOINT: confirm design tokens    → reply "ok" → tự chạy tiếp

4. GRAPH        (tự chạy tiếp)
   → .agent/graph.md: chia layer + task → tasks/<slug>/layer-{N}-task-{NN}.md
                      + layer-plan diagram docs/diagrams/layer-plan.html
                      + update .context/progress.json
   → ⏸ CHECKPOINT: duyệt layer plan        → reply "ok" → tự chạy tiếp

5. LOOP         (tự chạy tiếp sau khi duyệt plan)
   → .agent/loop.md thực thi layer 0 → ⏸ checkpoint → layer 1 → …
   → DevOps (.agent/devops.md) lo git init/CI-CD/deploy ở layer 0 + sau mỗi layer
   → Layer N+1 chỉ unlock khi Layer N PASS + user approve (vẫn là checkpoint, không cần gõ lệnh)
```

## Ghi chú
- `/start` = **một** cửa vào cho lần đầu dựng dự án; cả chuỗi tự chạy, user chỉ duyệt ở checkpoint.
- `/spec-init`, `/brainstorm`, `/design`, `/graph` vẫn tồn tại để **chạy tay/chạy lại** khi cần — không bắt buộc trong `/start`.
- Sau khi dự án build xong: mọi thay đổi đi qua `/change` (agent `change-request`), không chạy lại `/start`.
- HARD-GATE: không code trước khi design approve; không chạy layer N+1 trước khi layer N PASS + user approve.
