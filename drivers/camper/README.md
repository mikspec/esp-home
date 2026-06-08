# Camper Relay Controller (ESP32 Relay X4_V1.1)

ESPHome baseline configuration for an ESP32 relay X4_V1.1 board controlling camper loads.

## Scope

This initial profile focuses on safe startup defaults, secure connectivity, and Home Assistant discovery.

## Relay Roles

- Relay 1: Camper inverter (board relay 4)
- Relay 2: Camper fridge (board relay 3)
- Relay 3: N-G bond relay (board relay 2)
- Relay 4: Spare relay 1 (board relay 1)

Current GPIO mapping in [config/camper.yaml](../../config/camper.yaml):

- Camper inverter: GPIO25
- Camper fridge: GPIO26
- Camper N-G bond relay: GPIO33
- Camper spare relay 1: GPIO32
- Camper shore power sensor: GPIO21 (binary input, `pullup`, active-low, `inverted: true`)
- Camper inverter output sensor: GPIO22 (binary input, `pullup`, active-low, `inverted: true`)

## Startup And Recovery Policy

- Inverter relay: always off on boot (`ALWAYS_OFF`) — the internal GPIO switch is turned off directly on boot, bypassing the safe-shutdown script
- Fridge relay: always on on boot (`ALWAYS_ON`, with `inverted: true`) — fridge is powered immediately; `logfridge.py` re-evaluates within 10 seconds
- N-G bond relay: always off on boot (`ALWAYS_OFF`); automation re-evaluates after boot if `Camper N-G Bond Auto` was enabled
- Spare relay 2: always off on boot (`ALWAYS_OFF`)
- N-G bond auto switch: restores last persisted state on boot (`RESTORE_DEFAULT_OFF`)

On boot, the config explicitly turns off inverter (GPIO directly) and N-G bond relays, then re-evaluates the bond condition.

## Inverter Switch Architecture

The inverter is controlled via a **template switch** (`Camper Inverter`) wrapping an internal GPIO switch (`camper_inverter_relay_gpio`, GPIO25). This pattern enables the safe shutdown sequence to run before the GPIO is de-energised.

The internal GPIO switch is not exposed to Home Assistant or the REST API — all external callers use the `Camper Inverter` template switch entity as normal.

### Inverter Safe Shutdown Sequence

Turning off `Camper Inverter` (via Home Assistant, REST API, or any external caller) triggers the following sequence automatically:

```
1. If N-G bond relay is ON → turn it OFF immediately
2. Wait 5 seconds
3. Turn off inverter relay GPIO
```

This ensures the neutral-PE bond is opened while the inverter is still producing stable AC, preventing transients on the AC bus during shutdown.

> **Note:** The 5-second delay only applies when the N-G bond relay was ON at the time of the turn-off command. If the bond relay was already open, the inverter cuts immediately.

## Connectivity And Security

The config reuses shared secrets from [config/secrets.yaml.template](../../config/secrets.yaml.template):

- `wifi_ssid`, `wifi_password`
- `fallback_password`
- `api_encryption_key`
- `ota_password`

Enabled features:

- Wi-Fi station mode with fallback AP (`Camper Fallback`)
- Encrypted Home Assistant API
- Password-protected OTA updates
- Optional web server on port 80

## AC Topology

```
Shore (L) ─────────────┬─────────────────────► Inverter AC IN (L)
                       │ [Shore sensor GPIO21]
Shore (N) ─────────────┴─────────────────────► Inverter AC IN (N)
Shore (PE)──────────────────────────────────────► PE bus

                                    Inverter AC OUT (L) ──┬──────────────► L bus
                                                          │ [Output sensor GPIO22]
                                    Inverter AC OUT (N) ──┴──────────────► N bus

                    ┌─── [N-G Bond relay GPIO33] ────┐
                    │    Camper N-G Bond             │
                   N bus                            PE bus
                   (closed only when inverter generating)

L bus ──[Inverter relay GPIO25]──► Inverter DC on/off
L bus ──[Fridge relay GPIO26]────► Fridge
```

### N-G Bond Automation Logic

| Shore Power | Inverter Output | Auto ON | N-G Bond Relay                        |
|-------------|-----------------|---------|---------------------------------------|
| OFF         | ON              | ON      | CLOSED (after 5 s stabilisation delay)|
| ON          | ON              | ON      | OPEN                                  |
| OFF         | OFF             | ON      | OPEN                                  |
| ON          | OFF             | ON      | OPEN                                  |
| OFF         | *               | OFF     | OPEN → manual allowed                 |
| ON          | *               | OFF     | OPEN → manual **blocked**             |

> **Safety interlock:** the bond relay firmware rejects any turn-on command (automated or manual) when shore power is present, regardless of automation state. See [inverter-floating-phase.md](inverter-floating-phase.md) for details.

## Diagnostics

Exposed entities include:

- Wi-Fi signal sensor
- Uptime sensor
- Connected SSID text sensor
- IP address text sensor
- ESPHome version text sensor
- Shore power binary sensor (`Camper Shore Power`, GPIO21) — ON when grid phase is present
- Inverter output binary sensor (`Camper Inverter Output`, GPIO22) — ON when AC phase present on inverter output

### Phase Sensor States

| Shore Power | Inverter Output | Meaning                        |
|-------------|-----------------|--------------------------------|
| ON          | ON              | Grid pass-through              |
| OFF         | ON              | Inverter generating (off-grid) |
| OFF         | OFF             | No AC available                |
| ON          | OFF             | ⚠ Unusual — check inverter     |

## Fridge Relay

The fridge relay GPIO (GPIO26) is configured with `inverted: true` so that the ESPHome entity semantics match user intent:

- **`Camper Fridge` switch ON** → GPIO LOW → relay de-energised → NC closed → fridge **powered**
- **`Camper Fridge` switch OFF** → GPIO HIGH → relay energised → NO connected → fridge **not powered**

Note: effective fridge power also depends on AC source availability (`shore_power` or `inverter_output`). In the no-AC case, relay ON means NC path closed but the fridge is still effectively unpowered.

The fridge is managed automatically by `logfridge.py` on the Raspberry Pi, which applies context-aware power logic based on AC source state, engine RPM, and user button flags. See [fridge-power-managment.md](fridge-power-managment.md) for full logic description.

## RPi Integration

`logfridge.py` (`web2pyp3/applications/camp/private/logfridge.py`) runs as a 10-second polling loop on the Raspberry Pi. It:

- Reads `RPM`, `INV_BTN`, `FRDRIVE_BTN` from memcache
- Reads `BMSData` / `BMSTime` from memcache to evaluate RV battery SOC for the drive-mode guard
- Reads `Camper Shore Power` and `Camper Inverter Output` from the ESP REST API (port 80)
- Applies inverter desired-state sync and fridge power logic (including SOC guard in drive mode)
- Writes `camp/shore_power`, `camp/inverter_output`, `camp/fridge_relay`, `camp/fridge_powered`, `camp/inverter_relay` to memcache for the web2py dashboard

Configure the ESP IP/hostname via the `CAMPER_ESP_IP` environment variable (default: `camper.local`).

Additional env vars controlling the SOC guard:

| Variable | Default | Description |
|---|---|---|
| `SOC_DRIVE_THRESHOLD` | `50.0` | RV battery SOC % required to power fridge in drive mode |
| `BMS_MAX_AGE_SECS` | `300` | Seconds before BMSData is treated as stale (guard bypassed) |

### FRDRIVE Button Colour States

The dashboard FRDRIVE button uses tri-state background colour:

| Colour | Meaning |
|---|---|
| Green | Effective fridge power is ON (`fridge_powered = 1`) |
| Orange | Drive mode ON, but fridge not powered (engine stopped or SOC too low) |
| Gray | Drive mode OFF, fridge not powered |



1. Copy [config/secrets.yaml.template](../../config/secrets.yaml.template) to [config/secrets.yaml](../../config/secrets.yaml) and fill values.
2. Validate configuration syntax in ESPHome before flashing.
3. Verify relay polarity (active-low vs active-high).
4. Verify GPIO-to-relay wiring matches the board revision.
5. Power-cycle device and confirm startup policy:
   - inverter off
   - fridge on (ALWAYS_ON — powered immediately on boot; logfridge.py re-evaluates within 10 s)
   - spare relays off
6. Pair with Home Assistant and verify switch and diagnostic entities.
7. Verify `Camper Shore Power` binary sensor reports ON when shore power is connected.
8. Verify `Camper Inverter Output` binary sensor reports ON when inverter output phase is present.
9. Enable `Camper N-G Bond Auto`, disconnect shore power, and verify N-G bond relay closes.
10. Reconnect shore power and verify N-G bond relay opens.
11. With shore power connected, attempt to manually close `Camper N-G Bond` from Home Assistant — verify the relay does **not** close (safety interlock).

## Home Assistant Migration

If upgrading from a previous firmware that exposed `Camper Spare Relay 1`:
- Update any HA dashboard cards referencing the old entity.
- Update any HA automations using `switch.camper_spare_relay_1`.
- The new entity is `switch.camper_n_g_bond` (exact ID depends on HA device naming).

## Follow-Ups

- Confirm final GPIO mapping against the exact X4_V1.1 hardware revision and update config if needed.
- Consider making fridge startup policy configurable in a follow-up change (restore vs force on).
