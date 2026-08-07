# Secure Defaults & Sharp Edges Review (Mobile)

Identifies error-prone APIs, dangerous configurations, and "footgun" designs that enable security mistakes. Ensures code follows **secure-by-default** and **pit of success** principles.

> **BẮT BUỘC:** Áp dụng khi code/review config, auth, xử lý secret/token, hoặc thiết kế API/storage.

## When to Use

- Reviewing config schemas, env, storage choices
- Evaluating auth/session implementation
- Reviewing any code exposing security-relevant choices to developers
- Checking secure storage, permissions, deep links

## Check: Vulnerable Defaults

- **Fallback secrets** — hardcoded fallback API key/password/token in client. FAIL.
- **Insecure storage** — storing tokens/passwords in AsyncStorage/plaintext instead of `expo-secure-store`/Keychain/Keystore. FAIL.
- **Permissive access** — dangerous Android permissions (`READ_SMS`, `READ_CONTACTS`, `WRITE_EXTERNAL_STORAGE`) without need; iOS plist keys overdeclared. FAIL if unnecessary.
- **Default credentials** — default API keys, default tokens, debug keys shipped in prod. FAIL.
- **Fail-open security** — auth/config defaults to "allow/trust" on error. FAIL.
- **Debug features in prod** — verbose logs leaking tokens, debug menu enabled, `__DEV__`-only secrets. MAJOR.
- **Weak crypto** — MD5/SHA1, hardcoded IVs, deprecated encryption for sensitive data. FAIL.

## Secure-By-Default Rules (template-enforced)

1. **Secrets**: NEVER in client bundle. Load from env (`.env`) / backend proxy. `.env` NOT committed; use `EXPO_PUBLIC_` only for non-secret public values.
2. **Token storage**: `expo-secure-store` (or Keychain/Keystore) for tokens/refresh tokens/credentials. NEVER AsyncStorage for sensitive data.
3. **Network**: TLS enforced; no `cleartext`/`usesCleartextTraffic=true` for prod; pin certificate for high-sensitivity APIs.
4. **Validation**: every user input validated; sanitize anything rendering in WebView/Text.
5. **Crypto**: modern audited algorithms only (Argon2/bcrypt server-side; AES-GCM secure store).
6. **Errors**: sanitize — no stack traces / internal paths leaked in UI/analytics.

## Output

List findings with severity. **Any FAIL-severity finding blocks PASS** in review.

## Tone

Be specific — "access token stored in AsyncStorage at auth/storage.ts:8 — use expo-secure-store" not "storage insecure".
