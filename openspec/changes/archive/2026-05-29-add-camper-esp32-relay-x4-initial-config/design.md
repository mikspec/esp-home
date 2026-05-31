## Context

This change introduces a first-class ESPHome profile for an ESP32 relay X4_V1.1 board used in a camper. Existing drivers in this repository target other devices and mostly ESP8266 boards, with established conventions for secrets, fallback AP behavior, encrypted Home Assistant API, and OTA authentication. The initial release needs to prioritize safe startup behavior for high-impact loads (inverter, fridge) while keeping configuration simple enough for first deployment.

## Goals / Non-Goals

**Goals:**
- Provide an initial ESPHome configuration baseline for ESP32 relay X4_V1.1 with four relay channels.
- Define deterministic relay role mapping and startup behavior for inverter and fridge use.
- Reuse repository-standard connectivity/authentication blocks (`wifi`, fallback `ap`, encrypted `api`, and password-protected `ota`).
- Expose stable Home Assistant entities and basic diagnostics needed for bring-up.

**Non-Goals:**
- Implementing advanced automations, scheduling, or energy management logic.
- Hardware redesign, relay board reverse engineering, or adding external service dependencies.
- Defining long-term operational policies beyond baseline startup/recovery and connectivity.

## Decisions

1. Fixed role mapping over dynamic runtime remapping
- Decision: Keep four channels fixed in config (inverter, fridge, spare1, spare2) with stable IDs/names.
- Rationale: Reduces accidental miswiring/misoperation risk and improves HA entity stability.
- Alternative considered: Runtime-selectable role assignment via template helpers. Rejected for initial release due to higher complexity and misconfiguration risk.

2. Role-specific startup defaults
- Decision: Inverter defaults to safe-off on boot; fridge restores last persisted state; spare channels default off.
- Rationale: Inverter loads are safety-critical and should never energize implicitly; fridge benefits from continuity after transient power events.
- Alternative considered: All relays default off. Rejected because fridge continuity is a key camper requirement.

3. Reuse existing secrets schema and connectivity pattern
- Decision: Use existing secret keys (`wifi_ssid`, `wifi_password`, `fallback_password`, `api_encryption_key`, `ota_password`) and current ESPHome security blocks.
- Rationale: Keeps operational workflow consistent with existing nodes and avoids migration overhead.
- Alternative considered: New capability-specific secret keys. Rejected for initial config due to no immediate security benefit.

4. Keep initial observability minimal but sufficient
- Decision: Include Wi-Fi signal and core diagnostic entities only.
- Rationale: Supports deployment troubleshooting without delaying baseline rollout.
- Alternative considered: Rich telemetry package in initial scope. Deferred to follow-up change.

## Risks / Trade-offs

- [Incorrect GPIO mapping or relay polarity on X4_V1.1] -> Mitigation: Require board-level pin map verification during implementation and provide a validation checklist before production use.
- [Fridge restore behavior may not match all camper power strategies] -> Mitigation: Keep this as a documented default and add a follow-up change for configurable fridge restore policy if needed.
- [Fallback AP availability increases local attack surface if password is weak] -> Mitigation: Enforce secret-based strong password and keep API encryption + OTA password mandatory.
- [Baseline excludes higher-level automations] -> Mitigation: Explicitly scope advanced behavior to subsequent changes to preserve a safe MVP.

## Migration Plan

1. Add new ESPHome node configuration under `config/` and reference existing secrets.
2. Build/validate compilation for ESP32 target and verify relay boot states on cold and warm restarts.
3. Deploy to device in a controlled environment and verify HA entity discovery.
4. Rollback strategy: flash previous known-good firmware or disconnect node from critical loads until corrected image is deployed.

## Open Questions

- ~~Confirm final GPIO-to-relay mapping for ESP32 relay X4_V1.1 board revision in use.~~ Confirmed: inverter→GPIO25, fridge→GPIO26, spare1→GPIO33, spare2→GPIO32.
- Confirm whether fridge should restore last state or force ON for this installation.
- Confirm naming convention for camper entity prefixes to align with Home Assistant dashboard grouping.
