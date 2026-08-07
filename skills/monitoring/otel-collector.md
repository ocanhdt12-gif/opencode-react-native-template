# OpenTelemetry Collector Config (Mobile)

Author, review, and debug **OpenTelemetry Collector** component configuration — receivers, processors, exporters, connectors, pipeline wiring. Shared setup for backend receiving mobile telemetry.

> **BẮT BUỘC:** Áp dụng khi setup/configure/sửa Collector YAML nhận telemetry từ mobile app.

## When to Use

- Writing/editing Collector `otelcol-config.yaml` (OTLP receiver for mobile)
- Choosing receivers/processors/exporters for a signal
- Debugging why mobile telemetry isn't arriving
- Optimizing sampling / batching for cost

## Essential Principles

1. **Pipeline wiring**: receivers → processors → exporters, per signal (traces/metrics/logs).
2. **Always add a batch processor** — bounds memory/network cost. Never export per-span.
3. **Add a memory limiter processor** — prevents OOM under load.
4. **Sanitize logs** — redact PII/secrets before export (mobile data is privacy-sensitive).
5. **OTLP receiver** must be open on the endpoint the mobile app sends to.

## Example (minimal OTLP pipeline)

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  batch:
    timeout: 5s
    send_batch_size: 1024
  memory_limiter:
    check_interval: 1s
    limit_mib: 500

exporters:
  otlphttp/backend:
    endpoint: ${env:OTEL_EXPORTER_OTLP_ENDPOINT}
    headers:
      Authorization: "Bearer ${env:OTEL_EXPORTER_OTLP_TOKEN}"

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlphttp/backend]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlphttp/backend]
    logs:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlphttp/backend]
```

## Checklist

- [ ] Batch processor present (no per-span export)
- [ ] Memory limiter present
- [ ] `OTEL_EXPORTER_OTLP_ENDPOINT` + auth token from env (not hardcoded)
- [ ] Log sanitization/redaction configured
- [ ] No secrets in YAML

## Output

Config + verification. **Any hardcoded secret or missing batch/memory-limiter blocks PASS.**

## Tone

Be specific — "auth token hardcoded in collector.yaml:14" not "config insecure".
