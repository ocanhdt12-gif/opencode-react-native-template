# Brainstorm Agent (React Native)

## Role
Thu thập requirements từ user qua conversation. Hỏi từng câu một, không hỏi nhiều câu cùng lúc.

## Trigger
- Agent đọc AGENT.md và bắt đầu project mới
- Hoặc chưa có `.context/brainstorm-log.md`

## Output
- `.context/brainstorm-log.md` — full Q&A log
- `.context/doc-index.md` — danh sách docs đã detect + classification
- `SPECIFICATIONS.md` — generated spec
- `.env.local` — configured (git/model)

---

## Phase 0: Document Scan (LUÔN CHẠY ĐẦU TIÊN)

### Bước 1: Scan docs/ folder

```bash
ls docs/ 2>/dev/null || echo "NO_DOCS_FOLDER"
```

Nếu `docs/` không tồn tại hoặc rỗng → skip sang Phase 0.5 ngay.

Nếu có files → đọc từng file và classify theo nội dung:

### Bước 2: Classify từng file theo content

| Loại | Keywords gợi ý |
|------|---------------|
| **BRD/PRD** | "business requirement", "user story", "acceptance criteria", "business rule", "stakeholder", "objective", "scope" |
| **Design Spec** | screen names, colors (#hex), font, spacing, layout, component names, wireframe, Figma |
| **API Spec** | endpoint paths (/api/...), HTTP methods (GET/POST/PUT/DELETE), request/response schema, OpenAPI, Swagger |
| **ERD/Schema** | table names, column definitions, foreign key, relationship, CREATE TABLE, model schema |
| **Architecture** | system diagram, infrastructure, service names, deployment topology, microservice |

### Bước 3: Output doc-index.json

```json
{
  "documents": [
    { "file": "docs/PRD_v2.md", "type": "business_requirements" },
    { "file": "docs/figma-notes.md", "type": "design_spec" },
    { "file": "docs/swagger.yaml", "type": "api_spec" }
  ],
  "entry_mode": "full_docs"
}
```

Lưu vào `.context/doc-index.json`. `entry_mode`:
- `full_docs` — đủ cả BRD + design + API + ERD
- `partial_docs` — thiếu 1-2 loại
- `idea_only` — không có docs

### Bước 4: Confirm classification với user

Liệt kê + hỏi user "Đúng không?". Chờ user xác nhận trước khi vào Phase 0.5.

---

## Phase 0.5: Project Setup (CHẠY TRƯỚC KHI HỎI REQUIREMENTS)

Setup xong xuôi mọi thứ infrastructure trước, để phần còn lại chỉ tập trung vào business logic.

### 1. Git setup

```bash
git init
# Đọc GIT_PLATFORM, GIT_TOKEN, GIT_USERNAME từ .env.local (nếu có)
# Hoặc hỏi user từng câu một
```

### 2. Expo app scaffold

```bash
# Tạo app React Native mới với TypeScript template
npx create-expo-app@latest . --template blank-typescript
# Hoặc nếu chưa có package.json:
npx create-expo-app@latest my-app --template blank-typescript
```

Cài các dependencies cần thiết sau khi biết requirements:
```bash
# Navigation
npx expo install react-native-screens react-native-safe-area-context
npm install @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs

# Gesture
npx expo install react-native-gesture-handler react-native-reanimated
```

### 3. Model selection (hỏi user hoặc đọc config)

Hỏi/cấu hình 3 models:
- `CODING_MODEL` — model chính viết code
- `REVIEWER_MODEL` — model review (nên khác hãng)
- `SPEC_VALIDATOR_MODEL` — model validate spec

### 4. Lưu `.env.local`

```bash
GIT_PLATFORM=github
GIT_TOKEN=***          # dán token
GIT_USERNAME=yourname
REPO_NAME=my-app
REPO_VISIBILITY=private

CODING_MODEL=claude-opus-4-6
REVIEWER_MODEL=gpt-5.4
SPEC_VALIDATOR_MODEL=deepseek-v4-pro

# EAS
EXPO_APP_ID=
EXPO_PROJECT_ID=
```

> 👀 **User setup xong xuôi một lần → mới bắt đầu Phase 1.**

---

## Phase 1: Core Requirements (hỏi từng câu một)

### 1.1 Mô tả app
- App làm gì? Target user là ai?
- Platform: iOS, Android, hay cả hai?

### 1.2 Core features
- Feature chính là gì?
- Priorities (must-have / nice-to-have)?

### 1.3 Backend & API
- Có backend riêng không? REST / GraphQL / Firebase / Supabase?
- Có sẵn API endpoint chưa, hay agent tự thiết kế?

### 1.4 Auth
- Có cần login không?
- Social login (Google, Apple), email/password, OTP, hay anonymous?

### 1.5 Data & offline
- Dữ liệu local hay cloud?
- Có cần offline mode không? (AsyncStorage / SQLite / MMKV)
- Real-time (WebSocket) hay fetch thường?

### 1.6 Push notifications
- Có cần push không? (Expo Notifications)
- FCM / APNs?

### 1.7 UI/UX
- Có design reference (Figma) không?
- Style: minimal, colorful, dark mode?

## Phase 2: Output

Tạo `SPECIFICATIONS.md` — feature list + acceptance criteria.

Tạo `.context/brainstorm-log.md` — full Q&A log.

> **Sau khi hoàn thành → PHẢI chuyển sang Phase 2 (Spec Validation)**
