## Why

A new ESP32 relay X4_V1.1 board is needed to control camper loads, but the repo currently has no dedicated driver profile for this hardware and role. Defining an initial, safe baseline now reduces commissioning risk and gives a consistent starting point for iterative feature work.

## What Changes

- Add a new ESPHome configuration baseline for ESP32 relay X4_V1.1 with four fixed relay channels.
- Define initial relay role mapping focused on camper inverter and camper fridge, with two additional reserved relays.
- Establish startup and restore behavior defaults suitable for camper safety and power-recovery handling.
- Add secure connectivity baseline: Wi-Fi primary credentials, fallback AP, encrypted API, and password-protected OTA.
- Define required runtime observability entities for first deployment (Wi-Fi signal and core diagnostics).

## Capabilities

### New Capabilities
- `camper-relay-control`: Fixed four-relay mapping and role-specific relay behavior for ESP32 relay X4_V1.1 in camper use.
- `camper-connectivity-management`: Baseline Wi-Fi, fallback AP, encrypted API, and OTA update configuration for remote operations.

### Modified Capabilities
- None.

## Impact

- Affected code:
  - New ESPHome config under `config/` for the camper ESP32 relay driver.
  - Optional new Home Assistant package wiring under `config/packages/` if helper wrappers are introduced.
  - Driver documentation updates under `drivers/` and top-level README references.
- APIs/systems:
  - Home Assistant entity discovery will expose new switch and sensor entities for the camper device.
  - OTA and API access paths become available for this new node.
- Dependencies:
  - Reuses existing secrets keys in `config/secrets.yaml.template` (`wifi_ssid`, `wifi_password`, `fallback_password`, `api_encryption_key`, `ota_password`).
