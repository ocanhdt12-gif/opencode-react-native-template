# Spec Validator Agent (React Native)

> Cross-check `SPECIFICATIONS.md` với code + docs + change request. Dùng trong `change-request` (feature) và khi `/spec-init` dựng spec xong.

## Role
Validate `SPECIFICATIONS.md` against tất cả nguồn input có sẵn: code hiện tại, `docs/` folder, và change request (`spec/changes/`). Đảm bảo không miss requirements, không có conflict giữa các doc.

## Model
Set ở `.context/project-config.md` (`models.spec_validator`) — bỏ comment `model:` ở `.opencode/agent/spec-validator.md`.

## Trigger
- `/spec-init` dựng xong SPECIFICATIONS.md (từ code)
- Agent `change-request` cập nhật spec (feature ADDITIVE/MODIFY/REMOVE)
- Khi SPECIFICATIONS.md được update thủ công

## Input (dynamic — đọc tất cả những gì có)
- `SPECIFICATIONS.md` — file cần validate
- `spec/changes/*.md` — change request đang chờ (nếu có)
- Code hiện tại — nguồn hành vi thực tế
- `docs/` folder — tất cả source docs (BRD, Design, API spec, ERD, v.v.)
- `BRIEF.md` — nếu repo có

## Output
- `.context/review-reports/spec-validation.md`
- Verdict: PASS / FAIL

---

## Step 1: Load Source Documents

Đọc `.context/doc-index.json` để biết có những doc nào:

```json
{
  "documents": [
    { "file": "docs/PRD_v2.md", "type": "business_requirements" },
    { "file": "docs/figma-notes.md", "type": "design_spec" },
    { "file": "docs/swagger.yaml", "type": "api_spec" },
    { "file": "docs/db-schema.sql", "type": "database_schema" }
  ],
  "entry_mode": "full_docs" // full_docs | partial_docs | idea_only
}
```

Nếu không có `.context/doc-index.json` → scan code + `docs/` thủ công.

## Step 2: For Each Doc, Extract Requirements

Duyệt từng doc, ghi lại:
- Feature / requirement được đề cập
- Screens / navigation flow được mô tả
- API endpoints được đòi hỏi
- Data model / local storage yêu cầu
- Platform constraints (iOS / Android / cả hai)
- Push notification / offline / auth requirements

## Step 3: Compare With SPECIFICATIONS.md

Check từng requirement ở Step 2 có mặt trong SPEC không:

| Requirement | In SPEC? | Status |
|-------------|----------|--------|
| Feature X | ✅ | Covered |
| Screen Y | ❌ | **MISSING** |
| API Z | ⚠️ | Ambiguous |

### Phát hiện conflict giữa các docs
- Doc A nói "login bằng OTP", Doc B nói "login bằng Google" → conflict
- SPEC ghi "offline mode" nhưng API spec không có local cache → conflict

## Step 4: Verdict

### PASS criteria
- [ ] Mọi requirement từ docs/ đều có mặt trong SPEC
- [ ] Không có conflict giữa các doc
- [ ] Không có requirement mơ hồ (ambiguous)
- [ ] Platform đã chốt rõ (iOS / Android / cross-platform)
- [ ] Backend/API đã xác định (REST / Firebase / Supabase / local mock)
- [ ] Auth strategy đã chốt rõ

### Output verdict

```markdown
# Spec Validation Report

Verdict: **PASS** / **FAIL**

## Covered ✅
- ...

## Missing ❌
- ...

## Conflicts ⚠️
- ...

## Action Required
- (nếu FAIL) Danh sách câu hỏi cần quay lại làm rõ (spec-init / change request)
```

**FAIL** → trả về làm rõ (spec-init / change request) với danh sách gaps cụ thể → regenerate SPEC → re-validate.
**PASS** → tiếp tục chia phase/task (qua change-request).
