# Mobile Crash & Performance Monitoring

Detect, capture, and triage mobile app crashes, JS errors, hangs, and performance regressions. Ensures app stability and responsiveness are monitored across iOS/Android.

> **BẮT BUỘC:** Áp dụng khi setup/review crash capture, error tracking, hoặc performance monitoring cho React Native.

## When to Use

- Adding crash/error capture to the app
- Tracking screen performance / slow renders
- Reviewing release stability
- Investigating ANR/freeze reports on Android

## Essential Principles

1. **JS errors** — capture uncaught exceptions + unhandled promise rejections:
   ```ts
   ErrorUtils.setGlobalHandler((error, isFatal) => { /* report */ });
   ```
2. **Native crashes** — needs a native crash reporter (Sentry/OTel native module); JS-only catch misses native/OOM crashes.
3. **Non-fatal errors** — log recoverable errors with context (screen, action, OS, version) — most issues are non-fatal.
4. **Performance** — track screen render time, interaction latency, frame drops (via Performance Monitor / InteractionManager).
5. **De-duplicate** — group crashes by stack signature; alert on new/regressing signatures only.

## Checklist

- [ ] Uncaught JS errors + rejected promises captured
- [ ] Native crash capture enabled (not just JS)
- [ ] Error context: screen, OS version, app version, device
- [ ] Screen render / interaction latency tracked
- [ ] Crashes grouped by stack signature (no alert spam)
- [ ] No sensitive data in error payload
- [ ] Alert on crash-free-session regression

## Output

Crash/perf monitoring + checklist. **Any missing native crash capture or PII in payload blocks PASS.**

## Tone

Be specific — "ANR on Android Home screen not captured — needs native crash reporter" not "crashes not tracked".
