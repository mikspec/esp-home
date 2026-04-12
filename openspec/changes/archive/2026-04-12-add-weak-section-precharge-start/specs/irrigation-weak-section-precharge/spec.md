## ADDED Requirements

### Requirement: Weak section classification and global pre-charge parameter
The system SHALL support classifying irrigation sections as weak and SHALL provide one global weak-section pre-charge duration parameter used for all weak sections.

#### Scenario: Section weak flag is configurable
- **WHEN** irrigation section configuration is loaded
- **THEN** each section SHALL have a deterministic weak/non-weak classification available to runtime control logic

#### Scenario: Global pre-charge duration is configurable
- **WHEN** irrigation runtime configuration is loaded
- **THEN** a single global pre-charge duration value SHALL be available and applied to all weak sections

### Requirement: Weak section activation always uses staged startup
The system SHALL apply staged startup for weak sections regardless of schedule position or prior active section.

#### Scenario: Weak section is first section in run
- **WHEN** a weak section is selected while all sections are currently off
- **THEN** the controller SHALL perform global pre-charge before opening that weak section

#### Scenario: Weak section follows another running section
- **WHEN** a weak section is selected while a different section is currently active
- **THEN** the controller SHALL close the prior section and perform global pre-charge before opening the weak section
