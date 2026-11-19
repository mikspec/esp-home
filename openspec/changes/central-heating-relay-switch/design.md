# Design

## Context

See proposal.md for motivation. The [ESP32 Relay x1 reference](https://devices.esphome.io/devices/esp32-relay-x1/) maps the relay to GPIO16 and shows a non-inverted GPIO switch. `config/garage_gate.yaml` shows the shared home Wi-Fi, encrypted API, OTA, and fallback AP setup. Verify the physical board matches the reference.

## Goals / Non-Goals

**Goals:**
- Add a minimal ESPHome node named `central-heating` with one Home Assistant relay switch.
- Use GPIO16 and make logical switch ON close heater-control contacts.
- Ensure startup leaves heater-control contacts open.
- Enable Home Assistant connectivity and OTA using existing shared secrets.

**Non-Goals:**
- Implement temperature sensing, scheduling, hysteresis, or heating automation.
- Assume the central-heating board's relay input polarity is confirmed.

## Decisions

- Add `config/central_heating.yaml`, matching the repository's underscore convention for multiword config filenames. Use the `esp32dev` board definition and ESP-IDF framework for the ESP32-WROOM-32E.
- Configure one GPIO switch on GPIO16 with `restore_mode: ALWAYS_OFF`. Start without inversion, matching the board reference; change polarity only if commissioning shows logical ON does not actuate the relay as expected.
- Use the relay's normally-open contacts so the logical OFF/startup state leaves heater-control contacts open. Logical ON must close those contacts.
- Reuse shared home Wi-Fi secrets and garage-gate connectivity patterns, including encrypted API, OTA password, and fallback AP. Do not copy camper-specific Wi-Fi networks or add unrelated diagnostics.
- Prefer a direct GPIO switch over a template switch because no sequencing or additional safety logic is in scope.

## Risks / Trade-offs

- The physical board may differ from the referenced GPIO16 mapping → test switch actuation and contact continuity on the unconnected board before wiring heater controls.
- Relay input polarity or contact wiring may differ → confirm logical OFF is open and logical ON is closed; adjust GPIO inversion or contact selection before connecting to the heater.
- `esp32dev` may not match the exact board variant → verify ESPHome build and board startup during commissioning.

## Migration Plan

Flash the initial configuration and pair the node with Home Assistant. With heater wiring disconnected, verify boot leaves the relay contact open and that Home Assistant ON/OFF commands switch contact continuity. Roll back by restoring the previous firmware or disconnecting power; connect heater-control wiring only after the relay behavior is confirmed.
