### Requirement: Configurable maximum section runtime
The system SHALL provide a configurable maximum runtime limit for any active irrigation section with a default value of 120 minutes.

#### Scenario: Default runtime limit exists
- **WHEN** the irrigation controller is first deployed
- **THEN** the maximum runtime limit SHALL be set to 120 minutes

#### Scenario: Runtime limit can be adjusted
- **WHEN** an operator changes the maximum runtime configuration
- **THEN** the new runtime limit SHALL be applied for subsequent section runs

### Requirement: Runtime watchdog enforces forced safe-off
The system SHALL monitor active section runtime and force a safe-off state when the configured limit is exceeded.

#### Scenario: Active section exceeds runtime limit
- **WHEN** any section remains active longer than the configured maximum runtime
- **THEN** the system SHALL turn off all sections and turn off the pump

### Requirement: Runtime safety events notify operations group
The system SHALL send a Home Assistant notification to the existing `notify.all_phones` group when runtime watchdog forced-safe-off is triggered.

#### Scenario: Watchdog trip sends group notification
- **WHEN** a runtime limit overrun causes forced safe-off
- **THEN** the system SHALL send a notification through `notify.all_phones` indicating irrigation was stopped for safety

### Requirement: Runtime safety applies to all sections uniformly
Runtime safety enforcement SHALL apply equally to left, right, back, and lines sections.

#### Scenario: Lines section exceeds limit
- **WHEN** section lines remains active longer than the configured maximum runtime
- **THEN** the watchdog SHALL perform the same forced safe-off behavior as for other sections
