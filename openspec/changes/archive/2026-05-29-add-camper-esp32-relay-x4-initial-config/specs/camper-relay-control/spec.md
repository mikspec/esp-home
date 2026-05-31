## ADDED Requirements

### Requirement: Fixed four-relay mapping for camper loads
The system SHALL expose exactly four relay outputs for the ESP32 relay X4_V1.1 node with fixed roles: inverter relay, fridge relay, spare relay 1, and spare relay 2.

#### Scenario: Relay mapping is initialized
- **WHEN** the camper ESP32 relay configuration is loaded
- **THEN** each relay channel SHALL be assigned to its fixed role with stable IDs and names

### Requirement: Role-specific startup and recovery behavior
The system SHALL apply role-specific startup behavior: inverter relay MUST initialize off on boot, fridge relay SHALL restore its last persisted state, and spare relays MUST initialize off.

#### Scenario: Device boots after power recovery
- **WHEN** the ESP32 node starts after reboot or power interruption
- **THEN** inverter and spare relays SHALL be off and fridge relay SHALL restore its persisted state

### Requirement: Independent relay operation
The system SHALL allow each relay to be controlled independently so changing one relay SHALL NOT implicitly toggle any other relay.

#### Scenario: Inverter toggle does not alter fridge state
- **WHEN** inverter relay changes from off to on or on to off
- **THEN** fridge and spare relay states SHALL remain unchanged unless explicitly commanded

### Requirement: Stable Home Assistant switch exposure
The system SHALL expose Home Assistant switch entities for inverter, fridge, spare relay 1, and spare relay 2 using stable entity naming.

#### Scenario: Entity discovery after first pairing
- **WHEN** Home Assistant discovers the camper ESP32 relay node
- **THEN** all four relay switch entities SHALL be available for on/off control
