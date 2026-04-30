### Requirement: Switchable watchdog for carport light runtime
The system SHALL provide an operator-controlled watchdog for carport light (relay2) that can be enabled or disabled.

#### Scenario: Watchdog is disabled
- **WHEN** the light watchdog toggle is off
- **THEN** no automatic light runtime shutdown SHALL be enforced by the watchdog

#### Scenario: Watchdog is enabled
- **WHEN** the light watchdog toggle is on
- **THEN** light runtime SHALL be monitored against the configured maximum duration

### Requirement: Configurable maximum light active duration
The system SHALL provide a configurable maximum continuous active duration x (in minutes) for carport light watchdog enforcement.

#### Scenario: Max duration is configured
- **WHEN** an operator sets the watchdog maximum duration value
- **THEN** the new value SHALL be used for subsequent watchdog checks

### Requirement: Forced light off on watchdog timeout
When the watchdog is enabled, the system SHALL automatically turn carport light off if continuous active runtime exceeds the configured maximum duration x minutes.

#### Scenario: Active runtime exceeds configured maximum
- **WHEN** carport light remains on longer than configured x minutes while watchdog is enabled
- **THEN** the system SHALL switch carport light off automatically

### Requirement: Runtime timeout applies only to light relay
Watchdog timeout enforcement SHALL apply only to relay2 (carport light) and SHALL NOT alter relay1 (carport 230V plug) state.

#### Scenario: Watchdog timeout occurs while plug is on
- **WHEN** light watchdog timeout turns relay2 off
- **THEN** relay1 state SHALL remain unchanged unless explicitly commanded
