# Node to Flow Mapping

> Maps understanding tree nodes to generated flow documents

## Completed Mappings

| Understanding Node | Flow Type | Flow Path | Status | Topics |
|-------------------|-----------|-----------|--------|--------|
| call-management | SDD | flows/legacy/sdd-call-model/ | DRAFT | TeleCall model, URI parsing, duration calculation |
| endpoint | SDD | flows/legacy/sdd-endpoint/ | DRAFT | MethodChannel, EventChannel, Intent communication |
| android-telecom-integration | SDD | flows/legacy/sdd-android-telecom-integration/ | DRAFT | InCallService, Call.Callback, audio control |
| event-streaming | SDD | flows/legacy/sdd-event-streaming/ | DRAFT | EventChannel broadcast, event routing |
| dialer | SDD | flows/legacy/sdd-dialer/ | DRAFT | TeleDialer wrapper, flutter_dialer delegation |

## Pending Mappings

(none - all domains analyzed)

## Flow Index

### SDD Flows

| Flow | Module | Key Concepts | Status |
|------|--------|--------------|--------|
| sdd-call-model | call-management | TeleCall, URI parsing, duration calculation | DRAFT |
| sdd-endpoint | endpoint | MethodChannel, EventChannel, Intent-based communication | DRAFT |
| sdd-android-telecom-integration | android-telecom-integration | InCallService, Call.Callback, AudioManager, SIM slot selection | DRAFT |
| sdd-event-streaming | event-streaming | EventChannel broadcast, event type routing, StreamController management | DRAFT |
| sdd-dialer | dialer | TeleDialer wrapper, flutter_dialer, default dialer detection | DRAFT |

### ADR Flows

| Flow | Decision | Alternatives | Status |
|------|----------|--------------|--------|
| (none) | - | - | - |

### TDD Flows

| Flow | Module | Test Coverage | Status |
|------|--------|---------------|--------|
| (none) | - | - | - |

### DDD Flows

| Flow | Feature | Stakeholders | Status |
|------|---------|--------------|--------|
| (none) | - | - | - |

### VDD Flows

| Flow | UI Component | Visual Elements | Status |
|------|--------------|-----------------|--------|
| (none) | - | - | - |

---

*Legacy analysis complete - All domains mapped*
