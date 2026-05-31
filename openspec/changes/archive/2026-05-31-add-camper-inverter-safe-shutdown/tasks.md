## 1. ESP32 Firmware

- [x] 1.1 Rename `camper_inverter_relay` GPIO switch to `camper_inverter_relay_gpio` and add `internal: true`
- [x] 1.2 Add template switch `camper_inverter_relay` ("Camper Inverter") with `turn_on_action` targeting `camper_inverter_relay_gpio` and `turn_off_action` executing `inverter_safe_shutdown` script
- [x] 1.3 Add `inverter_safe_shutdown` script: check N-G bond ON → turn off N-G bond → delay 5 s → turn off `camper_inverter_relay_gpio`
- [x] 1.4 Update `on_boot` block to reference `camper_inverter_relay_gpio` directly (bypass template switch)

## 2. Documentation

- [x] 2.1 Update `drivers/camper/README.md` to document the template switch architecture and safe shutdown sequence
