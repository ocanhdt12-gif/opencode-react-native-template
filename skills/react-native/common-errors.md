# React Native — Common Errors & Fixes

> Error memory cho stack React Native/Expo. Append khi gặp lỗi mới.

## 1. Module version conflict (npx expo install)
**Error:** `Unable to resolve module ...` / version mismatch
**Root cause:** Cài package bằng `npm install` thay vì `npx expo install`, dẫn đến version không tương thích SDK.
**Fix:**
```bash
npx expo install <package>  # dùng cái này, tự chọn version đúng SDK
```

## 2. Metro bundle lỗi / cache cũ
**Error:** `Unable to resolve module` / stale bundle sau khi đổi file
**Fix:**
```bash
npx expo start -c   # clear cache
# hoặc
rm -rf .expo node_modules/.cache
```

## 3. SafeAreaView deprecated warning
**Error:** `SafeAreaView has been deprecated`
**Fix:** Dùng `react-native-safe-area-context`:
```tsx
import { SafeAreaView } from 'react-native-safe-area-context';
```

## 4. Hermes error / React Native version
**Error:** `Invariant Violation: Tried to register two views with the same name`
**Fix:** Khôi phục version native modules + restart:
```bash
npx expo install --fix
npx expo start -c
```

## 5. Keyboard/TextInput bị che
**Error:** Input bị che bởi keyboard trên iOS
**Fix:**
```tsx
<KeyboardAvoidingView behavior={Platform.OS === 'ios' ? 'padding' : 'height'}>
  {/* form */}
</KeyboardAvoidingView>
```

## 6. FlatList không render / không scroll
**Root cause:** Thiếu `keyExtractor` hoặc dùng `ScrollView` cho list lớn
**Fix:** Dùng `FlatList` + `keyExtractor` bắt buộc.

## 7. Push notification không hoạt động trên simulator
**Fix:** Push notification không hoạt động trên iOS simulator. Test trên device thật (Expo Go / Dev Client).

## 8. `expo-image-picker` cần permission config
**Fix:** Thêm vào `app.json`:
```json
{
  "expo": {
    "plugins": [
      ["expo-image-picker", { "photosPermission": "..." }]
    ]
  }
}
```

## 9. Memory leak: listeners không cleanup
**Warning:** `Can't perform a React state update on an unmounted component`
**Fix:**
```tsx
useEffect(() => {
  const sub = evt.addListener(cb);
  return () => sub.remove();  // cleanup
}, []);
```

## 10. Deep link không hoạt động
**Fix:** Đảm bảo `scheme` trong `app.json` + config trước khi build production:
```json
{ "expo": { "scheme": "myapp" } }
```

## 11. `npx tsc` báo lỗi types React Navigation
**Fix:** Import types đúng:
```tsx
import { NativeStackScreenProps } from '@react-navigation/native-stack';
```

## 12. EAS build fail vì thiếu app.json config
**Fix:** Chạy `npx eas-cli init` để tạo project ID + verify `app.json` có `name`, `slug`, `version`.

## 13. Android cleartext HTTP bị chặn (development)
**Error:** `CLEARTEXT communication to ... not permitted`
**Fix (dev only):** Dùng HTTPS cho API, hoặc config `usesCleartextTraffic` trong debug manifest. KHÔNG dùng cho production.

## 14. Reanimated báo lỗi babel plugin
**Error:** `[Reanimated] Mismatch between JavaScript part and native part`
**Fix:**
```bash
# Đảm bảo babel.config.js có plugin
# plugins: ['react-native-reanimated/plugin']
npx expo start -c
```
