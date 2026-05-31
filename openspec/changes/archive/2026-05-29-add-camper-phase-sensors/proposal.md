## Why

The camper driver currently has no visibility into AC power source state. Adding two binary phase sensors enables the system to distinguish whether AC power on the inverter output originates from shore grid or from the inverter itself, which is a prerequisite for safe, context-aware load management in future changes.

## What Changes

- Add two binary phase sensor entities to the camper ESPHome configuration using commercial optocoupler modules.
- Define GPIO assignments: GPIO21 for shore power detection (inverter input), GPIO22 for inverter output phase detection.
- Expose both sensors as Home Assistant binary sensor entities with stable, role-oriented names.
- Document wiring and GPIO rationale in the camper driver documentation.

## Capabilities

### New Capabilities
- `camper-phase-sensing`: Binary detection of AC phase presence at two measurement points — shore power input and inverter output — to expose AC source state to Home Assistant.

### Modified Capabilities
- `camper-relay-control`: No requirement changes — GPIO assignments for existing relays (GPIO25, GPIO26, GPIO32, GPIO33) are unaffected by this addition.

## Impact

- Affected code:
  - `config/camper.yaml`: New `binary_sensor` blocks for GPIO21 and GPIO22.
  - `drivers/camper/README.md`: Updated pin assignment table and sensor documentation.
  - `drivers/camper/phase-sensors.md`: Wiring diagram and connection notes.
- APIs/systems:
  - Home Assistant will discover two new binary sensor entities: `Camper Shore Power` and `Camper Inverter Output`.
- Dependencies:
  - No new secrets or external dependencies required.
