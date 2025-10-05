# MVP PHASE 4: Emergency Contacts & SOS

**Duration**: 3-4 days
**Prerequisites**: Phases 1, 2, and 3 completed
**Objective**: Implement emergency contact management and SMS-based SOS alerts

---

## OBJECTIVES

Create the emergency response system:
- ✅ Add/remove/edit emergency contacts
- ✅ Persist contacts in Redux with AsyncStorage
- ✅ SOS button sends SMS to all contacts
- ✅ SMS includes user's location and emergency message
- ✅ UI for managing emergency contacts in Settings
- ✅ One-tap SOS activation from CallScreen and HomeScreen

---

## DEPENDENCIES TO INSTALL

```bash
# SMS functionality (native - no npm install needed)
# Uses react-native Linking for SMS
```

**Android Configuration** - Edit `android/app/src/main/AndroidManifest.xml`:
```xml
<queries>
  <intent>
    <action android:name="android.intent.action.VIEW" />
    <data android:scheme="sms" />
  </intent>
</queries>
```

---

## TASKS

### Task 1: Update Emergency Redux Slice

**Update `src/store/slices/emergencySlice.ts`**:
```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

export interface EmergencyContact {
  id: string;
  name: string;
  phone: string;
  relationship: string;
  priority: number;
}

interface EmergencyState {
  contacts: EmergencyContact[];
  isSOSActive: boolean;
  sosTimestamp: number | null;
  sosLocation: { latitude: number; longitude: number } | null;
}

const initialState: EmergencyState = {
  contacts: [],
  isSOSActive: false,
  sosTimestamp: null,
  sosLocation: null,
};

const emergencySlice = createSlice({
  name: 'emergency',
  initialState,
  reducers: {
    addContact: (state, action: PayloadAction<EmergencyContact>) => {
      state.contacts.push(action.payload);
      // Sort by priority
      state.contacts.sort((a, b) => a.priority - b.priority);
    },
    removeContact: (state, action: PayloadAction<string>) => {
      state.contacts = state.contacts.filter(c => c.id !== action.payload);
    },
    updateContact: (state, action: PayloadAction<EmergencyContact>) => {
      const index = state.contacts.findIndex(c => c.id === action.payload.id);
      if (index !== -1) {
        state.contacts[index] = action.payload;
        state.contacts.sort((a, b) => a.priority - b.priority);
      }
    },
    activateSOS: (state, action: PayloadAction<{ latitude: number; longitude: number }>) => {
      state.isSOSActive = true;
      state.sosTimestamp = Date.now();
      state.sosLocation = action.payload;
    },
    deactivateSOS: (state) => {
      state.isSOSActive = false;
      state.sosTimestamp = null;
      state.sosLocation = null;
    },
  },
});

export const { addContact, removeContact, updateContact, activateSOS, deactivateSOS } =
  emergencySlice.actions;
export default emergencySlice.reducer;
```

---

### Task 2: Create Emergency Service

**Create `src/services/EmergencyService.ts`**:
```typescript
import { Linking, Alert } from 'react-native';
import { EmergencyContact } from '../store/slices/emergencySlice';
import { Location } from './LocationService';

class EmergencyService {
  async sendSOSViaSMS(contacts: EmergencyContact[], location: Location): Promise<void> {
    if (contacts.length === 0) {
      Alert.alert('No Emergency Contacts', 'Please add emergency contacts in Settings first');
      return;
    }

    const googleMapsUrl = `https://www.google.com/maps?q=${location.latitude},${location.longitude}`;
    const message = `🚨 EMERGENCY - I need help! My location: ${googleMapsUrl}`;

    for (const contact of contacts) {
      try {
        const smsUrl = this.buildSMSUrl(contact.phone, message);
        const canOpen = await Linking.canOpenURL(smsUrl);

        if (canOpen) {
          await Linking.openURL(smsUrl);
          // Add delay to allow each SMS to open
          await new Promise(resolve => setTimeout(resolve, 1000));
        } else {
          console.error(`Cannot send SMS to ${contact.phone}`);
        }
      } catch (error) {
        console.error(`Failed to send SMS to ${contact.name}:`, error);
      }
    }
  }

  private buildSMSUrl(phoneNumber: string, message: string): string {
    // Remove all non-numeric characters
    const cleanPhone = phoneNumber.replace(/[^0-9+]/g, '');

    // iOS and Android handle SMS URLs differently
    const separator = Platform.OS === 'ios' ? '&' : '?';
    return `sms:${cleanPhone}${separator}body=${encodeURIComponent(message)}`;
  }

  formatPhoneNumber(phone: string): string {
    // Format as (XXX) XXX-XXXX
    const cleaned = phone.replace(/\D/g, '');
    if (cleaned.length === 10) {
      return `(${cleaned.slice(0, 3)}) ${cleaned.slice(3, 6)}-${cleaned.slice(6)}`;
    }
    return phone;
  }

  validatePhoneNumber(phone: string): boolean {
    const cleaned = phone.replace(/\D/g, '');
    return cleaned.length >= 10;
  }
}

export default new EmergencyService();
```

---

### Task 3: Create Emergency Contacts Screen

**Update `src/screens/SettingsScreen/index.tsx`**:
```typescript
import React, { useState } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  TextInput,
  FlatList,
  Alert,
} from 'react-native';
import { COLORS } from '../../constants';
import { useAppDispatch, useAppSelector } from '../../store/hooks';
import { addContact, removeContact, EmergencyContact } from '../../store/slices/emergencySlice';
import EmergencyService from '../../services/EmergencyService';

export const SettingsScreen = () => {
  const dispatch = useAppDispatch();
  const contacts = useAppSelector(state => state.emergency.contacts);
  const [showAddForm, setShowAddForm] = useState(false);
  const [newContact, setNewContact] = useState({ name: '', phone: '', relationship: '' });

  const handleAddContact = () => {
    if (!newContact.name.trim() || !newContact.phone.trim()) {
      Alert.alert('Error', 'Please enter name and phone number');
      return;
    }

    if (!EmergencyService.validatePhoneNumber(newContact.phone)) {
      Alert.alert('Error', 'Please enter a valid 10-digit phone number');
      return;
    }

    const contact: EmergencyContact = {
      id: Date.now().toString(),
      name: newContact.name.trim(),
      phone: newContact.phone.trim(),
      relationship: newContact.relationship.trim() || 'Emergency Contact',
      priority: contacts.length + 1,
    };

    dispatch(addContact(contact));
    setNewContact({ name: '', phone: '', relationship: '' });
    setShowAddForm(false);
    Alert.alert('Success', `${contact.name} added to emergency contacts`);
  };

  const handleRemoveContact = (contact: EmergencyContact) => {
    Alert.alert(
      'Remove Contact',
      `Remove ${contact.name} from emergency contacts?`,
      [
        { text: 'Cancel', style: 'cancel' },
        {
          text: 'Remove',
          style: 'destructive',
          onPress: () => dispatch(removeContact(contact.id)),
        },
      ]
    );
  };

  const renderContact = ({ item }: { item: EmergencyContact }) => (
    <View style={styles.contactCard}>
      <View style={styles.contactInfo}>
        <Text style={styles.contactName}>{item.name}</Text>
        <Text style={styles.contactPhone}>
          {EmergencyService.formatPhoneNumber(item.phone)}
        </Text>
        <Text style={styles.contactRelationship}>{item.relationship}</Text>
      </View>
      <TouchableOpacity
        style={styles.removeButton}
        onPress={() => handleRemoveContact(item)}>
        <Text style={styles.removeButtonText}>Remove</Text>
      </TouchableOpacity>
    </View>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Emergency Contacts</Text>
      <Text style={styles.subtitle}>
        {contacts.length === 0
          ? 'Add contacts who will be notified in emergencies'
          : `${contacts.length} contact(s) will receive SOS alerts`}
      </Text>

      <FlatList
        data={contacts}
        renderItem={renderContact}
        keyExtractor={item => item.id}
        ListEmptyComponent={
          <Text style={styles.emptyText}>No emergency contacts yet</Text>
        }
        style={styles.contactList}
      />

      {showAddForm ? (
        <View style={styles.addForm}>
          <TextInput
            style={styles.input}
            placeholder="Name"
            placeholderTextColor={COLORS.textSecondary}
            value={newContact.name}
            onChangeText={text => setNewContact({ ...newContact, name: text })}
          />
          <TextInput
            style={styles.input}
            placeholder="Phone Number"
            placeholderTextColor={COLORS.textSecondary}
            keyboardType="phone-pad"
            value={newContact.phone}
            onChangeText={text => setNewContact({ ...newContact, phone: text })}
          />
          <TextInput
            style={styles.input}
            placeholder="Relationship (optional)"
            placeholderTextColor={COLORS.textSecondary}
            value={newContact.relationship}
            onChangeText={text => setNewContact({ ...newContact, relationship: text })}
          />
          <View style={styles.formButtons}>
            <TouchableOpacity
              style={[styles.button, styles.cancelButton]}
              onPress={() => {
                setShowAddForm(false);
                setNewContact({ name: '', phone: '', relationship: '' });
              }}>
              <Text style={styles.buttonText}>Cancel</Text>
            </TouchableOpacity>
            <TouchableOpacity
              style={[styles.button, styles.saveButton]}
              onPress={handleAddContact}>
              <Text style={styles.buttonText}>Save Contact</Text>
            </TouchableOpacity>
          </View>
        </View>
      ) : (
        <TouchableOpacity
          style={[styles.button, styles.addButton]}
          onPress={() => setShowAddForm(true)}>
          <Text style={styles.buttonText}>+ Add Emergency Contact</Text>
        </TouchableOpacity>
      )}
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
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 14,
    color: COLORS.textSecondary,
    marginBottom: 24,
  },
  contactList: {
    flex: 1,
    marginBottom: 20,
  },
  contactCard: {
    backgroundColor: COLORS.surface,
    padding: 16,
    borderRadius: 12,
    marginBottom: 12,
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },
  contactInfo: {
    flex: 1,
  },
  contactName: {
    fontSize: 18,
    fontWeight: '600',
    color: COLORS.text,
    marginBottom: 4,
  },
  contactPhone: {
    fontSize: 16,
    color: COLORS.textSecondary,
    fontFamily: 'Courier',
    marginBottom: 4,
  },
  contactRelationship: {
    fontSize: 14,
    color: COLORS.textSecondary,
  },
  removeButton: {
    paddingHorizontal: 16,
    paddingVertical: 8,
    backgroundColor: COLORS.danger,
    borderRadius: 8,
  },
  removeButtonText: {
    color: COLORS.text,
    fontSize: 14,
    fontWeight: '600',
  },
  emptyText: {
    fontSize: 16,
    color: COLORS.textSecondary,
    textAlign: 'center',
    marginTop: 40,
  },
  addForm: {
    backgroundColor: COLORS.surface,
    padding: 20,
    borderRadius: 12,
    marginBottom: 20,
  },
  input: {
    backgroundColor: COLORS.background,
    color: COLORS.text,
    padding: 12,
    borderRadius: 8,
    marginBottom: 12,
    fontSize: 16,
  },
  formButtons: {
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  button: {
    paddingVertical: 14,
    borderRadius: 12,
    alignItems: 'center',
  },
  addButton: {
    backgroundColor: COLORS.primary,
  },
  saveButton: {
    backgroundColor: '#34C759',
    flex: 1,
    marginLeft: 8,
  },
  cancelButton: {
    backgroundColor: '#8E8E93',
    flex: 1,
    marginRight: 8,
  },
  buttonText: {
    color: COLORS.text,
    fontSize: 16,
    fontWeight: '600',
  },
});
```

---

### Task 4: Update EmergencyScreen to Trigger SOS

**Update `src/screens/EmergencyScreen/index.tsx`**:
```typescript
import React, { useEffect } from 'react';
import { View, Text, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import { useNavigation, useRoute } from '@react-navigation/native';
import { COLORS } from '../../constants';
import { useAppDispatch, useAppSelector } from '../../store/hooks';
import { activateSOS, deactivateSOS } from '../../store/slices/emergencySlice';
import EmergencyService from '../../services/EmergencyService';

export const EmergencyScreen = () => {
  const navigation = useNavigation();
  const route = useRoute();
  const dispatch = useAppDispatch();
  const { level } = route.params as { level: number };

  const contacts = useAppSelector(state => state.emergency.contacts);
  const currentLocation = useAppSelector(state => state.location.currentLocation);
  const isSOSActive = useAppSelector(state => state.emergency.isSOSActive);

  useEffect(() => {
    if (level === 3 && currentLocation && !isSOSActive) {
      triggerSOS();
    }
  }, [level, currentLocation, isSOSActive]);

  const triggerSOS = async () => {
    if (!currentLocation) {
      Alert.alert('Error', 'Location not available. Cannot send SOS.');
      return;
    }

    if (contacts.length === 0) {
      Alert.alert(
        'No Emergency Contacts',
        'Please add emergency contacts in Settings before using SOS.',
        [
          { text: 'Cancel', onPress: () => navigation.goBack() },
          { text: 'Add Contacts', onPress: () => navigation.navigate('Settings') },
        ]
      );
      return;
    }

    // Activate SOS in Redux
    dispatch(
      activateSOS({
        latitude: currentLocation.latitude,
        longitude: currentLocation.longitude,
      })
    );

    // Send SMS to all contacts
    await EmergencyService.sendSOSViaSMS(contacts, currentLocation);

    Alert.alert(
      'SOS Activated',
      `Emergency alerts sent to ${contacts.length} contact(s). SMS apps will open for each contact.`,
      [{ text: 'OK' }]
    );
  };

  const handleCancel = () => {
    Alert.alert(
      'Cancel Emergency',
      'Are you sure you want to cancel the emergency alert?',
      [
        { text: 'No', style: 'cancel' },
        {
          text: 'Yes, Cancel',
          style: 'destructive',
          onPress: () => {
            dispatch(deactivateSOS());
            navigation.goBack();
          },
        },
      ]
    );
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>🚨 SOS ACTIVATED</Text>
      <Text style={styles.level}>Emergency Level: {level}</Text>

      <View style={styles.statusContainer}>
        <Text style={styles.statusText}>
          ✓ Location: {currentLocation ? 'Shared' : 'Unavailable'}
        </Text>
        <Text style={styles.statusText}>
          ✓ Contacts notified: {contacts.length}
        </Text>
        <Text style={styles.statusText}>✓ SMS alerts sent</Text>
      </View>

      <View style={styles.contactsContainer}>
        <Text style={styles.contactsTitle}>Notified Contacts:</Text>
        {contacts.map(contact => (
          <Text key={contact.id} style={styles.contactText}>
            • {contact.name} ({contact.phone})
          </Text>
        ))}
      </View>

      <TouchableOpacity style={styles.cancelButton} onPress={handleCancel}>
        <Text style={styles.buttonText}>Cancel Emergency</Text>
      </TouchableOpacity>

      <Text style={styles.helpText}>Stay calm. Help has been alerted.</Text>
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
    marginBottom: 16,
  },
  level: {
    fontSize: 20,
    color: COLORS.text,
    marginBottom: 40,
  },
  statusContainer: {
    marginBottom: 40,
  },
  statusText: {
    fontSize: 18,
    color: COLORS.text,
    marginBottom: 12,
  },
  contactsContainer: {
    backgroundColor: 'rgba(0,0,0,0.3)',
    padding: 20,
    borderRadius: 12,
    marginBottom: 40,
    width: '100%',
  },
  contactsTitle: {
    fontSize: 16,
    color: COLORS.text,
    fontWeight: '600',
    marginBottom: 12,
  },
  contactText: {
    fontSize: 14,
    color: COLORS.text,
    marginBottom: 6,
  },
  cancelButton: {
    paddingHorizontal: 32,
    paddingVertical: 16,
    backgroundColor: 'rgba(255,255,255,0.3)',
    borderRadius: 12,
    marginBottom: 20,
  },
  buttonText: {
    color: COLORS.text,
    fontSize: 18,
    fontWeight: '600',
  },
  helpText: {
    fontSize: 14,
    color: COLORS.text,
    opacity: 0.8,
    marginTop: 20,
  },
});
```

---

### Task 5: Add Missing Import

**Add to `src/services/EmergencyService.ts`**:
```typescript
import { Linking, Alert, Platform } from 'react-native';
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

1. **Add Emergency Contacts**:
   - Go to Settings
   - Tap "+ Add Emergency Contact"
   - Enter name, phone number (use your real phone for testing)
   - Save contact
   - Verify contact appears in list

2. **Test SOS from HomeScreen**:
   - Tap "Emergency SOS" button
   - Verify EmergencyScreen shows
   - SMS app should open with pre-filled message
   - Verify message includes Google Maps link with location

3. **Test SOS from CallScreen**:
   - Start a walk (tap "Start Walk")
   - Tap "SOS" button on CallScreen
   - Verify SOS triggered and SMS sent

4. **Test Emergency Phrase**:
   - During a call, say "I need help"
   - App should navigate to EmergencyScreen
   - SMS should be sent to all contacts

5. **Cancel Emergency**:
   - Activate SOS
   - Tap "Cancel Emergency"
   - Confirm cancellation
   - SOS should deactivate

---

## TROUBLESHOOTING

**SMS not opening**:
- Verify phone number is in correct format (10 digits)
- Check device has SMS capability (not all tablets do)
- Try on real device (simulator may not have SMS app)
- Check console for URL construction errors

**Location not in SMS**:
- Ensure location permission granted
- Verify currentLocation is not null
- Check Google Maps URL format

**Contacts not persisting**:
- Verify Redux Persist is configured correctly
- Check AsyncStorage whitelists 'emergency' slice
- Clear app data and re-add contacts

---

## COMPLETION CRITERIA

- ✅ Can add/remove emergency contacts in Settings
- ✅ Contacts persist after app restart
- ✅ Phone numbers validated (10 digits minimum)
- ✅ SOS button triggers SMS to all contacts
- ✅ SMS includes Google Maps location link
- ✅ Emergency phrase "help me" triggers SOS during call
- ✅ Can cancel emergency alert
- ✅ EmergencyScreen shows list of notified contacts
- ✅ Works on both iOS and Android

---

**Phase 4 Complete!** Proceed to `MVP_PHASE_5.md` for final testing and polish.
