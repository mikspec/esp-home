### Requirement: Daily irrigation run uses one sequence workflow
The system SHALL provide one daily irrigation automation that starts a single ordered sequence rather than multiple independent schedules.

#### Scenario: Daily trigger starts sequence
- **WHEN** local time reaches 05:00
- **THEN** the system SHALL start the irrigation sequence workflow

### Requirement: Optional second run for hot period
The system SHALL support an optional second daily sequence run for hot periods using a manual Home Assistant hot-period toggle and a default evening trigger time of 22:00.

#### Scenario: Hot-period second run enabled
- **WHEN** hot-period mode is enabled and the configured evening trigger time is reached
- **THEN** the system SHALL execute the same irrigation sequence workflow a second time on that day

#### Scenario: Hot-period second run disabled
- **WHEN** hot-period mode is disabled
- **THEN** only the morning daily sequence run SHALL execute

#### Scenario: Default evening trigger time
- **WHEN** hot-period mode is enabled and no custom evening time is set
- **THEN** the second run SHALL trigger at 22:00 local time

### Requirement: Sequence order and durations match target program
The sequence workflow SHALL execute sections in the defined order and duration: right 15 minutes, left 15 minutes, back 15 minutes, lines 30 minutes.

#### Scenario: Ordered section progression
- **WHEN** the daily sequence starts
- **THEN** section right SHALL run first for 15 minutes, then left for 15 minutes, then back for 15 minutes, then lines for 30 minutes

### Requirement: Sequence completion leaves safe state
The sequence workflow SHALL explicitly finalize by turning all sections and pump off.

#### Scenario: End-of-sequence cleanup
- **WHEN** the final section duration elapses
- **THEN** the workflow SHALL issue an all-off action for sections and pump
