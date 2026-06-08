### Requirement: Fridge power decision logic
The system SHALL evaluate fridge relay command and effective fridge power every 10 seconds using the following logic: `relay_on = (NOT power_present) OR shore_power OR (inv_btn AND NOT frdrive_btn) OR (inv_btn AND frdrive_btn AND rpm > 0 AND soc_ok)` and `fridge_powered = power_present AND relay_on`, where `power_present = shore_power OR inverter_output` and `soc_ok = True when BMS data is unavailable or stale, otherwise soc > SOC_DRIVE_THRESHOLD`. The FRIDGE button does not exist; INV_BTN governs whether the inverter is on and therefore whether fridge can be powered from it.

#### Scenario: Shore power connected
- **WHEN** shore power sensor reports ON
- **THEN** fridge switch SHALL be set to ON and effective fridge power SHALL be ON regardless of button states or RPM

#### Scenario: Inverter on, drive mode off
- **WHEN** INV_BTN is 1 and FRDRIVE_BTN is 0 (explicitly off)
- **AND** power_present is true
- **THEN** fridge switch SHALL be set to ON and effective fridge power SHALL be ON (no RPM or SOC condition)

#### Scenario: Inverter on, drive mode on, engine running, SOC sufficient
- **WHEN** INV_BTN is 1 and FRDRIVE_BTN is active (1 or default)
- **AND** RPM is greater than 0
- **AND** soc_ok is True (BMS unavailable, stale, or SOC > SOC_DRIVE_THRESHOLD)
- **AND** power_present is true
- **THEN** fridge switch SHALL be set to ON and effective fridge power SHALL be ON

#### Scenario: Inverter on, drive mode on, engine running, SOC too low
- **WHEN** INV_BTN is 1 and FRDRIVE_BTN is active
- **AND** RPM is greater than 0
- **AND** BMS data is fresh and SOC ≤ SOC_DRIVE_THRESHOLD
- **AND** power_present is true
- **THEN** fridge switch SHALL be set to OFF and effective fridge power SHALL be OFF

#### Scenario: Inverter on, drive mode on, engine not running
- **WHEN** INV_BTN is 1 and FRDRIVE_BTN is active
- **AND** RPM is 0 or unknown
- **AND** power_present is true
- **THEN** fridge switch SHALL be set to OFF and effective fridge power SHALL be OFF

#### Scenario: No AC power source available
- **WHEN** shore_power is OFF and inverter_output is OFF
- **THEN** fridge switch SHALL be set to ON (relay de-energised) regardless of button states
- **AND** effective fridge power SHALL be OFF

#### Scenario: Inverter off, no shore power
- **WHEN** INV_BTN is 0 and shore_power is OFF
- **AND** inverter_output is ON
- **THEN** fridge switch SHALL be set to OFF and effective fridge power SHALL be OFF

### Requirement: Inverter desired-state synchronisation
The system SHALL synchronise the ESP inverter switch to match the INV_BTN memcache value on each poll cycle, calling turn_on or turn_off only when the desired state differs from the current switch state.

#### Scenario: INV_BTN set to 1, inverter currently off
- **WHEN** INV_BTN is 1 in memcache
- **AND** camper_inverter switch reports OFF
- **THEN** the system SHALL call turn_on on the inverter switch

#### Scenario: INV_BTN and inverter already in sync
- **WHEN** INV_BTN matches current inverter switch state
- **THEN** the system SHALL NOT issue any switch command to the ESP

### Requirement: Sensor state mirrored to memcache
The system SHALL write shore power and inverter output sensor states to memcache keys `camp/shore_power` and `camp/inverter_output` on each poll cycle so the web2py dashboard can display AC source status without direct ESP access.

#### Scenario: Shore power sensor ON
- **WHEN** camper_shore_power binary sensor reports ON
- **THEN** `camp/shore_power` SHALL be set to 1 in memcache

#### Scenario: ESP unreachable
- **WHEN** the ESP REST API does not respond within the timeout
- **THEN** the poll cycle SHALL be skipped with a warning log and memcache state SHALL NOT be updated

### Requirement: Relay command and effective fridge power mirrored to memcache
The system SHALL write actual fridge and inverter relay states to memcache keys `camp/fridge_relay` and `camp/inverter_relay` after each sync cycle. The system SHALL also write effective fridge power state to memcache key `camp/fridge_powered`.

#### Scenario: Fridge relay state written after sync
- **WHEN** the fridge logic evaluation completes
- **THEN** `camp/fridge_relay` SHALL reflect the resulting fridge switch state (1 = on, 0 = off)

#### Scenario: Effective fridge power written after sync
- **WHEN** the fridge logic evaluation completes
- **THEN** `camp/fridge_powered` SHALL reflect effective fridge power (1 = powered, 0 = not powered)

#### Scenario: No AC source keeps relay on but fridge unpowered
- **WHEN** shore_power is OFF and inverter_output is OFF
- **THEN** `camp/fridge_relay` SHALL be 1
- **AND** `camp/fridge_powered` SHALL be 0

### Requirement: BMS SOC guard configuration
The system SHALL read SOC threshold and BMS staleness window from environment variables `SOC_DRIVE_THRESHOLD` (default `50.0`, percent float) and `BMS_MAX_AGE_SECS` (default `300`, seconds int), following the same module-level constant pattern as `_POLL_INTERVAL` and `_ESP_BASE`.

#### Scenario: Default threshold used when env var absent
- **WHEN** `SOC_DRIVE_THRESHOLD` environment variable is not set
- **THEN** the system SHALL use 50.0% as the SOC guard threshold

#### Scenario: Custom threshold applied
- **WHEN** `SOC_DRIVE_THRESHOLD` is set to `"60.0"`
- **THEN** the system SHALL only power the fridge in drive mode when SOC > 60.0%

### Requirement: BMS data freshness check
The system SHALL treat BMS data as unavailable (soc_ok = True, permissive) when `BMSTime` memcache key is absent or when the timestamp it contains is older than `_BMS_MAX_AGE_SECS` seconds relative to the current UTC time.

#### Scenario: BMSTime key absent
- **WHEN** `BMSTime` is not present in memcache
- **THEN** soc_ok SHALL be True (guard bypassed)

#### Scenario: BMSTime stale
- **WHEN** `BMSTime` is present but older than `_BMS_MAX_AGE_SECS` seconds
- **THEN** soc_ok SHALL be True (guard bypassed)

#### Scenario: BMSTime fresh, SOC above threshold
- **WHEN** `BMSTime` is within `_BMS_MAX_AGE_SECS` seconds
- **AND** decoded SOC from `BMSData` is greater than `_SOC_DRIVE_THRESHOLD`
- **THEN** soc_ok SHALL be True

#### Scenario: BMSTime fresh, SOC at or below threshold
- **WHEN** `BMSTime` is within `_BMS_MAX_AGE_SECS` seconds
- **AND** decoded SOC from `BMSData` is less than or equal to `_SOC_DRIVE_THRESHOLD`
- **THEN** soc_ok SHALL be False
