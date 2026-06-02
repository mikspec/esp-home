## Why

The fridge drive-mode guard currently only checks engine RPM, but running the fridge from the inverter while the RV battery is depleted risks stranding the vehicle. Additionally, the FRDRIVE button gives no feedback on whether the fridge is actually powered, making it hard to tell at a glance whether the drive-mode condition is met or blocked.

## What Changes

- `logfridge.py` gains a SOC guard: in drive mode the fridge is powered only when RPM > 0 **and** RV battery SOC > threshold (default 50%). If BMS data is absent or stale (> 5 min), the guard is bypassed (permissive fallback).
- `logfridge.py` gains two new env-var constants: `SOC_DRIVE_THRESHOLD` (default `50.0`) and `BMS_MAX_AGE_SECS` (default `300`), consistent with existing `_POLL_INTERVAL` / `CAMPER_ESP_IP` pattern.
- `logfridge.py` imports `BmsData` from `private/batmon-ha/bms_sample_v2_pb2.py` via `sys.path` injection.
- `obd_stat` endpoint in `default.py` adds `fridge_relay` to its JSON response (read from `camp/fridge_relay` memcache key).
- `index.html` FRDRIVE button gains tri-state colour logic: **green** when fridge is powered (regardless of drive mode), **orange** when drive mode is on but fridge is blocked (RPM/SOC condition unmet), **gray** when drive mode off and fridge not powered.

## Capabilities

### New Capabilities
<!-- none -->

### Modified Capabilities

- `camper-fridge-power-management`: Fridge drive-mode condition gains SOC guard; new env-var configuration for threshold and BMS staleness window.
- `camper-dashboard`: `obd_stat` response gains `fridge_relay` field; FRDRIVE button gains tri-state colour feedback (green/orange/gray).

## Impact

- `web2pyp3/applications/camp/private/logfridge.py` — core logic, imports, env vars
- `web2pyp3/applications/camp/controllers/default.py` — `obd_stat` function only
- `web2pyp3/applications/camp/views/default/index.html` — `update()` function and initial state
- No ESP firmware changes required
- No new dependencies (protobuf already in use via batmon-ha)
