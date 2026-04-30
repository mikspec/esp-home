# Carport Relay Controller

ESPHome retrofit for the legacy ESP8266 C driver (garage branch) controlling two carport outputs:
- relay1: 230V plug
- relay2: carport light

## Entity Mapping

New ESPHome entities:
- `switch.carport_carport_plug` -> relay1 (230V plug)
- `switch.carport_carport_light` -> relay2 (carport light)
- `switch.carport_carport_light_watchdog` -> watchdog enable/disable for light runtime cap
- `number.carport_carport_light_max_on_minutes` -> light watchdog timeout in minutes

Home Assistant script wrappers (from package):
- `script.carport_plug_on`
- `script.carport_plug_off`
- `script.carport_light_on`
- `script.carport_light_off`
- `script.carport_all_off`

## Power Recovery Behavior

- Plug (relay1): restore last state (`RESTORE_DEFAULT_OFF` behavior).
- Light (relay2): always starts off (`ALWAYS_OFF`).

## Light Watchdog Behavior

- Watchdog can be turned on/off via `switch.carport_carport_light_watchdog`.
- When enabled, light is forced off if active longer than `number.carport_carport_light_max_on_minutes`.
- When disabled, no auto-off timeout is enforced.

## Migration Plan

1. Flash [config/carport.yaml](config/carport.yaml) to target ESP8266.
2. Include [config/packages/carport_ha.yaml](config/packages/carport_ha.yaml) in Home Assistant packages.
3. Verify entities and scripts are visible in Home Assistant.
4. Validate restore and watchdog behavior with on-device tests.
5. Disable legacy carport C-driver automation/endpoints after successful verification.

## Rollback Plan

1. Disable Home Assistant package usage for carport scripts.
2. Re-enable legacy carport C-driver control path.
3. Flash previous firmware if required.

## Commissioning Checklist

- Relay pin mapping is correct:
  - relay1 -> GPIO12 -> carport plug
  - relay2 -> GPIO13 -> carport light
- Relay polarity is correct (on/off command matches hardware output).
- Reboot test confirms:
  - plug restores previous state
  - light starts off
- Independent control test confirms plug and light do not alter each other.
- Watchdog test confirms:
  - enabled mode forces light off after timeout
  - disabled mode does not force timeout-based off
