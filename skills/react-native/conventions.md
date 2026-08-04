# React Native — Coding Conventions

## Folder Structure

```
app/  (expo-router)  HOẶC  src/  (react-navigation)
├── app/                     # Routes (expo-router)
│   ├── _layout.tsx          # Root layout (Stack/Tabs)
│   ├── index.tsx            # Home screen
│   ├── (auth)/              # Auth group
│   └── (tabs)/              # Tab navigation
├── components/              # Reusable UI components
│   ├── ui/                  # Generic UI (Button, Input, Card)
│   └── forms/               # Form-specific
├── screens/                 # (react-navigation) Full screens
├── navigation/              # Navigation config
├── services/                # API calls, external services
├── hooks/                   # Custom React hooks
├── store/                   # Zustand stores
├── types/                   # TypeScript types/interfaces
├── theme/                   # Colors, spacing, typography tokens
├── utils/                   # Helper functions
└── constants/               # App constants
```

## Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Components | PascalCase | `UserCard.tsx` |
| Hooks | camelCase + use | `useAuth.ts` |
| Services | camelCase | `apiClient.ts` |
| Types | PascalCase | `UserProfile.ts` |
| Constants | UPPER_SNAKE | `API_BASE_URL` |
| Files (components) | PascalCase | `Button.tsx` |
| Files (non-component) | camelCase | `apiClient.ts` |

## Component Rules

```tsx
// ✅ Memoize when props are stable & re-render is costly
const UserCard = React.memo(({ user }: { user: User }) => {
  return <View><Text>{user.name}</Text></View>;
});

// ✅ Extract styles outside component (avoid inline re-creation)
const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
});

// ✅ Use typed props
type ButtonProps = {
  title: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary';
};
```

## Hooks Rules

```tsx
// ✅ useCallback for stable function refs passed to memoized children
const handlePress = useCallback(() => {
  // ...
}, [deps]);

// ✅ useMemo for expensive computations
const filtered = useMemo(() => list.filter(...), [list]);

// ✅ Cleanup listeners to avoid memory leaks
useEffect(() => {
  const sub = someListener.subscribe(cb);
  return () => sub.unsubscribe();  // cleanup!
}, []);
```

## Styling Conventions

- Use `StyleSheet.create` for static styles
- Use theme tokens, **không hardcode** màu/space inline trừ khi cần thiết
- Dark mode: support qua theme provider nếu requirement yêu cầu
- Safe area: dùng `SafeAreaView` từ `react-native-safe-area-context`

## Platform-Specific

```tsx
import { Platform } from 'react-native';

const isIOS = Platform.OS === 'ios';
// Dùng cho khác biệt behavior, không dùng cho layout (dùng Flexbox)
```

## ✅ Do
- Types đầy đủ, hạn chế `any`
- Handle loading / empty / error states cho mọi data fetch
- `keyExtractor` bắt buộc cho FlatList
- Tách business logic khỏi UI (services/hooks)
- Error boundaries cho screens

## ❌ Don't
- Không hardcode secrets / API keys trong code
- Không dùng `ScrollView` cho list lớn (dùng FlatList)
- Không block main thread với heavy compute
- Không để listeners không cleanup
- Không import types từ `@react-navigation/native` thiếu typing (dùng `NativeStackScreenProps`)
