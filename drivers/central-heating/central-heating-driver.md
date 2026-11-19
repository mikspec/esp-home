# Central Heating Driver

My house central heating  original driver has two parts - temperature sensor with display and scheduler installed in main room.
It allows to set night and day temperatures and allows to set schedule for every day of week. Scheduler allows to set for each
hour day or night temperature flag. Day and night temperature is set globally. Driver check temperature from sensor and compare
to current hour setup - if temperature is too low it sends signal `On` to second part of the driver.
When temperature is to high - signal `Off` is send. There is histerese algorithm implemented.

Second part of the driver receives radio signal `On\Off` and switches on/off relay which connects or disconnects two cables 
mounted to heater. 

Making changes in setup requires pressing phisical buttons many times - is time consuming and can't be done remotely.

# New driver

The idea is to replace original driver with custom one - I have relay board (single channel) with esp32. This board can play 
role of second driver module. I would like to include this board into home assistant system. HA can provide scheduling capabilities 
together with current temperature analysis. I bought thermometer which publishes readings through BT from main room, HA is able to 
read and analyse temp and humidity. 

# Initial configuration

The initial ESPHome configuration is [config/central_heating.yaml](../../config/central_heating.yaml). It targets an ESP32 (`esp32dev`) and exposes one Home Assistant switch for the relay.

The [ESP32 Relay x1 board reference](https://devices.esphome.io/devices/esp32-relay-x1/) lists the relay on GPIO16, the onboard button on GPIO0, and the LED on GPIO23. This configuration uses GPIO16 without inversion and with `restore_mode: ALWAYS_OFF`; it does not configure the button or LED. Logical ON is intended to close the heater-control contacts; OFF and startup are intended to leave them open. Confirm the board matches the reference and verify actual relay and contact behavior before connecting the heater.

The configuration reuses the shared home Wi-Fi, API encryption, fallback AP, and OTA secrets. Temperature analysis and scheduling remain in Home Assistant and are not implemented on this board.

## Commissioning

1. Validate the configuration with `docker compose exec esphome esphome config /config/central_heating.yaml`.
2. Flash the configuration and pair the device with Home Assistant. The reference says the board's micro-USB connector is power-only: initial serial flashing requires an external USB-to-serial adapter connected to the board's flashing pins. Verify adapter voltage and wiring against the board documentation.
3. Keep heater-control wires disconnected. With the board powered, verify the relay contact pair is open after boot, closes when the Home Assistant switch is ON, and opens when it is OFF.
4. If GPIO16 or relay polarity does not produce those states, correct the pin or inversion and repeat the isolated test.
5. Connect heater-control wires only after contact behavior is confirmed.

# BT thermometer

I have thermometer with custom firmware from https://github.com/pvvx/ATC_MiThermometer - it uses BTHome v2 protocol.
Home assistant is reading data from it. Thermometer is far away from RPi5 where HA is hosted and there are BT connection issues.
Heating driver will be in the middle between thermometer and RPi5 - I wonder if heating driver can also play BT proxy?

## Bluetooth proxy

The node forwards BLE advertisements to Home Assistant through its existing encrypted ESPHome API. Home Assistant's Bluetooth and BTHome integrations process the thermometer's readings; no duplicate ESPHome sensor or bindkey is configured on the node. Active GATT connections are disabled, while BLE scanning uses ESPHome's default settings.

### Commissioning and rollback

1. Validate and compile `config/central_heating.yaml`, then upload it using OTA if supported. First-time BLE enablement may require a partition-table update and serial flashing; the board's micro-USB port is power-only, so use an appropriate USB-to-serial adapter on its flashing pins if required.
2. Confirm the node is connected in Home Assistant and is available as a Bluetooth proxy. Verify the thermometer is seen by this proxy specifically and its existing temperature and humidity entities receive repeated updates from the intended location. Retain the existing BTHome bindkey if the thermometer uses encryption.
3. Keep heater-control wires disconnected while testing. With BLE scanning active, verify the relay responds to ON/OFF commands and remains OFF after reboot.
4. Compare Wi-Fi latency and monitor logs for disconnects, resets, or memory problems. BLE and Wi-Fi share the radio.
5. If the proxy causes instability, restore the previous ESPHome configuration and firmware. A partition-table change may require serial flashing to restore the previous firmware.

# Heater driver logic

I wonder if whole logic (scheduler, day/night temperatures setup, histerese algorithm) should be implemented by home assistant or
by heater esp. Having whole login on HA site will create strong dependency on wifi connection. Heater ESP now is receiving adds from 
thermometer. Explore options.

I have idea - let's introduce min temp limit - ex 16 st C (configured), heater driver active flag (if not set then no action on relay).
