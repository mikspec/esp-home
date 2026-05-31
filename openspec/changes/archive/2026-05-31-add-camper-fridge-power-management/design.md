## Context

The camper ESP32 manages the fridge and inverter relays. The Raspberry Pi Zero W runs the camp management system (web2py + memcache + polling scripts). These two systems are currently independent. The fridge should be powered intelligently based on AC source state, engine RPM, and user intent. A new Python script (`logfridge.py`) bridges them: it polls the ESP REST API and memcache, applies the fridge and inverter decision logic, and writes sensor state back to memcache for the dashboard. The web2py dashboard gains three control buttons and two AC source indicators.

## Goals / Non-Goals

**Goals:**
- Invert the fridge relay GPIO so ESPHome switch ON = fridge powered.
- Change fridge restore mode to `ALWAYS_ON` (fridge on at boot, script re-evaluates within 10 s).
- Implement `logfridge.py` as a 10-second polling loop following the existing private script pattern.
- Extend `obd_stat`/`obd_save` endpoints to carry the two new button states and two sensor flags.
- Add INV and FRDRIVE buttons and GRID/INV_OUT indicators to the index dashboard.

**Non-Goals:**
- No Home Assistant automations — all logic in `logfridge.py`.
- No persistent button state — memcache only, clears on RPi restart.
- No ESP web server authentication.
- No changes to inverter safe-shutdown sequence or N-G bond logic.

## Decisions

### HTTP REST over aioesphomeapi

All existing private scripts (logarduino, loggersync) are synchronous polling loops. `aioesphomeapi` requires asyncio and is a paradigm shift. The ESP `web_server` is already enabled on port 80. HTTP REST with the `requests` library is consistent, simple, and requires no new dependencies beyond what the existing environment is likely to have.

### Memcache as state bus

Button state is written to memcache by the web2py controller on user interaction, and read by `logfridge.py` on each poll cycle. Sensor state (shore_power, inverter_output, relay states) is written to memcache by `logfridge.py` and read by the web2py controller's `obd_stat` endpoint. No direct ESP calls from web2py — all ESP interaction is isolated in `logfridge.py`.

### Extend `obd_stat`/`obd_save` rather than new endpoints

The frontend already uses a single save/stat JSON round-trip for all button state. Adding the three new buttons to the existing payload keeps the frontend pattern uniform and avoids a second AJAX endpoint.

### ESP REST URL slugs

ESPHome web server derives REST paths from the entity `name` field (lowercase, spaces to underscores). Entity paths used:
- `GET /binary_sensor/camper_shore_power`
- `GET /binary_sensor/camper_inverter_output`
- `GET /switch/camper_inverter` + `/turn_on`, `/turn_off`
- `GET /switch/camper_fridge` + `/turn_on`, `/turn_off`

### Inverter desired-state model

`INV_BTN` in memcache represents the desired inverter state (1 = on, None/0 = off). `logfridge.py` reads the current switch state from ESP and only calls turn_on/turn_off when the desired and actual states differ. This avoids triggering the safe-shutdown script on every poll cycle.

### Fridge logic expression

```python
power_present = shore_power or inverter_output

fridge_on = power_present and (
    shore_power                                      # grid: always power fridge
    or (inv_btn and not frdrive_btn)                 # inverter on, no drive mode: always power fridge
    or (inv_btn and frdrive_btn and rpm > 0)         # inverter on, drive mode: engine running
)
```

When `power_present` is false, `fridge_on` is always false — no point activating the relay with no AC source.

## Risks / Trade-offs

- **ESP unreachable**: If the ESP is offline, `logfridge.py` will log a warning and skip the cycle. The fridge relay retains its last state (physically safe — `ALWAYS_ON` at boot means it defaults to powered). → Acceptable.
- **RPM staleness**: OBD reader may not update frequently. `rpm or 0` treats missing RPM as zero (engine off), which is the conservative safe choice — fridge won't stay on if engine state is unknown during drive mode.
- **10-second lag**: Button presses take up to 10 seconds to propagate to the ESP. Acceptable for a camper fridge management system.

## Migration Plan

1. Flash `config/camper.yaml` (fridge relay inverted + ALWAYS_ON). Fridge powers on immediately.
2. Deploy `logfridge.py` to RPi private scripts directory and start it (e.g., add to cron or systemd).
3. Deploy updated `controllers/default.py` and `views/default/index.html`.
4. Verify dashboard shows GRID/INV_OUT indicators and new buttons work.

## Open Questions

None.
