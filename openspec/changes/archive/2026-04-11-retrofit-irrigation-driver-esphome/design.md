## Context

The current irrigation behavior is split across legacy ESP8266 C firmware relay endpoints and external cron shell scripts that toggle relays by HTTP calls. The retrofit moves control into the esp-home platform using ESPHome on ESP8266 plus Home Assistant orchestration. Existing expected behavior must be preserved: one active section at a time, pump coupled to section activity, and a deterministic daily sequence.

Constraints:
- Hardware is ESP8266 with five relay outputs.
- Relay mapping is fixed: relay1 pump, relay2 left, relay3 right, relay4 back, relay5 lines.
- Section lines is treated exactly like the other sections for mutual exclusion.

## Goals / Non-Goals

**Goals:**
- Preserve legacy watering flow while moving control to ESPHome and Home Assistant.
- Enforce hard mutual exclusion among irrigation sections at the device layer.
- Ensure pump state is automatically derived from section activity.
- Expose pump running state in Home Assistant without exposing a direct manual pump switch control.
- Add configurable max runtime protection with default 120 minutes.
- Implement scheduling option 1 as a single Home Assistant sequence that runs daily and can optionally run a second time in hot periods.
- Route irrigation safety notifications to the existing Home Assistant `notify.all_phones` group.

**Non-Goals:**
- Reimplement legacy C firmware UI and CGI interface.
- Provide dynamic zone runtime planning beyond the fixed sequence defined in this change.
- Introduce new hardware abstractions or additional sensors.

## Decisions

1. Device-level section interlock in ESPHome
- Decision: Use ESPHome relay switches with one interlock group for section relays (left, right, back, lines).
- Rationale: Device-level exclusivity remains true regardless of command source (HA UI, service call, automation).
- Alternative considered: Enforce exclusivity only in HA scripts. Rejected because manual UI actions or external calls can bypass script assumptions.

2. Pump is derived, not independently scheduled
- Decision: Pump relay is automatically managed from section state transitions.
- Rationale: Avoids schedule drift where a section is active but pump is off (or vice versa), and preserves pump power through short interlock handoffs when one section replaces another.
- Alternative considered: Explicit pump on/off steps in each automation stage only. Rejected because failure in one step can leave unsafe state.

3. Runtime safety watchdog in ESPHome with configurable limit
- Decision: Add a configurable max runtime control (default 120 minutes) and force safe-off when exceeded.
- Rationale: Protects against hanging automation, communication loss, or accidental manual enable.
- Alternative considered: HA-only timer helpers. Rejected because they are not reliable if HA is unavailable.

4. Scheduling option 1 in Home Assistant
- Decision: Use a sequence script triggered by a morning daily automation at 05:00, with an optional second evening trigger at 22:00 controlled by a manual HA hot-period toggle helper.
- Sequence: right 15m -> left 15m -> back 15m -> lines 30m -> all off.
- Rationale: Matches current cron behavior while centralizing schedule edits in HA.
- Alternative considered: four or five independent time automations. Rejected because it is harder to reason about partial failures and run cancellation.

5. Startup-safe defaults
- Decision: Configure relays to restore as off and enforce pump-off when no section is active on boot.
- Rationale: Prevent unintended activation after reboot or power recovery.
- Alternative considered: restore last state. Rejected due to risk of restarting irrigation unexpectedly.

6. Pump visibility without manual pump control
- Decision: Keep the pump relay as an internal control output and expose pump state via a read-only entity in Home Assistant.
- Rationale: Operators can observe pump behavior while preventing accidental manual toggling that would violate derived pump semantics.
- Alternative considered: expose pump switch normally. Rejected because users can force incompatible pump state from UI.

7. Notification channel for safety events
- Decision: Send runtime watchdog forced-safe-off notifications to `notify.all_phones`.
- Rationale: Reuses existing grouped notification channel and avoids per-device duplication.
- Alternative considered: no notification or per-device notifications. Rejected due to weaker operational visibility and maintainability.

## Risks / Trade-offs

- [HA automation interrupted mid-run] -> Mitigation: Device interlock and runtime watchdog force eventual safe state; sequence finalization includes explicit all-off.
- [Relay polarity or boot strap mismatch on ESP8266 pins] -> Mitigation: Validate GPIO behavior in commissioning checklist before production use.
- [Configurable runtime set too high/low] -> Mitigation: expose runtime as user-editable number with sane defaults and bounds.
- [Manual operation during scheduled run] -> Mitigation: define manual-stop script and ensure interlock preserves one-section invariant.
- [Section handoff briefly leaves no active section] -> Mitigation: delay pump shutdown long enough to cover ESPHome interlock transitions so the pump is not power-cycled between sections.

## Migration Plan

1. Add new ESPHome irrigation device configuration with fixed relay mapping and safety logic.
2. Add Home Assistant package with one daily sequence automation and start/stop scripts.
3. Deploy to test environment and validate section exclusivity and pump coupling.
4. Verify max runtime default (120 min) and configurable behavior.
5. Disable legacy cron relay scripts after successful validation.
6. Rollback plan: re-enable legacy cron script and previous relay endpoint control if migration fails before cutover completes.

## Open Questions

- None for this phase. Evening run default is 22:00 and hot-period mode is manual toggle.
