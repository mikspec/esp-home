## 1. ESPHome Node Baseline

- [x] 1.1 Create `config/camper.yaml` with ESP32 target, node identity, and boot-safe initialization flow.
- [x] 1.2 Add standard connectivity/security blocks (`wifi`, fallback `ap`, encrypted `api`, `ota`) using existing secrets keys.
- [x] 1.3 Add baseline diagnostics entities (Wi-Fi signal and core node diagnostics) and verify entity naming prefix consistency.

## 2. Relay Mapping And Behavior

- [x] 2.1 Define four GPIO relay switches with fixed roles: inverter, fridge, spare1, spare2, with stable IDs and names.
- [x] 2.2 Implement role-specific restore policy (inverter off on boot, fridge restore persisted state, spare relays off on boot).
- [x] 2.3 Validate independent relay operation to ensure toggling one channel does not modify other relay states.

## 3. Integration Validation

- [x] 3.1 Compile and validate the ESPHome config for ESP32 board profile used by relay X4_V1.1.
- [x] 3.2 Perform boot and power-recovery tests to confirm expected relay startup states.
- [x] 3.3 Pair with Home Assistant and verify discovery/control of four switch entities plus connectivity diagnostics.

## 4. Documentation Updates

- [x] 4.1 Add/update driver documentation under `drivers/` for board mapping, safety defaults, and commissioning steps.
- [x] 4.2 Update top-level README to include the new camper ESP32 relay driver and usage path.
- [x] 4.3 Document open follow-ups (GPIO verification results, optional fridge policy override) for next change.
