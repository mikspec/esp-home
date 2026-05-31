## MODIFIED Requirements

### Requirement: Role-specific startup and recovery behavior
The system SHALL apply role-specific startup behavior: inverter relay MUST initialize off on boot via the internal GPIO switch (bypassing the safe-shutdown script), fridge relay SHALL initialize ON on boot (fridge powered by default), and spare relays MUST initialize off.

#### Scenario: Device boots after power recovery
- **WHEN** the ESP32 node starts after reboot or power interruption
- **THEN** the inverter relay GPIO SHALL be turned off directly (not via the template switch)
- **AND** spare relays SHALL be off
- **AND** fridge relay SHALL be ON (fridge powered) so the fridge is live until logfridge.py re-evaluates within 10 seconds

### Requirement: Fridge relay GPIO semantics inverted
The fridge relay GPIO pin SHALL be configured with `inverted: true` so that the ESPHome switch entity ON state corresponds to the relay de-energised (NC closed, fridge powered) and switch OFF state corresponds to relay energised (NO connected, fridge not powered). This makes the entity name semantically correct: switch ON = fridge powered.

#### Scenario: Fridge switch turned ON via Home Assistant or REST API
- **WHEN** a turn-on command is issued to the Camper Fridge switch entity
- **THEN** the fridge relay SHALL de-energise (NC closed) and the fridge SHALL receive power

#### Scenario: Fridge switch turned OFF via Home Assistant or REST API
- **WHEN** a turn-off command is issued to the Camper Fridge switch entity
- **THEN** the fridge relay SHALL energise (NO connected) and the fridge SHALL NOT receive power
