## MODIFIED Requirements

### Requirement: Fridge power decision logic
The system SHALL evaluate fridge power state every 10 seconds using the following logic: `fridge_on = power_present AND (shore_power OR (inv_btn AND NOT frdrive_btn) OR (inv_btn AND frdrive_btn AND rpm > 0 AND soc_ok))` where `power_present = shore_power OR inverter_output` and `soc_ok = True when BMS data is unavailable or stale, otherwise soc > SOC_DRIVE_THRESHOLD`. The FRIDGE button does not exist; INV_BTN governs whether the inverter is on and therefore whether fridge can be powered from it.

#### Scenario: Shore power connected
- **WHEN** shore power sensor reports ON
- **THEN** fridge switch SHALL be set to ON (fridge powered) regardless of button states or RPM

#### Scenario: Inverter on, drive mode off
- **WHEN** INV_BTN is 1 and FRDRIVE_BTN is 0 (explicitly off)
- **AND** power_present is true
- **THEN** fridge switch SHALL be set to ON (no RPM or SOC condition)

#### Scenario: Inverter on, drive mode on, engine running, SOC sufficient
- **WHEN** INV_BTN is 1 and FRDRIVE_BTN is active (1 or default)
- **AND** RPM is greater than 0
- **AND** soc_ok is True (BMS unavailable, stale, or SOC > SOC_DRIVE_THRESHOLD)
- **AND** power_present is true
- **THEN** fridge switch SHALL be set to ON

#### Scenario: Inverter on, drive mode on, engine running, SOC too low
- **WHEN** INV_BTN is 1 and FRDRIVE_BTN is active
- **AND** RPM is greater than 0
- **AND** BMS data is fresh and SOC ≤ SOC_DRIVE_THRESHOLD
- **THEN** fridge switch SHALL be set to OFF

#### Scenario: Inverter on, drive mode on, engine not running
- **WHEN** INV_BTN is 1 and FRDRIVE_BTN is active
- **AND** RPM is 0 or unknown
- **THEN** fridge switch SHALL be set to OFF

#### Scenario: No AC power source available
- **WHEN** shore_power is OFF and inverter_output is OFF
- **THEN** fridge switch SHALL be set to OFF regardless of button states

#### Scenario: Inverter off, no shore power
- **WHEN** INV_BTN is 0 and shore_power is OFF
- **THEN** fridge switch SHALL be set to OFF

## ADDED Requirements

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
