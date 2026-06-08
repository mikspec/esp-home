## 1. logfridge.py — split relay command from effective power

- [x] 1.1 In poll logic, compute relay_on and fridge_powered separately using current inputs (shore_power, inverter_output, INV_BTN, FRDRIVE_BTN, RPM, soc_ok)
- [x] 1.2 Apply no-AC rule: when shore_power == 0 and inverter_output == 0, set relay_on = 1 and fridge_powered = 0
- [x] 1.3 Keep inverter desired-state synchronisation unchanged
- [x] 1.4 Update debug logging to emit both relay_on and fridge_powered for each cycle

## 2. logfridge.py — memcache write contract

- [x] 2.1 Continue writing camp/fridge_relay and camp/inverter_relay as before
- [x] 2.2 Add camp/fridge_powered memcache key write on every successful sync cycle
- [x] 2.3 Ensure no-AC cycle writes camp/fridge_relay = 1 and camp/fridge_powered = 0

## 3. default.py — obd_stat response extension

- [x] 3.1 In obd_stat, read camp/fridge_powered from memcache with default 0 when key is absent
- [x] 3.2 Keep reading camp/fridge_relay with default 0 (backward compatibility)
- [x] 3.3 Return both fridge_relay and fridge_powered fields in obd_stat JSON

## 4. index.html — FRDRIVE colour semantics

- [x] 4.1 Add fridge_powered: 0 to initial client-side state object
- [x] 4.2 Update update() colour logic to use fridge_powered, not fridge_relay:
- [x] 4.2.1 Green when fridge_powered == 1
- [x] 4.2.2 Orange when frdrivebtn == 1 and fridge_powered == 0
- [x] 4.2.3 Gray when frdrivebtn == 0 and fridge_powered == 0
- [x] 4.3 Keep FRDRIVE label toggle logic unchanged (label reflects drive mode only)

## 5. tests and validation scenarios

- [x] 5.1 Validate no AC sources: expect fridge_relay = 1, fridge_powered = 0, UI not green
- [x] 5.2 Validate shore power present: expect fridge_relay = 1, fridge_powered = 1, UI green
- [x] 5.3 Validate inverter output present + INV on + FRDRIVE off: expect fridge_relay = 1, fridge_powered = 1
- [x] 5.4 Validate drive mode on + RPM 0: expect fridge_relay = 0, fridge_powered = 0
- [x] 5.5 Validate drive mode on + RPM > 0 + SOC low: expect fridge_relay = 0, fridge_powered = 0
- [x] 5.6 Validate drive mode on + RPM > 0 + SOC high: expect fridge_relay = 1, fridge_powered = 1
- [x] 5.7 Validate startup window with missing memcache keys: obd_stat returns fridge_relay = 0 and fridge_powered = 0

## 6. documentation updates

- [x] 6.1 Update drivers/camper/fridge-power-managment.md to describe relay_on versus fridge_powered split
- [x] 6.2 Update drivers/camper/README.md FRDRIVE colour description to reference fridge_powered semantics
- [x] 6.3 Confirm docs state that relay ON does not always mean fridge effectively powered (depends on AC source)

## 7. rollout sequence

- [x] 7.1 Deploy poller change first (writes camp/fridge_powered)
- [x] 7.2 Deploy default.py change second (returns fridge_powered)
- [x] 7.3 Deploy dashboard change third (consumes fridge_powered)
- [x] 7.4 Keep fridge_relay field during transition and verify no mixed-version regressions
