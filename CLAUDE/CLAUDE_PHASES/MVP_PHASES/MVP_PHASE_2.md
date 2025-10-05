# MVP PHASE 2: Basic Voice AI Integration

**Duration**: 3-4 days
**Prerequisites**: Phase 1 completed
**Objective**: Implement voice conversation with AI companion

---

## OBJECTIVES

Add voice AI capabilities to the CallScreen:
- ✅ User can speak and see transcription
- ✅ AI responds with contextual conversation
- ✅ AI voice speaks responses aloud
- ✅ Conversation feels natural and supportive
- ✅ Emergency trigger phrases detected

---

## DEPENDENCIES TO INSTALL

```bash
# Speech recognition
npm install @react-native-voice/voice

# Text-to-speech
npm install react-native-tts

# OpenAI API client
npm install openai

# iOS permissions
cd ios && pod install && cd ..
```

**Platform Configuration Required**:

**iOS** - Edit `ios/SallyAI/Info.plist`:
```xml
<key>NSSpeechRecognitionUsageDescription</key>
<string>Sally.ai needs microphone access for voice conversations</string>
<key>NSMicrophoneUsageDescription</key>
<string>Sally.ai uses your microphone to understand your voice</string>
```

**Android** - Edit `android/app/src/main/AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
```

---

## TASKS

### Task 1: Create Voice AI Service

**Create `src/services/voiceAI/VoiceAIService.ts`**:
```typescript
import OpenAI from 'openai';

// TODO: Move to secure env file post-MVP
const OPENAI_API_KEY = 'your-api-key-here';

const openai = new OpenAI({
  apiKey: OPENAI_API_KEY,
});

interface ConversationMessage {
  role: 'system' | 'user' | 'assistant';
  content: string;
}

class VoiceAIService {
  private conversationHistory: ConversationMessage[] = [];

  constructor() {
    this.conversationHistory = [
      {
        role: 'system',
        content: `You are Sally, a warm and supportive AI safety companion. You're talking to someone who is walking alone at night and wants company for safety. Keep responses:
- Brief (1-2 sentences max)
- Friendly and conversational
- Supportive and reassuring
- Ask occasional questions to keep them engaged
- If they say phrases like "help me", "call police", "emergency", immediately respond with "I'm activating emergency services now" and mark this as an emergency.`,
      },
    ];
  }

  async getAIResponse(userMessage: string): Promise<{ response: string; isEmergency: boolean }> {
    // Check for emergency triggers
    const emergencyPhrases = ['help me', 'call police', 'emergency', '911', 'i need help'];
    const isEmergency = emergencyPhrases.some(phrase =>
      userMessage.toLowerCase().includes(phrase)
    );

    this.conversationHistory.push({
      role: 'user',
      content: userMessage,
    });

    try {
      const completion = await openai.chat.completions.create({
        model: 'gpt-4',
        messages: this.conversationHistory,
        max_tokens: 100,
        temperature: 0.8,
      });

      const aiResponse = completion.choices[0]?.message?.content || "I'm here with you.";

      this.conversationHistory.push({
        role: 'assistant',
        content: aiResponse,
      });

      return { response: aiResponse, isEmergency };
    } catch (error) {
      console.error('OpenAI API Error:', error);
      return {
        response: "I'm still here with you. Keep walking, you're doing great.",
        isEmergency,
      };
    }
  }

  resetConversation() {
    this.conversationHistory = [this.conversationHistory[0]]; // Keep system prompt
  }
}

export default new VoiceAIService();
```

---

### Task 2: Create Speech Recognition Service

**Create `src/services/voiceAI/SpeechRecognitionService.ts`**:
```typescript
import Voice from '@react-native-voice/voice';

class SpeechRecognitionService {
  private onResultCallback: ((transcript: string) => void) | null = null;
  private onErrorCallback: ((error: string) => void) | null = null;

  constructor() {
    Voice.onSpeechResults = this.onSpeechResults;
    Voice.onSpeechError = this.onSpeechError;
  }

  private onSpeechResults = (event: any) => {
    const transcript = event.value[0];
    if (this.onResultCallback && transcript) {
      this.onResultCallback(transcript);
    }
  };

  private onSpeechError = (event: any) => {
    console.error('Speech recognition error:', event);
    if (this.onErrorCallback) {
      this.onErrorCallback(event.error?.message || 'Speech recognition failed');
    }
  };

  async startListening(
    onResult: (transcript: string) => void,
    onError?: (error: string) => void
  ) {
    this.onResultCallback = onResult;
    this.onErrorCallback = onError || null;

    try {
      await Voice.start('en-US');
    } catch (error) {
      console.error('Failed to start voice recognition:', error);
      if (onError) {
        onError('Failed to start listening');
      }
    }
  }

  async stopListening() {
    try {
      await Voice.stop();
    } catch (error) {
      console.error('Failed to stop voice recognition:', error);
    }
  }

  async destroy() {
    try {
      await Voice.destroy();
    } catch (error) {
      console.error('Failed to destroy voice instance:', error);
    }
  }
}

export default new SpeechRecognitionService();
```

---

### Task 3: Create Text-to-Speech Service

**Create `src/services/voiceAI/TextToSpeechService.ts`**:
```typescript
import Tts from 'react-native-tts';

class TextToSpeechService {
  constructor() {
    Tts.setDefaultLanguage('en-US');
    Tts.setDefaultRate(0.5); // Slightly slower for clarity
    Tts.setDefaultPitch(1.1); // Slightly higher for friendliness
  }

  async speak(text: string): Promise<void> {
    try {
      await Tts.speak(text);
    } catch (error) {
      console.error('TTS Error:', error);
    }
  }

  async stop(): Promise<void> {
    try {
      await Tts.stop();
    } catch (error) {
      console.error('Failed to stop TTS:', error);
    }
  }
}

export default new TextToSpeechService();
```

---

### Task 4: Update Call Redux Slice

**Update `src/store/slices/callSlice.ts`**:
```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface Message {
  speaker: 'user' | 'ai';
  text: string;
  timestamp: number;
}

interface CallState {
  isActive: boolean;
  duration: number;
  contactName: string;
  isListening: boolean;
  conversationHistory: Message[];
}

const initialState: CallState = {
  isActive: false,
  duration: 0,
  contactName: 'Sally',
  isListening: false,
  conversationHistory: [],
};

const callSlice = createSlice({
  name: 'call',
  initialState,
  reducers: {
    startCall: (state, action: PayloadAction<string>) => {
      state.isActive = true;
      state.contactName = action.payload;
      state.duration = 0;
      state.conversationHistory = [];
    },
    endCall: (state) => {
      state.isActive = false;
      state.isListening = false;
    },
    startListening: (state) => {
      state.isListening = true;
    },
    stopListening: (state) => {
      state.isListening = false;
    },
    addMessage: (state, action: PayloadAction<Message>) => {
      state.conversationHistory.push(action.payload);
    },
  },
});

export const { startCall, endCall, startListening, stopListening, addMessage } = callSlice.actions;
export default callSlice.reducer;
```

---

### Task 5: Update CallScreen with Voice AI

**Update `src/screens/CallScreen/index.tsx`**:
```typescript
import React, { useState, useEffect, useCallback } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  ScrollView,
  ActivityIndicator,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { COLORS } from '../../constants';
import { useAppDispatch, useAppSelector } from '../../store/hooks';
import { addMessage, startListening, stopListening } from '../../store/slices/callSlice';
import SpeechRecognitionService from '../../services/voiceAI/SpeechRecognitionService';
import TextToSpeechService from '../../services/voiceAI/TextToSpeechService';
import VoiceAIService from '../../services/voiceAI/VoiceAIService';

export const CallScreen = () => {
  const navigation = useNavigation();
  const dispatch = useAppDispatch();
  const { isListening, conversationHistory } = useAppSelector(state => state.call);
  const [duration, setDuration] = useState(0);
  const [isProcessing, setIsProcessing] = useState(false);

  useEffect(() => {
    const timer = setInterval(() => {
      setDuration(prev => prev + 1);
    }, 1000);

    // Start with AI greeting
    handleAIGreeting();

    return () => {
      clearInterval(timer);
      SpeechRecognitionService.destroy();
      TextToSpeechService.stop();
    };
  }, []);

  const handleAIGreeting = async () => {
    const greeting = "Hey! I'm Sally. I'm here to keep you company. How's your walk going?";
    dispatch(addMessage({ speaker: 'ai', text: greeting, timestamp: Date.now() }));
    await TextToSpeechService.speak(greeting);
  };

  const handleStartListening = useCallback(async () => {
    dispatch(startListening());
    await SpeechRecognitionService.startListening(
      async (transcript) => {
        dispatch(stopListening());
        handleUserSpeech(transcript);
      },
      (error) => {
        dispatch(stopListening());
        console.error('Speech error:', error);
      }
    );
  }, []);

  const handleUserSpeech = async (transcript: string) => {
    // Add user message
    dispatch(addMessage({ speaker: 'user', text: transcript, timestamp: Date.now() }));

    setIsProcessing(true);

    // Get AI response
    const { response, isEmergency } = await VoiceAIService.getAIResponse(transcript);

    // Add AI message
    dispatch(addMessage({ speaker: 'ai', text: response, timestamp: Date.now() }));

    // Speak AI response
    await TextToSpeechService.speak(response);

    setIsProcessing(false);

    // Trigger emergency if detected
    if (isEmergency) {
      navigation.navigate('Emergency', { level: 3 });
    }
  };

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

      {/* Conversation history */}
      <ScrollView style={styles.conversationContainer}>
        {conversationHistory.map((msg, index) => (
          <View
            key={index}
            style={[
              styles.messageBubble,
              msg.speaker === 'user' ? styles.userBubble : styles.aiBubble,
            ]}>
            <Text style={styles.messageText}>{msg.text}</Text>
          </View>
        ))}
      </ScrollView>

      {/* Voice button */}
      <View style={styles.controlsContainer}>
        {isProcessing ? (
          <ActivityIndicator size="large" color={COLORS.primary} />
        ) : (
          <TouchableOpacity
            style={[styles.voiceButton, isListening && styles.voiceButtonActive]}
            onPress={handleStartListening}
            disabled={isListening}>
            <Text style={styles.voiceButtonText}>
              {isListening ? '🎤 Listening...' : '🎤 Tap to Speak'}
            </Text>
          </TouchableOpacity>
        )}

        <View style={styles.buttonRow}>
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
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: COLORS.background,
    paddingTop: 60,
  },
  name: {
    fontSize: 36,
    fontWeight: 'bold',
    color: COLORS.text,
    textAlign: 'center',
    marginBottom: 8,
  },
  status: {
    fontSize: 16,
    color: COLORS.textSecondary,
    textAlign: 'center',
    marginBottom: 20,
  },
  duration: {
    fontSize: 24,
    color: COLORS.text,
    textAlign: 'center',
    marginBottom: 20,
  },
  conversationContainer: {
    flex: 1,
    paddingHorizontal: 20,
  },
  messageBubble: {
    padding: 12,
    borderRadius: 12,
    marginBottom: 12,
    maxWidth: '80%',
  },
  userBubble: {
    backgroundColor: COLORS.primary,
    alignSelf: 'flex-end',
  },
  aiBubble: {
    backgroundColor: COLORS.surface,
    alignSelf: 'flex-start',
  },
  messageText: {
    color: COLORS.text,
    fontSize: 16,
  },
  controlsContainer: {
    padding: 20,
  },
  voiceButton: {
    backgroundColor: COLORS.primary,
    paddingVertical: 20,
    borderRadius: 50,
    alignItems: 'center',
    marginBottom: 16,
  },
  voiceButtonActive: {
    backgroundColor: '#FF3B30',
  },
  voiceButtonText: {
    color: COLORS.text,
    fontSize: 18,
    fontWeight: '600',
  },
  buttonRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  button: {
    flex: 1,
    paddingVertical: 14,
    borderRadius: 12,
    alignItems: 'center',
    marginHorizontal: 6,
  },
  dangerButton: {
    backgroundColor: COLORS.danger,
  },
  sosButton: {
    backgroundColor: '#FF9500',
  },
  buttonText: {
    color: COLORS.text,
    fontSize: 16,
    fontWeight: '600',
  },
});
```

---

### Task 6: Request Microphone Permissions

**Create `src/utils/permissions.ts`**:
```typescript
import { Platform, PermissionsAndroid } from 'react-native';

export const requestMicrophonePermission = async (): Promise<boolean> => {
  if (Platform.OS === 'android') {
    try {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.RECORD_AUDIO,
        {
          title: 'Microphone Permission',
          message: 'Sally.ai needs access to your microphone for voice conversations',
          buttonNeutral: 'Ask Me Later',
          buttonNegative: 'Cancel',
          buttonPositive: 'OK',
        }
      );
      return granted === PermissionsAndroid.RESULTS.GRANTED;
    } catch (err) {
      console.error('Permission error:', err);
      return false;
    }
  }
  // iOS permissions handled via Info.plist
  return true;
};
```

**Update HomeScreen to request permission**:
```typescript
import { requestMicrophonePermission } from '../../utils/permissions';

// Inside HomeScreen component:
const handleStartWalk = async () => {
  const hasPermission = await requestMicrophonePermission();
  if (hasPermission) {
    navigation.navigate('Call', { isIncoming: false });
  } else {
    alert('Microphone permission is required for voice conversations');
  }
};

// Update button onPress:
<TouchableOpacity
  style={[styles.button, styles.primaryButton]}
  onPress={handleStartWalk}>
  <Text style={styles.buttonText}>Start Walk</Text>
</TouchableOpacity>
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
1. Grant microphone permission when prompted
2. Tap "Start Walk" → CallScreen opens
3. Sally greets you with voice
4. Tap "🎤 Tap to Speak" → button changes to "🎤 Listening..."
5. Say "How are you?" → see your message appear as blue bubble
6. AI responds with text (green bubble) and speaks it aloud
7. Say "I need help" → app navigates to EmergencyScreen
8. Conversation history persists during call

---

## TROUBLESHOOTING

**OpenAI API not working**:
- Replace `OPENAI_API_KEY` with your actual API key from https://platform.openai.com/api-keys
- Verify API key has GPT-4 access
- Check console for error messages

**Voice recognition not working**:
- Verify microphone permission granted
- Check iOS Info.plist has required keys
- Try uninstalling and reinstalling app
- Test on real device (simulator microphone may not work)

**TTS not speaking**:
- Check device volume is not muted
- On Android, ensure TTS engine is installed (Settings → Language → Text-to-speech)
- Try `await Tts.getInitStatus()` to verify TTS is ready

---

## COMPLETION CRITERIA

- ✅ User can speak and see transcription
- ✅ AI responds contextually to user input
- ✅ AI voice speaks responses aloud
- ✅ Conversation history displays correctly
- ✅ Emergency phrases trigger navigation to EmergencyScreen
- ✅ Microphone permission requested and handled
- ✅ Works on both iOS and Android

---

**Phase 2 Complete!** Proceed to `MVP_PHASE_3.md` to add location tracking.
