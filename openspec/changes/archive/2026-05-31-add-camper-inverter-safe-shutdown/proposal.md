## Why

When the inverter is shut down while the N-G bond relay is closed, cutting power to the inverter while the neutral-PE bond is still present can cause transients on the AC bus. The safe sequence is to open the N-G bond first — while the inverter is still producing stable AC — wait 5 seconds, then cut the inverter. Currently ESPHome provides no `before_turn_off` hook, so this sequencing cannot be enforced on the GPIO switch directly. A template switch wrapper is introduced to intercept the turn-off command and enforce the shutdown sequence.

## What Changes

### `config/camper.yaml`

- Rename the existing `camper_inverter_relay` GPIO switch to `camper_inverter_relay_gpio` and mark it `internal: true` (not exposed to Home Assistant or REST API).
- Add a new **template switch** `camper_inverter_relay` named `"Camper Inverter"` (same external name as today):
  - `turn_on_action`: directly turns on `camper_inverter_relay_gpio`.
  - `turn_off_action`: executes the new `inverter_safe_shutdown` script.
  - `restore_mode: ALWAYS_OFF`.
- Add a new script `inverter_safe_shutdown`:
  ```
  1. if camper_ng_bond_relay is ON → turn it OFF
  2. delay 5 s
  3. turn off camper_inverter_relay_gpio
  ```
- Update `on_boot` to target `camper_inverter_relay_gpio` directly (bypassing the template switch and its shutdown script during boot).
- No changes to `ng_bond_evaluate` — it already targets `camper_ng_bond_relay` directly and is unaffected.

### Behaviour from external callers

Any caller — Home Assistant, the ESP REST API (`POST /switch/camper_inverter_relay/turn_off`), or `logfridge.py` — issues a normal turn-off against `camper_inverter_relay`. The sequencing is fully transparent to the caller; the ESP enforces it internally.

```
Caller: turn_off "Camper Inverter"
             │
             ▼
  template switch turn_off_action
             │
             ▼
  inverter_safe_shutdown script
    ┌─ N-G bond ON? ─── yes ──► turn off camper_ng_bond_relay
    │                                      │
    │  no                               wait 5 s
    │                                      │
    └──────────────────────────────────────┘
             │
             ▼
      turn off camper_inverter_relay_gpio
```

## Capabilities

### Modified Capabilities
- `camper-relay-control`: Inverter switch turn-off now enforces a safe N-G bond → 5 s → inverter shutdown sequence. External interface (`Camper Inverter` entity name, REST endpoint) is unchanged.

## Impact

- Affected code:
  - `config/camper.yaml`: GPIO switch renamed and made internal; template switch added; `inverter_safe_shutdown` script added; `on_boot` updated to reference GPIO switch directly.
  - `drivers/camper/README.md`: Document the safe shutdown sequence and template switch architecture.
- APIs/systems:
  - Home Assistant and REST API entity name `Camper Inverter` is preserved — no dashboard or automation changes required.
  - `logfridge.py` (from `add-camper-fridge-power-management`) calls `turn_off` on `camper_inverter_relay` — unaffected, sequencing is transparent.
- Dependencies: None.

## Non-Goals

- No change to the N-G bond auto switch logic or `ng_bond_evaluate` script.
- No change to the fridge relay or other relays.
- No timeout or cancellation if the script is interrupted mid-sequence (acceptable for this use case).
