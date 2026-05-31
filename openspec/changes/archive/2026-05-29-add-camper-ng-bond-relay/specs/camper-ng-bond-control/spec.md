## ADDED Requirements

### Requirement: Automatic neutral-ground bond when inverter is generating
The system SHALL automatically close the N-G bond relay when shore power is absent and inverter output phase is present, and SHALL open it in all other conditions.

#### Scenario: Inverter generating with no shore power
- **WHEN** the shore power binary sensor is OFF and the inverter output binary sensor is ON and ng_bond_auto is ON
- **THEN** the N-G bond relay SHALL be closed

#### Scenario: Shore power present
- **WHEN** the shore power binary sensor is ON and ng_bond_auto is ON
- **THEN** the N-G bond relay SHALL be open

#### Scenario: No AC available
- **WHEN** both shore power and inverter output binary sensors are OFF and ng_bond_auto is ON
- **THEN** the N-G bond relay SHALL be open

### Requirement: Automation enable switch with persistent state
The system SHALL expose a switch entity `camper_ng_bond_auto` that enables or disables the N-G bond automation, and this switch state SHALL persist across device reboots.

#### Scenario: Automation enable switch survives reboot
- **WHEN** the ESP32 node reboots
- **THEN** the ng_bond_auto switch SHALL restore its last set state

### Requirement: Force-open on automation disable
The system SHALL open the N-G bond relay immediately when the automation enable switch is turned OFF.

#### Scenario: Automation is disabled
- **WHEN** ng_bond_auto is switched OFF
- **THEN** the N-G bond relay SHALL be opened immediately

### Requirement: Shore power safety interlock
The system SHALL prevent the N-G bond relay from being closed at any time when shore power is present, regardless of automation state or source of the command.

#### Scenario: Manual close attempted with shore power present
- **WHEN** the shore power binary sensor is ON and a turn-on command is issued to the N-G bond relay (via automation or manual HA control)
- **THEN** the N-G bond relay SHALL remain open (or open immediately if a brief close occurs)

#### Scenario: Shore power arrives while relay is manually closed
- **WHEN** the shore power binary sensor transitions to ON while the N-G bond relay is closed
- **THEN** the N-G bond relay SHALL be opened immediately

### Requirement: Manual relay control when automation is disabled and shore power is absent
The system SHALL allow the N-G bond relay to be manually controlled via Home Assistant when ng_bond_auto is OFF and shore power is absent.

#### Scenario: Manual close when automation disabled and no shore power
- **WHEN** ng_bond_auto is OFF and shore power is absent and the user commands the N-G bond relay ON via Home Assistant
- **THEN** the N-G bond relay SHALL close and remain closed until explicitly commanded otherwise

### Requirement: Stable Home Assistant entity exposure for N-G bond
The system SHALL expose `Camper N-G Bond` and `Camper N-G Bond Auto` as stable Home Assistant entities.

#### Scenario: Entity discovery after pairing
- **WHEN** Home Assistant discovers the camper ESP32 node
- **THEN** both `Camper N-G Bond` switch and `Camper N-G Bond Auto` switch SHALL be available
