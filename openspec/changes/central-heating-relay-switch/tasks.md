# Tasks

## 1. ESPHome Configuration

- [x] 1.1 Add `config/central_heating.yaml` with an ESP32 node, GPIO16 Home Assistant relay switch, OFF-on-boot behavior, and shared Wi-Fi/API/OTA configuration; verify it passes `docker compose exec esphome esphome config /config/central_heating.yaml`.
- [x] 1.2 Update `drivers/central-heating/central-heating-driver.md` with the initial configuration and bench commissioning procedure; verify GPIO16 and startup behavior match the config and heater wiring stays disconnected during relay testing.

## 2. Board Commissioning

- [ ] 2.1 Flash the configuration and pair the node with Home Assistant; verify the node connects and exposes the relay switch.
- [ ] 2.2 With heater wiring disconnected, verify contacts are open after boot, close when the switch is ON, and open when OFF; correct GPIO polarity or contact selection if needed before connecting heater controls.
