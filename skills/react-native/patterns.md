# React Native — Patterns

## Navigation Patterns

### Stack + Tabs (expo-router hoặc React Navigation)

```tsx
// expo-router: app/_layout.tsx
import { Stack } from 'expo-router';
import { SafeAreaProvider } from 'react-native-safe-area-context';

export default function RootLayout() {
  return (
    <SafeAreaProvider>
      <Stack>
        <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
        <Stack.Screen name="modal" options={{ presentation: 'modal' }} />
      </Stack>
      {/* <StatusBar style="dark" /> */}
    </SafeAreaProvider>
  );
}
```

```tsx
// React Navigation: navigation/index.tsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

export type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Settings: undefined;
};

const Stack = createNativeStackNavigator<RootStackParamList>();
```

**Typed navigation:**
```tsx
import { NativeStackScreenProps } from '@react-navigation/native-stack';

type Props = NativeStackScreenProps<RootStackParamList, 'Profile'>;
export default function ProfileScreen({ route, navigation }: Props) {
  const { userId } = route.params;
}
```

## Data Fetching Pattern (TanStack Query)

```tsx
// services/userService.ts
import axios from 'axios';

export const fetchUser = async (id: string): Promise<User> => {
  const { data } = await axios.get(`/users/${id}`);
  return data;
};

// hooks/useUser.ts
import { useQuery } from '@tanstack/react-query';

export function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    enabled: !!userId,
  });
}

// Component sử dụng
const { data, isLoading, error } = useUser(userId);
if (isLoading) return <LoadingSpinner />;
if (error) return <ErrorView message={error.message} />;
```

## State Management Pattern (Zustand)

```tsx
// store/authStore.ts
import { create } from 'zustand';

interface AuthState {
  user: User | null;
  token: string | null;
  setAuth: (user: User, token: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  token: null,
  setAuth: (user, token) => set({ user, token }),
  logout: () => set({ user: null, token: null }),
}));
```

## Auth Pattern (JWT)

```tsx
// services/authService.ts
import * as SecureStore from 'expo-secure-store';

const TOKEN_KEY = 'auth_token';

export async function saveToken(token: string) {
  await SecureStore.setItemAsync(TOKEN_KEY, token);
}

export async function getToken(): Promise<string | null> {
  return await SecureStore.getItemAsync(TOKEN_KEY);
}

export async function clearToken() {
  await SecureStore.deleteItemAsync(TOKEN_KEY);
}
```

**API client với token:**
```tsx
// services/apiClient.ts
import axios from 'axios';
import { getToken } from './authService';

export const apiClient = axios.create({ baseURL: 'https://api.example.com' });

apiClient.interceptors.request.use(async (config) => {
  const token = await getToken();
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

apiClient.interceptors.response.use(
  (res) => res,
  async (error) => {
    if (error.response?.status === 401) {
      // handle logout / refresh token
    }
    return Promise.reject(error);
  }
);
```

## Local Storage Pattern

```tsx
// Simple key-value: AsyncStorage
import AsyncStorage from '@react-native-async-storage/async-storage';

export const store = {
  get: async <T>(key: string): Promise<T | null> => {
    const raw = await AsyncStorage.getItem(key);
    return raw ? JSON.parse(raw) : null;
  },
  set: async <T>(key: string, value: T) => {
    await AsyncStorage.setItem(key, JSON.stringify(value));
  },
  remove: async (key: string) => AsyncStorage.removeItem(key),
};

// Sensitive data: SecureStore (tokens, credentials)
// Relational / offline: expo-sqlite
```

## Forms Pattern (React Hook Form + Zod)

```tsx
import { useForm, Controller } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

type FormData = z.infer<typeof schema>;

function LoginForm() {
  const { control, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data: FormData) => {
    // call auth service
  };

  return (
    <View>
      <Controller
        control={control}
        name="email"
        render={({ field: { onChange, value } }) => (
          <TextInput
            value={value}
            onChangeText={onChange}
            placeholder="Email"
            keyboardType="email-address"
          />
        )}
      />
      {errors.email && <Text style={{ color: 'red' }}>{errors.email.message}</Text>}
      <Button title="Login" onPress={handleSubmit(onSubmit)} />
    </View>
  );
}
```

## Push Notification Pattern (expo-notifications)

```tsx
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowBanner: true,
    shouldShowList: true,
    shouldPlaySound: false,
    shouldSetBadge: false,
  }),
});

export async function registerForPushNotifications() {
  if (!Device.isDevice) return;

  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;

  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }

  if (finalStatus !== 'granted') return;

  const token = (await Notifications.getExpoPushTokenAsync()).data;
  // gửi token lên server
  console.log('Push token:', token);
}
```

## Image Handling

```tsx
import { Image } from 'expo-image'; // hoặc react-native Image

// Lazy + resize
<Image
  source={{ uri: imageUrl }}
  contentFit="cover"
  transition={200}
  style={{ width: '100%', height: 200 }}
  placeholder={{ blurhash: '...' }}
/>
```

## List Pattern (FlatList — bắt buộc cho list lớn)

```tsx
<FlatList
  data={items}
  keyExtractor={(item) => item.id}
  renderItem={({ item }) => <ItemCard data={item} />}
  ListEmptyComponent={<EmptyState />}
  ListFooterComponent={<LoadingMore />}
  onEndReachedThreshold={0.5}
  onEndReached={loadMore}
  contentContainerStyle={{ padding: 16 }}
/>
```

## Error Handling

```tsx
// Error boundary
class ErrorBoundary extends React.Component<Props, State> {
  state = { hasError: false };
  static getDerivedStateFromError() {
    return { hasError: true };
  }
  render() {
    if (this.state.hasError) {
      return <ErrorView onRetry={() => this.setState({ hasError: false })} />;
    }
    return this.props.children;
  }
}
```

## API Error Handling

```tsx
export class ApiError extends Error {
  status: number;
  constructor(status: number, message: string) {
    super(message);
    this.status = status;
  }
}

try {
  const user = await fetchUser(id);
} catch (e) {
  if (e instanceof ApiError && e.status === 404) {
    // show "not found"
  } else {
    // generic error toast
  }
}
```
