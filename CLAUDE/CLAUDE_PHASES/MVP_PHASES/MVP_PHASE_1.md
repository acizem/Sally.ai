# MVP PHASE 1: Foundation & Navigation

**Duration**: 2-3 days
**Prerequisites**: macOS with Xcode, Android Studio, Node.js 18+
**Objective**: Create React Native TypeScript project with navigation and state management

---

## OBJECTIVES

Set up the foundational structure for Sally.ai MVP:
- ✅ Working React Native app on iOS and Android
- ✅ Navigation between 4 core screens
- ✅ Redux state management configured
- ✅ Dark mode UI theme
- ✅ Basic folder structure

---

## DEPENDENCIES TO INSTALL

```bash
# Core navigation
npm install @react-navigation/native @react-navigation/stack
npm install react-native-screens react-native-safe-area-context react-native-gesture-handler

# State management
npm install @reduxjs/toolkit react-redux redux-persist @react-native-async-storage/async-storage

# Dev dependencies
npm install --save-dev @types/react @types/react-native
```

**iOS Only**:
```bash
cd ios && pod install && cd ..
```

---

## TASKS

### Task 1: Initialize React Native Project

**Command**:
```bash
npx react-native@latest init SallyAI --template react-native-template-typescript
cd SallyAI
```

**Verify**:
```bash
npm run ios    # Should launch iOS simulator
npm run android # Should launch Android emulator
```

---

### Task 2: Create Folder Structure

**Command**:
```bash
mkdir -p src/{components/common,screens/{HomeScreen,CallScreen,EmergencyScreen,SettingsScreen},navigation,store/slices,types,constants}
```

**Create placeholder files**:
- `src/screens/HomeScreen/index.tsx`
- `src/screens/CallScreen/index.tsx`
- `src/screens/EmergencyScreen/index.tsx`
- `src/screens/SettingsScreen/index.tsx`
- `src/navigation/AppNavigator.tsx`
- `src/store/index.ts`
- `src/constants/index.ts`

---

### Task 3: Define Constants

**Create `src/constants/index.ts`**:
```typescript
export const COLORS = {
  primary: '#007AFF',
  danger: '#FF3B30',
  background: '#000000',
  surface: '#1C1C1E',
  text: '#FFFFFF',
  textSecondary: '#98989D',
} as const;

export const EMERGENCY_LEVELS = {
  NONE: 0,
  CONCERN: 1,
  ALERT: 2,
  EMERGENCY: 3,
} as const;
```

---

### Task 4: Create Navigation Types

**Create `src/types/navigation.ts`**:
```typescript
import { StackNavigationProp } from '@react-navigation/stack';

export type RootStackParamList = {
  Home: undefined;
  Call: { isIncoming?: boolean };
  Emergency: { level: number };
  Settings: undefined;
};

export type HomeScreenNavigationProp = StackNavigationProp<RootStackParamList, 'Home'>;
export type CallScreenNavigationProp = StackNavigationProp<RootStackParamList, 'Call'>;
```

---

### Task 5: Create Basic Screen Components

**`src/screens/HomeScreen/index.tsx`**:
```typescript
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { HomeScreenNavigationProp } from '../../types/navigation';
import { COLORS } from '../../constants';

export const HomeScreen = () => {
  const navigation = useNavigation<HomeScreenNavigationProp>();

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Sally.ai</Text>
      <Text style={styles.subtitle}>Your Safety Companion</Text>

      <TouchableOpacity
        style={[styles.button, styles.primaryButton]}
        onPress={() => navigation.navigate('Call', { isIncoming: false })}>
        <Text style={styles.buttonText}>Start Walk</Text>
      </TouchableOpacity>

      <TouchableOpacity
        style={[styles.button, styles.dangerButton]}
        onPress={() => navigation.navigate('Emergency', { level: 3 })}>
        <Text style={styles.buttonText}>Emergency SOS</Text>
      </TouchableOpacity>

      <TouchableOpacity
        style={[styles.button, styles.secondaryButton]}
        onPress={() => navigation.navigate('Settings')}>
        <Text style={styles.buttonText}>Settings</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: COLORS.background,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 48,
    fontWeight: 'bold',
    color: COLORS.text,
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 18,
    color: COLORS.textSecondary,
    marginBottom: 60,
  },
  button: {
    width: '80%',
    paddingVertical: 16,
    borderRadius: 12,
    alignItems: 'center',
    marginBottom: 16,
  },
  primaryButton: {
    backgroundColor: COLORS.primary,
  },
  dangerButton: {
    backgroundColor: COLORS.danger,
  },
  secondaryButton: {
    backgroundColor: '#5856D6',
  },
  buttonText: {
    color: COLORS.text,
    fontSize: 18,
    fontWeight: '600',
  },
});
```

**`src/screens/CallScreen/index.tsx`**:
```typescript
import React, { useState, useEffect } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { COLORS } from '../../constants';

export const CallScreen = () => {
  const navigation = useNavigation();
  const [duration, setDuration] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setDuration(prev => prev + 1);
    }, 1000);
    return () => clearInterval(timer);
  }, []);

  const formatDuration = (seconds: number) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
  };

  return (
    <View style={styles.container}>
      <Text style={styles.name}>Sally</Text>
      <Text style={styles.status}>Connected</Text>
      <Text style={styles.duration}>{formatDuration(duration)}</Text>

      <View style={styles.buttonContainer}>
        <TouchableOpacity
          style={[styles.button, styles.dangerButton]}
          onPress={() => navigation.goBack()}>
          <Text style={styles.buttonText}>End Call</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={[styles.button, styles.sosButton]}
          onPress={() => navigation.navigate('Emergency', { level: 3 })}>
          <Text style={styles.buttonText}>SOS</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: COLORS.background,
    justifyContent: 'center',
    alignItems: 'center',
  },
  name: {
    fontSize: 36,
    fontWeight: 'bold',
    color: COLORS.text,
    marginBottom: 8,
  },
  status: {
    fontSize: 16,
    color: COLORS.textSecondary,
    marginBottom: 40,
  },
  duration: {
    fontSize: 48,
    color: COLORS.text,
    fontWeight: '300',
    marginBottom: 80,
  },
  buttonContainer: {
    width: '80%',
  },
  button: {
    paddingVertical: 16,
    borderRadius: 12,
    alignItems: 'center',
    marginBottom: 16,
  },
  dangerButton: {
    backgroundColor: COLORS.danger,
  },
  sosButton: {
    backgroundColor: '#FF9500',
  },
  buttonText: {
    color: COLORS.text,
    fontSize: 18,
    fontWeight: '600',
  },
});
```

**`src/screens/EmergencyScreen/index.tsx`**:
```typescript
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { useNavigation, useRoute } from '@react-navigation/native';
import { COLORS } from '../../constants';

export const EmergencyScreen = () => {
  const navigation = useNavigation();
  const route = useRoute();
  const { level } = route.params as { level: number };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>🚨 SOS ACTIVATED</Text>
      <Text style={styles.level}>Emergency Level: {level}</Text>

      <View style={styles.statusContainer}>
        <Text style={styles.statusText}>✓ Location shared</Text>
        <Text style={styles.statusText}>✓ Contacts notified</Text>
        <Text style={styles.statusText}>✓ Recording started</Text>
      </View>

      <TouchableOpacity
        style={styles.cancelButton}
        onPress={() => navigation.goBack()}>
        <Text style={styles.buttonText}>Cancel Emergency</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: COLORS.danger,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    color: COLORS.text,
    marginBottom: 24,
  },
  level: {
    fontSize: 20,
    color: COLORS.text,
    marginBottom: 60,
  },
  statusContainer: {
    marginBottom: 60,
  },
  statusText: {
    fontSize: 18,
    color: COLORS.text,
    marginBottom: 12,
  },
  cancelButton: {
    paddingHorizontal: 32,
    paddingVertical: 16,
    backgroundColor: 'rgba(255,255,255,0.3)',
    borderRadius: 12,
  },
  buttonText: {
    color: COLORS.text,
    fontSize: 18,
    fontWeight: '600',
  },
});
```

**`src/screens/SettingsScreen/index.tsx`**:
```typescript
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { COLORS } from '../../constants';

export const SettingsScreen = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Settings</Text>
      <Text style={styles.placeholder}>Coming in Phase 4</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: COLORS.background,
    padding: 20,
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    color: COLORS.text,
    marginBottom: 24,
  },
  placeholder: {
    fontSize: 16,
    color: COLORS.textSecondary,
  },
});
```

---

### Task 6: Create App Navigator

**`src/navigation/AppNavigator.tsx`**:
```typescript
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { RootStackParamList } from '../types/navigation';
import { HomeScreen } from '../screens/HomeScreen';
import { CallScreen } from '../screens/CallScreen';
import { EmergencyScreen } from '../screens/EmergencyScreen';
import { SettingsScreen } from '../screens/SettingsScreen';
import { COLORS } from '../constants';

const Stack = createStackNavigator<RootStackParamList>();

export const AppNavigator = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator
        initialRouteName="Home"
        screenOptions={{
          headerStyle: { backgroundColor: COLORS.surface },
          headerTintColor: COLORS.text,
          headerTitleStyle: { fontWeight: 'bold' },
          cardStyle: { backgroundColor: COLORS.background },
        }}>
        <Stack.Screen
          name="Home"
          component={HomeScreen}
          options={{ headerShown: false }}
        />
        <Stack.Screen
          name="Call"
          component={CallScreen}
          options={{ headerShown: false, presentation: 'fullScreenModal' }}
        />
        <Stack.Screen
          name="Emergency"
          component={EmergencyScreen}
          options={{ headerShown: false, presentation: 'fullScreenModal' }}
        />
        <Stack.Screen
          name="Settings"
          component={SettingsScreen}
          options={{ title: 'Settings' }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
};
```

---

### Task 7: Configure Redux Store

**`src/store/slices/callSlice.ts`**:
```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface CallState {
  isActive: boolean;
  duration: number;
  contactName: string;
}

const initialState: CallState = {
  isActive: false,
  duration: 0,
  contactName: 'Sally',
};

const callSlice = createSlice({
  name: 'call',
  initialState,
  reducers: {
    startCall: (state, action: PayloadAction<string>) => {
      state.isActive = true;
      state.contactName = action.payload;
      state.duration = 0;
    },
    endCall: (state) => {
      state.isActive = false;
    },
  },
});

export const { startCall, endCall } = callSlice.actions;
export default callSlice.reducer;
```

**`src/store/slices/locationSlice.ts`**:
```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface Location {
  latitude: number;
  longitude: number;
  timestamp: number;
}

interface LocationState {
  currentLocation: Location | null;
  isTracking: boolean;
}

const initialState: LocationState = {
  currentLocation: null,
  isTracking: false,
};

const locationSlice = createSlice({
  name: 'location',
  initialState,
  reducers: {
    updateLocation: (state, action: PayloadAction<Location>) => {
      state.currentLocation = action.payload;
    },
    startTracking: (state) => {
      state.isTracking = true;
    },
    stopTracking: (state) => {
      state.isTracking = false;
    },
  },
});

export const { updateLocation, startTracking, stopTracking } = locationSlice.actions;
export default locationSlice.reducer;
```

**`src/store/slices/emergencySlice.ts`**:
```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface EmergencyContact {
  id: string;
  name: string;
  phone: string;
}

interface EmergencyState {
  contacts: EmergencyContact[];
  isSOSActive: boolean;
}

const initialState: EmergencyState = {
  contacts: [],
  isSOSActive: false,
};

const emergencySlice = createSlice({
  name: 'emergency',
  initialState,
  reducers: {
    addContact: (state, action: PayloadAction<EmergencyContact>) => {
      state.contacts.push(action.payload);
    },
    removeContact: (state, action: PayloadAction<string>) => {
      state.contacts = state.contacts.filter(c => c.id !== action.payload);
    },
    activateSOS: (state) => {
      state.isSOSActive = true;
    },
    deactivateSOS: (state) => {
      state.isSOSActive = false;
    },
  },
});

export const { addContact, removeContact, activateSOS, deactivateSOS } = emergencySlice.actions;
export default emergencySlice.reducer;
```

**`src/store/index.ts`**:
```typescript
import { configureStore, combineReducers } from '@reduxjs/toolkit';
import { persistStore, persistReducer } from 'redux-persist';
import AsyncStorage from '@react-native-async-storage/async-storage';
import callReducer from './slices/callSlice';
import locationReducer from './slices/locationSlice';
import emergencyReducer from './slices/emergencySlice';

const persistConfig = {
  key: 'root',
  storage: AsyncStorage,
  whitelist: ['emergency'], // Only persist emergency contacts
};

const rootReducer = combineReducers({
  call: callReducer,
  location: locationReducer,
  emergency: emergencyReducer,
});

const persistedReducer = persistReducer(persistConfig, rootReducer);

export const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE'],
      },
    }),
});

export const persistor = persistStore(store);

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

**`src/store/hooks.ts`**:
```typescript
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './index';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

---

### Task 8: Update App Entry Point

**Edit `App.tsx`**:
```typescript
import React from 'react';
import { StatusBar } from 'react-native';
import { Provider } from 'react-redux';
import { PersistGate } from 'redux-persist/integration/react';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { AppNavigator } from './src/navigation/AppNavigator';
import { store, persistor } from './src/store';

const App = () => {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <Provider store={store}>
        <PersistGate loading={null} persistor={persistor}>
          <SafeAreaProvider>
            <StatusBar barStyle="light-content" />
            <AppNavigator />
          </SafeAreaProvider>
        </PersistGate>
      </Provider>
    </GestureHandlerRootView>
  );
};

export default App;
```

---

## VERIFICATION

Run these commands to verify Phase 1 is complete:

```bash
# 1. Build and run iOS
npm run ios

# 2. Build and run Android
npm run android
```

**Manual Tests**:
1. App launches without errors
2. HomeScreen displays with "Sally.ai" title
3. Tap "Start Walk" → navigates to CallScreen
4. CallScreen shows timer counting up
5. Tap "End Call" → returns to HomeScreen
6. Tap "Emergency SOS" → shows red EmergencyScreen
7. Tap "Settings" → shows SettingsScreen
8. All navigation works smoothly

---

## COMPLETION CRITERIA

- ✅ App runs on both iOS and Android without crashes
- ✅ All 4 screens render correctly
- ✅ Navigation between screens works
- ✅ Redux store configured and connected
- ✅ Dark mode UI visible
- ✅ Call timer increments on CallScreen
- ✅ No TypeScript errors

---

**Phase 1 Complete!** Proceed to `MVP_PHASE_2.md` to add voice AI integration.
