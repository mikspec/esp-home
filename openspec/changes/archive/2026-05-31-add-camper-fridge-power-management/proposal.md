## Why

The camper has an ESP32 relay controller managing the fridge and inverter, and a Raspberry Pi Zero W running the camp management system (web2py + memcache). These two systems currently operate independently. The fridge should be powered intelligently based on driving state (RPM), AC power source, and user intent — saving battery when the engine is off and the camper is off-grid. A new Python script bridges the two systems, and the web2py dashboard gets two new control buttons and two power-source indicators.

## What Changes

### ESP32 firmware (`config/camper.yaml`)
- Invert the fridge relay GPIO pin (`inverted: true`) so that the ESPHome switch entity matches user intent: switch ON = fridge powered (relay de-energised = NC closed).
- Change fridge relay `restore_mode` from `ALWAYS_OFF` to `ALWAYS_ON` so the fridge is powered immediately on ESP boot (script re-evaluates within 10 s).

### New RPi script (`logfridge.py`)
- Poll loop running every 10 seconds.
- Reads from memcache: `RPM`, `FRDRIVE_BTN`, `INV_BTN`.
- Reads from ESP REST API (port 80): `camper_shore_power`, `camper_inverter_output` binary sensor states.
- Writes to memcache: `camp/shore_power`, `camp/inverter_output`, `camp/fridge_relay`, `camp/inverter_relay` (for dashboard display).
- Applies inverter desired-state: syncs `camper_inverter_relay` to `INV_BTN`.
- Applies fridge logic (see below) and syncs `camper_fridge_relay` accordingly.

#### Fridge logic

```
power_present = shore_power OR inverter_output

fridge_on = power_present AND (
    shore_power                                      # grid connected — always power fridge
    OR (INV_BTN AND NOT FRDRIVE_BTN)                 # inverter on, no drive mode — always power fridge
    OR (INV_BTN AND FRDRIVE_BTN AND RPM > 0)         # inverter on, drive mode — engine running
)
```

If `power_present` is false there is no AC source to run the fridge regardless of button state; the fridge switch is left OFF (safe idle). Shore power always takes precedence over drive-mode logic.

### web2py camp dashboard (`views/default/index.html`, `controllers/default.py`)
- Add two display flags: **Grid Power** (`camp/shore_power`) and **Inverter Output** (`camp/inverter_output`), read from memcache.
- Add two toggle buttons:
  - **[INV]** — sets/clears `INV_BTN` in memcache (desired inverter state).
  - **[FRDRIVE]** — sets/clears `FRDRIVE_BTN` in memcache (drive mode; active by default on RPi restart).
- Button state is not persisted; clears on RPi restart (by design).

## Capabilities

### New Capabilities
- `camper-fridge-power-management`: Context-aware fridge power control driven by AC source state, engine RPM, and user intent flags. Implemented as a polling bridge script on the RPi.

### Modified Capabilities
- `camper-relay-control`: Fridge relay GPIO inverted for intuitive switch semantics; restore mode changed to `ALWAYS_ON`.
- `camper-dashboard`: Two new control buttons (INV, FRDRIVE) and two AC source indicator flags added to the main index view.

## Impact

- Affected code:
  - `config/camper.yaml`: `inverted: true` and `restore_mode: ALWAYS_ON` on `camper_fridge_relay`.
  - `web2pyp3/applications/camp/private/logfridge.py`: New script (created).
  - `web2pyp3/applications/camp/controllers/default.py`: New memcache reads and button toggle endpoints.
  - `web2pyp3/applications/camp/views/default/index.html`: New buttons and indicator flags.
- APIs/systems:
  - ESP32 REST API (port 80) polled by `logfridge.py` for sensor states and relay control.
  - Memcache used as the state bus between logfridge, web2py controller, and web2py view.
- Dependencies:
  - `requests` (Python) — standard HTTP calls to ESP REST API; likely already available.
  - No new ESPHome secrets or GPIO changes beyond the fridge relay pin inversion.

## Non-Goals

- No Home Assistant automations — all logic lives in `logfridge.py`.
- No persistent button state — memcache only, clears on restart.
- No authentication on ESP web server — acceptable on the isolated Android-phone LAN.
- No changes to inverter safety interlocks or N-G bond logic.
