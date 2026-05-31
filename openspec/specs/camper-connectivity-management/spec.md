### Requirement: Wi-Fi client configuration with fallback AP
The system SHALL connect using configured Wi-Fi credentials and SHALL provide a password-protected fallback access point when normal Wi-Fi connection fails.

#### Scenario: Primary Wi-Fi available
- **WHEN** valid Wi-Fi credentials are configured and network is reachable
- **THEN** the node SHALL join the primary network as a client

#### Scenario: Primary Wi-Fi unavailable
- **WHEN** the node cannot join the configured Wi-Fi network
- **THEN** the node SHALL start fallback AP mode protected by a configured password

### Requirement: Authenticated OTA update channel
The system SHALL provide OTA firmware update capability protected by an OTA password.

#### Scenario: OTA update is requested
- **WHEN** an OTA upload is initiated for the node
- **THEN** the update SHALL require valid OTA authentication before applying firmware

### Requirement: Encrypted Home Assistant API access
The system SHALL expose Home Assistant API access using configured encryption key material.

#### Scenario: Home Assistant establishes API session
- **WHEN** Home Assistant connects to the node API endpoint
- **THEN** communication SHALL use configured API encryption

### Requirement: Baseline connectivity diagnostics
The system SHALL expose Wi-Fi signal diagnostics to support deployment validation and troubleshooting.

#### Scenario: Operator checks signal quality
- **WHEN** Home Assistant reads connectivity diagnostics from the node
- **THEN** Wi-Fi signal strength data SHALL be available as a sensor entity
