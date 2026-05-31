## Context

The camper driver exposes four relay channels on an ESP32 relay X4_V1.1 board. Spare relay 1 (GPIO33) is currently a generic unassigned channel. The phase sensors implemented in the prior change (GPIO21 shore power, GPIO22 inverter output) provide the exact binary signals needed to detect when the inverter is generating standalone AC power — the condition under which neutral floats relative to protective earth.

The N-G bond must be present when the inverter is generating and absent when shore power is supplying the load (grid has its own N-G bond at the distribution transformer; two simultaneous bonds create a ground loop).

## Goals / Non-Goals

**Goals:**
- Repurpose spare relay 1 (GPIO33) as a dedicated N-G bond relay (`camper_ng_bond_relay`).
- Add a persistent virtual switch (`camper_ng_bond_auto`) to enable/disable the automation.
- Implement ESPHome-local automation: close bond when shore=OFF and inverter output=ON; open in all other conditions (fail-open).
- When automation is disabled, force relay open once then allow full manual HA control.
- Update driver documentation with the complete AC topology diagram.

**Non-Goals:**
- Load management, fridge scheduling, or any other automation logic — deferred to future changes.
- Voltage or frequency monitoring on the AC bus.
- Modifying relay 2 (GPIO32, spare relay 2) — remains unassigned.

## Decisions

1. ESPHome-local automation over Home Assistant automation
   - Decision: All N-G bond logic runs in ESPHome firmware, not HA.
   - Rationale: Bond state is safety-relevant. HA unavailability (restart, network loss) must not prevent correct bonding during a power source transition. ESPHome-local automation is independent of HA connectivity.
   - Alternative considered: HA automation referencing sensor entities. Rejected due to HA availability dependency.

2. `globals` + `template switch` for the automation-enable switch
   - Decision: Use a `globals` bool with `restore_value: true` as the backing store for `camper_ng_bond_auto`, exposed as a `template switch`.
   - Rationale: Persists across ESP32 reboots without needing a physical pin. Survives power loss. ESPHome `globals` with flash restore is the standard pattern for persistent virtual state.
   - Alternative considered: `input_boolean` helper in HA. Rejected — same HA dependency problem as decision 1.

3. Fail-open on all non-generating conditions
   - Decision: Relay is OPEN unless the exact condition (shore=OFF, inv_out=ON, auto=ON) is met.
   - Rationale: An unexpected closed bond during grid pass-through creates a ground loop. Fail-open is the electrically safer default.
   - Alternative considered: Fail-closed (bond always present). Rejected — creates ground loop when grid is connected.

4. Force-open on automation disable, then yield to manual control
   - Decision: When `ng_bond_auto` transitions to OFF, the automation fires once to open the relay, then stops interfering. User can manually close via HA if needed.
   - Rationale: Disabling the automation shouldn't silently leave the bond closed from a prior auto-close. Clean state on disable. Manual override remains possible for specific use cases.
   - Alternative considered: Leave relay in current state when disabling. Rejected — could leave bond closed when grid is connected.

## Risks / Trade-offs

- [N-G bond relay carries fault current in a line-neutral fault scenario] → Mitigation: Relay X4_V1.1 board relay is rated for AC loads; verify fault current does not exceed relay contact rating for the installation.
- [Transition gap: brief moment when both sensors are OFF during source switchover] → Mitigation: Fail-open handles this safely — relay opens during transition and closes again once inverter output is confirmed present.
- [ESP32 reboot while inverter is generating] → Mitigation: `ALWAYS_OFF` restore mode means relay starts open; automation re-evaluates on boot and closes bond if conditions are met. Short gap accepted and documented.
- [Breaking HA entity name for spare relay 1] → Mitigation: Document migration step — update any HA dashboard cards or automations referencing `Camper Spare Relay 1`.

## Migration Plan

1. Update `config/camper.yaml`: rename entity, add globals, template switch, and automation blocks.
2. Flash updated firmware to the camper ESP32 node.
3. In Home Assistant: remove or update any dashboard cards/automations referencing `Camper Spare Relay 1`.
4. Verify N-G bond behavior: enable automation, disconnect shore power, confirm relay closes; reconnect shore power, confirm relay opens.
5. Rollback: flash previous firmware — relay reverts to generic spare, floating neutral returns.

## Open Questions

- None. All design decisions resolved during exploration.
