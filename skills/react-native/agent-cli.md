# @expo/agent-cli — Agent Harness cho Expo

`@expo/agent-cli` **1.0.20** (hợp với Expo SDK 58) — CLI agent-native bọc quanh họ Expo CLI.
Nó chạy `expo`, `eas-cli`, `expo-doctor` như **subprocess**, không import internals → output
**machine-readable** (`--json`) để agent đọc trực tiếp thay vì parse log người.

Bin: `expo-agent-cli`. Chạy không cần cài: `npx @expo/agent-cli <cmd>`.

## Vì sao agent nên dùng

- Có exit code rõ ràng + `--json` → agent biết pass/fail mà không phải grep log màu.
- Tự trả lời "dev server / app đang chạy gì" bằng lệnh, không phải đọc output Metro cuộn.
- Không đoán version native module: `install` gọi `expo install` rồi đồng bộ skill của package.

## Vòng lặp chuẩn cho agent

| Bước | Lệnh | Nhận được |
|------|------|-----------|
| 1. Khảo sát project | `npx @expo/agent-cli status --json` | project là gì, có cần rebuild không, nên chạy gì tiếp |
| 2. Bật app | `npx @expo/agent-cli dev --ios --detach` | dev server chạy nền, terminal trả lại |
| 3. Mở route | `npx @expo/agent-cli navigate /` | app mở route trên device/simulator |
| 4. Sửa + reload | `npx @expo/agent-cli runtime:reload` | app chạy đúng code trên đĩa |
| 5. Xem lỗi | `npx @expo/agent-cli runtime:errors` | lỗi runtime trong khoảng thời gian |
| 6. Xem màn hình | `npx @expo/agent-cli runtime:tree` | cây view + `testID` đang hiển thị |
| 7. Verify | `npx @expo/agent-cli smoke --ios` | app trên code hiện tại + 1 exit code duy nhất |
| 8. Type check | `npx @expo/agent-cli typecheck` | lỗi type mà smoke không bắt được |

## Kiểm tra dev server / tiến trình (thay cho đọc log Metro)

Dùng nhóm `dev:*` + `status` — KHÔNG parse text log người:

- `dev:logs` — đọc log của dev server đã detach (kèm `--json` để lấy mảng log).
- `dev:stop` — dừng dev server đã detach.
- `status --json` — trạng thái project + gợi ý bước kế tiếp.

## Lệnh hay dùng khác

| Lệnh | Việc |
|------|------|
| `runtime:tap` / `runtime:type` | Điều khiển app theo `testID` |
| `runtime:eval` | Eval JS trong app đang chạy |
| `runtime:stop` | Dừng app trên device |
| `doctor` | `expo-doctor` đã chuẩn hoá output |
| `inspect:build-log` | Chỉ ra đúng dòng làm native build fail |
| `install <pkg>` | `expo install` + đồng bộ skill của package |
| `deploy` | Web app lên EAS Hosting; `--native` để ship native |
| `agents:setup` | Cấu hình agent cho project hoặc user home |
| `skills:sync` / `skills:list` | Đồng bộ/liệt kê skill do module cài sẵn |
| `docs:sync` / `docs:search` | Docs Expo của đúng SDK thành Markdown local để grep |

Nhóm lệnh dùng dạng `group:action`, viết cách cũng như nhau: `skills list` = `skills:list`.

## Lệnh KHÔNG thuộc CLI này

Các lệnh sau vẫn gọi `expo` CLI trực tiếp (agent-cli không bọc): `run`, `run:ios`, `run:android`,
`prebuild`, `config`, `export`, `export:web`, `export:embed`, `serve`, `customize`, `lint`,
`login`, `logout`, `register`, `whoami`.

## Lưu ý

- Mọi lệnh có `--json` → **luôn thêm `--json`** khi gọi từ agent/CI.
- `npx @expo/agent-cli help workflow` tóm tắt cả vòng lặp + exit code trên 1 màn hình.
- Dùng `npx @expo/agent-cli <cmd> --help` để xem option + key `--json` của từng lệnh.
