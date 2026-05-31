## Context

The camper ESP32 drives the inverter via a GPIO relay (`camper_inverter_relay`, GPIO25). When the N-G bond relay is closed (bonding neutral to PE while the inverter generates off-grid), turning the inverter off abruptly while the bond is still closed can produce transients on the AC bus. The safe sequence is: open the N-G bond first (while inverter AC is still stable), wait 5 seconds, then cut the inverter.

ESPHome GPIO switches do not provide a `before_turn_off` hook — `on_turn_off` fires *after* the GPIO has already changed. This prevents sequencing any actions prior to the relay opening.

## Goals / Non-Goals

**Goals:**
- Enforce N-G bond → 5 s delay → inverter GPIO shutdown sequence on every inverter turn-off command.
- Keep the external entity name and REST endpoint unchanged so Home Assistant, logfridge.py, and any other callers require no modification.
- Keep boot behaviour unchanged: inverter off on startup.

**Non-Goals:**
- No change to the N-G bond auto switch or `ng_bond_evaluate` script.
- No timeout or abort path if the script is interrupted mid-sequence.
- No change to fridge relay, spare relay, or phase sensor behaviour.

## Decisions

### Template switch wrapper over direct GPIO switch

**Decision**: Rename the GPIO switch to `camper_inverter_relay_gpio` (`internal: true`) and introduce a template switch `camper_inverter_relay` as the user-facing entity.

**Rationale**: A template switch's `turn_off_action` fires *before* the entity changes state and can execute an arbitrary script, including delays. This is the only ESPHome pattern that allows pre-turn-off sequencing without custom components.

**Alternatives considered**:
- `on_turn_off` on the GPIO switch: fires after relay opens — too late to sequence N-G bond first.
- Separate "safe shutdown" button entity: would require all callers to use a different entity for shutdown — breaks existing HA automations and the logfridge script.

### N-G bond turned off before inverter cuts

**Decision**: The shutdown script checks whether `camper_ng_bond_relay` is ON and opens it before the 5-second delay, regardless of `camper_ng_bond_auto` state.

**Rationale**: The bond relay could be ON via manual control (auto OFF) or via automation. The shutdown sequence must be safe in both cases. After the inverter cuts, `ng_bond_evaluate` will run via the `camper_inverter_output` phase sensor's `on_state` trigger and confirm the bond relay is off.

### `on_boot` targets internal GPIO switch directly

**Decision**: The `on_boot` block references `camper_inverter_relay_gpio` (the internal switch) rather than the template switch.

**Rationale**: Routing the boot turn-off through the template switch would trigger the shutdown script on every reboot, unnecessarily waiting 5 seconds and potentially interfering with the bond relay state during initialisation.

## Risks / Trade-offs

- **Script non-interruptible**: Once `inverter_safe_shutdown` starts, ESPHome will run it to completion. A rapid on/off/on sequence could leave the system in an unexpected state. → Acceptable for this use case; the 5-second window is short.
- **Template switch state**: Template switch state is `optimistic: true` — it reflects the commanded state, not a GPIO readback. On reboot, the template switch restores to `ALWAYS_OFF`, consistent with the internal GPIO. → No divergence risk.
- **`ng_bond_evaluate` unaffected**: The evaluate script calls `switch.turn_off: camper_ng_bond_relay` directly and `switch.turn_on: camper_ng_bond_relay` directly — it never touches the inverter switch. No interaction. ✓

## Migration Plan

1. Flash updated `config/camper.yaml` to the ESP32.
2. Home Assistant will see the existing `Camper Inverter` entity update in-place (template switch retains the same `name:`). No HA dashboard or automation changes required.
3. `logfridge.py` calls `POST /switch/camper_inverter_relay/turn_off` — endpoint unchanged. ✓
4. Verify in HA that turning off `Camper Inverter` while the N-G bond is ON produces the expected sequence (bond opens, 5 s pause, inverter relay off).

## Open Questions

None.
