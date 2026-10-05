# 📱 AI-Powered Project Template (React Native)

> A model-agnostic multi-agent template for **mobile repos that already have code** (bug/feature/update).

> 📌 **Luồng làm việc:** `/start` (TỰ ĐỘNG): đọc spec (`/spec-init` nếu chưa có) → `/brainstorm` (clear yêu cầu + config) → `/graph` (chia layer/task) → **loop** → `/change` cho mọi thay đổi sau đó (agent `change-request`). `/brainstorm` + `/graph` là lệnh manual, tự động được gọi trong `/start`.
>
> ⭐ **Sau khi spec đã có, MỌI thay đổi (feature mới + fix bug) đi qua MỘT agent: `change-request`.**
> Cửa vào: **`/change`** (đọc hết `spec/changes/*.md`) · `/bug` · `/feature`.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Works with](https://img.shields.io/badge/Works%20with-Cursor%20%7C%20Opencode%20%7C%20Windsurf%20%7C%20Copilot-blue)](https://opencode.ai)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Workflow Router (maintenance)](#workflow-router-maintenance)
- [🔒 Security Integration](#-security-integration)
- [📊 Monitoring Integration](#-monitoring-integration)
- [⚙️ Workflow Skills Integration](#️-workflow-skills-integration)
- [How It Works](#how-it-works)
- [Agent Roles](#agent-roles)
- [Getting Started](#getting-started)
- [Smoke Test Without App Code](#smoke-test-without-app-code)
- [Agent Models](#agent-models)
- [Git & CI/CD](#git--cicd)
- [Change Requests](#change-requests)
- [License](#license)

---

## Overview

This template provides a **multi-agent workflow** for **mobile repos that already have code** (bug / feature / update). Bắt đầu bằng `/start` (tự động): đọc spec → brainstorm (clear yêu cầu + config) → graph (chia layer/task) → loop thực thi task → `/change` cho mọi thay đổi sau đó (agent `change-request`). `AGENTS.md` routes every request; specialized subagents handle build, independent review, spec validation, and close-out.

**Key features:**
- 🧭 **Intent router** — `AGENTS.md` classifies every request (bug / sweep / feature / review / research) before any code is touched
- 🔒 **Bug discipline** — root cause before fix (Iron Law), repro verification, no fix-by-symptom
- 🧩 **Change Request workflow** — classify ADDITIVE/MODIFY/REMOVE → spec delta → phase/task → build/review/validate
- 🤖 **Model-agnostic** — works with any AI tool (Cursor, Opencode, Windsurf, GitHub Copilot); roles are real subagents in `.opencode/agent/`
- 🔍 **Independent review** — reviewer / spec-validator run as separate subagents (`edit: deny`) with different models to reduce bias
- 💾 **Session handoff** — Run Journal in `.context/runs/` + `/resume`, so a new session continues from the last safe step
- 🧠 **Error memory** — agents learn from mistakes, avoid repeating them
- 📱 **Expo-first** — built on Expo + Expo Go, EAS Build for production
- 🗜️ **Auto context compaction** — context compressed every 3 tasks and after each layer

---

## Architecture

This template combines **4 architectural patterns**:

| Pattern | Role |
|---------|------|
| **Event-Driven** | Classify intent, route to correct agent |
| **Graph** | Dependency-driven layer breakdown (Layer 0 → 1 → 2 → ...) |
| **Loop (ReAct)** | Execute tasks, test, fix, retry until passing |
| **Blackboard** | Shared state in `.context/` — enables pause/resume |

---

## Project Structure

```
project-template/
├── AGENTS.md                     ← ✅ Entry point (router) — always loaded
├── SPECIFICATIONS.md             ← Spec canonical (dựng bởi /spec-init từ code)
├── opencode.jsonc                ← Permission gate (builder-strong = ask)
├── .env.local                    ← Git/deploy secrets (gitignored)
├── .env.local.example            ← Template for .env.local
│
├── .opencode/
│   ├── agent/
│   │   ├── builder.md            ← Default code+test (model code chính)
│   │   ├── builder-strong.md     ← Hard task (opt-in only; gated)
│   │   ├── change-request.md     ← ⭐ agent DUY NHẤT cho mọi thay đổi (feature + bug)
│   │   ├── spec-init.md          ← reverse-engineer spec cho project CŨ
│   │   ├── spec-publisher.md     ← tự động phát hành spec + test-scope cho template test
│   │   ├── reviewer.md           ← Independent review (edit: deny)
│   │   └── spec-validator.md     ← Spec/phase cross-check (edit: deny)
│   ├── command/
│   │   ├── brainstorm.md      ← /brainstorm → onboarding repo thật (project-config)
│   │   ├── change.md             ← /change → change request hậu-build (đọc spec/changes/ → agent change-request)
│   │   ├── bug-check.md          ← /bug-check → read-only sweep, list defects
│   │   ├── bug.md                ← /bug  → fix ONE known bug (→ agent change-request)
│   │   ├── feature.md            ← /feature → Change Request workflow (→ agent change-request)
│   │   ├── spec-init.md          ← /spec-init → reverse-engineer spec cho project CŨ
│   │   ├── spec-publish.md       ← /spec-publish → phát hành spec cho template test
│   │   └── resume.md             ← /resume → continue from Run Journal (cross-session)
│   └── plugins/loop-guard.ts     ← Doom-loop guard + usage() gate
│
├── spec/                         ← Spec versioning + change requests
│   ├── CHANGELOG.md              ← lịch sử version spec
│   ├── updates/                  ← 1 file / lần update (spec delta)
│   ├── archive/                  ← spec đóng băng theo release
│   ├── changes/                  ← ⭐ change request hậu-build (pending → archive)
│   └── test-scope/current.json   ← hợp đồng bàn giao cho template test
│
├── docs/                         ← Drop your project docs here (optional)
│   ├── INDEX.md                  ← Canonical vs historical classification
│   ├── BRD.md                    ← Business requirements (template)
│   ├── DESIGN.md                 ← Design spec / Figma notes (template)
│   ├── API_SPEC.md               ← API overview + pointer (no hand-embedded code)
│   ├── ERD.md                    ← Schema overview + pointer
│   ├── PERMISSION.md             ← Roles + guard order (synced from code)
│   ├── smoke-tests/              ← MAINTENANCE_TEMPLATE_SMOKE.md (template, no app code)
│   └── generated/                ← AUTO-GENERATED inventory (do not edit)
│       └── inventory.md
│
├── scripts/
│   ├── generate-inventory.mjs         ← Deterministic inventory generator
│   ├── detect-profile.mjs             ← Detect stack → gợi ý project-config (dùng bởi /brainstorm)
│   └── apply-verify-permissions.mjs   ← Sync allow rule verify command vào reviewer/spec-validator
│
├── .agent/
│   ├── FEATURE_WORKFLOW.md       ← ✅ Workflow entry (bug/feature/update)
│   ├── project-config.md        ← ✅ Project values (branch, pm, checks, models)
│   ├── references/               ← taste-skill-v2.md (anti-slop design reference)
│   ├── spec-validator.md         ← Validate spec vs code/docs
│   ├── spec-init.md              ← /spec-init: reverse-engineer spec cho project CŨ
│   ├── spec-publish.md           ← /spec-publish: phát hành spec + test-scope
│   ├── loop.md                   ← Execute tasks (ReAct pattern)
│   ├── reviewer.md               ← Independent code review
│   ├── error-analyzer.md         ← Root cause analysis + error memory
│   └── change-request.md        ← ⭐ Agent DUY NHẤT cho mọi thay đổi (feature + bug) — đọc spec/changes/
│
├── skills/
│   ├── react-native/
│   │   ├── conventions.md        ← Coding style, folder structure
│   │   ├── stack.md              ← Libraries, tools, versions
│   │   ├── patterns.md           ← Navigation, state, API, testing patterns
│   │   ├── common-errors.md     ← Known issues + fixes
│   │   ├── SKILL.md             ← Wrapper skill (frontmatter)
│   │   └── e2e-maestro.md       ← Maestro E2E testing (YAML flows)
│   ├── security/                 ← 🔒 Security skills (mandatory)
│   │   ├── semgrep-scan.md          ← Static analysis security scan
│   │   ├── api-owasp.md             ← OWASP API Top 10 checklist
│   │   ├── mobile-auth.md           ← Token storage & mobile auth hardening
│   │   ├── sharp-edges.md           ← Secure defaults & footgun config
│   │   ├── supply-chain-audit.md    ← dependency audit + dependency risk
│   │   └── codex-security.md        ← OpenAI Codex Security CLI scan/fix (AI-driven, curated)
│   └── monitoring/               ← 📊 Monitoring skills (mandatory)
│       ├── otel-instrumentation.md  ← OTel traces/metrics/logs (RN)
│       ├── mobile-crash-performance.md ← Crash reporting + performance
│       ├── otel-collector.md        ← Collector config (receivers/exporters)
│       ├── otel-semantic-conventions.md ← OTel naming compliance
│       ├── production-monitoring.md ← Release health, alerting, secure logs
│       └── eas-observe.md          ← EAS Observe: startup perf (TTI/launch) production cho Expo/EAS
│   ├── superpowers/             ← 🧠 Debug Iron Law + TDD test-first (obra/superpowers)
│   ├── brainstorming/           ← 💬 Clear yêu cầu → propose approaches → design doc trước khi code (nửa "clarify" của /brainstorm)
│   ├── ponytail/                ← 🪶 Lazy senior dev ladder, chống over-engineering
│   ├── impeccable/              ← 🎨 UI craft-floor + polish gate (pbakaus) — mobile UI
│   ├── anti-slop/               ← 🧬 Oxlint rules chống low-evidence TS/JS (dmmulroy/anti-slop)
│   ├── ai-readable-codebase/    ← 🧠 Code cho 2 độc giả (người + AI) — README+ARCHITECTURE
│   ├── scalability-architecture/ ← 📦 OPTIONAL scalability tiers — only when user enables the option
│   ├── karpathy-guidelines/      ← ✂️ Surgical changes + think before coding (andrej-karpathy-skills)
│   ├── aislop/                   ← 🧹 AI-slop detection gate (scanaislop/aislop, curated)
│   ├── open-code-review/         ← 🔍 Alibaba OCR gate (alibaba/open-code-review, curated) — CRITICAL → FAIL
│   └── blitzstrike/              ← ⚡ MCP pentest toolbelt (shinthink/blitzstrike, curated) — security optional
│
├── tasks/                        ← Maintenance task board
│   ├── README.md                 ← Task file format + rules
│   ├── feature-<slug>/phase-<N>-task-<NN>.md
│   └── bug-<slug>/               ← scan.md (/bug-check) + phase-<N>-task-<NN>.md
│
├── .context/
│   ├── progress.json             ← Current work-item status (features[] / bugs[])
│   ├── session-policy.json       ← Usage gate thresholds (loop-guard)
│   ├── runs/                     ← Run Journal per task (cross-session resume)
│   │   └── _TEMPLATE.md
│   ├── decisions.md              ← Architecture decisions log
│   ├── spec-notes.md             ← Ghi chú dựng spec (/spec-init)
│   ├── error-memory.md           ← Errors encountered + fixes
│   └── review-reports/           ← Reviewer / spec-validator reports
│
└── .devops/
    ├── templates/
    │   ├── eas-preview.md        ← EAS Preview (internal distribution)
    │   ├── eas-production.md     ← EAS Store submit (App Store + Play Store)
    │   └── generic.md            ← Generic build/deploy notes
    ├── environments.md
    └── deploy-log.md
```

```

---

## Workflow Router (maintenance)

`AGENTS.md` (always-loaded) phân loại mọi request trước khi code:

| Intent (user says…) | Route |
|---|---|
| "fix bug", "lỗi", "broken", crash (đã biết bug) | **`/change`** hoặc **`/bug`** — agent `change-request` (class BUG): root cause → fix → review |
| "soi/kiểm tra màn", "cảm giác nhiều lỗi" | **`/bug-check`** — READ-ONLY sweep, liệt kê defect, dừng chờ duyệt |
| "thêm/sửa/bỏ tính năng" | **`/change`** hoặc **`/feature`** — agent `change-request` (class ADDITIVE/MODIFY/REMOVE) |
| thay đổi đã ghi trong `spec/changes/` | **`/change`** — đọc hết file pending → agent `change-request` |
| "implement feature" (đã có spec/task) | subagent **builder** theo task file |
| "review/check/soát" | subagent **reviewer** — KHÔNG tự sửa code |
| hỏi / điều tra | research-only — không edit |

Không rõ intent → hỏi 1 câu ngắn, không đoán. Chi tiết: `AGENTS.md` + `.agent/FEATURE_WORKFLOW.md`.

## 🔒 Security Integration

The template ships with built-in security rules (read and applied mandatorily by multiple agents) to ensure the generated code meets security standards.

### Skills (`skills/security/`)

| Skill | Purpose |
|-------|---------|
| `semgrep-scan.md` | Static analysis security scan before commit |
| `api-owasp.md` | OWASP API Top 10 checklist for every endpoint |
| `mobile-auth.md` | Token storage (`expo-secure-store`) + mobile auth hardening |
| `sharp-edges.md` | Secure defaults & footgun config/secret/storage |
| `supply-chain-audit.md` | `npm audit` + dependency takeover risk |
| `codex-security.md` | OpenAI Codex Security CLI — AI-driven scan/fix (curated, optional) |

### 3 Mandatory Checkpoints

1. **When coding** (`loop.md`) → read the security skill before writing any file that handles auth/storage/input/API
2. **When reviewing** (`reviewer.md`) → run semgrep + `npm audit` + mobile security checklist before PASS
3. **When pushing** (`devops.md`/CI) → `npm audit --audit-level=high` + semgrep scan in CI

> 🔴 ERROR-severity security finding or high/critical CVE → **DO NOT PASS / do not merge**.

---

## 📊 Monitoring Integration

The template ships with built-in production monitoring (release health + runtime observability) based on **OpenTelemetry** — vendor-neutral, plus crash/performance tracking for mobile.

### Skills (`skills/monitoring/`)

| Skill | Purpose |
|-------|---------|
| `otel-instrumentation.md` | OTel traces/metrics/logs for React Native (network, screens) |
| `mobile-crash-performance.md` | Crash reporting + JS/native errors + performance |
| `otel-collector.md` | Collector config (receivers/processors/exporters) |
| `otel-semantic-conventions.md` | OTel naming compliance (span/attribute) |
| `production-monitoring.md` | Release health, alerting, secure logs, offline batch |
| `eas-observe.md` | EAS Observe — startup perf (TTI/TTR/launch/frame drops), per-route timings, CLI query (Expo/EAS) |

### Keys setup in Phase 0.5

Monitor keys/tokens (OTLP endpoint, service name, Sentry DSN) lưu vào `.env.local` khi `/brainstorm`.

### 3 Mandatory Checkpoints

1. **When coding** (`loop.md`) → read the monitoring skill before writing API/network/performance/logging files
2. **When reviewing** (`reviewer.md`) → check crash capture, no PII in telemetry, OTel naming before PASS
3. **When pushing** (`devops.md`) → verify OTel init + env keys in Health Check

---

## ⚙️ Workflow Skills Integration

The template ships with 2 curated workflow skills (curated from well-known open-source repos — picking the essence, not copying verbatim) to raise code quality throughout the pipeline. These are stack-agnostic methodologies, applicable to React Native.

### Skills (`skills/`)

| Skill | Source | When used / Purpose |
|-------|-------|---------------------|
| `superpowers/` | obra/superpowers (270k⭐) | Every coding task — **Iron Law debug** (no fix without root cause) + **TDD test-first** |
| `brainstorming/` | curated (in-house) | **Trước khi code** feature/thay đổi lớn — clear/clarify yêu cầu từng câu một, propose 2-3 approaches + trade-offs, present design, viết `docs/specs/*-design.md`, user approve trước khi implement (HARD-GATE). Nửa "clear yêu cầu" của `/brainstorm` |
| `ponytail/` | DietrichGebert/ponytail (100k⭐) | Loop while implementing — **lazy senior dev ladder**, stop at the simplest solution, avoid over-engineering |
| `aislop/` | scanaislop/aislop (curated, MIT) | Reviewer reviews **code changes** — deterministic AI-slop scan (narrative comments, swallowed errors, hidden fallbacks, `as any`, duplication, dead code, todo stubs), score 0-100 ≥80 gate, `fix --safe` mechanical, offline no API key |
| `open-code-review/` | alibaba/open-code-review (curated, Apache-2.0) | Reviewer reviews **code changes** — hybrid deterministic + LLM code review, precise line-level comments, built-in ruleset (NPE, thread-safety, XSS, SQLi). Delegation mode = no API key needed; CRITICAL finding → FAIL |
| `blitzstrike/` | shinthink/blitzstrike (curated, MIT) | Reviewer — **optional** MCP pentest toolbelt (BLITZ recon → EAGLE-EYE source trace → STRIKE live validate). Task nhạy cảm (auth/API/input) → chỉ report finding đã verify live |
| `security/codex-security.md` | openai/codex-security (curated, npm `@openai/codex-security`) | Reviewer task nhạy cảm (optional) — AI-driven scan/fix; **CRITICAL verified** → FAIL, ≥3 MAJOR → FAIL; chưa login/không network → ghi `N/A`, không chặn PASS |
| `react-native/e2e-maestro.md` | mobile-dev-inc/maestro (curated, Apache-2.0) | Builder viết E2E + Reviewer — YAML flows (`launchApp`/`tapOn`/`assertVisible`) chạy `maestro test .maestro/`; flow fail → FAIL; không có device/môi trường → ghi `N/A` |

### 3 Mandatory Checkpoints

1. **When debugging** (`error-analyzer.md`) → Iron Law: **NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST**. Read `superpowers/systematic-debugging.md` before proposing a fix. ≥3 failed fixes = suspect the architecture, don't try fix #4
2. **When implementing** (`loop.md`) → TDD test-first (`superpowers/test-driven-development.md`) + ponytail ladder; test fails → write minimal code to pass
3. **When reviewing** (`reviewer.md`) → confirm test-first was followed, no over-engineering (YAGNI/DRY)

> 💡 `ui-ux-pro-max` (web design intelligence) không áp dụng cho RN — mobile dùng design tokens từ Phase 2.5 Design Agent. Còn `impeccable` (UI craft-floor) **có áp dụng** cho mobile UI review (xem Mobile UI Checklist Gate).

---

## How It Works

> 📌 **Start dự án:** `/start` (TỰ ĐỘNG) — đọc spec (`/spec-init` nếu chưa có) → `/brainstorm` (clear yêu cầu + design doc + config) → `/graph` (chia layer/task) → **loop**. Mỗi bước dừng ở human checkpoint. Sau khi build xong → `/change` cho mọi thay đổi.
> `/brainstorm` và `/graph` là lệnh **manual** — chạy tay để chạy lại/update; trong `/start` chúng tự động được gọi.

### Pipeline

```
🚀 /start  (tự động chuỗi khởi tạo)
├─ 1. Đọc spec          → có rồi thì dùng; chưa có → /spec-init (đọc code → SPECIFICATIONS.md + spec/)
├─ 2. /brainstorm       → clear yêu cầu (design doc docs/specs/) + config .context/project-config.md   ← DỪNG chờ approve design
└─ 3. /graph            → chia layer/task (tasks/<slug>/layer-N-task-NN.md) + layer-plan diagram         ← DỪNG chờ duyệt plan
    │
    ▼
Loop Agent — execute từng task (ReAct)
├─ Builder code + test
├─ Test FAIL → Error Analyzer → fix → retry
└─ Test PASS → git commit (rollback point)
    │
    ▼
Review Agent (different model)
├─ FAIL → fix + log error memory
└─ PASS → close-out
    │
    ▼
👀 Human Checkpoint → layer/phase tiếp theo (Layer N+1 chỉ unlock khi Layer N PASS + user approve)

────────── Sau đó: mọi thay đổi ──────────
spec/changes/<file>.md → /change → agent change-request
├─ classify: BUG / ADDITIVE / MODIFY / REMOVE
├─ spec delta + ★ Spec Publisher (spec/updates/ + spec/test-scope/current.json)
├─ loop(builder/reviewer) → spec-validator
└─ progress + commit-first → change file archive
```

> 💡 **Test loop chạy như lần đầu:** mọi thay đổi để lại `spec/test-scope/current.json` → bên template test `/autotest --full` (lần đầu) rồi `/test-scope`, `/regression`.

---

## Agent Roles

**Maintenance (default)** — real subagents live in `.opencode/agent/` (clean context, `edit: deny` where applicable):

| Agent | File | Description |
|-------|------|-------------|
| **Builder** | `.opencode/agent/builder.md` | Implement exactly one task + tests; no scope creep; never commits/pushes |
| **Builder (strong)** | `.opencode/agent/builder-strong.md` | Same, for hard tasks — **opt-in only**, gated by `permission.task` |
| **Reviewer** | `.opencode/agent/reviewer.md` | Independent review, risk level FAST/NORMAL/STRICT (`edit: deny`) |
| **Spec Validator** | `.opencode/agent/spec-validator.md` | Cross-check spec/phase vs requirements (`edit: deny`) |

**Prompt-level agents in `.agent/`:**

| Agent | File | Description |
|-------|------|-------------|
| **Spec Validator** | `.agent/spec-validator.md` | Cross-validates spec against code/docs; detects conflicts |
| **Spec Init** | `.agent/spec-init.md` | Reverse-engineer spec cho project CŨ (chưa có spec) |
| **Spec Publisher** | `.agent/spec-publish.md` | Phát hành spec + test-scope cho template test |
| **Loop Builder** | `.agent/loop.md` | Implements tasks using ReAct (read → plan → code → test → fix) |
| **Reviewer** | `.agent/reviewer.md` | Independent code review with a different model |
| **Error Analyzer** | `.agent/error-analyzer.md` | Root cause analysis; builds error memory to prevent recurrence |
| **Change Request** ⭐ | `.agent/change-request.md` | **Agent DUY NHẤT cho mọi thay đổi** (feature + bug) — đọc `spec/changes/`, spec-publish + test-scope. Cửa vào: `/change`, `/bug`, `/feature` |
| **Spec Init** | `.opencode/agent/spec-init.md` | Reverse-engineer spec cho project CŨ (chưa có spec) |
| **Spec Publisher** | `.opencode/agent/spec-publisher.md` | Tự động bump spec + sinh `spec/test-scope/current.json` cho template test |

---

## Getting Started

### Quickstart (repo đã có code)

```bash
# 1. Clone repo code thật + copy phần template workflow vào (hoặc dùng template này làm base)
git clone <your-existing-repo> my-app && cd my-app

# 2. Chạy /start (chuỗi TỰ ĐỘNG): đọc spec → /brainstorm (clear yêu cầu + config) → /graph (chia layer/task)
#    - /brainstorm: auto-detect stack → điền .context/project-config.md (target_branch, package_manager, verify commands, models theo vai)
#    - /graph: chia layer/task + layer-plan diagram, dừng chờ anh duyệt plan
#    (chạy tay /brainstorm hoặc /graph bất cứ lúc nào để chạy lại/update)

# 3. Restart opencode (config không hot-reload) — bắt buộc sau khi sửa .opencode/

# 4. Sau khi project build xong → dùng /change cho mọi thay đổi
/spec-init                       # đọc code → SPECIFICATIONS.md + spec/ + test-scope (nếu chưa có spec)
/bug-check "soi màn Settings"     # read-only sweep → list defect → anh chọn
/change "thêm tính năng Y"         # ⭐ cửa vào chính cho mọi thay đổi (agent change-request)
/resume                          # làm tiếp từ Run Journal (session mới)
```

### Resuming

```
Read AGENTS.md and resume the project
# hoặc:  /resume feature/<slug>   /resume bug/<slug>
```

Agent reads `.context/progress.json` + Run Journal `.context/runs/<type>-<slug>-<phaseTask>.md` and continues from the last checkpoint.

---

## docs/ Folder

The `docs/` folder is where you drop any existing project documentation. The agent automatically classifies each file by content — no need to follow a naming convention.

### Supported Doc Types

| Type | Detected by | Template file |
|------|-------------|---------------|
| Business Requirements (BRD/PRD) | "user story", "acceptance criteria", "business rule" | `docs/BRD.md` |
| Design Spec | screen names, colors, layout, component names | `docs/DESIGN.md` |
| API Spec | endpoints, request/response schemas, HTTP methods | `docs/API_SPEC.md` |
| Database Schema (ERD) | table definitions, relationships, indexes | `docs/ERD.md` |

### docs/INDEX.md (optional)

If you want to skip auto-detection, create `docs/INDEX.md`:

```markdown
- PRD_v2.md: business requirements
- figma-export.md: design reference
- swagger.yaml: api spec
```

---

## Smoke Test Without App Code

Repo template (chưa có app code) vẫn test được workflow — `docs/smoke-tests/MAINTENANCE_TEMPLATE_SMOKE.md`:

- **A. `/bug-check` Smoke** — giả lập màn Settings: phải tạo đúng `tasks/bug-<slug>/scan.md`, không gọi Builder, không sửa file khác (final `git status --short` check).
- **A2. Cross-Cutting** — dark mode toàn hệ thống: phải enumerate toàn bộ surface theo `source_roots`, CẤM sampling, có `## Coverage` + `## Chưa soi`.
- **B. Permission gates** — `git push origin main` deny, `--force` deny, `db push` deny; `builder-strong` bị `ask`.
- **C. Close-out** — Reviewer PASS + report đúng tên + progress.json cập nhật → commit; FAIL → không commit.

---

## Agent Models

Models theo vai nằm trong **frontmatter** `.opencode/agent/*.md` (`builder`, `builder-strong`,
`reviewer`, `spec-validator`) — khai ở `.context/project-config.md` (`models:`) rồi **bỏ comment**
dòng `model:` và copy giá trị sang. Dùng **provider khác họ** giữa builder và reviewer để tránh bias.

| Agent | File | Gợi ý |
|-------|------|-------|
| Builder | `.opencode/agent/builder.md` | model code chính |
| Builder (strong) | `.opencode/agent/builder-strong.md` | model mạnh hơn, **opt-in only** |
| Reviewer | `.opencode/agent/reviewer.md` | provider khác builder |
| Spec Validator | `.opencode/agent/spec-validator.md` | họ thứ 3 nếu có |

> ⚠️ Biến model trong `.env.local` KHÔNG có tác dụng với opencode (không đọc). Đã bỏ.
> Sau khi sửa `.opencode/*` hoặc `opencode.jsonc`, **restart opencode** (config không hot-reload).

---

## Git & CI/CD

### Branch Strategy (maintenance)

- **Default staging-direct**: làm việc trên `target_branch` (khai trong `.context/project-config.md`), commit trực tiếp ở đó.
- **Cấm push `forbidden_branch`** (mặc định `main`) — chặn cứng ở `opencode.jsonc` → `permission.bash`.
- **Cấm `--force` / `-f`** — chặn cứng.
- Feature branch (`feature/<slug>` / `bug/<slug>`) chỉ khi user yêu cầu rõ; PR chỉ mở khi user yêu cầu.
- `auto_push_after_pass: true` chỉ auto-push `target_branch` sau PASS — commit sau PASS luôn bắt buộc.

### EAS Build Profiles (`.devops/templates/`)

| Profile | Purpose |
|---------|---------|
| **Preview** | Internal distribution via EAS (testers, QA) |
| **Production** | App Store + Play Store submission |

### Git Token Setup

| Platform | Token URL | Scope |
|----------|-----------|-------|
| **GitHub** | https://github.com/settings/tokens | `repo` |
| **GitLab** | https://gitlab.com/-/user_settings/personal_access_tokens | `api` |
| **Bitbucket** | https://bitbucket.org/account/settings/app-passwords | `Repositories Read+Write` |

---

## Change Requests

After the project is complete, use **Change Request Agent** for modifications:

| Type | Example |
|------|---------|
| **ADDITIVE** | "Add offline sync feature" |
| **MODIFY** | "Change checkout flow" |
| **REMOVE** | "Remove push notifications" |

SPECIFICATIONS.md is automatically versioned on each change.

---

## License

MIT
