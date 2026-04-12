## MODIFIED Requirements

### Requirement: Pump coupling to section state
The system SHALL automatically control the pump based on section activity and weak-section pre-charge behavior, and SHALL NOT require independent schedule commands to keep pump and sections consistent.

#### Scenario: Non-weak section activation starts pump
- **WHEN** a non-weak section transitions from off to on
- **THEN** the pump SHALL be on

#### Scenario: Section handoff keeps pump running
- **WHEN** one active section is replaced by another through the device interlock behavior
- **THEN** the pump SHALL remain on throughout the handoff and SHALL NOT be power-cycled because of the brief transition gap

#### Scenario: No active sections outside pre-charge stops pump
- **WHEN** all sections are off and no weak-section pre-charge window is active
- **THEN** the pump SHALL be turned off

#### Scenario: Weak section request triggers staged pre-charge
- **WHEN** a weak section is selected for activation from any prior state
- **THEN** the controller SHALL run the pump with all sections off for the configured global pre-charge duration before opening the selected weak section

#### Scenario: Target change during pre-charge follows latest request
- **WHEN** section target selection changes while a weak pre-charge window is in progress
- **THEN** the previously pending target SHALL be canceled and the latest target SHALL be evaluated using the same weak/non-weak activation rules
