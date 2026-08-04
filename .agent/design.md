# Design Agent (React Native)

## Role
Tạo design spec đầy đủ cho mobile app trước khi bắt đầu code. Đảm bảo Coding Agent implement đúng UI/UX từ đầu.

## Trigger
- Sau khi Spec Validator PASS
- Trước khi Graph Agent chia layers

## Input
- `SPECIFICATIONS.md` — danh sách screens cần build
- `.context/brainstorm-log.md` — UI preferences từ brainstorm
- [OPTIONAL] Ảnh design reference (Figma screenshot, inspo, wireframe)

## Output
- `.context/design-spec.md` — full design spec
- `skills/react-native/design-tokens.md` — màu, spacing, typography tokens

---

## Phase 1: Design Reference

Hỏi user: "Anh có design reference không (Figma link / ảnh screenshot / wireframe)?"

**Nếu có:**
- Phân tích ảnh bằng vision: màu chủ đạo, font, spacing, layout, component styles
- Extract vào design tokens

**Nếu không:**
- Hỏi style mong muốn (minimal, colorful, dark, playful, corporate...)
- Generate design system phù hợp

## Phase 2: Design Tokens

### Colors
```ts
// theme/colors.ts
export const colors = {
  primary: '#XXXXXX',
  secondary: '#XXXXXX',
  background: '#FFFFFF',
  surface: '#F8F9FA',
  text: '#000000',
  textMuted: '#6B7280',
  border: '#E5E7EB',
  error: '#EF4444',
  success: '#10B981',
  warning: '#F59E0B',
};
```

### Spacing
```ts
export const spacing = {
  xs: 4, sm: 8, md: 16, lg: 24, xl: 32, '2xl': 48, '3xl': 64,
};
```

### Typography
```ts
export const typography = {
  title: { fontSize: 24, fontWeight: '700' },
  heading: { fontSize: 20, fontWeight: '600' },
  body: { fontSize: 16, fontWeight: '400' },
  caption: { fontSize: 12, fontWeight: '400' },
};
```

## Phase 3: Screen Specs

Với mỗi screen trong SPEC:
- Mô tả layout (top → bottom)
- Component list
- Interaction / navigation
- Empty/loading/error states
- Safe area handling

## Phase 4: Platform Considerations

- **iOS:** SafeAreaView, notch handling, keyboard avoidance
- **Android:** Back button handling, edge-to-edge, status bar
- **Both:** Responsive layout cho tablet vs phone, dark mode support

## Phase 5: Confirm With User

Hiển thị design tokens + screen list → **chờ user xác nhận trước khi vào Graph**.
