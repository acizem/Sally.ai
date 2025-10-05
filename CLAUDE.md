# CLAUDE.md - Sally.ai Development Guide

This file provides comprehensive guidance for building Sally.ai, an AI-powered safety companion mobile application.

---

## 📋 How to Use This Documentation

**IMPORTANT**: Before starting any development work, refer to these documents in the following order:

### For MVP Development (Start Here):
1. **Read this file (CLAUDE.md)** - Understand the full technical specification and architecture
2. **Open `CLAUDE/CLAUDE_PHASES/MVP_EXECUTION_GUIDE.md`** - Master guide for MVP development approach
3. **Follow MVP Phase Documents Sequentially**:
   - `CLAUDE/CLAUDE_PHASES/MVP_PHASES/MVP_PHASE_1.md` - Foundation & Navigation
   - `CLAUDE/CLAUDE_PHASES/MVP_PHASES/MVP_PHASE_2.md` - Voice AI Integration
   - `CLAUDE/CLAUDE_PHASES/MVP_PHASES/MVP_PHASES/MVP_PHASE_3.md` - Location Tracking
   - `CLAUDE/CLAUDE_PHASES/MVP_PHASES/MVP_PHASE_4.md` - Emergency Contacts & SOS
   - `CLAUDE/CLAUDE_PHASES/MVP_PHASES/MVP_PHASE_5.md` - Testing & Polish

### For Full Production Development (Post-MVP):
1. **Read `CLAUDE/CLAUDE_PLAN.md`** - Complete 12-phase implementation plan
2. **Reference individual phase documents** in `CLAUDE/CLAUDE_PHASES/` as needed

### When Coding:
- **Always check the current MVP phase document** for specific implementation details
- **Use code examples from phase documents** as templates
- **Complete all tasks in a phase sequentially** before moving to the next
- **Verify completion criteria** before proceeding to the next phase
- **Refer back to this CLAUDE.md** for architectural context and interfaces

---

## Detailed Technical Specification

Sally.ai is an AI-powered safety companion mobile application designed to provide simulated phone call support for users walking alone at night, with seamless escalation to emergency safety features. The app combines advanced voice AI technology with robust safety infrastructure to create a natural, supportive experience that can instantly transition to emergency response when needed.[1][2]

### System Architecture Overview### Core Technology Stack**Frontend Development:**
- **Framework**: React Native with TypeScript for cross-platform compatibility[3][4]
- **Voice Integration**: @react-native-voice/voice for speech recognition[5][6]
- **Real-time Communication**: react-native-webrtc for low-latency voice processing[7][8]
- **Navigation**: React Navigation v6 for seamless screen transitions[3]
- **State Management**: Redux Toolkit with Redux Persist for reliable state handling[9]

**Backend Architecture:**
- **Voice AI Engine**: OpenAI Realtime API with WebRTC for natural conversation[10][11]
- **Speech Processing**: Whisper for speech-to-text, ElevenLabs for text-to-speech[12]
- **Server Framework**: Node.js with Express.js and Socket.IO for real-time features[13][14]
- **Database**: MongoDB with encryption for sensitive data storage[13]
- **Authentication**: JSON Web Tokens (JWT) with bcrypt password hashing[13]

**Safety Services Infrastructure:**
- **Location Services**: Google Maps API with continuous GPS tracking[15][16]
- **Emergency Communications**: Twilio for SMS/voice, Firebase for push notifications[13]
- **Audio/Video Storage**: AWS S3 with encryption for emergency recordings[17]
- **Real-time Monitoring**: WebSocket connections for instant escalation[2]

### Detailed Feature Specifications#### Voice AI Companion System
**Conversational Engine:**
- **Natural Language Processing**: GPT-4 powered conversation with safety-aware prompting[2]
- **Context Awareness**: Real-time location, time, and ambient noise analysis[12]
- **Personality Configuration**: Warm, supportive friend persona with emergency response training[1]
- **Response Optimization**: Sub-800ms latency for natural conversation flow[2]

**Voice Processing Pipeline:**
```
User Speech → WebRTC Capture → Whisper STT → GPT-4 Processing → ElevenLabs TTS → Audio Output
```

**Call Simulation Features:**
- **Outgoing Calls**: User-initiated safety conversations with customizable duration
- **Incoming Calls**: Scheduled or location-triggered automatic "check-ins"
- **Call Authenticity**: Realistic phone UI with call controls and background compatibility[10]

#### Safety Escalation System
**Emergency Detection Logic:**
- **Voice Trigger Phrases**: "Help me", "Call police", "Emergency", customizable safe words[18]
- **Ambient Threat Detection**: AI analysis of background audio for distress indicators[1]
- **Location Anomaly Detection**: Deviation from expected route or extended stops[19]
- **Manual Escalation**: One-tap SOS button accessible during conversation[19][18]

**Multi-Level Response System:**
1. **Level 1 - Concern**: Increased check-in frequency, location monitoring
2. **Level 2 - Alert**: Contact trusted contacts with location updates
3. **Level 3 - Emergency**: Full SOS activation with emergency services contact[18][19]

**SOS Features:**
- **Instant Alerts**: SMS/push notifications to emergency contacts with GPS location[19][18]
- **Audio/Video Recording**: Automatic 30-second recordings sent to contacts[19]
- **Live Location Sharing**: Real-time GPS tracking for 24 hours[20]
- **Emergency Services Integration**: Direct connection to local 911/emergency services[18]

#### Advanced Safety Features
**Location Intelligence:**
- **Route Deviation Alerts**: Notifications when user strays from planned path[15][1]
- **Safe Zone Geofencing**: Alerts when entering/leaving designated safe areas[18][19]
- **Real-time Tracking**: Continuous GPS with 30-second update intervals[15]

**Contact Management System:**
- **Emergency Circle**: Priority contacts with instant notification capabilities[21]
- **Escalation Hierarchy**: Automatic contact progression if primary contacts don't respond[18]
- **Contact Verification**: Two-way confirmation system for emergency situations[19]

### Mobile App Architecture#### Project Structure (React Native)
```
src/
├── components/
│   ├── common/
│   │   ├── Button/
│   │   ├── SafetyButton/
│   │   └── VoiceInterface/
│   ├── voice/
│   │   ├── CallSimulator/
│   │   ├── VoiceRecognition/
│   │   └── ConversationEngine/
│   └── safety/
│       ├── EmergencyPanel/
│       ├── LocationTracker/
│       └── ContactList/
├── screens/
│   ├── HomeScreen/
│   ├── CallScreen/
│   ├── EmergencyScreen/
│   └── SettingsScreen/
├── services/
│   ├── VoiceAI/
│   ├── LocationService/
│   ├── EmergencyService/
│   └── NotificationService/
├── utils/
│   ├── permissions/
│   ├── encryption/
│   └── constants/
└── navigation/
    └── AppNavigator.tsx
```

#### Security Architecture
**Data Protection:**
- **End-to-End Encryption**: AES-256 encryption for all sensitive communications[17]
- **Local Data Security**: Secure storage using React Native Keychain/Keystore[17]
- **Privacy Controls**: User-controlled data retention and deletion policies[17]
- **Secure Communication**: SSL/TLS for all API communications with certificate pinning[17]

**Permission Management:**
- **Microphone Access**: Continuous recording permission for voice AI functionality[5]
- **Location Services**: High-accuracy GPS access for safety features[16]
- **Camera Access**: Optional video recording during emergency situations[18]
- **Contact Access**: Emergency contact management and quick dial features[19]

### Real-time Communication Implementation**WebRTC Integration:**
```javascript
// Voice AI WebRTC Configuration
const webRTCConfig = {
  iceServers: [{ urls: 'stun:stun.l.google.com:19302' }],
  iceCandidatePoolSize: 10
};

// Emergency escalation during voice call
const handleEmergencyEscalation = async () => {
  await startLocationTracking();
  await notifyEmergencyContacts();
  await beginAudioRecording();
};
```

**Performance Optimization:**
- **Audio Latency**: <500ms round-trip for natural conversation flow[2]
- **Battery Optimization**: Intelligent power management during extended use[16]
- **Network Resilience**: Offline capability for critical safety features[1]
- **Background Processing**: Continuous operation when screen is locked[22]

---

## Claude Code Prompt for Sally.ai Mobile App Development```markdown
# Claude Code Prompt: Sally.ai - AI Safety Companion Mobile App

You are an expert React Native developer tasked with building Sally.ai, a safety companion app that provides simulated AI phone conversations for users walking alone at night, with seamless emergency escalation capabilities.

## Project Overview
Build a complete React Native mobile application with TypeScript that:
1. Simulates realistic phone calls with an AI companion using voice interaction
2. Provides natural, supportive conversation during nighttime walks
3. Seamlessly escalates to emergency features when safety concerns arise
4. Integrates real-time voice AI, location tracking, and emergency response systems

## Core Technical Requirements

### 1. Project Structure
Create a well-organized React Native project structure with:
- TypeScript configuration for type safety
- Modular component architecture (common, voice, safety, screens)
- Service layer for AI, location, emergency, and notification services
- Proper navigation setup with React Navigation v6
- Redux Toolkit for state management with persistence

### 2. Voice AI Integration
Implement real-time voice conversation system:
- Use @react-native-voice/voice for speech recognition
- Integrate react-native-webrtc for low-latency audio processing
- Connect to OpenAI Realtime API for conversational AI responses
- Implement text-to-speech for AI companion voice output
- Create natural conversation flow with context awareness
- Target <800ms response latency for natural interaction

### 3. Safety Features Implementation
Build comprehensive safety system:
- SOS button with one-tap emergency activation
- Real-time GPS location tracking and sharing
- Emergency contact notification system with SMS/calls
- Audio/video recording during emergency situations
- Geofencing for route deviation detection
- Multi-level escalation (concern → alert → emergency)

### 4. Call Simulation System
Create realistic phone call interface:
- Authentic call UI that mimics native phone apps
- Support for both outgoing and incoming call simulation
- Background call functionality when app is minimized
- Call controls (mute, speaker, hang up) that work with AI
- Scheduled "check-in" calls at user-defined intervals

### 5. Security & Privacy
Implement robust security measures:
- End-to-end encryption for sensitive communications
- Secure local storage using React Native Keychain
- Permission management for microphone, location, camera, contacts
- Privacy controls for data retention and deletion
- SSL/TLS with certificate pinning for API communications

## Specific Implementation Guidelines

### Voice AI Component Structure:
```
interface VoiceAIService {
  startConversation(): Promise<void>;
  processVoiceInput(audio: AudioData): Promise<string>;
  generateResponse(input: string, context: ConversationContext): Promise<AudioResponse>;
  detectEmergencyTriggers(transcript: string): EmergencyLevel;
  escalateToEmergency(): Promise<void>;
}

interface ConversationContext {
  location: GPSCoordinates;
  timeOfDay: string;
  routeProgress: RouteInfo;
  ambientNoise: NoiseLevel;
  userMood: EmotionalState;
}
```

### Safety Service Architecture:
```
interface SafetyService {
  activateSOS(level: EmergencyLevel): Promise<void>;
  trackLocation(): Promise<GPSCoordinates>;
  notifyEmergencyContacts(alert: EmergencyAlert): Promise<void>;
  startEmergencyRecording(): Promise<RecordingId>;
  checkRouteDeviation(): Promise<boolean>;
  sendLocationUpdates(): void;
}

enum EmergencyLevel {
  CONCERN = 1,
  ALERT = 2,  
  EMERGENCY = 3
}
```

### UI/UX Requirements:
- Clean, intuitive interface optimized for night-time use
- Large, easily accessible emergency buttons
- Dark mode with high contrast for low-light visibility  
- Voice-first interaction design with minimal screen interaction needed
- Quick access to safety features from any screen
- Accessibility support for users with disabilities

### Real-time Features:
- WebSocket connections for instant emergency notifications
- Live location sharing with trusted contacts
- Real-time conversation processing with minimal latency
- Background operation for continuous safety monitoring
- Push notifications for check-ins and alerts

### Performance Requirements:
- Battery optimization for extended use during walks
- Offline capability for core safety functions
- Fast app startup (<3 seconds) for emergency situations
- Reliable operation in low-network conditions
- Memory efficient audio processing

## Development Approach
1. Start with basic project structure and navigation
2. Implement voice recognition and basic AI conversation
3. Add call simulation UI and functionality
4. Integrate location services and tracking
5. Build emergency escalation system
6. Add contact management and notifications
7. Implement security and encryption features
8. Optimize performance and add error handling
9. Create comprehensive testing suite
10. Add accessibility and localization support

## Code Quality Standards
- Use TypeScript for all components and services
- Follow React Native best practices and patterns
- Implement proper error handling and user feedback
- Add comprehensive logging for debugging
- Use proper state management patterns
- Include unit and integration tests
- Document all public APIs and complex functions
- Follow security best practices for sensitive data handling

## Testing Requirements
- Unit tests for all core services and utilities
- Integration tests for voice AI and safety features
- End-to-end testing for complete user flows
- Performance testing for audio processing and battery usage
- Security testing for data encryption and API communications
- Accessibility testing for inclusive design
- Network resilience testing for offline functionality

Please build this application incrementally, starting with the foundation and gradually adding complexity. Focus on creating a production-ready, secure, and user-friendly safety application that could genuinely help protect users walking alone at night.

Prioritize clean, maintainable code with proper error handling, comprehensive testing, and detailed documentation. The app should feel natural and supportive while maintaining robust emergency response capabilities.
```

***

This comprehensive technical specification and Claude code prompt provides everything needed to build Sally.ai as a production-ready safety companion application. The specification covers all technical aspects from architecture to implementation details, while the Claude prompt provides specific development guidance for creating a robust, secure, and user-friendly mobile application that combines advanced AI voice technology with critical safety features.][2][17][19][18]

[1](https://www.ijrti.org/papers/IJRTI2505120.pdf)
[2](https://webrtc.ventures/2025/07/how-to-build-voice-ai-applications-a-complete-developer-guide/)
[3](https://dev.to/paulocappa/how-to-organize-your-components-in-react-native-folder-structure-and-project-organization-1hke)
[4](https://www.tricentis.com/learn/react-native-project-structure)
[5](https://dev.to/amitkumar13/how-to-implement-voice-to-text-in-react-native-3j0d)
[6](https://blog.logrocket.com/build-react-native-speech-to-text-dictation-app/)
[7](https://reactnativeexpert.com/blog/building-real-time-communication-in-react-native-with-webrtc/)
[8](https://videosdk.live/developer-hub/webrtc/webrtc-react-native)
[9](https://dev.to/rushitjivani/ultimate-folder-structure-for-your-react-native-project-1k27)
[10](https://community.openai.com/t/real-time-model-is-hearing-and-talking-to-itself-in-a-loop/1066520)
[11](https://www.youtube.com/watch?v=t12ekS7Vgh4)
[12](https://www.forasoft.com/blog/article/voice-activated-mobile-apps-ai-nlp-integration)
[13](https://www.i-webservices.com/blog/personal-security-app-development/)
[14](https://appmaster.io/blog/developing-personal-safety-app)
[15](https://www.topdevelopers.co/blog/location-tracking-apps/)
[16](https://play.google.com/store/apps/details?id=com.app.realtimetracker&hl=en_US)
[17](https://dig8ital.com/post/secure-mobile-apps/)
[18](https://www.aalpha.net/articles/how-to-build-sos-mobile-application/)
[19](https://acquaintsoft.com/blog/how-to-develop-emergency-sos-mobile-application)
[20](https://insights.samsung.com/2025/03/31/how-to-set-up-emergency-sos-on-your-galaxy-mobile-device/)
[21](https://github.com/Thabhelo/sos)
[22](https://support.google.com/android/answer/9319337?hl=en)
[23](https://www.reddit.com/r/reactnative/comments/1lkdkta/whats_the_best_architecture_for_building_a_mobile/)
[24](https://github.com/react-native-voice/voice)
[25](https://www.rapidevelopers.com/build-ideas-with-no-code/build-personal-safety-and-emergency-alert-app-no-code)
[26](https://vapi.ai)
[27](https://www.reddit.com/r/reactnative/comments/1kotpsj/reactnativevoice2text_add_voicetotext_to_your/)
[28](https://moldstud.com/articles/p-essential-tips-and-tricks-for-designing-user-friendly-interfaces-in-public-safety-apps)
[29](https://voice.ai)
[30](https://alan.app/docs/tutorials/react-native/integrating-react-native/)
[31](https://tateeda.com/blog/fundamentals-of-mobile-application-architecture)
[32](https://docs.livekit.io/agents/start/voice-ai/)
[33](https://github.com/react-native-voice)
[34](https://www.linkedin.com/pulse/personal-safety-app-market-enhancing-real-time-protection-through-vw1wc)
[35](https://www.youtube.com/watch?v=sc1_ltGGF4M)
[36](https://thisisglance.com/blog/key-principles-of-effective-mobile-application-architecture-design)
[37](https://www.simform.com/blog/mobile-application-architecture/)
[38](https://www.safetydetectives.com/blog/best-family-locator-apps-for-iphone-and-android/)
[39](https://owasp.org/www-project-mobile-application-security-design-guide/)
[40](https://www.gpswox.com/en/gps-trackers-shop/all/mobile-gps-tracker-3)
[41](https://softwaremind.com/blog/mobile-app-architecture-best-practices/)
[42](https://siriussignal.com/the-power-and-limitations-of-smartphone-emergency-sos-features/)
[43](https://family1st.io/best-family-tracking-apps-for-iphone-android/)
[44](https://developer.android.com/topic/architecture)
[45](https://www.reddit.com/r/androidapps/comments/1i5ge0k/looking_for_a_good_app_that_tracks_a_phone/)
[46](https://www.linkedin.com/advice/0/what-most-common-design-patterns-mobile-app-ygtee)
[47](https://support.apple.com/guide/iphone/use-emergency-sos-via-satellite-iph2968440de/ios)
[48](https://apidog.com/blog/claude-code-prompts/)
[49](https://www.linkedin.com/pulse/how-integrate-ai-chatbots-your-mobile-app-topdevelopers-co-kju4f)
[50](https://www.indragie.com/blog/i-shipped-a-macos-app-built-entirely-by-claude-code)
[51](https://www.reactnative.express/app/project_structure)
[52](https://www.mobiloud.com/blog/in-app-ai-chatbots)
[53](https://www.youtube.com/watch?v=U8bZwtp5PAQ)
[54](https://www.solzit.com/how-ai-chatbots-are-revolutionizing-mobile-applications/)
[55](http://plankenau.com/blog/post/claude-coding-an-ios-app)
[56](https://www.brilworks.com/blog/integrating-ai-in-mobile-apps/)
[57](https://www.anthropic.com/news/claude-sonnet-4-5)
[58](https://www.reddit.com/r/reactnative/comments/1gk4kgb/how_do_you_structure_your_react_native_projects/)
[59](https://perpet.io/blog/a-detailed-guide-to-ai-chatbot-integration-in-mobile-apps/)
[60](https://www.reddit.com/r/ClaudeCode/comments/1nepkej/has_anyone_used_claude_code_for_iosandroid_app/)
[61](https://www.youtube.com/watch?v=PjmAVh4Syvk)
[62](https://technologyadvice.com/blog/voip/ai-chatbot-examples/)
[63](https://www.builder.io/blog/claude-code)
[64](https://github.com/suryakarmakar/ReactNative-ProjectStructure)
[65](https://www.octalsoftware.com/blog/emergency-alert-app-development)
[66](https://www.muvi.com/live/audio-live-streaming/)
[67](https://www.researchpublish.com/upload/book/Emergency%20Notification%20Services%20Application-4657.pdf)
[68](https://play.google.com/store/apps/details?id=io.livevoice.client&hl=en_US)
[69](https://www.reddit.com/r/react/comments/1f2ektq/realtime_voice_filters_with_react_and_webrtc/)
[70](https://www.simublade.com/case-study/incident-alert)
[71](https://www.limestreamer.com)
[72](https://www.brihaspatitech.com/blog/emergency-notification-app-a-step-by-step-development-guide/)
[73](https://streamlabs.com/mobile-app)
[74](https://docs.livekit.io/home/quickstarts/react-native/)
[75](https://peasi.com/blog/mobile-app-alerts-for-emergency-notification)
[76](https://play.google.com/store/apps/details?id=com.mixlr.android&hl=en_US)
[77](https://www.appmaisters.com/case-study/emergency-alert-safety-app/)
[78](https://www.whathifi.com/best-buys/best-free-music-apps-free-music-on-android-and-iphone)
[79](https://github.com/react-native-webrtc/react-native-webrtc)