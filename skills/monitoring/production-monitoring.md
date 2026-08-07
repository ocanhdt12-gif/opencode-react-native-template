# Production Monitoring Setup (Mobile)

Set up end-to-end production monitoring for a React Native / Expo app: crash reporting, performance tracking, structured logging, and Alert. Ensures the shipped app is observable and issues surface quickly.

> **BẮT BUỘC:** Áp dụng khi setup/deploy production monitoring, crash reporting, performance, hoặc error tracking cho mobile app.

## When to Use

- Post-launch monitoring setup (store release)
- Configuring crash reporting
- Tracking app performance (slow screens, hangs, ANRs)
- Structured logging + error capture in the app
- Setting up release health / alerting

## Essential Principles

1. **Crash reporting** — capture crashes + non-fatal errors; include stack trace + context (screen, OS version, app version).
2. **Performance tracking** — track screen render times, network latency, frame rate; flag slow interactions.
3. **Secure telemetry** — NEVER log tokens, passwords, PII. Redact request bodies and query params.
4. **Offline batch** — queue telemetry locally and flush when online (mobile networks drop).
5. **Release health** — track crash-free sessions, DAU, error rate per version; alert on regression.

## Setup Sketch (React Native / Expo)

- Initialize monitoring SDK at app entry (before any logic).
- Set `service.name`, `deployment.environment.name`, app version from env.
- Instrument network calls (track latency + status per endpoint).
- Capture JS errors (`ErrorUtils`), unhandled promise rejections, and native crashes.
- Batch + retry exports; never block UI thread.

## Checklist (review gate)

- [ ] Crash reporting enabled on release builds (not just dev)
- [ ] Non-fatal JS errors captured
- [ ] Network requests tracked (latency + status)
- [ ] No tokens/PII/passwords/request bodies in telemetry
- [ ] Offline batching/retry (telemetry survives network drops)
- [ ] App version + environment logged for triage
- [ ] Alert configured on error-rate/crash-free regression

## Output

Monitoring config + checklist. **Any token/PII in telemetry or missing crash capture blocks PASS.**

## Tone

Be specific — "crash reporter only enabled in dev, release builds silently drop errors" not "monitoring incomplete".
