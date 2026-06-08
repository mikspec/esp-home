## Context

The RPi polling script (`logfridge.py`) currently computes a single fridge switch decision and mirrors it as `camp/fridge_relay`. The web2py dashboard consumes that value in `obd_stat` and uses it for FRDRIVE coloring.

With the new behavior, the no-AC state must keep the relay de-energized (coil-saving, NC path closed) while effective fridge power remains OFF because no upstream AC source exists. This requires a state model split and API/UI contract update.

## Goals / Non-Goals

**Goals:**
- Represent relay command state and effective fridge power as separate runtime values
- Preserve current inverter sync behavior and existing relay key compatibility
- Prevent dashboard false-positive "powered" display in no-AC conditions
- Keep rollout safe across mixed versions

**Non-Goals:**
- ESP firmware or wiring changes
- New dashboard widgets beyond current FRDRIVE color behavior
- Changing FRDRIVE toggle semantics

## Decisions

### D1: Two-value state model

Compute and carry both values in each poll cycle:

- `relay_on = (NOT power_present) OR shore_power OR (inv_btn AND NOT frdrive_btn) OR (inv_btn AND frdrive_btn AND rpm > 0 AND soc_ok)`
- `fridge_powered = power_present AND relay_on`
- `power_present = shore_power OR inverter_output`

This explicitly encodes no-AC behavior while preserving existing policy branches.

Scenario-level behavior carried from spec:
- No AC source (`shore_power = 0` and `inverter_output = 0`): `relay_on = 1`, `fridge_powered = 0`
- Inverter off, no shore power, inverter output present (`INV_BTN = 0`, `shore_power = 0`, `inverter_output = 1`): `relay_on = 0`, `fridge_powered = 0`

### D2: Memcache contract

Keep existing key:
- `camp/fridge_relay`

Add key:
- `camp/fridge_powered`

Rationale: preserves compatibility for old readers while enabling correct semantics for new readers.

### D3: API compatibility strategy

`obd_stat` returns both:
- `fridge_relay`
- `fridge_powered`

Both default to `0` when key is absent. This avoids undefined states during startup or staggered deployment.

### D4: UI semantics source of truth

FRDRIVE color uses `fridge_powered` only:
- green when `fridge_powered == 1`
- orange when `frdrivebtn == 1 && fridge_powered == 0`
- gray when `frdrivebtn == 0 && fridge_powered == 0`

Button label remains tied to drive-mode toggle state.

This matches dashboard spec semantics where FRDRIVE color reflects effective fridge power, not relay command state.

### D5: Rollout order

Deploy in this order:
1. `logfridge.py` writes `camp/fridge_powered`
2. `default.py` exposes `fridge_powered`
3. Dashboard consumes `fridge_powered`

Rationale: each step remains safe if the next step is not yet deployed.

## Risks / Trade-offs

- **Mixed-version display mismatch**: old UI still tied to `fridge_relay` may show green in no-AC state. Mitigation: sequence rollout and prioritize UI update after API deployment.
- **Startup key absence**: first polls may not have both memcache keys. Mitigation: explicit default `0` in `obd_stat`.
- **Operator interpretation drift**: historical docs equate relay ON with powered. Mitigation: update docs in the same change.

## Verification Plan

- No AC (`shore=0`, `inv_out=0`): expect `fridge_relay=1`, `fridge_powered=0`, non-green FRDRIVE
- Shore present: expect `1/1`, green
- Inverter output + INV on + FRDRIVE off: expect `1/1`, green
- Drive mode on + RPM 0: expect `0/0`, non-green
- Drive mode on + RPM>0 + SOC low: expect `0/0`, non-green
- Drive mode on + RPM>0 + SOC high: expect `1/1`, green
- Missing keys on startup: `obd_stat` returns `fridge_relay=0`, `fridge_powered=0`
