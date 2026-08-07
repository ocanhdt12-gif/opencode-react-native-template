# OpenTelemetry Semantic Conventions (Mobile)

OpenTelemetry semantic convention lookup and naming guidance — attribute and span naming rules for consistent, compliant telemetry from your mobile app.

> **BẮT BUỘC:** Áp dụng khi đặt tên span/attribute/metric, hoặc kiểm tra OTel naming compliance.

## When to Use

- Naming spans, attributes, metrics, log fields
- Checking semantic convention compliance
- Ensuring consistent naming for good dashboards

## Essential Rules

1. **Span names** — use the low-cardinality operation name, not full URL with IDs. `GET /users/{id}` not `GET /users/12345`.
2. **Standard attributes** — prefer released semantic convention groups:
   - `http.request.method`, `http.response.status_code`, `url.path`
   - `service.name`, `service.version`, `deployment.environment.name`
   - `device.model.identifier`, `os.type`, `os.version`
   - `error.type`, `exception.type`
3. **Avoid high-cardinality attributes** in metrics (user IDs, raw URLs, email) — explode cardinality and cost.
4. **Units** — metric names use SI: `db.client.operation.duration` (s), memory (By), latency (s).
5. **Consistency** — same attribute names across app + backend enables correlation.

## Quick Reference

| Concept | Use |
|---------|-----|
| Service identity | `service.name`, `service.version`, `deployment.environment.name` |
| HTTP client | `http.request.method`, `http.response.status_code`, `server.address`, `url.path` |
| Device | `device.model.identifier`, `os.type`, `os.version` |
| Errors | `error.type`, `exception.message` |
| DB | `db.system.name`, `db.collection.name`, `db.operation.name` |

## Checklist

- [ ] Span names low-cardinality (no raw IDs/URLs)
- [ ] `service.name` + `deployment.environment.name` set
- [ ] Device/OS attributes attached (mobile triage)
- [ ] No PII/high-cardinality in attribute values
- [ ] Standard attributes used over custom ones when a convention exists

## Output

Naming applied/compliant. **Any raw-ID span name or high-cardinality/PII attribute blocks PASS.**

## Tone

Be specific — "span uses raw device id GET /user/{uuid} — use GET /user/{id}" not "naming off".
