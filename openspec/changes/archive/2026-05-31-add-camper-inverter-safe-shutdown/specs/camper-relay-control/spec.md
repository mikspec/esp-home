## ADDED Requirements

### Requirement: Inverter N-G bond safe shutdown sequence
When a turn-off command is issued to the inverter switch entity, the system SHALL open the N-G bond relay first (if closed), wait 5 seconds, and only then de-energise the inverter relay GPIO. This sequence SHALL be enforced regardless of whether the N-G bond relay was closed via automation or manual control.

#### Scenario: Turn off inverter while N-G bond is closed
- **WHEN** a turn-off command is issued to the `Camper Inverter` switch entity
- **AND** the N-G bond relay is ON (closed)
- **THEN** the N-G bond relay SHALL be turned off immediately
- **AND** the inverter relay GPIO SHALL be turned off no sooner than 5 seconds later

#### Scenario: Turn off inverter while N-G bond is already open
- **WHEN** a turn-off command is issued to the `Camper Inverter` switch entity
- **AND** the N-G bond relay is OFF (open)
- **THEN** the inverter relay GPIO SHALL be turned off immediately with no delay

#### Scenario: External callers require no changes
- **WHEN** Home Assistant, the ESP REST API, or any external script issues `turn_off` on the `Camper Inverter` entity
- **THEN** the safe shutdown sequence SHALL execute transparently without requiring the caller to use a different endpoint or entity

## MODIFIED Requirements

### Requirement: Role-specific startup and recovery behavior
The system SHALL apply role-specific startup behavior: inverter relay MUST initialize off on boot via the internal GPIO switch (bypassing the safe-shutdown script), fridge relay SHALL restore its last persisted state, and spare relays MUST initialize off.

#### Scenario: Device boots after power recovery
- **WHEN** the ESP32 node starts after reboot or power interruption
- **THEN** the inverter relay GPIO SHALL be turned off directly (not via the template switch)
- **AND** spare relays SHALL be off
- **AND** fridge relay SHALL restore its persisted state
- **AND** the safe shutdown script SHALL NOT be triggered during boot
