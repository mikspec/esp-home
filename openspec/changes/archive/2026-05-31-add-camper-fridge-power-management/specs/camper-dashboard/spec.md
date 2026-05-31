## ADDED Requirements

### Requirement: INV and FRDRIVE control buttons
The camp dashboard SHALL expose two new toggle buttons — INV and FRDRIVE — that write their state to memcache keys `INV_BTN` and `FRDRIVE_BTN` respectively via the `obd_save` endpoint.

### Requirement: Grid Power and Inverter Output indicators
The camp dashboard SHALL display two read-only AC source indicators — GRID and INV_OUT — reflecting `camp/shore_power` and `camp/inverter_output` memcache values written by logfridge.py. Indicators SHALL be updated on each `obd_stat` poll (every 5 seconds).

#### Scenario: Shore power present
- **WHEN** `camp/shore_power` is 1 in memcache
- **THEN** the GRID indicator SHALL display as active (highlighted)

#### Scenario: No AC sources
- **WHEN** both `camp/shore_power` and `camp/inverter_output` are 0 or absent
- **THEN** both indicators SHALL display as inactive
