### Requirement: Fixed relay mapping for irrigation outputs
The system SHALL expose five relay outputs on ESP8266 with fixed mapping: relay1 as pump, relay2 as section left, relay3 as section right, relay4 as section back, and relay5 as section lines.

#### Scenario: Relay mapping is configured
- **WHEN** the irrigation device configuration is loaded
- **THEN** each logical output SHALL map to the assigned physical relay index and GPIO for pump, left, right, back, and lines

### Requirement: Single active irrigation section
The system SHALL enforce that at most one irrigation section is active at any time across section left, section right, section back, and section lines.

#### Scenario: New section activation deactivates previous section
- **WHEN** one section is active and a different section is turned on
- **THEN** the previously active section SHALL be turned off and only the newly selected section SHALL remain on

#### Scenario: Manual section activation still enforces exclusivity
- **WHEN** a section is activated through manual control while another section is already on
- **THEN** the system SHALL still keep exactly one active section by turning the prior section off

### Requirement: Pump coupling to section state
The system SHALL automatically control the pump based on section activity and SHALL NOT require independent schedule commands to keep pump and sections consistent.

#### Scenario: Section activation starts pump
- **WHEN** any section transitions from off to on
- **THEN** the pump SHALL be turned on

#### Scenario: Section handoff keeps pump running
- **WHEN** one active section is replaced by another through the device interlock behavior
- **THEN** the pump SHALL remain on throughout the handoff and SHALL NOT be power-cycled because of the brief transition gap

#### Scenario: No active sections stops pump
- **WHEN** all sections are off
- **THEN** the pump SHALL be turned off

### Requirement: Pump state visibility without direct user pump control
The system SHALL expose pump running state in Home Assistant while preventing direct user operation of the pump relay from Home Assistant controls.

#### Scenario: Pump state is visible
- **WHEN** pump relay state changes
- **THEN** a Home Assistant entity SHALL reflect the current pump running state

#### Scenario: Direct user pump switch is not exposed
- **WHEN** a user opens Home Assistant controls for irrigation
- **THEN** no direct pump toggle control SHALL be presented for manual switching

### Requirement: Safe startup relay state
The system SHALL initialize irrigation outputs to a safe off state after reboot unless an explicit command is issued.

#### Scenario: Device boots without irrigation command
- **WHEN** the device starts after reset or power recovery
- **THEN** all sections SHALL be off and the pump SHALL be off
