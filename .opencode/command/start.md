---
description: Start project — chạy TỰ ĐỘNG chuỗi khởi tạo: đọc spec (spec-init nếu chưa có) → brainstorm (clear yêu cầu + config) → design (design spec + tokens) → graph (chia layer/task). Dùng khi bắt đầu dự án mới/đưa repo vào pipeline.
---

Chạy **Project Start** — chuỗi khởi tạo tự động cho repo vào pipeline. Người dùng không phải gõ tay từng bước.

`$ARGUMENTS`

## Chuỗi tự động (chạy lần lượt, dừng ở human checkpoint)

```
1. ĐỌC SPEC
   ├─ Có SPECIFICATIONS.md rồi  → đọc + dùng luôn
   └─ Chưa có                    → chạy /spec-init (reverse-engineer từ code)

2. BRAINSTORM  (đọc spec xong → tự gọi)
   → thực thi .agent/brainstorm.md:
     • Clear yêu cầu (skills/brainstorming/SKILL.md) → design doc docs/specs/
     • Chốt config dự án → .context/project-config.md
   → DỪNG chờ user approve design (HARD-GATE)

3. DESIGN  (brainstorm approve → tự gọi, nếu project có UI)
   → thực thi .agent/design.md:
     • Sinh design tokens skills/<stack>/design-tokens.md
     • Sinh .context/design-spec.md (screen specs)
     • Diagram (nếu cần) docs/diagrams/
   → DỪNG confirm design tokens với user

4. GRAPH  (design xong → tự gọi)
   → thực thi .agent/graph.md:
     • Chia layer + task → tasks/<slug>/layer-{N}-task-{NN}.md
     • Vẽ layer-plan diagram docs/diagrams/layer-plan.html
     • Update .context/progress.json (totalLayers/currentLayer/…)
   → DỪNG chờ user duyệt plan

5. LOOP  (sau khi user duyệt plan)
   → .agent/loop.md thực thi layer 0 → checkpoint → layer 1 → …
   → DevOps (.agent/devops.md) lo git init/CI-CD/deploy ở layer 0 + sau mỗi layer
```

## Ghi chú
- `/start` là cửa vào cho **lần đầu** dựng dự án. Các bước con cũng chạy tay được: `/spec-init`, `/brainstorm`, `/design`, `/graph`.
- `/brainstorm`, `/design`, `/graph` **tự động được gọi** trong chuỗi này; khi đã có spec/config rồi thì chúng dùng lại, không hỏi lại thứ đã có.
- Sau khi dự án đã build xong: mọi thay đổi đi qua `/change` (agent `change-request`), không chạy lại `/start`.
- Không code trước khi design approve; không chạy layer N+1 trước khi layer N PASS + user approve.
