## Why

The carport controller is still based on legacy ESP8266 C firmware (garage branch from irrigationEsp), which is outside the current esp-home ESPHome workflow. We need a native esp-home retrofit so carport 230V plug and carport light are controlled consistently from Home Assistant and maintained in one platform.

## What Changes

- Add an ESPHome-based carport relay driver for ESP8266 with fixed mapping:
  - relay1 as carport 230V plug
  - relay2 as carport light
- Expose both controls as Home Assistant entities with stable names and IDs.
- Enforce deterministic startup behavior by relay role:
  - carport 230V plug (relay1) restores last known state after power recovery
  - carport light (relay2) starts in off state after reboot unless explicitly commanded
- Add a Home Assistant package for carport controls (service/script wrappers and optional helper entities) to keep integration consistent with existing esp-home drivers.
- Add optional watchdog functionality for carport light (relay2):
  - watchdog can be enabled/disabled
  - when enabled, light runtime is capped at configurable x minutes and forced off when exceeded
- Document migration and cutover from legacy carport C firmware behavior to ESPHome-managed entities.

## Capabilities

### New Capabilities
- `carport-relay-control`: Defines fixed relay mapping, Home Assistant entity exposure, and safe relay lifecycle behavior for carport plug and light.
- `carport-light-runtime-safety`: Defines optional watchdog controls for maximum continuous carport light runtime.

### Modified Capabilities
- None.

## Impact

- New ESPHome configuration for a carport device under config.
- New Home Assistant package under config/packages for carport control integration.
- New driver documentation under drivers for setup, mapping, and migration/cutover notes.
- Legacy carport behavior from the C firmware path is superseded after validation.
