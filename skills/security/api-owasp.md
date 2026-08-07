# API Security — OWASP Top 10 (Mobile)

Systematically assesses REST/GraphQL API endpoints consumed by the mobile app against the OWASP API Security Top 10 (2023). Ensures the mobile app talks to hardened endpoints and handles responses safely.

> **BẮT BUỘC:** Áp dụng cho mọi task tạo/sửa API client, auth logic, hoặc xử lý dữ liệu từ server.

## When to Use

- Creating/modifying API client (fetchers, TanStack Query hooks)
- Adding auth, token refresh, or session logic
- Handling server responses (rendering, storage)
- Reviewing network layer security

## The OWASP API Top 10 Checks (mobile context)

### 1. Broken Object Level Authorization (BOLA/IDOR)
- Ensure the app doesn't rely on "logged in" alone; backend must verify ownership.
- App should pass the correct scoped IDs; never blindly trust deep-link params.

### 2. Broken Authentication
- Secure token handling on device: store in `expo-secure-store`/Keychain/Keystore, NOT AsyncStorage.
- Implement token refresh securely; logout clears all sensitive state.

### 3. Broken Object Property Level Authorization
- Never send/save more user data than needed; sanitize objects before rendering.

### 4. Unrestricted Resource Consumption
- App-side: limit payload size, paginate large lists via TanStack Query.

### 5. Broken Function Level Authorization
- Role-gated UI/actions must be re-validated server-side; don't just hide buttons client-side.

### 6. Unrestricted Access to Sensitive Business Flows
- Guard high-value flows (payments, account actions) with biometrics/re-auth and rate limits.

### 7. SSRF
- If the app can provide URLs the server fetches, ensure server validates allowlist.

### 8. Security Misconfiguration
- TLS enforced; no `cleartext` traffic in prod; auth/security headers; sanitize error payloads shown in UI.

### 9. Improper Inventory Management
- No deprecated/unsupported API versions; keep Expo SDK + deps updated.

### 10. Unsafe Consumption of APIs
- Validate/limit data from third-party APIs; never eval external responses; sanitize anything rendered in WebView.

## Validation & Output

- Validate API response shape (zod schemas) before use; never trust unvalidated server data.
- For every finding, state severity: **FAIL** (blocks PASS) vs **MAJOR** (should fix).
- **Any FAIL finding blocks review PASS** until resolved.

## Tone

Be specific — "API response rendered raw in WebView without sanitization at screens/news.tsx:40" not "insecure API usage".
