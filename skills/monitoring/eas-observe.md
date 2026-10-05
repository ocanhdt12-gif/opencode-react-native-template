---
name: eas-observe
description: "EAS Observe (Expo) — performance monitoring production cho Expo/React Native: đo TTI, cold/warm launch, bundle load, frame drops, per-route timings, gắn với từng build/OTA update và query qua CLI. Dùng khi setup performance monitoring Expo/EAS hoặc review startup performance. Trigger: eas observe, expo-observe, performance monitoring Expo, startup TTI."
---

# EAS Observe — Performance Monitoring cho Expo/RN (Curated)

> Curated từ docs.expo.dev/eas/observe (GA 08/2026) — performance monitoring production của Expo, dành cho app **Expo SDK 55+ chạy EAS**. Đo startup thật trên thiết bị người dùng, gắn metric với build/OTA update, query được qua CLI cho agent.

## Yêu cầu / giới hạn

- **Expo SDK ≥ 55** (SDK 54 trở xuống không dùng được). SDK 56+ dùng API `Observe`; SDK 55 dùng tên `AppMetrics` cũ.
- Chỉ chạy trong **development/production build** — **KHÔNG chạy trong Expo Go**.
- Cần `eas` project (`extra.eas.projectId`) + đăng nhập EAS CLI.

## Cài đặt (verified — SDK 57)

```bash
npm install -g eas-cli && eas whoami          # đăng nhập eas login nếu chưa
npx expo install expo-observe                 # tự chọn version khớp SDK
```

Wrap root layout component + export làm default:

```tsx
import { Observe } from 'expo-observe';
Observe.configure({ /* dispatchInDebug: true để test trên dev build */ });
export default function Root() { /* ... */ }
```

`eas.json` — bật source maps cho stack trace đúng file:

```json
{ "build": { "production": { "uploadSourceMaps": true } } }  // cần EAS CLI ≥ 22.0.0
```

Debug build mặc định KHÔNG dispatch metric — thêm `dispatchInDebug: true`.

## Query bằng CLI (agent dùng được)

```bash
eas observe:versions                  # các app version/build/update để lọc
eas observe:metrics-summary           # median, p90, p99 theo version (--metric tti --platform ios --days 14)
eas observe:metrics                   # sample cá nhân (--sort slowest --limit 20) → tìm session chậm
eas observe:routes                    # per-route timings (SDK 56+, Expo Router/React Navigation)
eas observe:session                   # full timeline 1 session (điều tra sau khi thấy sample chậm)
eas observe:events                    # sự kiện user-defined (Observe.logEvent) + expo.memory.warning...
```

**Metric có sẵn:** `tti` (time to interactive) · `ttr` (first render) · `cold_launch` · `warm_launch` · `bundle_load` · `update_download` · per-route `nav_cold_ttr` / `nav_warm_ttr` / `nav_tti` + **frame drops**, kèm device/battery/thermal/network.

## Hook vào template (mobile monitoring)

- **Setup (Phase 0.5 monitor):** project Expo + EAS → thêm `expo-observe` + `Observe.configure` ở root layout + `uploadSourceMaps` production, dùng CLI verify metric sau build đầu tiên.
- **Reviewer task liên quan performance/release:** chạy `eas observe:metrics-summary --metric tti` để bắt regression giữa 2 release; `eas observe:routes` khi đổi navigation.
- Bổ trợ: Sentry/OTel lo error tracking/APM chung — Observe lo **startup performance + pipeline EAS** (chúng bổ sung, không thay thế).

## Lưu ý

- `Observe.configure({ sampleRate: 0.25 })` — sampling theo installation, ổn định; free plan ~100K events/tháng.
- Dữ liệu gửi qua **OTLP over HTTP** — đổi được `endpointUrl` về OpenTelemetry Collector tuỳ biến.
- Không dùng Observe trên app chưa có EAS project / sắp tách khỏi Expo.

## Output

Báo: các metric đã lấy (median/p90/p99 theo version), regression nào phát hiện giữa các release, session chậm cần soi, hoặc ghi chú `N/A` (app không phải Expo/EAS, chưa cài expo-observe).