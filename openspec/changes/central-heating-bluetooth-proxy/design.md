# Design

## Context

The existing ESP32-WROOM-32E node exposes the heater relay via the encrypted ESPHome API and uses ESP-IDF with Wi-Fi power saving disabled. Its relay must continue to behave as before; Home Assistant already integrates the BTHome thermometer.

## Goals / Non-Goals

**Goals:**
- Forward nearby BLE advertisements through the existing Home Assistant API.
- Verify reception through this proxy, Wi-Fi stability, and relay behavior after deployment.

**Non-Goals:**
- Decode thermometer data on the ESP32, create duplicate ESPHome sensor entities, or modify the existing BTHome integration.
- Implement active GATT connections, temperature control, schedules, or stale-reading safety logic.

## Decisions

- Add ESPHome `esp32_ble_tracker` and `bluetooth_proxy` with `active: false`, leaving tracker scan parameters at their defaults. BTHome measurements are broadcast; GATT connection slots are not needed. Proxy active connections and active scanning are separate settings.
- Retain the existing API encryption, Wi-Fi settings, web server, and relay configuration. The alternative of decoding the sensor on the node would duplicate Home Assistant's existing BTHome integration.
- Home Assistant's Bluetooth integration aggregates the proxy with other adapters; keep the existing thermometer and its bindkey if encryption is enabled. No sensor MAC or bindkey is added to ESPHome.
- Use default coexistence and scan settings initially rather than aggressive scanning, and assess reception and network behavior on real hardware.

## Risks / Trade-offs

- BLE and Wi-Fi share the radio and BLE consumes RAM → monitor Wi-Fi latency, reconnects, and ESP32 stability while verifying the relay. If resources are inadequate, investigate a separate proxy or reducing optional components.
- A closer node may still receive weak BLE signals → verify Home Assistant sees the thermometer through this specific proxy and observe repeated readings at the intended placement.
- First BLE enablement may require a partition-table change → verify upgrade feasibility and use the board's serial flashing header and an appropriate USB-to-serial adapter if needed; its micro-USB port is power-only.
- Missing or stale measurements can affect future HA heating automations → thermostat behavior and stale-reading safeguards require separate design and are not provided by the proxy.

## Migration Plan

Validate and compile the updated configuration, then flash using OTA if supported or serial if required. Pair or confirm the ESPHome node in Home Assistant and verify proxy reception and relay behavior. If unstable, restore the previous ESPHome configuration and firmware; the previous firmware may require serial flashing if the partition table changed.
