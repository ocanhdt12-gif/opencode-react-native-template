# Supply Chain Risk Audit (Mobile)

Identifies dependencies at heightened risk of exploitation or takeover. Assesses the dependency attack surface and flags unmaintained, low-popularity, or risky packages.

> **BẮT BUỘC:** Chạy `npm audit` + review dependency risk trong mọi task install dependency, và ở bước CI/DevOps trước khi deploy.

## When to Use

- After adding/updating any dependency (npm, Expo SDK packages)
- Before push/merge to CI
- During final review / pre-deploy
- When onboarding a new npm/Expo package

## When NOT to Use

- Active vulnerability scanning at runtime (use `npm audit` live) — this complements it
- License compliance auditing

## Risk Criteria (a dependency is high-risk if it has ANY of:)

- **Single/unknown maintainer** — not organization-backed; anonymous identity.
- **Unmaintained/stale** — no updates for a long period, archived, or explicitly deprecated.
- **Low popularity** — few stars/downloads relative to peers.
- **High-risk features** — FFI, deserialization, third-party code execution, native modules with eval.
- **Past high/critical CVEs** — especially many relative to popularity.
- **No security contact** — missing `.github/SECURITY.md`, `CONTRIBUTING.md`, or security disclosure.

## Commands

```bash
# Active vulnerability scan — REQUIRED gate
npm audit --audit-level=high
# → exit non-zero if high/critical vulns (blocks CI)

npx expo-doctor   # check Expo dependency compatibility

# List direct deps + their popularity for review
npm ls --depth=0
npm view <pkg> time.modified maintainers 2>/dev/null
```

## Workflow

1. Run `npm audit --audit-level=high` — **any high/critical vuln blocks PASS/commit** until resolved or documented exception.
2. For newly added direct dependencies, evaluate against the Risk Criteria above.
3. Flag high-risk dependencies with the specific risk factor + suggested alternative (more popular / well-maintained).
4. Summarize overall dependency security posture + recommendations.

## Output

Findings list with severity. **High/critical CVE or flagged high-risk new dep blocks PASS.**

## Tone

Be specific — "`rn-qr-scanner-fork@1.0.0` single anonymous maintainer, unmaintained 1.5yr, suggests `expo-camera`" not "some deps are risky".
