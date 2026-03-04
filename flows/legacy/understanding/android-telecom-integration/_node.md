# Understanding: Android Telecom Integration

> Native Android InCallService implementation for flutter_tele

## Phase: SYNTHESIZING

## Hypothesis

This domain handles:
- **InCallService implementation**: Extending android.telecom.InCallService
- **Call lifecycle management**: onCallAdded, onCallRemoved callbacks
- **Call state synchronization**: Tracking Android Call objects and syncing to Flutter
- **Audio routing**: Speaker/earpiece, mute/unmute via AudioManager
- **TelecomManager integration**: System telephony service interaction

## Sources

- `android/src/main/kotlin/org/telon/tele/flutter_tele/TeleService.kt` - InCallService implementation (Kotlin)

## Validated Understanding

### TeleService Class Structure

**Inheritance**: `class TeleService : InCallService()`

**Service Lifecycle**:
```kotlin
override fun onCreate() {
    super.onCreate()
    instance = this  // Singleton pattern
    mHandler = Handler(Looper.getMainLooper())
    mAudioManager = getSystemService(Context.AUDIO_SERVICE) as AudioManager
    mPowerManager = getSystemService(Context.POWER_SERVICE) as PowerManager
    mTelephonyManager = getSystemService(Context.TELEPHONY_SERVICE) as TelephonyManager
}

override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
    intent?.let { handleIntent(it) }
    return START_STICKY  // Service restarted if killed
}
```

**State Management**:
```kotlin
private var mInitialized = false
private val mCalls = mutableListOf<TeleCall>()  // Active calls tracking
private var currentCall: Call? = null  // Current Android Call
private var teleCallIds = 0  // ID generator
private val callMapping = mutableMapOf<Int, Call>()  // TeleCall ID → Android Call
```

### Intent Handling

**Action Strings**:
| Action | Handler | Description |
|--------|---------|-------------|
| START_TELEPHONY_SERVICE | initializeService() | Initialize service with config |
| MAKE_CALL | makeCall() | Initiate outgoing call |
| ANSWER_CALL | answerCall() | Answer incoming call |
| HANGUP_CALL | hangupCall() | End call |
| DECLINE_CALL | declineCall() | Reject incoming call |
| HOLD_CALL | holdCall() | Hold call |
| UNHOLD_CALL | unholdCall() | Resume held call |
| MUTE_CALL | muteCall() | Mute microphone |
| UNMUTE_CALL | unMuteCall() | Unmute microphone |
| USE_SPEAKER | useSpeaker() | Enable speakerphone |
| USE_EARPIECE | useEarpiece() | Enable earpiece |

### InCallService Callbacks

**onCallAdded(call: Call)**:
```kotlin
override fun onCallAdded(call: Call) {
    super.onCallAdded(call)
    currentCall = call
    
    // Extract call details
    val callDetails = call.details
    val handle = callDetails.handle
    val remoteNumber = handle?.schemeSpecificPart ?: ""
    val remoteName = callDetails.callerDisplayName ?: remoteNumber
    
    // Find existing TeleCall or create new one
    var teleCall = mCalls.find { it.destination == remoteNumber && it.state == "INITIATING" }
    
    if (teleCall == null) {
        // Incoming call: create new TeleCall
        teleCallIds++
        teleCall = TeleCall(
            id = teleCallIds,
            destination = remoteNumber,
            sim = 1,
            state = "INCOMING",
            direction = "DIRECTION_INCOMING",
            remoteNumber = remoteNumber,
            remoteName = remoteName
        )
        mCalls.add(teleCall)
    } else {
        // Outgoing call: update existing TeleCall
        teleCall.state = "CONNECTING"
        teleCall.remoteNumber = remoteNumber
        teleCall.remoteName = remoteName
    }
    
    // Map Android Call to TeleCall
    callMapping[teleCall.id] = call
    
    // Send event to Flutter
    FlutterTelePlugin.getInstance()?.sendEvent("call_received", teleCall.toMap() as Map<String, Any>)
    
    // Register Call.Callback for state changes
    call.registerCallback(object : Call.Callback() {
        override fun onStateChanged(call: Call, state: Int) {
            teleCall.state = when (state) {
                Call.STATE_RINGING -> "RINGING"
                Call.STATE_DISCONNECTED -> "DISCONNECTED"
                Call.STATE_ACTIVE -> "ACTIVE"
                Call.STATE_HOLDING -> "HOLDING"
                Call.STATE_DIALING -> "DIALING"
                Call.STATE_CONNECTING -> "CONNECTING"
                else -> "UNKNOWN"
            }
            FlutterTelePlugin.getInstance()?.sendEvent("call_changed", teleCall.toMap() as Map<String, Any>)
        }
        
        override fun onCallDestroyed(call: Call) {
            teleCall.state = "DISCONNECTED"
            FlutterTelePlugin.getInstance()?.sendEvent("call_terminated", teleCall.toMap() as Map<String, Any>)
            mCalls.remove(teleCall)
            callMapping.remove(teleCall.id)
        }
    })
}
```

**onCallRemoved(call: Call)**:
```kotlin
override fun onCallRemoved(call: Call) {
    super.onCallRemoved(call)
    
    // Find associated TeleCall
    val teleCallId = callMapping.entries.find { it.value == call }?.key
    val teleCall = if (teleCallId != null) findCall(teleCallId) else findCallByCall(call)
    
    if (teleCall != null) {
        teleCall.state = "DISCONNECTED"
        FlutterTelePlugin.getInstance()?.sendEvent("call_terminated", teleCall.toMap() as Map<String, Any>)
        mCalls.remove(teleCall)
        callMapping.remove(teleCall.id)
    }
    
    if (currentCall == call) {
        currentCall = null
    }
}
```

### Call Control Operations

**makeCall(sim, destination, callSettings, msgData)**:
```kotlin
private fun makeCall(sim: Int, destination: String, ...) {
    // Create TeleCall for tracking
    teleCallIds++
    val teleCall = TeleCall(id = teleCallIds, destination = destination, sim = sim, state = "INITIATING")
    mCalls.add(teleCall)
    
    // Send call_received event
    FlutterTelePlugin.getInstance()?.sendEvent("call_received", teleCall.toMap())
    
    // Make actual phone call using Intent.ACTION_CALL
    val url = "tel:$destination"
    val callIntent = Intent(Intent.ACTION_CALL, Uri.parse(url))
    callIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
    
    // Set SIM slot information
    val simSlotNames = arrayOf(
        "android.intent.extra.SLOT_ID",
        "android.intent.extra.SIM_SLOT_INDEX",
        "android.intent.extra.SUB_ID"
    )
    for (slotName in simSlotNames) {
        callIntent.putExtra(slotName, sim - 1)  // 0-based SIM slots
    }
    
    // Try to set PhoneAccountHandle for specific SIM
    try {
        val subscriptionManager = getSystemService(Context.TELEPHONY_SUBSCRIPTION_SERVICE) as SubscriptionManager
        val telecomManager = getSystemService(Context.TELECOM_SERVICE) as TelecomManager
        val phoneAccounts = telecomManager.callCapablePhoneAccounts
        if (phoneAccounts.isNotEmpty()) {
            val phoneAccount = phoneAccounts.getOrNull(sim - 1) ?: phoneAccounts.first()
            callIntent.putExtra("android.telecom.extra.PHONE_ACCOUNT_HANDLE", phoneAccount)
        }
    } catch (e: Exception) {
        Log.w(TAG, "Could not set phone account handle: ${e.message}")
    }
    
    // Start call activity
    startActivity(callIntent)
    
    // Simulate state change after delay
    mHandler?.postDelayed({
        teleCall.state = "CONNECTING"
        FlutterTelePlugin.getInstance()?.sendEvent("call_changed", teleCall.toMap())
    }, 1000)
}
```

**Audio Control Operations**:
```kotlin
private fun muteCall(callId: Int) {
    val teleCall = findCall(callId)
    if (teleCall != null) {
        mAudioManager?.isMicrophoneMute = true
        teleCall.muted = true
        FlutterTelePlugin.getInstance()?.sendEvent("call_changed", teleCall.toMap())
    }
}

private fun useSpeaker(callId: Int) {
    val teleCall = findCall(callId)
    if (teleCall != null) {
        mAudioManager?.mode = AudioManager.MODE_NORMAL
        mAudioManager?.isSpeakerphoneOn = true
        teleCall.speaker = true
        FlutterTelePlugin.getInstance()?.sendEvent("call_changed", teleCall.toMap())
    }
}

private fun useEarpiece(callId: Int) {
    val teleCall = findCall(callId)
    if (teleCall != null) {
        mAudioManager?.mode = AudioManager.MODE_IN_COMMUNICATION
        mAudioManager?.isSpeakerphoneOn = false
        teleCall.speaker = false
        FlutterTelePlugin.getInstance()?.sendEvent("call_changed", teleCall.toMap())
    }
}
```

**Hold/Unhold Operations**:
```kotlin
private fun holdCall(callId: Int) {
    val teleCall = findCall(callId)
    if (teleCall != null) {
        val androidCall = callMapping[callId] ?: currentCall
        if (androidCall != null) {
            androidCall.hold()
        }
        teleCall.held = true
        FlutterTelePlugin.getInstance()?.sendEvent("call_changed", teleCall.toMap())
    }
}

private fun unholdCall(callId: Int) {
    val teleCall = findCall(callId)
    if (teleCall != null) {
        val androidCall = callMapping[callId] ?: currentCall
        if (androidCall != null) {
            androidCall.unhold()
        }
        teleCall.held = false
        FlutterTelePlugin.getInstance()?.sendEvent("call_changed", teleCall.toMap())
    }
}
```

### Call State Mapping

**Android Call.State → TeleCall State**:
| Android State | TeleCall State |
|---------------|----------------|
| STATE_RINGING | RINGING |
| STATE_DISCONNECTED | DISCONNECTED |
| STATE_ACTIVE | ACTIVE |
| STATE_HOLDING | HOLDING |
| STATE_DIALING | DIALING |
| STATE_CONNECTING | CONNECTING |
| (other) | UNKNOWN |

### Key Insights

1. **Callback-based architecture**: Uses Call.Callback for real-time state updates
2. **Call mapping**: Maintains mapping between TeleCall IDs and Android Call objects
3. **Dual call tracking**: Tracks both outgoing (INITIATING → CONNECTING) and incoming (INCOMING → RINGING) calls
4. **AudioManager integration**: Direct audio control via AudioManager API
5. **SIM slot handling**: Tries multiple approaches (extras, PhoneAccountHandle) for dual-SIM
6. **State synchronization**: Every state change sends event to Flutter
7. **Service singleton**: Uses instance pattern for Plugin → Service communication

## Children

| Child | Status |
|-------|--------|
| (none - telecom integration is leaf concept) | N/A |

## Flow Recommendation

**Type**: SDD (Spec-Driven Development)
**Confidence**: high
**Rationale**: Internal service logic, Android framework integration.

## Bubble Up

- InCallService callbacks provide call lifecycle events
- Call mapping enables tracking multiple simultaneous calls
- AudioManager controls audio routing (speaker/earpiece, mute)
- SIM slot selection uses multiple fallback strategies

## Synthesis

> Complete understanding after analysis

### Architecture Summary

The android-telecom-integration domain implements an **InCallService callback pattern** with:
1. **InCallService extension**: Receives system callbacks for call lifecycle events
2. **Call.Callback registration**: Real-time state change notifications for each Call object
3. **Call mapping layer**: Maps TeleCall IDs to Android Call objects for bidirectional control
4. **AudioManager integration**: Direct audio routing control (speaker/earpiece, mute)
5. **Intent-based initiation**: Outgoing calls via Intent.ACTION_CALL with SIM slot extras

### Critical Design Decisions

1. **Callback chaining**: InCallService callbacks → Call.Callback → EventChannel → Flutter
2. **State synchronization**: Every Android state change triggers event to Flutter
3. **Call tracking**: Maintains mCalls list and callMapping for multi-call scenarios
4. **SIM slot fallback**: Tries 3 extra keys + PhoneAccountHandle for dual-SIM support
5. **Service singleton**: Plugin and Service communicate via static instance pattern

### Potential Issues

1. **State change simulation**: makeCall() posts delayed state change (1 second) instead of waiting for real callback
   - **Risk**: State may not match actual call state if call fails quickly
   - **Recommendation**: Rely on Call.Callback for state changes

2. **Call mapping lookup**: `findCallByCall()` uses `lastOrNull()` - may return wrong call in multi-call scenarios
   - **Risk**: Operations may affect wrong call when multiple calls active
   - **Recommendation**: Always use callMapping for lookups

3. **Audio mode changes**: useSpeaker() sets MODE_NORMAL, useEarpiece() sets MODE_IN_COMMUNICATION
   - **Risk**: May conflict with other audio-using apps
   - **Recommendation**: Save/restore previous audio mode

4. **Service lifecycle**: START_STICKY means service restarted if killed, but state is lost
   - **Risk**: mCalls and callMapping lost on restart
   - **Recommendation**: Persist call state or handle restart gracefully

5. **Null safety**: Many operations silently fail if call not found
   - **Risk**: No error feedback to Flutter
   - **Recommendation**: Send error events for failed operations

### Flow Generation Strategy

**Create SDD**: `flows/sdd-android-telecom-integration/`
- Document InCallService callback architecture
- Specify Call state mapping (Android → TeleCall)
- Document call control operations (answer, hangup, hold, mute, speaker)
- Specify SIM slot selection strategy
- Note known issues and limitations

---

*Updated by /legacy SYNTHESIZING phase*
