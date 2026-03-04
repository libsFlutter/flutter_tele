# Understanding: Endpoint

> Flutter-Native communication layer for flutter_tele

## Phase: SYNTHESIZING

## Hypothesis

This domain handles:
- **MethodChannel communication**: Flutter → Android commands (start, makeCall, answerCall, etc.)
- **EventChannel streaming**: Android → Flutter events (call_received, call_changed, etc.)
- **Permission management**: Phone state and call permissions
- **Service lifecycle**: Telephony service start/stop coordination

## Sources

- `lib/src/endpoint.dart` - TeleEndpoint class (Dart)
- `android/.../FlutterTelePlugin.kt` - Plugin method/event handlers (Kotlin)

## Validated Understanding

### TeleEndpoint Class (Dart)

**Channel Setup**:
```dart
static const MethodChannel _channel = MethodChannel('flutter_tele')
static const EventChannel _eventChannel = EventChannel('flutter_tele_events')
```

**Methods** (15 public methods):

| Method | Return | Description |
|--------|--------|-------------|
| requestPermissions() | Future<bool> | Request phone permissions |
| hasPermissions() | Future<bool> | Check if permissions granted |
| start(configuration) | Future<Map> | Start telephony service |
| makeCall(sim, destination, settings, msgData) | Future<TeleCall> | Make outgoing call |
| answerCall(call) | Future<dynamic> | Answer incoming call |
| hangupCall(call) | Future<dynamic> | End call |
| declineCall(call) | Future<dynamic> | Reject incoming call |
| holdCall(call) | Future<dynamic> | Hold call |
| unholdCall(call) | Future<dynamic> | Resume held call |
| muteCall(call) | Future<dynamic> | Mute microphone |
| unMuteCall(call) | Future<dynamic> | Unmute microphone |
| useSpeaker(call) | Future<dynamic> | Enable speakerphone |
| useEarpiece(call) | Future<dynamic> | Enable earpiece |
| sendEnvelope(call) | Future<String> | Send SIM envelope command |
| on(eventType) | Stream<dynamic> | Get event stream for type |
| dispose() | void | Clean up resources |

**Event Handling**:
- `_setupEventChannel()`: Listens to EventChannel broadcast stream
- `_handleEvent(event)`: Routes events to StreamController by type
- `_eventControllers`: Map<eventType, StreamController> for broadcasting
- `on(eventType)`: Returns broadcast stream for event type

**Error Handling**:
- All methods wrapped in try-catch
- PlatformException caught and re-thrown as Exception
- Extensive debug logging (print statements)

### FlutterTelePlugin (Kotlin)

**Channel Registration**:
```kotlin
channel = MethodChannel(flutterPluginBinding.binaryMessenger, "flutter_tele")
eventChannel = EventChannel(flutterPluginBinding.binaryMessenger, "flutter_tele_events")
eventChannel.setStreamHandler(object : EventChannel.StreamHandler {
    override fun onListen(arguments: Any?, events: EventChannel.EventSink?) {
        eventSink = events  // Store for later use
    }
    override fun onCancel(arguments: Any?) {
        eventSink = null
    }
})
```

**Method Handlers** (15 methods):

| Method | Handler | Action |
|--------|---------|--------|
| requestPermissions | requestPermissions() | Check/request phone permissions |
| hasPermissions | hasPermissions() | Check if permissions granted |
| start | startTelephonyService() | Start TeleService with config |
| makeCall | makeCall() | Start service with MAKE_CALL intent |
| answerCall | answerCall() | Start service with ANSWER_CALL intent |
| hangupCall | hangupCall() | Start service with HANGUP_CALL intent |
| declineCall | declineCall() | Start service with DECLINE_CALL intent |
| holdCall | holdCall() | Start service with HOLD_CALL intent |
| unholdCall | unholdCall() | Start service with UNHOLD_CALL intent |
| muteCall | muteCall() | Start service with MUTE_CALL intent |
| unMuteCall | unMuteCall() | Start service with UNMUTE_CALL intent |
| useSpeaker | useSpeaker() | Start service with USE_SPEAKER intent |
| useEarpiece | useEarpiece() | Start service with USE_EARPIECE intent |
| sendEnvelope | sendEnvelope() | Return "TESTTEST" (stub) |

**Event Broadcasting**:
```kotlin
fun sendEvent(eventType: String, eventData: Map<String, Any?>) {
    val event = mapOf("type" to eventType, "data" to eventData)
    eventSink?.success(event)  // Send to Flutter
}
```

**Permissions**:
```kotlin
val permissions = arrayOf(
    Manifest.permission.READ_PHONE_STATE,
    Manifest.permission.CALL_PHONE,
    Manifest.permission.READ_CALL_LOG,
    Manifest.permission.ANSWER_PHONE_CALLS,
    Manifest.permission.MANAGE_OWN_CALLS
)
```

### Communication Pattern

**MethodChannel (Flutter → Android)**:
```
Flutter                          Android
  │                                │
  │ invokeMethod(name, args)       │
  ├───────────────────────────────►│ onMethodCall(call, result)
  │                                │ handleIntent(intent)
  │                                │ startService(intent)
  │ success(result) / error()      │
  ◄───────────────────────────────┤
  │                                │
```

**EventChannel (Android → Flutter)**:
```
Flutter                          Android
  │                                │
  │ receiveBroadcastStream()       │
  │ listen((event) => ...)         │
  ◄───────────────────────────────┤ eventSink.success(event)
  │                                │ sendEvent(type, data)
  │                                │
```

### Key Insights

1. **Intent-based communication**: Plugin doesn't call TeleService directly, uses Intents
2. **Service always started**: Every method call starts TeleService via Intent
3. **EventChannel singleton**: Single eventSink shared for all event types
4. **Stub implementation**: sendEnvelope() returns hardcoded "TESTTEST"
5. **Permission limitation**: requestPermissions() returns false (needs Activity for real request)

## Children

| Child | Status |
|-------|--------|
| (none - endpoint is leaf concept) | N/A |

## Flow Recommendation

**Type**: SDD (Spec-Driven Development)
**Confidence**: high
**Rationale**: Internal service logic, platform channel implementation.

## Bubble Up

- Uses MethodChannel for commands, EventChannel for events
- Intent-based communication with TeleService
- Permission handling incomplete (needs Activity)
- sendEnvelope() is stub implementation

## Synthesis

> Complete understanding after analysis

### Architecture Summary

The endpoint domain implements a **platform channel abstraction pattern** with:
1. **MethodChannel**: Synchronous command/response communication (Flutter → Android)
2. **EventChannel**: Asynchronous event streaming (Android → Flutter)
3. **Intent-based service communication**: Plugin acts as bridge, TeleService handles logic
4. **Broadcast event distribution**: Single eventSink → multiple StreamControllers → multiple listeners

### Critical Design Decisions

1. **Intent decoupling**: Plugin doesn't directly call TeleService methods, uses Intents for loose coupling
2. **Service-per-call pattern**: Every method starts TeleService (may cause multiple startService calls)
3. **Singleton eventSink**: Single EventChannel.StreamHandler shares eventSink for all events
4. **Event type routing**: Event type string used to route to appropriate StreamController

### Potential Issues

1. **Permission request incomplete**: `requestPermissions()` returns `false` - needs Activity for runtime permission request
2. **sendEnvelope() stub**: Returns hardcoded "TESTTEST" - not implemented
3. **Service lifecycle**: `startService()` called for every method - may cause redundant service starts
4. **Error handling**: PlatformException re-thrown as generic Exception - loses type information
5. **Debug logging**: Production code has extensive `print()` statements - should use proper logging

### Flow Generation Strategy

**Create SDD**: `flows/sdd-endpoint/`
- Document MethodChannel API specification
- Document EventChannel event types and payloads
- Specify permission requirements and handling
- Document Intent-based service communication pattern
- Note stub implementations and limitations

---

*Updated by /legacy SYNTHESIZING phase*
