# OpenTelemetry Instrumentation (Mobile)

Configures trace spans, metrics, and log exporters for OpenTelemetry in a **React Native / Expo** app. Use when instrumenting the mobile app with traces, metrics, or network/error observability.

> **BẮT BUỘC:** Áp dụng khi setup/implement observability, telemetry, tracing, network monitoring, hoặc logging integration trong app mobile.

## When to Use

- Setting up OTel/observability in React Native
- Adding network request tracing (API calls)
- Capturing app lifecycle / screen events as spans
- Configuring exporters (OTLP) and offline batching

## Essential Principles

1. **Signals**: traces (network/screen flows), metrics (counters/histograms), logs (structured).
2. **Never write package names/versions from memory** — verify against npm registry first (`npm view <pkg> version`).
3. **Installing SDK is not enough** — must initialize the SDK AND enable exporters.
4. **Sensitive data**: NEVER capture tokens, passwords, PII, or full request bodies in spans/attributes. Sanitize.
5. **Offline-friendly**: mobile networks drop — batch + retry exports, don't lose telemetry silently.

## Commands

```bash
# Verify OTel RN packages exist
npm view @opentelemetry/sdk-trace-base version
npm view @opentelemetry/exporter-trace-otlp-http version
npm view @opentelemetry/instrumentation-fetch version
```

## Setup Sketch (React Native / Expo)

```ts
// instrumentation.ts — init once at app entry
import { WebTracerProvider } from '@opentelemetry/sdk-trace-web';  // RN uses web-compatible tracer
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { FetchInstrumentation } from '@opentelemetry/instrumentation-fetch';

const provider = new WebTracerProvider({
  resource: { 'service.name': process.env.OTEL_SERVICE_NAME ?? 'my-mobile-app' },
});
provider.addSpanProcessor(new BatchSpanProcessor(new OTLPTraceExporter({
  url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT,
})));
provider.register();
// instrument fetch/axios to trace API calls
```

## Checklist

- [ ] OTel SDK initialized at app entry (before network calls)
- [ ] API requests traced (fetch/axios instrumentation)
- [ ] Screen navigation / key flows as spans
- [ ] OTLP exporter endpoint + auth from env (not hardcoded)
- [ ] No tokens/PII/request bodies in attributes
- [ ] Batch + retry for offline tolerance
- [ ] Service name + environment set

## Output

Instrumentation done + checklist. **Any token/PII in span attribute blocks PASS.**

## Tone

Be specific — "access token passed in fetch URL logged as attribute at lib/api.ts:30" not "telemetry leaks data".
