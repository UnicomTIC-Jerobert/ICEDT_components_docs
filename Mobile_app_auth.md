Excellent. Moving to the mobile app is the perfect next step. You are absolutely right in your analysis—we can and **should reuse a significant amount of the logic** we've already built.

The `apiClient` and the token refresh flow are completely reusable. This is a huge advantage of separating your logic into distinct layers.

Let's create a clear, step-by-step plan to integrate the full authentication and progress flow into your React Native (Expo) mobile app. The process will be very similar to what we did for the admin panel, but adapted for a mobile environment.

---

### **Plan for Mobile App Integration**

We'll follow the same successful pattern:
1.  **Set up the API Client and Services:** Reuse our API logic.
2.  **Implement Auth State Management:** Use React Context for a global user session.
3.  **Create Login and Protected Screens:** Build the UI for login and protect the main app content.

---

### **Step 1: Reuse API Logic and Services**

Copy the following files and folders from your `admin-app` project directly into your Expo project's source directory (e.g., `expo-app/src/`):

1.  **`src/api/` folder:**
    *   `apiClient.ts`
    *   `axiosClient.ts`
    *   `authApi.ts`
    *   `progressApi.ts` (You'll need to create this)
    *   `lessonApi.ts` (For fetching lesson summaries)
2.  **`src/types/` folder:**
    *   `auth.ts`
    *   `progress.ts` (Create this for progress-related DTOs)
    *   `lesson.ts`, etc.
3.  **`src/services/` folder:**
    *   `authService.ts`

**Crucial Mobile-Specific Change:**
In a mobile app, `localStorage` is not available. The standard for persistent, secure storage in React Native is **`AsyncStorage`**.

*   **Install AsyncStorage:**
    ```bash
    npx expo install @react-native-async-storage/async-storage
    ```

*   **Modify `src/services/authService.ts` for React Native:**
    ```typescript
    import AsyncStorage from '@react-native-async-storage/async-storage';
    import { AuthResponse } from '../types/auth';

    const TOKEN_KEY = 'accessToken';
    const REFRESH_TOKEN_KEY = 'refreshToken';

    export const saveTokens = async (tokens: AuthResponse): Promise<void> => {
        if (tokens.token) {
            await AsyncStorage.setItem(TOKEN_KEY, tokens.token);
        }
        if (tokens.refreshToken) {
            await AsyncStorage.setItem(REFRESH_TOKEN_KEY, tokens.refreshToken);
        }
    };

    export const getAccessToken = async (): Promise<string | null> => {
        return await AsyncStorage.getItem(TOKEN_KEY);
    };

    export const getRefreshToken = async (): Promise<string | null> => {
        return await AsyncStorage.getItem(REFRESH_TOKEN_KEY);
    };

    export const clearTokens = async (): Promise<void> => {
        await AsyncStorage.removeItem(TOKEN_KEY);
        await AsyncStorage.removeItem(REFRESH_TOKEN_KEY);
    };
    ```
    *Note: `AsyncStorage` is asynchronous, so all functions now return a `Promise`.* You will need to update your `axiosClient.ts` to `await` these calls (e.g., `const token = await getAccessToken();`).

---

### **Step 2: Create the `AuthContext` for React Native**

This will be almost identical to the web version, but it will use the new `async` `authService`.

**File: `expo-app/src/context/AuthContext.tsx`**
```typescript
import React, { createContext, useState, useContext, ReactNode, useEffect } from 'react';
import { AuthResponse, LoginRequest } from '../types/auth';
import * as authApi from '../api/authApi';
import * as authService from '../services/authService';
import { jwtDecode } from 'jwt-decode'; // npm install jwt-decode

// ... (UserInfo and AuthContextType interfaces are the same)

export const AuthProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
    const [user, setUser] = useState<UserInfo | null>(null);
    const [accessToken, setAccessToken] = useState<string | null>(null);
    const [isLoading, setIsLoading] = useState<boolean>(true);

    useEffect(() => {
        const bootstrapAsync = async () => {
            const token = await authService.getAccessToken();
            if (token) {
                // You should also validate token expiry here in a real app
                setAccessToken(token);
                // Decode user info from the token
                const decoded: { unique_name: string, role: string } = jwtDecode(token);
                setUser({ username: decoded.unique_name, role: decoded.role });
            }
            setIsLoading(false);
        };
        bootstrapAsync();
    }, []);

    const login = async (credentials: LoginRequest): Promise<AuthResponse> => {
        const response = await authApi.login(credentials);
        if (response.isSuccess && response.token) {
            await authService.saveTokens(response);
            setAccessToken(response.token);
            const decoded: { unique_name: string, role: string } = jwtDecode(response.token);
            setUser({ username: decoded.unique_name, role: decoded.role });
        }
        return response;
    };

    const logout = async () => {
        await authService.clearTokens();
        setAccessToken(null);
        setUser(null);
    };

    const value = { isAuthenticated: !!user, user, accessToken, isLoading, login, logout };

    return (
        <AuthContext.Provider value={value}>
            {children}
        </AuthContext.Provider>
    );
};

// useAuth hook remains the same
export const useAuth = () => { /* ... */ };
```

---

### **Step 3: Implement the App's Root Navigator**

This is the equivalent of the `App.tsx` router in the admin panel. It will decide whether to show the Login screen or the main app based on the authentication state.

We will use **React Navigation**.

*   **Install React Navigation:**
    ```bash
    npx expo install @react-navigation/native @react-navigation/native-stack react-native-screens react-native-safe-area-context
    ```

**File: `expo-app/src/navigation/RootNavigator.tsx`**
```typescript
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { useAuth } from '../context/AuthContext';
import { View, ActivityIndicator } from 'react-native';

// Import your screen components
import LoginScreen from '../screens/LoginScreen';
import HomeScreen from '../screens/HomeScreen'; // The main screen after login
import LessonScreen from '../screens/LessonScreen';

const Stack = createNativeStackNavigator();

const RootNavigator = () => {
    const { isAuthenticated, isLoading } = useAuth();

    // Show a loading spinner while the AuthContext is checking for a token
    if (isLoading) {
        return (
            <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
                <ActivityIndicator size="large" />
            </View>
        );
    }

    return (
        <NavigationContainer>
            <Stack.Navigator>
                {isAuthenticated ? (
                    // Screens for logged-in users
                    <>
                        <Stack.Screen name="Home" component={HomeScreen} />
                        <Stack.Screen name="Lesson" component={LessonScreen} />
                        {/* Add other authenticated screens here */}
                    </>
                ) : (
                    // Screens for logged-out users
                    <>
                        <Stack.Screen name="Login" component={LoginScreen} options={{ headerShown: false }} />
                        {/* You can add RegisterScreen, ForgotPasswordScreen here */}
                    </>
                )}
            </Stack.Navigator>
        </NavigationContainer>
    );
};

export default RootNavigator;
```

#### **Step 4: Wrap your app with the `AuthProvider`**

**File: `expo-app/App.tsx` (or your app's entry point)**
```typescript
import React from 'react';
import { AuthProvider } from './src/context/AuthContext';
import RootNavigator from './src/navigation/RootNavigator';

export default function App() {
    return (
        <AuthProvider>
            <RootNavigator />
        </AuthProvider>
    );
}
```

#### **Step 5: Create the `LoginScreen.tsx`**

This will be a simple form similar to the web version, but using React Native components.
```typescript
// In src/screens/LoginScreen.tsx
import React, { useState } from 'react';
import { View, TextInput, Button, Text, StyleSheet } from 'react-native';
import { useAuth } from '../context/AuthContext';

const LoginScreen = () => {
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');
    const [error, setError] = useState('');
    const { login } = useAuth();

    const handleLogin = async () => {
        try {
            const response = await login({ username, password });
            if (!response.isSuccess) {
                setError(response.message);
            }
            // If successful, the RootNavigator will automatically switch screens
        } catch (err) {
            setError('An error occurred.');
        }
    };

    return (
        <View style={styles.container}>
            <Text style={styles.title}>Login</Text>
            <TextInput
                style={styles.input}
                placeholder="Username"
                value={username}
                onChangeText={setUsername}
            />
            <TextInput
                style={styles.input}
                placeholder="Password"
                value={password}
                onChangeText={setPassword}
                secureTextEntry
            />
            {error ? <Text style={styles.error}>{error}</Text> : null}
            <Button title="Sign In" onPress={handleLogin} />
        </View>
    );
};

const styles = StyleSheet.create({ /* ... your styles here ... */ });

export default LoginScreen;
```

This provides a complete, step-by-step plan to integrate the exact same authentication architecture into your Expo app. The core logic remains the same; only the UI components and the secure storage mechanism (`AsyncStorage`) are different.
