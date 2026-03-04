# Understanding: Event Streaming

> Real-time event broadcasting architecture for flutter_tele

## Phase: SYNTHESIZING

## Hypothesis

This domain handles:
- **EventChannel broadcast**: Android → Flutter event streaming
- **Event type routing**: Routing events to appropriate StreamControllers
- **State synchronization**: Keeping Flutter state in sync with Android
- **Event ordering**: Ensuring events arrive in correct sequence
- **Multi-listener support**: Broadcast streams for multiple subscribers

## Sources

- `lib/src/endpoint.dart` - Event handling in TeleEndpoint (Dart)
- `android/.../FlutterTelePlugin.kt` - EventChannel StreamHandler (Kotlin)
- `android/.../TeleService.kt` - Event sending (Kotlin)

## Validated Understanding

### EventChannel Architecture (Dart Side)

**Channel Setup**:
```dart
static const EventChannel _eventChannel = EventChannel('flutter_tele_events')
StreamSubscription? _eventSubscription
final Map<String, StreamController<dynamic>> _eventControllers = {}
```

**Event Reception Pipeline**:
```dart
void _setupEventChannel() {
  _eventSubscription = _eventChannel.receiveBroadcastStream().listen((event) {
    _handleEvent(event)
  })
}

void _handleEvent(dynamic event) {
  if (event is Map) {
    final eventStringMap = event.map((key, value) => MapEntry(key.toString(), value))
    final eventType = eventStringMap['type'] as String?
    final eventData = eventStringMap['data']
    
    // Route to appropriate StreamController
    if (eventType != null && _eventControllers.containsKey(eventType)) {
      _eventControllers[eventType]!.add(eventData)
    }
  }
}
```

**StreamController Management**:
```dart
Stream<dynamic> on(String eventType) {
  // Lazy creation of StreamController
  if (!_eventControllers.containsKey(eventType)) {
    _eventControllers[eventType] = StreamController<dynamic>.broadcast()
  }
  return _eventControllers[eventType]!.stream
}

void dispose() {
  _eventSubscription?.cancel()
  for (final controller in _eventControllers.values) {
    controller.close()
  }
  _eventControllers.clear()
}
```

### Event Broadcasting (Android Side)

**EventChannel StreamHandler**:
```kotlin
eventChannel.setStreamHandler(object : EventChannel.StreamHandler {
    override fun onListen(arguments: Any?, events: EventChannel.EventSink?) {
        eventSink = events  // Store for later use
    }
    
    override fun onCancel(arguments: Any?) {
        eventSink = null
    }
})
```

**Event Sending**:
```kotlin
fun sendEvent(eventType: String, eventData: Map<String, Any?>) {
    val event = mapOf("type" to eventType, "data" to eventData)
    Log.d(TAG, "Sending event to Flutter: $event")
    eventSink?.success(event)  // Send to Flutter
}
```

### Event Types

| Event Type | Source | Payload | Description |
|------------|--------|---------|-------------|
| service_started | TeleService | { status: "initialized" } | Service initialization complete |
| call_received | TeleService | TeleCall map | New call detected (incoming or outgoing) |
| call_changed | TeleService | TeleCall map | Call state changed |
| call_terminated | TeleService | TeleCall map | Call ended |
| call_error | TeleService | { error, destination, sim } | Call error occurred |

### Event Flow Examples

**Incoming Call Flow**:
```
1. Android Telecom: onCallAdded(call)
2. TeleService: Create TeleCall, register Call.Callback
3. TeleService: sendEvent("call_received", teleCall.toMap())
4. FlutterTelePlugin: eventSink.success(event)
5. EventChannel: Broadcast to Flutter
6. TeleEndpoint: _handleEvent(event)
7. TeleEndpoint: _eventControllers["call_received"]!.add(data)
8. Flutter UI: stream.listen() receives event
```

**Call State Change Flow**:
```
1. Android Call.Callback: onStateChanged(call, state)
2. TeleService: Update teleCall.state
3. TeleService: sendEvent("call_changed", teleCall.toMap())
4. FlutterTelePlugin: eventSink.success(event)
5. EventChannel: Broadcast to Flutter
6. TeleEndpoint: Route to _eventControllers["call_changed"]
7. Flutter UI: stream.listen() receives updated state
```

**Call Termination Flow**:
```
1. Android Call.Callback: onCallDestroyed(call)
2. TeleService: Update teleCall.state = "DISCONNECTED"
3. TeleService: sendEvent("call_terminated", teleCall.toMap())
4. TeleService: Remove from mCalls, callMapping
5. FlutterTelePlugin: eventSink.success(event)
6. EventChannel: Broadcast to Flutter
7. TeleEndpoint: Route to _eventControllers["call_terminated"]
8. Flutter UI: stream.listen() receives termination event
```

### Key Insights

1. **Broadcast streams**: Uses `StreamController.broadcast()` for multiple listeners
2. **Lazy initialization**: StreamControllers created on first subscription
3. **Event type routing**: String-based routing to separate controllers
4. **Single eventSink**: One EventChannel.StreamHandler shares eventSink for all events
5. **Event format**: Consistent `{ type: String, data: Map }` structure
6. **No ordering guarantee**: Events may arrive out of order under load
7. **No retry mechanism**: Dropped events if eventSink is null

### Potential Issues

1. **Event ordering**: No sequence numbers or ordering guarantees
   - **Risk**: UI may show stale state if events arrive out of order
   - **Recommendation**: Add sequence numbers or timestamps

2. **Null eventSink**: Events silently dropped if no listener
   - **Risk**: State desynchronization if Flutter misses events
   - **Recommendation**: Buffer events or request state refresh

3. **Type safety**: Dynamic event data, no compile-time checking
   - **Risk**: Runtime errors from malformed events
   - **Recommendation**: Add event validation/schema

4. **Memory leak risk**: StreamControllers not closed if dispose() not called
   - **Risk**: Memory leaks if TeleEndpoint not disposed
   - **Recommendation**: Document dispose() requirement clearly

5. **No backpressure**: Broadcast streams don't handle backpressure
   - **Risk**: Event flood may overwhelm listeners
   - **Recommendation**: Add buffering or throttling

## Children

| Child | Status |
|-------|--------|
| (none - event streaming is leaf concept) | N/A |

## Flow Recommendation

**Type**: SDD (Spec-Driven Development)
**Confidence**: high
**Rationale**: Internal service logic, event broadcasting implementation.

## Bubble Up

- EventChannel broadcasts events from Android to Flutter
- Event type routing via Map<String, StreamController>
- Broadcast streams support multiple listeners
- No ordering guarantees or retry mechanism

## Synthesis

> Complete understanding after analysis

### Architecture Summary

The event-streaming domain implements an **EventChannel broadcast pattern** with:
1. **EventChannel**: Single channel for all event types (Android → Flutter)
2. **Event type routing**: String-based routing to separate StreamControllers
3. **Broadcast streams**: StreamController.broadcast() for multiple listeners
4. **Lazy initialization**: StreamControllers created on first subscription
5. **Centralized cleanup**: dispose() cancels subscription and closes all controllers

### Critical Design Decisions

1. **Single EventChannel**: All events flow through one channel, differentiated by type string
2. **Map-based routing**: Map<String, StreamController> for event type → stream mapping
3. **Broadcast mode**: All streams are broadcast streams (multiple listeners allowed)
4. **Dynamic event data**: No compile-time type checking, runtime type casting
5. **Fire-and-forget**: No acknowledgment, retry, or ordering guarantees

### Potential Issues

1. **Event ordering**: No sequence numbers or ordering guarantees
   - **Risk**: UI may show stale state if events arrive out of order
   - **Recommendation**: Add sequence numbers or timestamps

2. **Null eventSink**: Events silently dropped if no listener
   - **Risk**: State desynchronization if Flutter misses events
   - **Recommendation**: Buffer events or request state refresh

3. **Type safety**: Dynamic event data, no compile-time checking
   - **Risk**: Runtime errors from malformed events
   - **Recommendation**: Add event validation/schema

4. **Memory leak risk**: StreamControllers not closed if dispose() not called
   - **Risk**: Memory leaks if TeleEndpoint not disposed
   - **Recommendation**: Document dispose() requirement clearly

5. **No backpressure**: Broadcast streams don't handle backpressure
   - **Risk**: Event flood may overwhelm listeners
   - **Recommendation**: Add buffering or throttling

### Flow Generation Strategy

**Create SDD**: `flows/sdd-event-streaming/`
- Document EventChannel broadcast architecture
- Specify event types and payloads
- Document event routing mechanism
- Specify StreamController lifecycle management
- Note known issues and limitations

---

*Updated by /legacy SYNTHESIZING phase*
