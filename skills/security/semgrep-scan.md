# Semgrep Security Scan

Run a Semgrep static analysis scan on the codebase to catch security vulnerabilities, bugs, and risky patterns before commit.

> **BẮT BUỘC:** Chạy scan này trên mọi task liên quan input handling, auth, storage, hoặc trước khi commit bất kỳ code mới nào.

## When to Use

- Before committing new/changed code
- During code review (reviewer agent)
- When adding API calls, auth logic, or secure storage
- Before push/merge to CI

## Essential Principles

1. **Always use `--metrics=off`** — Semgrep sends telemetry by default; `--config auto` phones home. Every `semgrep` command MUST include `--metrics=off` to prevent data leakage.
2. **Scan plan must be explicit** — State exact rulesets and scan targets before running. For reviewer gate, scan changed files with high-confidence rules.
3. **Use security rulesets** — Include official `p/security-audit`, `p/owasp-top-ten`, and language-specific security rules (TypeScript/JavaScript for RN).

## Commands

```bash
# Quick scan (high-confidence security only) — use for reviewer gate / CI
semgrep --metrics=off --config p/security-audit --config p/owasp-top-ten \
  --severity ERROR --error --quiet \
  --include 'src/**' .

# Full scan (all rulesets) — use before final commit
semgrep --metrics=off --config auto --severity ERROR --error \
  --include 'src/**' .

# TypeScript / React Native specific
semgrep --metrics=off --config p/typescript --config p/javascript \
  --config p/owasp-top-ten --severity ERROR --error --include 'src/**' .

# Scan only changed files (fast reviewer gate)
semgrep --metrics=off --config p/security-audit --severity ERROR \
  --error --include 'src/**' $(git diff --name-only HEAD)
```

## Rules of Thumb (check when reviewing findings)

- **Secrets**: hardcoded API keys, tokens, passwords in client code → FAIL
- **XSS**: `dangerouslySetInnerHTML`/`WebView` with unsanitized input → FAIL
- **Insecure storage**: sensitive data in AsyncStorage/plaintext instead of SecureStore → FAIL
- **Command injection**: `exec`/`child_process` with user input → FAIL
- **Deep link / URI scheme**: unvalidated URIs opening external/sensitive screens → MAJOR

## Output

Produce a findings list with severity. **Any ERROR-severity security finding blocks PASS** in review; resolve before commit.

## Tone

Be specific — "API key hardcoded in api/client.ts:12" not "security issues exist".
