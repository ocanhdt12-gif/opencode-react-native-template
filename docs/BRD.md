# Business Requirements Document (BRD)

> Fill in this template if your project has formal business requirements.
> Agent will auto-detect and use this file during brainstorm.

---

## Project Overview

**Project Name:** [Project name]
**Version:** 1.0
**Date:** YYYY-MM-DD
**Author:** [Name]

## Business Context

**Problem Statement:**
[What problem are we solving? Why does it exist?]

**Business Objectives:**
- [Objective 1]
- [Objective 2]

**Success Metrics:**
- [KPI 1 — e.g. 1000 users in first month]
- [KPI 2]

---

## Stakeholders

| Role | Name | Responsibility |
|------|------|----------------|
| Product Owner | | Final decisions |
| Developer | | Implementation |
| End User | | Primary consumer |

---

## Scope

### In Scope
- [Feature / module 1]
- [Feature / module 2]

### Out of Scope
- [Explicitly excluded items]

---

## Functional Requirements

### Module 1: [Name]
**FR-001:** [Requirement description]
- Priority: High / Medium / Low
- Acceptance Criteria: [When is this done?]

**FR-002:** [Requirement description]
- Priority:
- Acceptance Criteria:

### Module 2: [Name]
**FR-010:** [Requirement description]
- Priority:
- Acceptance Criteria:

---

## Non-Functional Requirements

| Category | Requirement |
|----------|------------|
| Performance | Page load < 2s, API response < 500ms |
| Security | JWT auth, HTTPS only, input validation |
| Scalability | Support 10k concurrent users |
| Availability | 99.9% uptime |
| Accessibility | WCAG 2.1 AA |

---

## User Roles & Permissions

| Role | Permissions |
|------|------------|
| Admin | Full access |
| User | Read + write own data |
| Guest | Read only |

---

## Business Rules

- **BR-001:** [e.g. Order cannot be cancelled after 24h]
- **BR-002:** [e.g. User must verify email before purchasing]

---

## Constraints

- **Technical:** [e.g. Must use existing PostgreSQL database]
- **Budget:** [e.g. No paid third-party services]
- **Timeline:** [e.g. MVP in 8 weeks]
- **Regulatory:** [e.g. GDPR compliance required]

---

## Assumptions

- [Assumption 1]
- [Assumption 2]

---

## Open Questions

- [ ] [Question that needs answer before development]
- [ ] [Another open question]
