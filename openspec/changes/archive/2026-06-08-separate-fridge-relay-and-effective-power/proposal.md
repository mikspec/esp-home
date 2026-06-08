## Why

The fridge control currently uses relay command state as a proxy for whether the fridge is powered. This breaks in the no-AC case: we want the relay de-energized to avoid wasting coil power, but the fridge is still effectively unpowered when both shore and inverter AC are absent.

Without separating these states, the dashboard can show a misleading green state and operators cannot distinguish relay-path status from actual available fridge power.

## What Changes

- Split fridge decision output into two values in `logfridge.py`:
  - relay command state (`camp/fridge_relay`)
  - effective fridge power state (`camp/fridge_powered`)
- Use the exact decision model from spec:
  - `relay_on = (NOT power_present) OR shore_power OR (inv_btn AND NOT frdrive_btn) OR (inv_btn AND frdrive_btn AND rpm > 0 AND soc_ok)`
  - `fridge_powered = power_present AND relay_on`
  - `power_present = shore_power OR inverter_output`
- Apply explicit no-AC behavior:
  - if `shore_power == 0` and `inverter_output == 0`, set relay ON (de-energized NC path) and effective power OFF
- Preserve the explicit scenario: if `INV_BTN == 0`, `shore_power == 0`, and `inverter_output == 1`, set relay OFF and effective power OFF
- Extend `obd_stat` response with `fridge_powered` while keeping `fridge_relay` for compatibility
- Update dashboard FRDRIVE color semantics to use `fridge_powered` instead of `fridge_relay`
- Align driver docs to clarify that relay ON does not always imply effective fridge power

## Capabilities

### New Capabilities
<!-- none -->

### Modified Capabilities

- `camper-fridge-power-management`: Control loop now derives relay-path state and effective power independently
- `camper-dashboard`: FRDRIVE tri-state color is now driven by effective power
- `camper-relay-control`: Wording clarified to separate relay-path closure from AC source availability

## Impact

- `web2pyp3/applications/camp/private/logfridge.py` — control logic split, memcache write extension
- `web2pyp3/applications/camp/controllers/default.py` — `obd_stat` payload extension
- `web2pyp3/applications/camp/views/default/index.html` — FRDRIVE color logic input switch
- `drivers/camper/fridge-power-managment.md` and `drivers/camper/README.md` — semantic clarification
- No ESP firmware changes required
- No new dependencies required
