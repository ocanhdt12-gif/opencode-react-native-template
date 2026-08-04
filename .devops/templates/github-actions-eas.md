# GitHub Actions + EAS Build CI/CD

## Workflow: Build on Push to Main

```yaml
# .github/workflows/eas-build.yml
name: EAS Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  EXPO_TOKEN: ${{ secrets.EXPO_TOKEN }}

jobs:
  lint-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test -- --coverage

  build-preview:
    needs: lint-test
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - run: npm ci
      - run: eas build --profile preview --platform all --non-interactive

  build-production:
    needs: lint-test
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - run: npm ci
      - run: eas build --profile production --platform all --non-interactive

  submit:
    needs: build-production
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - run: npm ci
      - run: eas submit --platform all --latest --non-interactive
```

## Workflow: OTA Update on Push

```yaml
# .github/workflows/eas-update.yml
name: EAS Update

on:
  push:
    branches: [main]
    paths-ignore:
      - '*.md'
      - '.github/**'

env:
  EXPO_TOKEN: ${{ secrets.EXPO_TOKEN }}

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - run: npm ci
      - run: eas update --channel production --message "${{ github.event.head_commit.message }}" --non-interactive
```

## Workflow: PR Preview with QR Code

```yaml
# .github/workflows/pr-preview.yml
name: PR Preview

on:
  pull_request:
    types: [opened, synchronize]

env:
  EXPO_TOKEN: ${{ secrets.EXPO_TOKEN }}

jobs:
  preview:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - uses: expo/expo-github-action@v8
        with:
          eas-version: latest
          token: ${{ secrets.EXPO_TOKEN }}

      - run: npm ci

      - name: Create update
        id: update
        run: |
          OUTPUT=$(eas update --branch pr-${{ github.event.number }} --message "PR #${{ github.event.number }}" --non-interactive --json)
          echo "update_url=$(echo $OUTPUT | jq -r '.[0].manifestPermalink')" >> $GITHUB_OUTPUT

      - name: Comment PR
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `📱 **Preview ready!**\n\nScan with Expo Go:\n\`\`\`\n${{ steps.update.outputs.update_url }}\n\`\`\``
            })
```

## Required Secrets

| Secret | Where to Get |
|--------|-------------|
| `EXPO_TOKEN` | expo.dev → Account Settings → Access Tokens |
| `APPLE_ID` | Your Apple ID email (for submit) |
| `ASC_API_KEY` | App Store Connect → Users → Keys |
| `GOOGLE_SERVICE_ACCOUNT` | Google Cloud Console → Service Accounts |

## Setup Steps

1. **Create Expo Token:**
   ```bash
   # Or at expo.dev dashboard
   eas login
   ```

2. **Add GitHub Secrets:**
   - Repo → Settings → Secrets → Actions → New secret
   - Add `EXPO_TOKEN`

3. **Configure EAS for CI:**
   ```json
   // eas.json - add to build profiles
   {
     "build": {
       "production": {
         "autoIncrement": true,
         "cache": {
           "key": "production-v1"
         }
       }
     }
   }
   ```

4. **Store submission credentials as EAS Secrets:**
   ```bash
   eas secret:create --name GOOGLE_SERVICES_JSON --value "$(cat google-services.json)" --type file
   ```

## Tips

- Use `--non-interactive` flag in CI (no prompts)
- Cache node_modules with `actions/setup-node` cache option
- Use `eas build --profile preview` for PR builds (faster, no signing)
- OTA updates don't need full rebuild — much faster for JS-only changes
- Set up webhooks at expo.dev for build completion notifications
