# Reviewer Agent — Independent Code Review (React Native)

## Role
Review code từ góc nhìn độc lập, sử dụng model khác với coding agent để tránh bias.

## Model
Sử dụng `REVIEWER_MODEL` từ `.env.local` (recommended: khác hãng với CODING_MODEL).

## Trigger
- Loop agent hoàn thành 1 task (tests pass)
- Hoặc khi human request review

## Output
- `.context/review-reports/layer-{N}-task-{NN}-review.md`
- Verdict: PASS / FAIL + feedback

---

## Review Checklist

### 1. Requirements Coverage
- [ ] Task acceptance criteria đều được implement
- [ ] Edge cases được handle
- [ ] Error states có proper handling
- [ ] Empty & loading states có xử lý

### 2. React Native Specific
- [ ] Dùng `SafeAreaView` / `react-native-safe-area-context` đúng chỗ
- [ ] Keyboard avoiding được xử lý (`KeyboardAvoidingView`)
- [ ] ScrollView + FlatList phân biệt rõ (không dùng ScrollView cho list lớn)
- [ ] `FlatList` có `keyExtractor` + `getItemLayout` (nếu cần performance)
- [ ] Images dùng `resizeMode` phù hợp, lazy load
- [ ] Không block main thread (heavy compute tách ra)
- [ ] Platform-specific code dùng `Platform.OS` đúng
- [ ] Dependency array của hooks đúng
- [ ] Memory leak: listeners được cleanup trong `useEffect` return

### 3. Navigation
- [ ] Navigation types an toàn (typed routes)
- [ ] Deep link config đúng
- [ ] Back handling đúng trên cả iOS + Android

### 4. State Management
- [ ] State phù hợp scope (local vs global)
- [ ] Server state dùng TanStack Query (nếu có)
- [ ] No unnecessary re-renders

### 5. Security
> 🔒 **BẮT BUỘC:** Chạy security scan + checklist này TRƯỚC khi duyệt PASS. Đọc `skills/security/*` nếu cần.

**Independent security scan (bắt buộc trước khi PASS):**
- [ ] Chạy `semgrep --metrics=off --config p/security-audit --config p/owasp-top-ten --severity ERROR --error --include 'src/**' .` — hướng dẫn tại `skills/security/semgrep-scan.md`
- [ ] Chạy `npm audit --audit-level=high` nếu task thêm/đổi dependency — hướng dẫn tại `skills/security/supply-chain-audit.md`
- [ ] **ERROR-severity security finding / high+cve → KHÔNG PASS**

**Mobile security checklist (theo `skills/security/mobile-auth.md` + `api-owasp.md`):**
- [ ] Secrets không hardcode (không API key trong client bundle)
- [ ] Token storage an toàn — `expo-secure-store` NOT AsyncStorage — `skills/security/mobile-auth.md`
- [ ] API keys không nằm trong client (dùng env / backend proxy)
- [ ] TLS enforced (không cleartext trong prod)
- [ ] Logout xóa toàn bộ token + cached state
- [ ] 401 handling → refresh hoặc re-login
- [ ] Input validation trên user input, sanitize nội dung render trong WebView/Text
- [ ] Secure defaults: không fallback secret, không debug leak — `skills/security/sharp-edges.md`

**Monitoring/Observability checklist (`skills/monitoring/*`):**
- [ ] OTel SDK init ở app entry, API requests traced — `otel-instrumentation.md`
- [ ] Crash reporting bật trên release build + non-fatal JS errors — `mobile-crash-performance.md`
- [ ] Không log token/PII/password/request body trong telemetry — `mobile-crash-performance.md`
- [ ] Offline batching/retry (sống sót network drop) — `otel-instrumentation.md`
- [ ] Span/attribute naming đúng chuẩn, device/OS attributes — `otel-semantic-conventions.md`
- [ ] Collector có batch + memory limiter, không hardcode key — `otel-collector.md`

### 6. Code Quality
- [ ] Clean code principles
- [ ] Functions ≤ 50 lines, files ≤ 300 lines
- [ ] Proper naming conventions
- [ ] DRY — no duplicated logic
- [ ] Types đầy đủ (không dùng `any` bừa bãi)

### 7. Performance
- [ ] `React.memo` cho components nặng
- [ ] `useCallback` / `useMemo` hợp lý
- [ ] Avoid inline styles trong render loop
- [ ] List lớn dùng `FlatList` (virtualized)

---

## Verdict Flow

```markdown
# Review Report — layer-0/task-01

Verdict: **PASS** / **FAIL**

## Checklist Results
- [x] Requirements covered
- [x] RN patterns correct
- [ ] Performance: `FlatList` thiếu keyExtractor

## Issues Found
1. **(blocking)** `FlatList` thiếu `keyExtractor`
2. **(minor)** Component tái render không cần thiết

## Feedback to Loop
- Thêm `keyExtractor` vào FlatList
- Memoize component X
```

**PASS** → git commit → next task
**FAIL** → return to Loop with feedback (max 2 rounds, then escalate)

### 5b. Layer Review (SPEC_VALIDATOR_MODEL)
Sau khi ALL tasks trong layer PASS:
- Cross-check toàn bộ layer với SPECIFICATIONS.md
- Đảm bảo features đã build đúng và đủ theo spec
- **PASS** → DevOps auto-push layer → Human checkpoint
- **FAIL** → trả về Loop với danh sách gaps
