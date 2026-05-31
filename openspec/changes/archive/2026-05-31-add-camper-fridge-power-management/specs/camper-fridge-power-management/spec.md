## ADDED Requirements

### Requirement: Fridge power decision logic
The system SHALL evaluate fridge power state every 10 seconds using the following logic: `fridge_on = power_present AND (shore_power OR (inv_btn AND NOT frdrive_btn) OR (inv_btn AND frdrive_btn AND rpm > 0))` where `power_present = shore_power OR inverter_output`. When no AC source is present, the fridge switch SHALL be set to off regardless of button state.

#### Scenario: Shore power connected
- **WHEN** shore power sensor reports ON
- **THEN** fridge switch SHALL be set to ON (fridge powered) regardless of button states or RPM

#### Scenario: Inverter on, no drive mode, power present
- **WHEN** shore power is OFF and inverter output is ON
- **AND** INV_BTN is 1 and FRDRIVE_BTN is 0
- **THEN** fridge switch SHALL be set to ON

#### Scenario: Drive mode, engine running, power present
- **WHEN** INV_BTN is 1 and FRDRIVE_BTN is 1
- **AND** RPM is greater than 0
- **AND** power_present is true
- **THEN** fridge switch SHALL be set to ON

#### Scenario: Drive mode, engine not running
- **WHEN** INV_BTN is 1 and FRDRIVE_BTN is 1
- **AND** RPM is 0 or unknown
- **THEN** fridge switch SHALL be set to OFF regardless of power source

#### Scenario: No AC power source available
- **WHEN** shore_power is OFF and inverter_output is OFF
- **THEN** fridge switch SHALL be set to OFF regardless of button states

#### Scenario: Inverter off, no shore power
- **WHEN** INV_BTN is 0
- **AND** shore_power is OFF
- **THEN** fridge switch SHALL be set to OFF

### Requirement: Inverter desired-state synchronisation
The system SHALL synchronise the ESP inverter switch to match the INV_BTN memcache value on each poll cycle, calling turn_on or turn_off only when the desired state differs from the current switch state.

#### Scenario: INV_BTN set to 1, inverter currently off
- **WHEN** INV_BTN is 1 in memcache
- **AND** camper_inverter switch reports OFF
- **THEN** the system SHALL call turn_on on the inverter switch

#### Scenario: INV_BTN and inverter already in sync
- **WHEN** INV_BTN matches current inverter switch state
- **THEN** the system SHALL NOT issue any switch command to the ESP

### Requirement: Sensor state mirrored to memcache
The system SHALL write shore power and inverter output sensor states to memcache keys `camp/shore_power` and `camp/inverter_output` on each poll cycle so the web2py dashboard can display AC source status without direct ESP access.

#### Scenario: Shore power sensor ON
- **WHEN** camper_shore_power binary sensor reports ON
- **THEN** `camp/shore_power` SHALL be set to 1 in memcache

#### Scenario: ESP unreachable
- **WHEN** the ESP REST API does not respond within the timeout
- **THEN** the poll cycle SHALL be skipped with a warning log and memcache state SHALL NOT be updated

### Requirement: Relay state mirrored to memcache
The system SHALL write actual fridge and inverter relay states to memcache keys `camp/fridge_relay` and `camp/inverter_relay` after each sync cycle.

#### Scenario: Fridge relay state written after sync
- **WHEN** the fridge logic evaluation completes
- **THEN** `camp/fridge_relay` SHALL reflect the resulting fridge switch state (1 = on, 0 = off)
