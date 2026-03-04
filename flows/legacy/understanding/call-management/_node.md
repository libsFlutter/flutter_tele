# Understanding: Call Management

> Core call model and state tracking for flutter_tele

## Phase: SYNTHESIZING

## Hypothesis

This domain handles:
- **TeleCall data model**: Comprehensive call state representation
- **Call lifecycle**: From initiation to termination
- **State tracking**: Duration calculation, formatting, state transitions
- **SIP URI parsing**: Extracting caller info from SIP/tel URIs

## Sources

- `lib/src/call.dart` - TeleCall model (Dart)
- `android/.../TeleService.kt` - TeleCall data class (Kotlin)

## Validated Understanding

### TeleCall Model (Dart)

**Properties** (40+ fields):
- **Identity**: id, callId, accountId, callHashCode
- **Participants**: localContact, localUri, remoteContact, remoteUri, remoteNumber, remoteName
- **State**: state, stateText, held, muted, speaker, direction, disconnectCause
- **Timing**: connectDuration, totalDuration, creationTime, connectTime, connectTimeMillis, creationTimeMillis
- **Media**: audioCount, videoCount, remoteAudioCount, remoteVideoCount, remoteOfferer, media, provisionalMedia
- **Status**: lastStatusCode, lastReason, details, extras
- **SIM**: simSlot, simSlot1, simSlot2

**Key Methods**:
- `fromMap()`: Parses map data, extracts remoteNumber/name from SIP/tel URIs using regex
- `toMap()`: Serializes to Map
- `getTotalDuration()`: Calculates total duration including elapsed time
- `getConnectDuration()`: Calculates connected duration
- `getFormattedTotalDuration()` / `getFormattedConnectDuration()`: MM:SS formatting
- `isTerminated()`: Checks if call is disconnected
- Getters for all properties

**SIP URI Parsing**:
```dart
// Extracts name and number from: "John Doe" <sip:123@domain.com>
RegExp(r'"([^"]+)" <sip:([^@]+)@')
// Extracts from: tel:+1234567890
RegExp(r'tel:([^@]+)')
```

### TeleCall Model (Kotlin)

Simplified data class with:
- Basic fields: id, destination, sim, state, held, muted, speaker, direction, remoteNumber, remoteName
- `toMap()`: Converts to Map for event streaming to Flutter

### Key Insights

1. **Dual representation**: Dart has rich model (40+ fields), Kotlin has minimal model (10 fields)
2. **State synchronization**: Events flow from Kotlin → Dart via EventChannel
3. **Duration calculation**: Dart calculates durations dynamically using creationTimeMillis offset
4. **URI parsing complexity**: Handles multiple URI formats (SIP with name, SIP bare, tel:)

## Children

| Child | Status |
|-------|--------|
| (none - model is leaf concept) | N/A |

## Flow Recommendation

**Type**: SDD (Spec-Driven Development)
**Confidence**: high
**Rationale**: Internal service logic. Model defines data structure for entire plugin.

## Bubble Up

- Core data model used by endpoint, event-streaming, and android-telecom-integration
- Dart model is comprehensive, Kotlin model is minimal (needs synchronization)
- SIP URI parsing is critical for caller ID display

## Synthesis

> Complete understanding after analysis

### Architecture Summary

The call-management domain implements a **rich data model pattern** with:
1. **Dart side**: Comprehensive TeleCall (40+ fields) with business logic (duration calculation, URI parsing)
2. **Kotlin side**: Minimal TeleCall (10 fields) for event transmission
3. **Synchronization**: EventChannel streams Kotlin events → Dart, which enriches to full model

### Critical Design Decisions

1. **Dynamic duration calculation**: Uses `creationTimeMillis` offset to calculate elapsed time without constant updates
2. **Multi-format URI parsing**: Handles SIP with name (`"Name" <sip:num@domain>`), bare SIP, and tel: URIs
3. **Bidirectional conversion**: `fromMap()`/`toMap()` for serialization across platform channel

### Potential Issues

1. **Model mismatch**: Kotlin model lacks 30+ fields from Dart model - may cause data loss in events
2. **Time zone handling**: Duration calculations use `DateTime.now()` without timezone consideration
3. **Regex fragility**: SIP URI parsing assumes specific format, may fail on edge cases

### Flow Generation Strategy

**Create SDD**: `flows/sdd-call-model/`
- Document TeleCall data structure specification
- Define serialization/deserialization contracts
- Specify duration calculation behavior
- Document URI parsing requirements

---

*Updated by /legacy SYNTHESIZING phase*
