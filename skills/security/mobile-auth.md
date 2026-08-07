# Mobile API Authentication & Token Security

Tests and hardens authentication/authorization in the mobile app's API layer: secure token storage, session management, token refresh, and preventing auth bypass.

> **BẮT BUỘC:** Áp dụng khi code/review bất kỳ logic auth: login, token storage, refresh token, session, logout, hoặc tích hợp OAuth2/OIDC.

## When to Use

- Creating/modifying login/logout flows
- Token storage, refresh, and session management
- Adding auth headers to API client
- Any OAuth2 / social login integration

## Critical Do's (secure mobile auth)

1. **Secure storage** — store access token + refresh token in `expo-secure-store` (Keychain on iOS / Keystore on Android). NEVER AsyncStorage for tokens/credentials.
   ```ts
   // ✅ CORRECT
   import * as SecureStore from 'expo-secure-store';
   await SecureStore.setItemAsync('access_token', token);
   // ❌ WRONG — plaintext, not encrypted
   // await AsyncStorage.setItem('access_token', token);
   ```
2. **No secrets in bundle** — API keys/URLs in client must be non-sensitive; real secrets go to backend. `EXPO_PUBLIC_` is NOT secret.
3. **Token refresh** — implement refresh flow; on failure, clear session + force re-login. Don't cache tokens in app state only.
4. **Logout** — clear ALL sensitive data (tokens, cached user state) from secure store + in-memory.
5. **Biometrics** (optional) — Gate high-value actions with `LocalAuthentication`/`expo-local-authentication`.
6. **TLS** — enforce HTTPS; no clear-text endpoints in prod; consider cert pinning for high-sensitivity APIs.

## Test/Review Checklist

- [ ] Tokens stored via `expo-secure-store` (not AsyncStorage/plaintext)
- [ ] No API keys/secrets hardcoded in client bundle
- [ ] Refresh token rotation + secure refresh flow
- [ ] Logout fully clears token + cached user state
- [ ] TLS enforced (no cleartext in prod)
- [ ] Auth header attached to all protected requests
- [ ] 401 handling → refresh or re-login, not silent fail
- [ ] No sensitive data (tokens, passwords) in logs / analytics / error reports

## Output

Findings with severity. **Any FAIL (plaintext token storage, hardcoded secret, cleartext traffic) blocks PASS** in review. Resolve before commit.

## Tone

Be specific — "access token stored in AsyncStorage at auth/storage.ts:8 — use expo-secure-store" not "auth not secure".
