# EAS Build - Expo Template

## Prerequisites
- Expo account (expo.dev)
- EAS CLI: `npm install -g eas-cli`
- Logged in: `eas login`

## eas.json Configuration

```json
{
  "cli": {
    "version": ">= 12.0.0",
    "appVersionSource": "remote"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": {
        "simulator": true
      },
      "env": {
        "EXPO_PUBLIC_API_URL": "http://localhost:3000"
      }
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "simulator": false
      },
      "env": {
        "EXPO_PUBLIC_API_URL": "https://staging-api.example.com"
      },
      "channel": "preview"
    },
    "production": {
      "autoIncrement": true,
      "env": {
        "EXPO_PUBLIC_API_URL": "https://api.example.com"
      },
      "channel": "production"
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "your@apple.id",
        "ascAppId": "1234567890",
        "appleTeamId": "XXXXXXXXXX"
      },
      "android": {
        "serviceAccountKeyPath": "./google-services.json",
        "track": "internal"
      }
    }
  }
}
```

## app.json / app.config.js

```json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.company.myapp",
      "buildNumber": "1"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "package": "com.company.myapp",
      "versionCode": 1
    },
    "plugins": [
      "expo-router",
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#ffffff"
        }
      ]
    ],
    "extra": {
      "eas": {
        "projectId": "your-project-id"
      }
    },
    "runtimeVersion": {
      "policy": "appVersion"
    },
    "updates": {
      "url": "https://u.expo.dev/your-project-id"
    }
  }
}
```

## Build Commands

```bash
# Development build (with dev tools)
eas build --profile development --platform ios
eas build --profile development --platform android

# Preview build (for internal testing)
eas build --profile preview --platform all

# Production build
eas build --profile production --platform all

# Build for specific platform
eas build --profile production --platform ios
eas build --profile production --platform android

# Local build (no EAS servers)
eas build --profile development --platform ios --local
```

## Submit to Stores

```bash
# Submit iOS to App Store Connect
eas submit --platform ios --latest

# Submit Android to Google Play
eas submit --platform android --latest

# Submit specific build
eas submit --platform ios --id <build-id>
```

## OTA Updates (EAS Update)

```bash
# Publish update to preview channel
eas update --channel preview --message "Bug fix: login screen"

# Publish to production
eas update --channel production --message "v1.2.1 hotfix"

# Check update status
eas update:list
```

## Credentials Management

```bash
# View/manage credentials
eas credentials

# iOS: Generate new certificates
eas credentials --platform ios

# Android: Manage keystore
eas credentials --platform android

# Reset credentials (careful!)
eas credentials --platform ios # then select "Remove"
```

## Build Lifecycle

```
1. eas build triggered
2. EAS clones repo
3. Install dependencies (npm/yarn)
4. Run prebuild (generate native projects)
5. iOS: pod install
6. Compile native code
7. Sign binary
8. Upload artifact
9. Notify (webhook/email)
```

## Monitoring Builds

```bash
# List recent builds
eas build:list

# View specific build
eas build:view <build-id>

# Cancel running build
eas build:cancel <build-id>

# Download build artifact
eas build:download --id <build-id>
```

## Tips

- Use `autoIncrement: true` for production to auto-bump version
- Set `appVersionSource: "remote"` to sync versions across team
- Use `channel` for OTA update routing
- Test preview builds before production
- Keep `google-services.json` in `.gitignore`, use EAS Secrets for CI
