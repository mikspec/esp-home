## Context

The current carport controller behavior is tied to legacy ESP8266 C firmware from the irrigationEsp garage branch. This makes carport relay behavior harder to evolve in the esp-home platform, where new drivers are standardized on ESPHome and Home Assistant package integration.

Constraints:
- Hardware is ESP8266 relay board behavior inherited from legacy firmware assumptions.
- Required fixed mapping is relay1 -> carport 230V plug and relay2 -> carport light.
- Migration should preserve simple manual control semantics while improving boot-time safety and maintainability.

## Goals / Non-Goals

**Goals:**
- Move carport relay control to native ESPHome in this repository.
- Keep relay mapping deterministic and explicitly documented.
- Expose plug and light controls in Home Assistant with stable entity IDs.
- Ensure deterministic startup by relay role: plug restores previous state, light starts off.
- Add optional light watchdog controls so carport light can be automatically switched off after configurable x minutes.
- Provide a clean migration/cutover path from legacy C driver behavior.

**Non-Goals:**
- Reimplement the legacy web UI/API from the C firmware.
- Add complex sequencing logic like irrigation section workflows.
- Introduce new hardware interfaces beyond the two required relay outputs.

## Decisions

1. Device-level relay mapping in ESPHome
- Decision: Define two explicit relay entities in ESPHome with fixed semantic mapping (relay1 plug, relay2 light).
- Rationale: Keeps mapping stable and avoids accidental name/relay drift between firmware and Home Assistant.
- Alternative considered: Generic relay naming with only HA-level labels. Rejected because it increases operational ambiguity during maintenance and migration.

2. Safe startup defaults
- Decision: Use role-specific restore behavior: relay1 (carport 230V plug) restores last state after power recovery, while relay2 (carport light) boots to off unless explicitly commanded.
- Rationale: Plug behavior should survive power interruptions as requested, while keeping light startup predictable and non-surprising.
- Alternative considered: Force both relays off at boot. Rejected because it breaks expected plug continuity after recovery.

3. Simple independent control model
- Decision: Treat plug and light as independent controls with no mandatory interlock or coupling.
- Rationale: User requirement is straightforward direct control of two outputs.
- Alternative considered: Add coupling/interlock policy (for example one output disabling the other). Rejected for this phase because there is no stated need.

4. Home Assistant package wrapper
- Decision: Provide package-level scripts/helpers for standardized integration and easier future automation growth.
- Rationale: Matches existing repository convention used by other drivers and keeps HA behavior cohesive.
- Alternative considered: Expose raw entities only without package layer. Rejected because it fragments operational setup and documentation.

5. Optional runtime watchdog for carport light
- Decision: Implement watchdog only for relay2 (carport light) with two operator-facing controls: enable/disable toggle and max-on duration in minutes.
- Rationale: Matches user requirement for limiting how long lights can remain active while allowing the feature to be turned off when not needed.
- Alternative considered: Always-on fixed timeout. Rejected because it removes operator control and can interrupt intentional long-duration use.

## Risks / Trade-offs

- [Boot relay polarity differs from expectations] -> Mitigation: Add commissioning checklist to verify real boot state and invert GPIO logic if needed.
- [Plug restores ON unexpectedly after outage] -> Mitigation: Document restore behavior clearly and provide optional follow-up helper to force plug off on prolonged outage if needed.
- [Watchdog turns light off during intended long use] -> Mitigation: Provide explicit watchdog enable/disable control and clear max-minutes setting in HA.
- [Entity names differ from legacy references] -> Mitigation: Document canonical entity IDs and migration mapping in driver docs.
- [Watchdog state unclear after restart] -> Mitigation: Define and test restore behavior for watchdog toggle and timeout settings.

## Migration Plan

1. Add new ESPHome carport device configuration with fixed relay mapping and safe startup behavior.
2. Add Home Assistant package for carport controls and integration helpers.
3. Deploy to test environment and validate plug/light manual control and reboot safety.
4. Validate reboot behavior matches role-specific restore policy (plug restores prior state, light boots off).
5. Validate watchdog behavior: disabled mode does not enforce timeout; enabled mode turns light off when runtime exceeds configured minutes.
6. Update docs to map legacy control endpoints to new entities/scripts.
7. Disable legacy carport C-driver path after successful validation.
8. Rollback plan: temporarily re-enable legacy control path and disable ESPHome entities if production behavior deviates.

## Open Questions

- Is any default automation desired (for example evening light schedule), or should this remain manual-only in this change?
