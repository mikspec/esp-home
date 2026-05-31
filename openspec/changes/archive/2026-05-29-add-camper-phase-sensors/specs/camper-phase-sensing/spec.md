## ADDED Requirements

### Requirement: Shore power phase detection
The system SHALL expose a binary sensor that indicates whether AC phase from the grid is present at the inverter input (GPIO21).

#### Scenario: Shore power is connected
- **WHEN** AC phase from the grid is present at the inverter input
- **THEN** the `Camper Shore Power` binary sensor SHALL report ON state in Home Assistant

#### Scenario: Shore power is disconnected
- **WHEN** no AC phase from the grid is present at the inverter input
- **THEN** the `Camper Shore Power` binary sensor SHALL report OFF state in Home Assistant

### Requirement: Inverter output phase detection
The system SHALL expose a binary sensor that indicates whether AC phase is present at the inverter output (GPIO22), regardless of whether the source is grid pass-through or inverter generation.

#### Scenario: Phase present on inverter output
- **WHEN** AC phase is present at the inverter output
- **THEN** the `Camper Inverter Output` binary sensor SHALL report ON state in Home Assistant

#### Scenario: No phase on inverter output
- **WHEN** no AC phase is present at the inverter output
- **THEN** the `Camper Inverter Output` binary sensor SHALL report OFF state in Home Assistant

### Requirement: Independent sensor operation
The system SHALL allow each phase sensor to report its state independently so that one sensor state SHALL NOT implicitly affect the other.

#### Scenario: Shore power lost while inverter is running
- **WHEN** shore power phase is lost and inverter is generating output phase
- **THEN** `Camper Shore Power` SHALL report OFF and `Camper Inverter Output` SHALL report ON independently

### Requirement: Stable Home Assistant binary sensor exposure
The system SHALL expose both phase sensors as Home Assistant binary sensor entities using stable, role-oriented names and IDs.

#### Scenario: Entity discovery after pairing
- **WHEN** Home Assistant discovers the camper ESP32 node
- **THEN** both `Camper Shore Power` and `Camper Inverter Output` binary sensor entities SHALL be available for state monitoring
