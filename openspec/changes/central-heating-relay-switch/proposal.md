# Proposal

## Why

The existing heating controller requires time-consuming physical setup and cannot be controlled remotely. A minimal ESPHome node will let Home Assistant operate the heater relay and verify the board's GPIO16 mapping before scheduling or temperature logic is added.

## What Changes

- Add an initial ESPHome configuration for the central-heating ESP32 relay board.
- Expose the GPIO16 relay as a Home Assistant switch; switch ON requests heat by closing the heater-control contacts.
- Default the logical switch to OFF at boot and include connectivity needed for Home Assistant control and OTA updates.
- Keep scheduling, temperature analysis, and other automation out of this first iteration.

## Capabilities

### New Capabilities
- `central-heating-relay-control`: Home Assistant control and startup behavior for the central-heating relay.

### Modified Capabilities
- None.

## Impact

- Add `config/central_heating.yaml`, reusing the repository's shared Wi-Fi, API encryption, and OTA secrets.
- Add the `central-heating-relay-control` OpenSpec capability.
- Commissioning must confirm GPIO16 drives the intended relay and that the configured output polarity makes switch ON close the heater contacts. Board identity, relay polarity, and contact wiring remain hardware assumptions until verified.
