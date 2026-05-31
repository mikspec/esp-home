## 1. ESPHome Config Update

- [x] 1.1 Add `binary_sensor` block for `Camper Shore Power` on GPIO21 with `device_class: power` and `ALWAYS_OFF` disabled restore mode.
- [x] 1.2 Add `binary_sensor` block for `Camper Inverter Output` on GPIO22 with `device_class: power` and `ALWAYS_OFF` disabled restore mode.
- [x] 1.3 Verify `inverted:` polarity flag matches the optocoupler module output (active-high vs active-low) and adjust if needed.

## 2. Validation

- [x] 2.1 Compile ESPHome config and confirm no errors for both new binary sensor entities.
- [x] 2.2 Flash device and confirm `Camper Shore Power` and `Camper Inverter Output` entities appear in Home Assistant.
- [x] 2.3 Verify shore power sensor toggles correctly when shore power is connected and disconnected.
- [x] 2.4 Verify inverter output sensor toggles correctly for both grid pass-through and inverter-generated phase.

## 3. Documentation Updates

- [x] 3.1 Update `drivers/camper/README.md` pin assignment table to include GPIO21 and GPIO22 sensor entries.
- [x] 3.2 Complete wiring diagram and connection notes in `drivers/camper/phase-sensors.md`.
- [x] 3.3 Confirm and document final signal polarity (active-high or active-low) for the optocoupler module used.
