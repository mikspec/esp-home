# Proposal

## Why

The room thermometer's BTHome advertisements do not reliably reach Home Assistant's Bluetooth adapter. The central-heating ESP32 is closer and can forward BLE advertisements over the existing home network.

## What Changes

- Add an advertisement-only Bluetooth proxy to the central-heating ESPHome node.
- Allow Home Assistant's existing Bluetooth and BTHome integrations to receive thermometer advertisements via the node.
- Preserve the existing heating relay, connectivity, and OFF-on-boot policy.
- Document commissioning, possible serial flashing, and verification of BLE reception and relay operation.
- Leave thermostat logic, stale-reading handling, and active GATT connections outside this change.

## Capabilities

### New Capabilities

- `central-heating-bluetooth-proxy`: Forward nearby Bluetooth Low Energy advertisements to Home Assistant through the central-heating node.

### Modified Capabilities

- None.

## Impact

- Update `config/central_heating.yaml` and the central-heating driver documentation.
- Increase ESP32 memory and shared radio usage; verify Wi-Fi and relay behavior after deployment.
- Initial BLE enablement may require a partition-table update and serial flashing.
