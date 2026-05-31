### Requirement: INV and FRDRIVE control buttons
The camp dashboard SHALL expose two toggle buttons — INV and FRDRIVE — that write their state to memcache keys `INV_BTN` and `FRDRIVE_BTN` respectively via the `obd_save` endpoint. The FRIDGE button does not exist.

FRDRIVE is **active by default**: the button SHALL render as active (✓) on page load, and `FRDRIVE_BTN` SHALL be treated as ON when absent from memcache.

#### Scenario: User toggles FRDRIVE button OFF
- **WHEN** user clicks FRDRIVE button and state transitions to OFF
- **THEN** `obd_save` SHALL set `FRDRIVE_BTN` to 0 in memcache (explicit off, not delete)

#### Scenario: User toggles FRDRIVE button ON
- **WHEN** user clicks FRDRIVE button and state transitions to ON
- **THEN** `obd_save` SHALL set `FRDRIVE_BTN` to 1 in memcache

#### Scenario: RPi restart — FRDRIVE default
- **WHEN** the RPi restarts and `FRDRIVE_BTN` key is absent from memcache
- **THEN** `logfridge.py` SHALL treat FRDRIVE as active (ON)

### Requirement: Grid Power and Inverter Output indicators
The camp dashboard SHALL display two read-only AC source indicators — GRID and INV_OUT — reflecting `camp/shore_power` and `camp/inverter_output` memcache values written by logfridge.py. Indicators SHALL be updated on each `obd_stat` poll (every 5 seconds).

#### Scenario: Shore power present
- **WHEN** `camp/shore_power` is 1 in memcache
- **THEN** the GRID indicator SHALL display as active (highlighted)

#### Scenario: No AC sources
- **WHEN** both `camp/shore_power` and `camp/inverter_output` are 0 or absent
- **THEN** both indicators SHALL display as inactive
