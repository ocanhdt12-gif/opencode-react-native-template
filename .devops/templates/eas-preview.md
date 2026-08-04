# EAS Preview Build (Internal Distribution)

## Overview
Build APK/IPA cho internal distribution — dùng để test trên device trước khi release. Không cần lên store.

## Profile (trong `eas.json`)
```json
{
  "build": {
    "preview": {
      "distribution": "internal"
    }
  }
}
```

## Build Commands

```bash
# Build cả hai platform cho tester
npx eas-cli build --platform all --profile preview

# Hoặc từng platform
npx eas-cli build --platform ios --profile preview
npx eas-cli build --platform android --profile preview

# Build Android APK (có thể cài trực tiếp)
npx eas-cli build --platform android --profile preview
```

## Kết Quả
- iOS: build được cài qua TestFlight / Dev Client
- Android: APK/AAB có thể tải và cài trực tiếp
- Link install được gửi qua email từ EAS

## Yêu Cầu Trước Khi Build
- [ ] Login Expo: `npx eas-cli login`
- [ ] `eas.json` đã cấu hình profile preview
- [ ] `app.json` có đúng `name`, `slug`, `version`
- [ ] Đã `npx eas-cli init` (có project ID)

## Test Trước Khi Build
```bash
# 1. Typecheck
npx tsc --noEmit

# 2. Lint
npx expo lint

# 3. Unit tests
npx jest

# 4. Chạy thử local
npx expo start
```

## Checklist Trước Khi Submit Production
- [ ] Preview build đã được test trên thiết bị thật
- [ ] Các luồng chính (login, core feature, navigation) hoạt động
- [ ] Không có crash / lỗi nghiêm trọng
- [ ] User đã confirm test xong → mới chuyển sang EAS production submit
