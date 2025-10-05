# MVP PHASE 3: Location Tracking Basics

**Duration**: 2-3 days
**Prerequisites**: Phase 1 & 2 completed
**Objective**: Track user's GPS location in real-time

---

## OBJECTIVES

Implement location tracking for safety monitoring:
- ✅ Request location permissions
- ✅ Track user's current location continuously
- ✅ Display location on HomeScreen
- ✅ Store location history in Redux
- ✅ Location updates every 30 seconds during active call

---

## DEPENDENCIES TO INSTALL

```bash
npm install react-native-geolocation-service
```

**iOS Configuration** - Edit `ios/SallyAI/Info.plist`:
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>Sally.ai needs your location to keep you safe during walks</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>Sally.ai tracks your location to send to emergency contacts if needed</string>
```

**iOS** - Run pod install:
```bash
cd ios && pod install && cd ..
```

**Android Configuration** - Edit `android/app/src/main/AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

---

## TASKS

### Task 1: Update Permissions Utility

**Update `src/utils/permissions.ts`**:
```typescript
import { Platform, PermissionsAndroid, Alert } from 'react-native';
import Geolocation from 'react-native-geolocation-service';

export const requestMicrophonePermission = async (): Promise<boolean> => {
  if (Platform.OS === 'android') {
    try {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.RECORD_AUDIO,
        {
          title: 'Microphone Permission',
          message: 'Sally.ai needs access to your microphone for voice conversations',
          buttonPositive: 'OK',
        }
      );
      return granted === PermissionsAndroid.RESULTS.GRANTED;
    } catch (err) {
      console.error('Microphone permission error:', err);
      return false;
    }
  }
  return true;
};

export const requestLocationPermission = async (): Promise<boolean> => {
  if (Platform.OS === 'android') {
    try {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION,
        {
          title: 'Location Permission',
          message: 'Sally.ai needs your location to keep you safe during walks',
          buttonPositive: 'OK',
        }
      );
      return granted === PermissionsAndroid.RESULTS.GRANTED;
    } catch (err) {
      console.error('Location permission error:', err);
      return false;
    }
  }

  // iOS - request permission
  return new Promise((resolve) => {
    Geolocation.requestAuthorization('whenInUse').then((result) => {
      resolve(result === 'granted');
    });
  });
};
```

---

### Task 2: Create Location Service

**Create `src/services/LocationService.ts`**:
```typescript
import Geolocation from 'react-native-geolocation-service';

export interface Location {
  latitude: number;
  longitude: number;
  accuracy: number;
  timestamp: number;
}

class LocationService {
  private watchId: number | null = null;

  getCurrentLocation(): Promise<Location> {
    return new Promise((resolve, reject) => {
      Geolocation.getCurrentPosition(
        (position) => {
          resolve({
            latitude: position.coords.latitude,
            longitude: position.coords.longitude,
            accuracy: position.coords.accuracy,
            timestamp: position.timestamp,
          });
        },
        (error) => {
          console.error('Get location error:', error);
          reject(error);
        },
        {
          enableHighAccuracy: true,
          timeout: 15000,
          maximumAge: 10000,
        }
      );
    });
  }

  startWatching(callback: (location: Location) => void): void {
    this.watchId = Geolocation.watchPosition(
      (position) => {
        callback({
          latitude: position.coords.latitude,
          longitude: position.coords.longitude,
          accuracy: position.coords.accuracy,
          timestamp: position.timestamp,
        });
      },
      (error) => {
        console.error('Watch location error:', error);
      },
      {
        enableHighAccuracy: true,
        distanceFilter: 10, // Update every 10 meters
        interval: 30000, // Check every 30 seconds
        fastestInterval: 15000,
      }
    );
  }

  stopWatching(): void {
    if (this.watchId !== null) {
      Geolocation.clearWatch(this.watchId);
      this.watchId = null;
    }
  }

  formatLocation(location: Location): string {
    return `${location.latitude.toFixed(6)}, ${location.longitude.toFixed(6)}`;
  }

  getGoogleMapsUrl(location: Location): string {
    return `https://www.google.com/maps?q=${location.latitude},${location.longitude}`;
  }
}

export default new LocationService();
```

---

### Task 3: Update Location Redux Slice

**Update `src/store/slices/locationSlice.ts`**:
```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

export interface Location {
  latitude: number;
  longitude: number;
  accuracy: number;
  timestamp: number;
}

interface LocationState {
  currentLocation: Location | null;
  locationHistory: Location[];
  isTracking: boolean;
  lastUpdate: number | null;
}

const initialState: LocationState = {
  currentLocation: null,
  locationHistory: [],
  isTracking: false,
  lastUpdate: null,
};

const locationSlice = createSlice({
  name: 'location',
  initialState,
  reducers: {
    updateLocation: (state, action: PayloadAction<Location>) => {
      state.currentLocation = action.payload;
      state.locationHistory.push(action.payload);
      state.lastUpdate = Date.now();

      // Keep only last 100 locations to avoid memory issues
      if (state.locationHistory.length > 100) {
        state.locationHistory = state.locationHistory.slice(-100);
      }
    },
    startTracking: (state) => {
      state.isTracking = true;
    },
    stopTracking: (state) => {
      state.isTracking = false;
    },
    clearHistory: (state) => {
      state.locationHistory = [];
    },
  },
});

export const { updateLocation, startTracking, stopTracking, clearHistory } = locationSlice.actions;
export default locationSlice.reducer;
```

---

### Task 4: Update HomeScreen with Location Display

**Update `src/screens/HomeScreen/index.tsx`**:
```typescript
import React, { useEffect } from 'react';
import { View, Text, TouchableOpacity, StyleSheet, Alert, Linking } from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { HomeScreenNavigationProp } from '../../types/navigation';
import { COLORS } from '../../constants';
import { requestMicrophonePermission, requestLocationPermission } from '../../utils/permissions';
import { useAppDispatch, useAppSelector } from '../../store/hooks';
import { updateLocation, startTracking } from '../../store/slices/locationSlice';
import LocationService from '../../services/LocationService';

export const HomeScreen = () => {
  const navigation = useNavigation<HomeScreenNavigationProp>();
  const dispatch = useAppDispatch();
  const { currentLocation, isTracking } = useAppSelector(state => state.location);

  useEffect(() => {
    initializeLocation();

    return () => {
      LocationService.stopWatching();
    };
  }, []);

  const initializeLocation = async () => {
    const hasPermission = await requestLocationPermission();
    if (hasPermission) {
      // Get initial location
      try {
        const location = await LocationService.getCurrentLocation();
        dispatch(updateLocation(location));
      } catch (error) {
        console.error('Failed to get initial location:', error);
      }

      // Start watching location
      dispatch(startTracking());
      LocationService.startWatching((location) => {
        dispatch(updateLocation(location));
      });
    } else {
      Alert.alert(
        'Location Required',
        'Sally.ai needs location access to keep you safe. Please enable location in settings.',
        [
          { text: 'Cancel', style: 'cancel' },
          { text: 'Open Settings', onPress: () => Linking.openSettings() },
        ]
      );
    }
  };

  const handleStartWalk = async () => {
    const hasMicPermission = await requestMicrophonePermission();
    if (!hasMicPermission) {
      Alert.alert('Microphone permission is required for voice conversations');
      return;
    }

    if (!currentLocation) {
      Alert.alert('Waiting for location...', 'Please try again in a moment');
      return;
    }

    navigation.navigate('Call', { isIncoming: false });
  };

  const openLocationInMaps = () => {
    if (currentLocation) {
      const url = LocationService.getGoogleMapsUrl(currentLocation);
      Linking.openURL(url);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Sally.ai</Text>
      <Text style={styles.subtitle}>Your Safety Companion</Text>

      {/* Location Display */}
      <View style={styles.locationCard}>
        <Text style={styles.locationLabel}>Current Location</Text>
        {currentLocation ? (
          <>
            <Text style={styles.locationText}>
              {LocationService.formatLocation(currentLocation)}
            </Text>
            <Text style={styles.accuracyText}>
              Accuracy: ±{currentLocation.accuracy.toFixed(0)}m
            </Text>
            <TouchableOpacity onPress={openLocationInMaps}>
              <Text style={styles.mapsLink}>View on Maps →</Text>
            </TouchableOpacity>
          </>
        ) : (
          <Text style={styles.locationText}>
            {isTracking ? 'Getting location...' : 'Location disabled'}
          </Text>
        )}
      </View>

      <TouchableOpacity
        style={[styles.button, styles.primaryButton]}
        onPress={handleStartWalk}>
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
    marginBottom: 40,
  },
  locationCard: {
    backgroundColor: COLORS.surface,
    padding: 20,
    borderRadius: 12,
    width: '100%',
    marginBottom: 40,
  },
  locationLabel: {
    fontSize: 12,
    color: COLORS.textSecondary,
    marginBottom: 8,
    textTransform: 'uppercase',
  },
  locationText: {
    fontSize: 16,
    color: COLORS.text,
    fontFamily: 'Courier',
    marginBottom: 4,
  },
  accuracyText: {
    fontSize: 12,
    color: COLORS.textSecondary,
    marginBottom: 8,
  },
  mapsLink: {
    fontSize: 14,
    color: COLORS.primary,
    marginTop: 4,
  },
  button: {
    width: '100%',
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

---

### Task 5: Add Location to AI Context

**Update `src/services/voiceAI/VoiceAIService.ts`** to include location in conversation:

```typescript
import OpenAI from 'openai';
import { Location } from '../LocationService';

// Add this method to VoiceAIService class:

setLocationContext(location: Location | null): void {
  if (!location) return;

  const locationInfo = `User's current location: ${location.latitude.toFixed(4)}, ${location.longitude.toFixed(4)}`;

  // Update system message with location
  this.conversationHistory[0] = {
    role: 'system',
    content: `You are Sally, a warm and supportive AI safety companion. You're talking to someone who is walking alone at night and wants company for safety. ${locationInfo}. Keep responses:
- Brief (1-2 sentences max)
- Friendly and conversational
- Supportive and reassuring
- Ask occasional questions to keep them engaged
- If they say phrases like "help me", "call police", "emergency", immediately respond with "I'm activating emergency services now" and mark this as an emergency.`,
  };
}
```

**Update CallScreen** to pass location to AI:
```typescript
// Add inside CallScreen useEffect:
const currentLocation = useAppSelector(state => state.location.currentLocation);

useEffect(() => {
  // ... existing code ...
  if (currentLocation) {
    VoiceAIService.setLocationContext(currentLocation);
  }
}, [currentLocation]);
```

---

## VERIFICATION

**Test on iOS**:
```bash
npm run ios
```

**Test on Android**:
```bash
npm run android
```

**Manual Tests**:
1. App requests location permission on launch → Grant permission
2. HomeScreen displays current coordinates
3. Location accuracy shown (should be <50m on real device)
4. Tap "View on Maps" → opens Google Maps at your location
5. Start a call → location continues updating in background
6. Check Redux DevTools → location history should grow

**Location Accuracy Test**:
- Compare displayed coordinates with Maps app
- Should be within 20-50 meters on real device
- Simulator may show fixed location (Apple HQ)

---

## TROUBLESHOOTING

**Location permission denied**:
- On iOS: Settings → Privacy → Location Services → SallyAI → While Using App
- On Android: Settings → Apps → SallyAI → Permissions → Location → Allow

**Location not updating**:
- Check location services are enabled on device
- Verify Info.plist (iOS) or AndroidManifest.xml (Android) has required keys
- Try on real device (simulator may have issues)
- Check console for errors

**Inaccurate location**:
- GPS works best outdoors with clear sky view
- Indoor locations may be off by 50-200m
- Wi-Fi must be enabled for improved accuracy
- Wait 30 seconds for GPS to stabilize

---

## COMPLETION CRITERIA

- ✅ Location permission requested and handled
- ✅ Current location displays on HomeScreen
- ✅ Location updates every 30 seconds
- ✅ Location history stored in Redux
- ✅ "View on Maps" link works
- ✅ Location accuracy shown (<50m on real device)
- ✅ Location tracking works during active calls
- ✅ Works on both iOS and Android

---

**Phase 3 Complete!** Proceed to `MVP_PHASE_4.md` to add emergency contacts and SOS functionality.
