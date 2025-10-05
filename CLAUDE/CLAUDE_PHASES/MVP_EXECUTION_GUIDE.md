# Sally.ai MVP Execution Guide

## Purpose of This Document

This guide explains how to use the MVP phase documents to build a functional Minimum Viable Product (MVP) of Sally.ai. The MVP focuses on core safety features while deferring advanced security, optimization, and backend infrastructure to post-MVP iterations.

---

## MVP Scope & Philosophy

### What the MVP Includes
The MVP demonstrates the **core value proposition** of Sally.ai:
1. **Simulated AI voice conversations** during walks
2. **Real-time location tracking** for safety monitoring
3. **Emergency SOS system** with contact notifications
4. **Basic call simulation** with realistic UI

### What the MVP Defers (Post-MVP)
- Advanced encryption & security infrastructure
- Backend server with database
- WebRTC optimization for sub-800ms latency
- Geofencing & route deviation detection
- Audio/video recording during emergencies
- Full accessibility features
- App store deployment
- Comprehensive testing suite

### MVP Success Criteria
✅ User can start a simulated call with AI companion
✅ AI responds to voice input with natural conversation
✅ App tracks user's location in real-time
✅ User can add emergency contacts
✅ SOS button sends SMS alerts with location to emergency contacts
✅ App runs on iOS and Android simulators without crashing

---

## MVP Phases Overview

The MVP is broken into **5 sequential phases**, each building on the previous:

| Phase | Focus | Duration | Key Deliverables |
|-------|-------|----------|------------------|
| **Phase 1** | Foundation & Navigation | 2-3 days | React Native project, navigation, state management |
| **Phase 2** | Basic Voice AI | 3-4 days | Speech recognition, OpenAI integration, TTS |
| **Phase 3** | Location Tracking | 2-3 days | GPS tracking, location history, permissions |
| **Phase 4** | Emergency SOS | 3-4 days | Contact management, SOS alerts via SMS |
| **Phase 5** | Testing & Polish | 2-3 days | Bug fixes, UI polish, basic testing |

**Total MVP Timeline**: 12-17 days (approx. 2.5-3.5 weeks)

---

## How to Use the MVP Phase Documents

Each MVP phase document (`MVP_PHASE_X.md`) is a **concise execution spec** that tells you:

1. **What to build** - Clear objectives and scope
2. **Technical requirements** - Packages to install, APIs to integrate
3. **Step-by-step tasks** - Ordered checklist of implementation steps
4. **Code examples** - Key code snippets for critical functionality
5. **Verification steps** - How to test that the phase is complete
6. **Completion criteria** - Checklist to confirm before moving to next phase

### Execution Workflow

```
┌─────────────────────────────────────────┐
│ 1. Read MVP_PHASE_X.md                  │
│    - Understand objectives              │
│    - Review technical requirements      │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 2. Install Dependencies                 │
│    - Run npm install commands           │
│    - Configure platform-specific setup  │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 3. Execute Tasks Sequentially           │
│    - Follow task checklist in order     │
│    - Implement each feature             │
│    - Use provided code examples         │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 4. Verify Functionality                 │
│    - Run verification commands          │
│    - Test on iOS and Android            │
│    - Confirm all features work          │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 5. Complete Checklist                   │
│    - Ensure all criteria met            │
│    - Fix any failing items              │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 6. Proceed to Next Phase                │
└─────────────────────────────────────────┘
```

---

## AI Agent Instructions

When executing these MVP phases, follow these guidelines:

### 1. Sequential Execution
- **Always complete phases in order**: 1 → 2 → 3 → 4 → 5
- Do NOT skip ahead or work on multiple phases simultaneously
- Each phase depends on the previous phase's foundation

### 2. Task Completion
- Mark each task as complete in your todo list as you finish it
- If a task fails, debug and fix it before moving to the next task
- Do NOT move to the next phase until ALL tasks in the current phase are complete

### 3. Code Quality (MVP Standards)
- Use TypeScript with proper types
- Follow the provided code examples closely
- Keep implementations simple - avoid over-engineering
- Hardcode API keys temporarily (security will be added post-MVP)
- Add TODO comments for features deferred to post-MVP

### 4. Testing Approach
- Test each feature immediately after implementing it
- Verify on both iOS and Android simulators
- If something doesn't work, fix it immediately
- Keep the app in a runnable state at all times

### 5. When Stuck
- Re-read the phase document carefully
- Check the CLAUDE.md spec for additional context
- Try simpler implementations first
- Ask for clarification if requirements are unclear

---

## MVP Phase Summaries

### MVP Phase 1: Foundation & Navigation
**Goal**: Set up React Native project with navigation and state management

**Key Tasks**:
- Initialize React Native TypeScript project
- Configure folder structure
- Set up React Navigation with 4 screens (Home, Call, Emergency, Settings)
- Configure Redux Toolkit for state management
- Create basic UI components

**Output**: Working app with navigation and dark mode UI

---

### MVP Phase 2: Basic Voice AI Integration
**Goal**: Implement voice conversation with AI companion

**Key Tasks**:
- Integrate @react-native-voice/voice for speech recognition
- Connect to OpenAI API for conversational responses
- Add react-native-tts for text-to-speech output
- Create CallScreen with voice conversation flow
- Implement basic conversation context

**Output**: User can have a voice conversation with Sally AI during a simulated call

---

### MVP Phase 3: Location Tracking Basics
**Goal**: Track user's location in real-time

**Key Tasks**:
- Request location permissions
- Integrate react-native-geolocation-service
- Implement GPS tracking with 30-second intervals
- Store location history in Redux
- Display current location on HomeScreen

**Output**: App continuously tracks user location and displays it

---

### MVP Phase 4: Emergency Contacts & SOS
**Goal**: Implement emergency contact management and SOS alerts

**Key Tasks**:
- Create emergency contact management UI
- Store contacts in Redux with AsyncStorage persistence
- Implement SOS button on CallScreen and HomeScreen
- Integrate Twilio or native SMS for emergency alerts
- Send SMS with user's location when SOS is activated

**Output**: User can add contacts and send emergency SMS alerts with location

---

### MVP Phase 5: Testing & Polish
**Goal**: Ensure MVP is stable and user-friendly

**Key Tasks**:
- Test complete user flow (start call → conversation → SOS)
- Fix critical bugs
- Polish UI/UX (button sizes, colors, spacing)
- Add loading states and error messages
- Write basic README for testing

**Output**: Stable, testable MVP ready for demo

---

## Post-MVP Roadmap

After completing the MVP, refer to the full CLAUDE_PLAN.md for:

1. **Security hardening** (encryption, secure storage, certificate pinning)
2. **Backend infrastructure** (Node.js API, MongoDB, Socket.IO)
3. **Advanced features** (geofencing, route deviation, video recording)
4. **Performance optimization** (battery, latency, offline mode)
5. **Production deployment** (app stores, monitoring, CI/CD)

---

## Success Metrics for MVP

Track these metrics to validate MVP success:

| Metric | Target | Measurement |
|--------|--------|-------------|
| App startup time | < 5 seconds | Manual timing |
| Voice recognition accuracy | > 80% | Test with 20 phrases |
| Location accuracy | < 50m | Compare with Google Maps |
| SOS alert delivery | 100% | Send 10 test alerts |
| Crash-free sessions | > 95% | Use Firebase Crashlytics (optional) |
| User flow completion | 100% | Manual E2E testing |

---

## Emergency Shortcuts

If you encounter blockers during MVP development:

### Voice AI Not Working
- Fallback: Use mock AI responses with setTimeout
- Test with hardcoded conversation examples
- Verify OpenAI API key is valid

### Location Permissions Denied
- Add clear permission request prompts
- Test with manual location input as fallback
- Check Info.plist (iOS) and AndroidManifest.xml permissions

### SMS Not Sending
- Use console.log to verify SOS triggered
- Test with Twilio test credentials first
- Fallback to in-app alert if SMS fails

### App Crashes
- Check console for error messages
- Clear Metro cache: `npm start -- --reset-cache`
- Rebuild: `cd ios && pod install && cd .. && npm run ios`

---

## Final Notes

**Remember**: The MVP is about **proving the concept**, not building a production-ready app.

Focus on:
- ✅ Core functionality working end-to-end
- ✅ Demonstrable user value
- ✅ Foundation for future features

Defer:
- ❌ Perfect code architecture
- ❌ Complete error handling
- ❌ Edge case coverage
- ❌ Performance optimization

**Once the MVP works, you can iterate and improve!**

---

**Ready to start?** Open `MVP_PHASE_1.md` and begin building! 🚀
