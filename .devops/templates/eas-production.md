# EAS Production Build & Store Submit

## Overview
Build production + submit lên App Store (iOS) và Google Play (Android). **Chỉ chạy khi user approve.**

## Profile (trong `eas.json`)
```json
{
  "build": {
    "production": {
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {}
  }
}
```

## Store Credentials

### iOS (App Store)
- Apple Developer account (paid, $99/year)
- App Store Connect: login interactive hoặc API key
- **Bundle identifier:** `com.yourcompany.yourapp` (phải khớp `app.json`)

### Android (Google Play)
- Google Play Console account ($25 one-time)
- Service account JSON hoặc login interactive
- **Package name:** `com.yourcompany.yourapp` (phải khớp `app.json`)

## Steps

### 1. Build Production
```bash
npx eas-cli build --platform all --profile production
```

### 2. iOS Submit (App Store)
```bash
npx eas-cli submit --platform ios --profile production
```
Yêu cầu:
- App Store Connect app đã tạo
- Apple credentials configured:
  ```bash
  npx eas-cli credentials
  ```

### 3. Android Submit (Google Play)
```bash
npx eas-cli submit --platform android --profile production
```
Yêu cầu:
- Play Console app đã tạo
- Service account JSON / login

## App Config (`app.json`)
```json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain"
    },
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.yourcompany.yourapp"
    },
    "android": {
      "package": "com.yourcompany.yourapp",
      "versionCode": 1
    }
  }
}
```

## Checklist Trước Khi Submit

### App Content
- [ ] App icon đúng kích thước
- [ ] Splash screen
- [ ] Store metadata (title, description, keywords)
- [ ] Screenshots (iOS 6.7"/6.5"/5.5", Android phone)
- [ ] Privacy policy URL (nếu thu thập data)

### Technical
- [ ] TypeScript compile pass
- [ ] Tests pass
- [ ] Preview build đã test OK trên device
- [ ] Không có hardcode URL môi trường dev
- [ ] App version + build number đúng

> 👀 **HUMAN CHECKPOINT — Trước khi submit production:**
> "Preview build xong và test OK. Anh confirm để em submit lên store không?"
> **KHÔNG tự động submit. Chờ user approve.**
