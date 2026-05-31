## 1. ESPHome Config — Entity Rename

- [x] 1.1 Rename `camper_spare_1_relay` entity ID to `camper_ng_bond_relay` and name to `"Camper N-G Bond"` in `config/camper.yaml`.

## 2. ESPHome Config — Automation Infrastructure

- [x] 2.1 Add `globals` block with a `bool` variable (restore from flash) to back the automation-enable switch state.
- [x] 2.2 Add `template switch` entity `camper_ng_bond_auto` (`"Camper N-G Bond Auto"`) reading and writing the global.
- [x] 2.3 Add automation triggered by `camper_shore_power`, `camper_inverter_output`, and `camper_ng_bond_auto` state changes: when auto=ON and shore=OFF and inv_out=ON → close bond relay; otherwise → open bond relay.
- [x] 2.4 Add automation triggered when `camper_ng_bond_auto` turns OFF: force `camper_ng_bond_relay` open immediately.
- [x] 2.5 Add `on_turn_on` safety interlock to `camper_ng_bond_relay`: if shore power is present at the moment of turn-on, immediately turn the relay back off.

## 3. Validation

- [x] 3.1 Compile ESPHome config and confirm no errors.
- [x] 3.2 Flash device; verify `Camper N-G Bond` and `Camper N-G Bond Auto` entities appear in Home Assistant.
- [x] 3.3 With auto=ON: disconnect shore power and confirm N-G bond relay closes; reconnect shore and confirm it opens.
- [x] 3.4 With auto=ON: turn auto OFF and confirm relay opens immediately; manually close relay via HA and confirm it stays closed.

## 4. Documentation Updates

- [x] 4.1 Update `drivers/camper/README.md` relay roles table: spare relay 1 → N-G bond relay.
- [x] 4.2 Add full AC topology diagram to `drivers/camper/README.md` covering shore power, inverter, phase sensors, fridge relay, inverter relay, and N-G bond relay.
- [x] 4.3 Update `drivers/camper/inverter-floating-phase.md` to reference the implemented solution.
- [x] 4.4 Note HA migration step: update any dashboard cards or automations referencing old `Camper Spare Relay 1` entity.
