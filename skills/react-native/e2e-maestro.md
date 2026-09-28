---
name: e2e-maestro
description: "Maestro E2E testing cho React Native/Expo — UI + end-to-end test bằng YAML flows (launchApp/tapOn/assertVisible), chạy trên emulator/simulator/thiết bị thật, không cần driver/SDK. Dùng khi viết E2E test, review mobile app có luồng UI quan trọng, hoặc build flow test mới. Trigger: E2E test, UI automation, maestro."
---

# Maestro — Mobile E2E Testing (Curated)

> Curated từ [mobile-dev-inc/Maestro](https://github.com/mobile-dev-inc/maestro) — framework E2E cho Android/iOS/web: YAML flows dễ đọc, cài single binary, không cần driver/SDK, chạy qua accessibility layer. Là lựa chọn chính cho E2E mobile (Detox giữ làm phương án thay thế).

## Vì sao Maestro (thay vì Detox mặc định)

1. **YAML flows** — khai báo `launchApp` / `tapOn` / `assertVisible` / `scrollUntilVisible`, không cần viết test code.
2. **Không driver/SDK** — cài binary, chạy được trên emulator, simulator, browser, hoặc device thật Android (iOS physical chưa hỗ trợ).
3. **Agent-friendly** — `maestro mcp` ship sẵn trong CLI, coding agent điều khiển app live để tự verify khi build.
4. **Nhanh** — viết flow đầu tiên trong <5 phút; chạy local hoặc Maestro Cloud (parallel).

## Cài đặt + chạy

```bash
curl -Ls "https://get.maestro.mobile.dev" | bash     # hoặc brew install maestro
maestro test .maestro/                               # chạy tất cả flows
maestro test .maestro/smoke.yaml                     # chạy 1 flow
maestro studio                                       # record flow bằng UI
maestro mcp                                          # MCP server cho coding agent
```

## Flow mẫu — `.maestro/smoke.yaml`

```yaml
appId: com.example.app
---
- launchApp
- assertVisible: "Home"
- tapOn: "Profile"
- assertVisible: "Đăng nhập"
- inputText: "user@example.com"
```

## Hook vào template

- **Builder** viết E2E cho luồng quan trọng (auth, checkout, publish) → tạo flow trong `.maestro/` + chạy `maestro test .maestro/` trên emulator/simulator.
- **Reviewer** task có E2E test → yêu cầu chạy Maestro flow liên quan; flow fail → FAIL (trừ khi lỗi môi trường, ghi rõ lý do).
- Chưa có emulator/simulator/device trong môi trường review → ghi `N/A`, không chặn PASS.

## Lưu ý

- Dùng `testID` (hoặc accessibilityLabel) cho selector ổn định — đừng phụ thuộc text nội dung dễ đổi.
- Flow phải deterministic — tránh sleep cứng, dùng `waitForAnimationToEnd` / assertion thay vì `delay`.
- Không chạy E2E trên môi trường production thật (tốn data + rủi ro); dùng build dev/preview.
- Maestro Cloud (parallel) chỉ khi cần scale — local đủ cho hầu hết task.

## Output

Trả về: danh sách flow đã chạy (pass/fail), tổng thời gian, screenshots/video khi fail, ghi chú giới hạn môi trường.