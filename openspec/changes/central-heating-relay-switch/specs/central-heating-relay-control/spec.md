# Spec Delta

## Purpose

Provides Home Assistant with direct control of the central-heating relay, with predictable contact behavior during normal operation and device startup.

## ADDED Requirements

### Requirement: Home Assistant controls heater demand
The system SHALL expose a Home Assistant switch whose ON state closes the heater-control contacts to request heat and whose OFF state opens them to stop requesting heat.

#### Scenario: Heat is requested
- **WHEN** the Home Assistant switch is turned ON
- **THEN** the heater-control contacts close

#### Scenario: Heat request is stopped
- **WHEN** the Home Assistant switch is turned OFF
- **THEN** the heater-control contacts open

### Requirement: Startup does not request heat
The system SHALL start with the heater-control contacts open after boot, regardless of the switch state before shutdown.

#### Scenario: Device boots or reboots
- **WHEN** the ESP32 completes startup
- **THEN** the heater-control contacts are open and no heat is requested
