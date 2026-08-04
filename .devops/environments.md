# Environments (React Native / Expo)

## Environment Strategy

React Native dùng `.env` files (qua `expo-constants` + babel plugin hoặc `process.env.EXPO_PUBLIC_*`).

> ⚠️ **Quan trọng:** Biến bắt đầu bằng `EXPO_PUBLIC_` được expose trong client bundle. **Không bao giờ** đặt secrets (API key server, DB password) với prefix này — chúng sẽ bị leak trong app.

## Env Files

| File | Dùng cho | Commit? |
|------|----------|---------|
| `.env` | Development / local | ❌ No |
| `.env.production` | Production build | ❌ No |
| `.env.example` | Template (expose variable NAMES thôi) | ✅ Yes |

## Cách dùng

```bash
# Đặt biến trong .env
EXPO_PUBLIC_API_URL=https://api.example.com
EXPO_PUBLIC_APP_NAME=MyApp
```

```tsx
// Đọc trong code
const apiUrl = process.env.EXPO_PUBLIC_API_URL;
```

## Thứ tự ưu tiên

1. `.env.production` khi build production (`eas build --profile production`)
2. `.env` mặc định

## Secrets KHÔNG được đặt trong client

| Loại | Nơi lưu đúng |
|------|--------------|
| Backend API key | Server-side (backend proxy) |
| Database credentials | Server-side |
| JWT secret | Server-side |
| Push notification server key | Server-side |
| Apple/Google store credentials | EAS credentials (không commit) |

Table: **Những gì được đặt `EXPO_PUBLIC_`** (public, không nhạy cảm):
- API public base URL
- App name / version
- Feature flags (public)
- Analytics keys (nếu public client-side)

## EAS Environment Variables

EAS Build hỗ trợ env vars qua `app.config.ts`:

```ts
// app.config.ts
import { ConfigContext } from 'expo/config';

export default ({ config }: ConfigContext) => ({
  ...config,
  extra: {
    apiUrl: process.env.EXPO_PUBLIC_API_URL,
    env: process.env.APP_ENV,
  },
});
```

```tsx
// Đọc qua Constants
import Constants from 'expo-constants';
const apiUrl = Constants.expoConfig?.extra?.apiUrl;
```

## Checklist

- [ ] `.env` / `.env.production` trong `.gitignore`
- [ ] `.env.example` đã commit (chỉ tên biến, không giá trị)
- [ ] Secrets không nằm trong client bundle
- [ ] Không hardcode URL môi trường trong code
