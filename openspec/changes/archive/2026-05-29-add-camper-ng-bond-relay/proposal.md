## Why

When the camper inverter generates AC power in standalone mode (no shore grid), the neutral conductor has no fixed reference to protective earth, causing a floating neutral. This produces noise and interference on touch screens and equipment with EMI filters. Using the already-available spare relay 1 (GPIO33) to create an automatic neutral-ground bond resolves the issue without additional hardware.

## What Changes

- **BREAKING** Rename `camper_spare_1_relay` (GPIO33) to `camper_ng_bond_relay` with HA entity name `"Camper N-G Bond"` — this relay is no longer a generic spare.
- Add a virtual automation-enable switch `camper_ng_bond_auto` (`"Camper N-G Bond Auto"`) backed by a persistent global.
- Add ESPHome automations: when `ng_bond_auto` is ON, automatically close the N-G bond relay when inverter is generating (shore absent, inverter output present) and open it in all other conditions.
- When `ng_bond_auto` is switched OFF, force the relay open once then yield to manual HA control.
- Update driver documentation with the full AC topology diagram.

## Capabilities

### New Capabilities
- `camper-ng-bond-control`: Automatic neutral-ground bonding for the camper inverter output, with an automation-enable switch and manual override.

### Modified Capabilities
- `camper-relay-control`: Spare relay 1 role changes from generic spare to dedicated N-G bond relay with a fixed, named purpose. The "fixed four-relay mapping" requirement is updated to reflect the new role.

## Impact

- Affected code:
  - `config/camper.yaml`: Rename spare relay 1 entity ID/name; add `globals` block, `template switch` for auto enable, and `automation` blocks referencing phase sensor binary sensors.
  - `drivers/camper/README.md`: Update relay roles table, GPIO mapping, and add full AC topology diagram.
  - `drivers/camper/inverter-floating-phase.md`: Superseded by implementation — update to reference config and diagram.
- APIs/systems:
  - Home Assistant: `Camper Spare Relay 1` entity replaced by `Camper N-G Bond` and `Camper N-G Bond Auto`. Any existing HA automations or dashboards using the old entity name will need updating.
- Dependencies:
  - Requires `camper-phase-sensing` capability (GPIO21 shore power, GPIO22 inverter output sensors) — already implemented.
