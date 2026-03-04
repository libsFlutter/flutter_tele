# Understanding: Project Root

> Entry point for recursive understanding. Children are top-level logical domains.

## Project Overview

**flutter_tele** is a Flutter plugin providing telecommunication capabilities for Android devices. It bridges Flutter applications with Android's native telephony APIs (android.telecom and InCallService) to enable:
- Making/receiving phone calls
- Call management (hold, mute, speaker control)
- Dual-SIM support
- Call event streaming

**Architecture**: Flutter Plugin (Dart + Kotlin)
- **Dart side**: Method channels, event channels, data models
- **Android side**: InCallService implementation, telecom integration

## Identified Domains

> Logical domains discovered. Each becomes a child directory for deeper exploration.

| Domain | Hypothesis | Priority | Status |
|--------|------------|----------|--------|
| call-management | Core call model and state tracking | HIGH | PENDING |
| endpoint | Flutter-Native communication layer | HIGH | PENDING |
| dialer | Default dialer replacement functionality | MEDIUM | PENDING |
| android-telecom-integration | Native Android InCallService bridge | HIGH | PENDING |
| permissions | Phone state and call permissions | MEDIUM | PENDING |
| event-streaming | Real-time call event broadcasting | HIGH | PENDING |

## Source Mapping

> Which source paths map to which logical domains

| Source Path | -> Domain |
|-------------|----------|
| lib/src/call.dart | call-management |
| lib/src/endpoint.dart | endpoint |
| lib/src/dialer.dart | dialer |
| android/.../FlutterTelePlugin.kt | endpoint, permissions |
| android/.../TeleService.kt | android-telecom-integration, event-streaming |

## Cross-Cutting Concerns

> Things that span multiple domains (may become ADRs)

- **Method Channel Protocol**: Communication between Dart and Kotlin
- **Call State Synchronization**: Keeping Dart/Android call states in sync
- **Event Channel Broadcasting**: Real-time event streaming architecture
- **Dual-SIM Handling**: SIM slot selection across layers

## Children Spawned

```
[completed] call-management → flows/sdd-call-model/ (DRAFT)
[completed] endpoint → flows/sdd-endpoint/ (DRAFT)
[completed] android-telecom-integration → flows/sdd-android-telecom-integration/ (DRAFT)
[completed] event-streaming → flows/sdd-event-streaming/ (DRAFT)
[completed] dialer → flows/sdd-dialer/ (DRAFT)
```

## Synthesis

> Updated after all children complete

### Completed Analysis - All Domains

**call-management** (DONE):
- TeleCall model: 40+ fields Dart, 10 fields Kotlin
- URI parsing: 3 patterns (SIP with name, bare SIP, tel:)
- Duration calculation: Dynamic using creationTimeMillis offset
- Model mismatch risk: Data loss in Kotlin → Dart events
- Flow generated: flows/sdd-call-model/ (DRAFT)

**endpoint** (DONE):
- MethodChannel: 15 methods (Flutter → Android)
- EventChannel: 5 event types (Android → Flutter)
- Intent-based service communication
- Permission handling incomplete (stub)
- sendEnvelope() stub implementation
- Flow generated: flows/sdd-endpoint/ (DRAFT)

**android-telecom-integration** (DONE):
- InCallService: onCallAdded, onCallRemoved callbacks
- Call.Callback: Real-time state change notifications
- Call mapping: TeleCall ID → Android Call object
- Audio control: AudioManager integration
- SIM slot selection: Multiple fallback strategies
- State change simulation (1 second delay)
- Flow generated: flows/sdd-android-telecom-integration/ (DRAFT)

**event-streaming** (DONE):
- EventChannel broadcast: Android → Flutter
- Event type routing: Map<String, StreamController>
- Broadcast streams: Multiple listeners support
- No ordering guarantees or retry mechanism
- Null eventSink drops events
- Flow generated: flows/sdd-event-streaming/ (DRAFT)

**dialer** (DONE):
- TeleDialer: Thin wrapper around flutter_dialer
- 4 static methods: isDefaultDialer, setDefaultDialer, canSetDefaultDialer, requestDefaultDialer
- Android-specific functionality
- Method naming confusion (requestDefaultDialer aliases setDefaultDialer)
- Flow generated: flows/sdd-dialer/ (DRAFT)

### Architecture Summary

**flutter_tele** is a Flutter plugin for Android telephony integration with:

1. **Platform Channel Architecture**:
   - MethodChannel: 15 commands (Flutter → Android)
   - EventChannel: 5 event types (Android → Flutter)
   - Intent-based Plugin → Service communication

2. **Call Management**:
   - Rich Dart model (40+ fields) vs minimal Kotlin model (10 fields)
   - URI parsing for caller ID extraction
   - Dynamic duration calculation

3. **Android Integration**:
   - InCallService extension for call callbacks
   - Call.Callback for real-time state changes
   - AudioManager for audio routing
   - Dual-SIM support with fallback strategies

4. **Event Streaming**:
   - Broadcast pattern with type-based routing
   - Multiple listeners per event type
   - No ordering guarantees

5. **Dialer Integration**:
   - Wrapper around flutter_dialer package
   - Default dialer detection and request

### Issues Summary

| Severity | Count | Key Issues |
|----------|-------|------------|
| **High** | 1 | Permission request stub |
| **Medium** | 6 | Model mismatch, state simulation, event ordering |
| **Low** | 9 | Time zone, regex, logging, type safety |

**Total**: 16 issues identified across all domains

---

*Legacy analysis COMPLETE - All 5 domains documented*
