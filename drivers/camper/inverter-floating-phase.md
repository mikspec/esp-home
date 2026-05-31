# Inverter Floating Phase

When the camper inverter generates AC phase in standalone mode (no shore power), the neutral conductor has no fixed reference to protective earth. This causes a floating neutral, which produces noise on touch screens and equipment with EMI filters.

## Solution

Spare relay 1 (GPIO33) is repurposed as a neutral-ground bond relay (`Camper N-G Bond`). When the inverter is generating and no shore power is present, the relay is closed to bond neutral to PE, stabilising the reference.

The bonding is managed automatically by the `Camper N-G Bond Auto` switch. See [README.md](README.md) for the full AC topology diagram, N-G bond automation logic, and commissioning steps.

## Safety logic

The N-G bond relay **must never be closed while shore power is present**. Closing the bond when the grid is connected would create a N-PE short through the shore supply — a dangerous condition regardless of whether the automation is enabled.

This is enforced in firmware via an `on_turn_on` interlock on `camper_ng_bond_relay`: any turn-on command (automated or manual) is checked against `camper_shore_power.state` and immediately reversed if shore power is active. The `on_state` trigger on the shore power sensor also calls `ng_bond_evaluate`, which opens the relay if shore turns on while it is closed.

### Inverter stabilisation delay

When inverter output becomes active, the `ng_bond_evaluate` script schedules a dedicated `ng_bond_delayed_close` script rather than closing the relay immediately. `ng_bond_delayed_close` waits **5 seconds** and then re-checks all conditions before closing the relay. This gives the inverter AC output time to stabilise before the neutral-PE bond is applied.

The delayed close is cancelled (via `script.stop`) in all forced-open paths:
- `ng_bond_evaluate` `else` branch (conditions no longer met)
- `camper_ng_bond_auto` turn-off action (automation disabled)
- `inverter_safe_shutdown` (inverter is being shut down)