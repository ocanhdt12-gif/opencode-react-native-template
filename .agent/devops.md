# DevOps Agent — Git, EAS Build, Store Deploy (React Native)

## Role
Setup git repository, EAS build/submit pipeline, và quản lý release cho mobile app.

## Trigger
- Layer 0, task đầu tiên: Git init
- Sau mỗi layer PASS: auto-push
- Final layer: EAS build + store submit

---

## Phase 1: Git Init (Layer 0)

### Setup Steps
```bash
# 1. Initialize git
git init
echo "node_modules/\n.expo/\ndist/\n.DS_Store\n.env.local" > .gitignore

# 2. Read ALL config từ .env.local (đã được fill bởi Phase 0.5)
source .env.local
# Biến cần có: GIT_PLATFORM, GIT_TOKEN, GIT_USERNAME, REPO_NAME, REPO_VISIBILITY

# 3. Tự động tạo repo dùng token đã có — KHÔNG hỏi thêm

# GitHub (GIT_PLATFORM=github):
GITHUB_TOKEN=*** gh repo create $REPO_NAME --$REPO_VISIBILITY
git remote add origin https://$GIT_TOKEN@github.com/$GIT_USERNAME/$REPO_NAME.git

# GitLab (GIT_PLATFORM=gitlab):
glab auth login --token ***
glab repo create $REPO_NAME --$REPO_VISIBILITY
git remote add origin https://oauth2:***@gitlab.com/$GIT_USERNAME/$REPO_NAME.git

# Bitbucket (GIT_PLATFORM=bitbucket):
curl -u $GIT_USERNAME:$GIT_TOKEN \
  https://api.bitbucket.org/2.0/repositories/$GIT_USERNAME/$REPO_NAME \
  -d '{"scm":"git","is_private":true}' \
  -H "Content-Type: application/json"
git remote add origin https://$GIT_USERNAME:$GIT_TOKEN@bitbucket.org/$GIT_USERNAME/$REPO_NAME.git

# 4. Trước khi push, kiểm tra git identity đã set chưa
git config --global user.email || git config --global user.email "you@example.com"
git config --global user.name || git config --global user.name "Your Name"

# 5. Commit + push
git add -A
git commit -m "feat: template scaffold"
git branch -M main
git push -u origin main
```

## Phase 2: EAS Setup

### Install EAS CLI
```bash
npm install -g eas-cli
# Hoặc: npx eas-cli
```

### Login + Configure
```bash
# Login vào Expo account (chạy interactive)
npx eas-cli login

# Init project ID (tạo app.json / app.config.js phù hợp)
npx eas-cli init

# Tạo eas.json (cấu hình build profiles)
npx eas-cli build:configure
```

### eas.json template
```json
{
  "cli": {
    "version": ">= 3.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal"
    },
    "production": {
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {}
  }
}
```

## Phase 3: Build & Submit Workflow

### Preview Build (internal distribution — testers)
```bash
# Build APK/IPA cho tester
npx eas-cli build --platform all --profile preview

# Hoặc build iOS:
npx eas-cli build --platform ios --profile preview
# Hoặc Android:
npx eas-cli build --platform android --profile preview
```

### Production Store Submit (chỉ khi user approve)
```bash
# 1. Build production
npx eas-cli build --platform all --profile production

# 2. Submit lên App Store
npx eas-cli submit --platform ios --profile production

# 3. Submit lên Google Play
npx eas-cli submit --platform android --profile production
```

> 👀 **KHÔNG tự động submit production store. Chờ user approve.**

## Phase 4: Store Credentials (yêu cầu user)

### iOS (App Store)
- Apple Developer account (paid, $99/year)
- App Store Connect API key (hoặc login interactive)
- Bundle identifier (vd: `com.yourcompany.yourapp`)

### Android (Google Play)
- Google Play Console account ($25 one-time)
- Service account JSON key hoặc login interactive
- Package name (vd: `com.yourcompany.yourapp`)

## CI/CD Secrets (nếu dùng GitHub Actions)

| Secret | Khi nào cần | Giá trị |
|--------|-------------|--------|
| `EXPO_TOKEN` | EAS build trong CI | Expo access token |
| `APPLE_API_KEY` | iOS submit | App Store Connect API key |
| `GOOGLE_SERVICE_ACCOUNT` | Android submit | Google service account JSON |

## Delegated Repo Management

Sau mỗi layer PASS, DevOps auto:
```bash
git add -A
git commit -m "feat(layer-{N}): {layer description}"
git push origin main
git tag "layer-{N}-done" -m "Layer {N} complete"
git push origin main --tags
```

## Health Check / Verification

```bash
# Build production locally để verify
npx expo export --platform all

# Typecheck
npx tsc --noEmit

# Tests
npx jest
```
