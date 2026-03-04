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
[none yet - awaiting EXPLORING phase]
```

## Synthesis

> Updated after all children complete

[pending children completion]

---

*Created by /legacy ENTERING phase*
