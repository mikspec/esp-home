## Why

The existing irrigation controller behavior is implemented in legacy ESP8266 C firmware and cron scripts, which makes scheduling and safety logic harder to maintain in the current esp-home stack. We need a native ESPHome and Home Assistant based retrofit that preserves proven behavior while adding safer runtime controls.

## What Changes

- Add an ESPHome-based irrigation relay driver for ESP8266 with fixed relay mapping:
  - relay1 pump, relay2 left, relay3 right, relay4 back, relay5 lines.
- Enforce single-active-section behavior across left, right, back, and lines.
- Couple pump behavior to section state:
  - pump on when any section is active, pump off when no sections are active.
  - keep pump running during normal section-to-section handoffs so interlock transitions do not power-cycle the pump.
- Add configurable max section runtime protection with default 120 minutes.
- Implement schedule orchestration using one Home Assistant sequence automation (option 1):
  - 05:00 right on, 05:15 left on, 05:30 back on, 05:45 lines on, 06:15 lines off.
- Run the sequence daily and support optional second daily run in hot period (morning and evening) via Home Assistant configuration.
- Keep pump state visible in Home Assistant while preventing direct user pump switching from Home Assistant UI controls.
- Send irrigation safety notifications to the existing `notify.all_phones` group.
- Add deterministic startup/off safety behavior to prevent unintended activation after reboot.

## Capabilities

### New Capabilities
- `irrigation-relay-control`: ESPHome relay mapping and state machine for section interlock and pump coupling.
- `irrigation-sequence-scheduling`: Home Assistant daily sequence automation equivalent to the prior cron flow.
- `irrigation-runtime-safety`: Configurable max runtime watchdog and forced safe-off handling.

### Modified Capabilities
- None.

## Impact

- New ESPHome device configuration for irrigation controller under config.
- New or updated Home Assistant package for irrigation scheduling and control scripts.
- Legacy cron-based relay orchestration is superseded by Home Assistant automation logic.
- Affects irrigation behavior and relay lifecycle management, but no external API contract changes are required.
