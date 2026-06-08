### Requirement: Fixed four-relay mapping for camper loads
The system SHALL expose exactly four relay outputs for the ESP32 relay X4_V1.1 node with fixed roles: inverter relay, fridge relay, N-G bond relay, and spare relay 2.

#### Scenario: Relay mapping is initialized
- **WHEN** the camper ESP32 relay configuration is loaded
- **THEN** each relay channel SHALL be assigned to its fixed role with stable IDs and names

### Requirement: Role-specific startup and recovery behavior
The system SHALL apply role-specific startup behavior: inverter relay MUST initialize off on boot via the internal GPIO switch (bypassing the safe-shutdown script), fridge relay SHALL initialize ON on boot (relay de-energised, NC path closed), and spare relays MUST initialize off.

#### Scenario: Device boots after power recovery
- **WHEN** the ESP32 node starts after reboot or power interruption
- **THEN** the inverter relay GPIO SHALL be turned off directly (not via the template switch)
- **AND** spare relays SHALL be off
- **AND** fridge relay SHALL be ON (relay de-energised, NC path closed) until logfridge.py re-evaluates within 10 seconds
- **AND** the safe shutdown script SHALL NOT be triggered during boot

### Requirement: Independent relay operation
The system SHALL allow each relay to be controlled independently so changing one relay SHALL NOT implicitly toggle any other relay.

#### Scenario: Inverter toggle does not alter fridge state
- **WHEN** inverter relay changes from off to on or on to off
- **THEN** fridge and spare relay states SHALL remain unchanged unless explicitly commanded

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

### Requirement: Fridge relay GPIO semantics inverted
The fridge relay GPIO pin SHALL be configured with `inverted: true` so that the ESPHome switch entity ON state corresponds to relay de-energised (NC path closed) and switch OFF state corresponds to relay energised (NO connected, path open). Effective fridge power also depends on upstream AC source availability.

#### Scenario: Fridge switch turned ON via Home Assistant or REST API
- **WHEN** a turn-on command is issued to the Camper Fridge switch entity
- **THEN** the fridge relay SHALL de-energise (NC path closed)
- **AND** the fridge SHALL receive power only when an AC source is available

#### Scenario: Fridge switch turned OFF via Home Assistant or REST API
- **WHEN** a turn-off command is issued to the Camper Fridge switch entity
- **THEN** the fridge relay SHALL energise (NO connected) and the fridge SHALL NOT receive power

### Requirement: Stable Home Assistant switch exposure
The system SHALL expose Home Assistant switch entities for inverter, fridge, N-G bond relay, and spare relay 2 using stable entity naming.

#### Scenario: Entity discovery after first pairing
- **WHEN** Home Assistant discovers the camper ESP32 relay node
- **THEN** all four relay switch entities SHALL be available for on/off control
