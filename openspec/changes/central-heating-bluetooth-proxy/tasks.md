# Tasks

## 1. Proxy Configuration

- [ ] 1.1 Add ESP32 BLE tracking and an advertisement-only Bluetooth proxy to `config/central_heating.yaml` without changing the existing relay, API, or Wi-Fi settings; verify ESPHome config validation and firmware compilation succeed.
- [x] 1.2 Document the proxy, Home Assistant integration, serial-flashing/partition caveat, rollback, and bench verification in `drivers/central-heating/central-heating-driver.md`; verify the instructions match the config and keep heater wiring disconnected for relay tests.

## 2. Device Commissioning

- [ ] 2.1 Upload the firmware using OTA if supported or serial if required; verify the node reconnects to Home Assistant and appears as an advertisement-only Bluetooth proxy.
- [ ] 2.2 Verify the intended thermometer is detected through the central-heating proxy specifically and existing BTHome temperature and humidity entities receive repeated updates at the intended placement.
- [ ] 2.3 With heater wiring disconnected, verify relay ON/OFF commands and OFF-on-boot behavior while scanning; check Wi-Fi latency and logs for disconnects or memory-related instability.
