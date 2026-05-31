## MODIFIED Requirements

### Requirement: Fixed four-relay mapping for camper loads
The system SHALL expose exactly four relay outputs for the ESP32 relay X4_V1.1 node with fixed roles: inverter relay, fridge relay, N-G bond relay, and spare relay 2.

#### Scenario: Relay mapping is initialized
- **WHEN** the camper ESP32 relay configuration is loaded
- **THEN** each relay channel SHALL be assigned to its fixed role with stable IDs and names

### Requirement: Stable Home Assistant switch exposure
The system SHALL expose Home Assistant switch entities for inverter, fridge, N-G bond relay, and spare relay 2 using stable entity naming.

#### Scenario: Entity discovery after first pairing
- **WHEN** Home Assistant discovers the camper ESP32 relay node
- **THEN** all four relay switch entities SHALL be available for on/off control
