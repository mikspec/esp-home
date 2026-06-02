## MODIFIED Requirements

### Requirement: INV and FRDRIVE control buttons
The camp dashboard SHALL expose two toggle buttons — INV and FRDRIVE — that write their state to memcache keys `INV_BTN` and `FRDRIVE_BTN` respectively via the `obd_save` endpoint. The FRIDGE button does not exist.

FRDRIVE is **active by default**: the button SHALL render as active (✓) on page load, and `FRDRIVE_BTN` SHALL be treated as ON when absent from memcache.

The FRDRIVE button SHALL use tri-state background colour to reflect both drive-mode state and fridge relay status:
- **Green** (`#00aa00`) when `fridge_relay` is 1 (fridge powered), regardless of drive-mode state.
- **Orange** (`#ff9600`) when `frdrivebtn` is 1 and `fridge_relay` is 0 (drive mode on but fridge not powered).
- **Gray** (`#555`) when `frdrivebtn` is 0 and `fridge_relay` is 0.

The button label (✓/✗) SHALL continue to reflect the drive-mode toggle state independently of colour.

#### Scenario: User toggles FRDRIVE button OFF
- **WHEN** user clicks FRDRIVE button and state transitions to OFF
- **THEN** `obd_save` SHALL set `FRDRIVE_BTN` to 0 in memcache (explicit off, not delete)

#### Scenario: User toggles FRDRIVE button ON
- **WHEN** user clicks FRDRIVE button and state transitions to ON
- **THEN** `obd_save` SHALL set `FRDRIVE_BTN` to 1 in memcache

#### Scenario: RPi restart — FRDRIVE default
- **WHEN** the RPi restarts and `FRDRIVE_BTN` key is absent from memcache
- **THEN** `logfridge.py` SHALL treat FRDRIVE as active (ON)

#### Scenario: Fridge is powered — FRDRIVE button green
- **WHEN** `obd_stat` returns `fridge_relay: 1`
- **THEN** FRDRIVE button SHALL display with green background regardless of `frdrivebtn` value

#### Scenario: Drive mode on, fridge not powered — FRDRIVE button orange
- **WHEN** `obd_stat` returns `frdrivebtn: 1` and `fridge_relay: 0`
- **THEN** FRDRIVE button SHALL display with orange background

#### Scenario: Drive mode off, fridge not powered — FRDRIVE button gray
- **WHEN** `obd_stat` returns `frdrivebtn: 0` and `fridge_relay: 0`
- **THEN** FRDRIVE button SHALL display with gray background

## ADDED Requirements

### Requirement: Fridge relay status in obd_stat response
The `obd_stat` endpoint SHALL include a `fridge_relay` field (integer 1 or 0) in its JSON response, read from the `camp/fridge_relay` memcache key written by `logfridge.py`. When the key is absent, the value SHALL default to 0.

#### Scenario: Fridge relay state available in memcache
- **WHEN** `camp/fridge_relay` is set to 1 in memcache
- **THEN** `obd_stat` response SHALL include `"fridge_relay": 1`

#### Scenario: Fridge relay key absent from memcache
- **WHEN** `camp/fridge_relay` is not present in memcache (e.g. before first logfridge.py cycle)
- **THEN** `obd_stat` response SHALL include `"fridge_relay": 0`
