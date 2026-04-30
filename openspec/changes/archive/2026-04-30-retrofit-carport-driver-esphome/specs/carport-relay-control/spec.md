## ADDED Requirements

### Requirement: Fixed relay mapping for carport outputs
The system SHALL expose two relay outputs on ESP8266 with fixed mapping: relay1 as carport 230V plug and relay2 as carport light.

#### Scenario: Relay mapping is configured
- **WHEN** the carport device configuration is loaded
- **THEN** relay1 SHALL control the carport 230V plug and relay2 SHALL control the carport light

### Requirement: Home Assistant control exposure for both relays
The system SHALL expose both carport plug and carport light as controllable Home Assistant entities with stable names and IDs.

#### Scenario: Entities are available in Home Assistant
- **WHEN** Home Assistant discovers the ESPHome carport device
- **THEN** users SHALL be able to issue on/off commands for both the carport plug and the carport light

### Requirement: Role-specific power recovery behavior
The system SHALL apply relay-specific startup behavior after reboot: carport plug relay SHALL restore its last known state, and carport light relay SHALL initialize to off unless explicitly commanded.

#### Scenario: Plug relay restores prior state on power recovery
- **WHEN** the device starts after reset or power recovery
- **THEN** the carport plug relay SHALL return to its last persisted on/off state

#### Scenario: Light relay starts off after power recovery
- **WHEN** the device starts after reset or power recovery
- **THEN** the carport light relay SHALL be off until explicitly commanded

### Requirement: Independent relay operation
The system SHALL allow independent operation of carport plug and carport light so either output can be on or off regardless of the other output state.

#### Scenario: Plug and light are controlled independently
- **WHEN** one relay is turned on or off
- **THEN** the other relay state SHALL remain unchanged unless explicitly commanded
