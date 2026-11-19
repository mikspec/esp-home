# Spec Delta

## Purpose

Extends Home Assistant's Bluetooth reception through the central-heating node so nearby BTHome sensors can report measurements.

## ADDED Requirements

### Requirement: Forward BLE advertisements
The central-heating node SHALL forward received BLE advertisements to Home Assistant's Bluetooth integration when connected to Home Assistant.

#### Scenario: Thermometer is within range of the proxy
- **WHEN** the connected node receives an advertisement from the BTHome thermometer
- **THEN** the advertisement is made available to Home Assistant's Bluetooth integration
- **AND** Home Assistant's BTHome integration can update the existing thermometer entities when the advertisement is valid and any required bindkey is configured

### Requirement: Proxy does not offer active GATT connections
The node SHALL provide advertisement forwarding without offering active GATT connection slots through the proxy.

#### Scenario: Proxy becomes available
- **WHEN** Home Assistant connects to the central-heating node
- **THEN** the proxy is available for BLE advertisements without active GATT connections

### Requirement: Preserve relay control while proxy is operating
The node SHALL retain its existing Home Assistant relay control and OFF-on-boot policy while Bluetooth proxying is enabled.

#### Scenario: Relay operated during BLE scanning
- **WHEN** Home Assistant switches the heater relay ON or OFF while the proxy is scanning
- **THEN** the relay responds according to the existing switch semantics

#### Scenario: Restart with proxy enabled
- **WHEN** the node completes startup with Bluetooth proxying enabled
- **THEN** the relay is OFF, regardless of its previous state
