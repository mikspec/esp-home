## Context

`logfridge.py` is a 10-second polling loop on the RPi Zero W that controls the camper fridge relay via the ESP32 REST API. It reads RPM and button state from memcache, reads shore-power and inverter-output sensors from the ESP, and applies fridge-on logic. The web2py dashboard polls `obd_stat` every ~5 seconds to render the INV/FRDRIVE buttons and GRID/INV_OUT indicators.

Current drive-mode condition: `INV_BTN AND FRDRIVE_BTN AND RPM > 0`. No SOC check. The FRDRIVE button shows only drive-mode state (orange/gray); it carries no fridge relay status.

BMS data is written to memcache as a protobuf blob under key `BMSData` by the batmon-ha daemon. The canonical decoder is `private/batmon-ha/bms_sample_v2_pb2.py`. `logfridge.py` currently does not read BMS data.

## Goals / Non-Goals

**Goals:**
- Add SOC guard to drive-mode fridge condition in `logfridge.py`
- Make guard permissive when BMS data absent or stale
- Expose configurable threshold and staleness window as env vars
- Surface `fridge_relay` state in `obd_stat` JSON response
- Tri-state FRDRIVE button colour in the dashboard

**Non-Goals:**
- Changing ESP firmware
- Adding new memcache keys beyond what already exists (`camp/fridge_relay` already written by `logfridge.py`)
- Making SOC threshold dynamically adjustable from the UI
- Adding tooltips or secondary indicators explaining *why* fridge is blocked

## Decisions

### D1: Import path for `bms_sample_v2_pb2`

`logfridge.py` lives in `private/`, not `controllers/`, so it cannot import from `modules/` via the web2py module system. Two copies exist: `modules/bms_sample_v2_pb2.py` and `private/batmon-ha/bms_sample_v2_pb2.py`.

**Decision:** Inject `private/batmon-ha/` into `sys.path` at the top of `logfridge.py` and import from there.

*Rationale:* batmon-ha is the writer of `BMSData` — its pb2 file is the canonical one. Using a `sys.path` injection is consistent with how other private scripts in the project resolve local imports, and avoids duplication.

*Alternative considered:* Copy pb2 into `private/` alongside `logfridge.py`. Rejected — creates a second copy that can drift out of sync.

### D2: SOC staleness strategy

BMSData stays in memcache at last-written value if the BMS BLE connection drops. A stale 80% SOC would never trigger the guard.

**Decision:** Check `BMSTime` memcache key. If absent or older than `_BMS_MAX_AGE_SECS` (default 300 s), treat BMS data as unavailable → `soc_ok = True` (permissive fallback).

*Rationale:* Permissive fallback degrades gracefully to prior behaviour; the user is never surprised by a fridge that won't turn on simply because the BMS Bluetooth dropped. A 5-minute window is generous enough to survive brief BLE reconnection delays.

`BMSTime` format from memcache: `'2026-05-29 20:09:37.780496+00:00'` — parsed with `datetime.fromisoformat()`.

### D3: Env-var naming and defaults

| Env var | Default | Notes |
|---|---|---|
| `SOC_DRIVE_THRESHOLD` | `50.0` | Percent; float |
| `BMS_MAX_AGE_SECS` | `300` | Seconds; int |

Named and placed with existing vars (`_POLL_INTERVAL`, `_ESP_BASE`) at module top-level, prefixed `_` as module-private constants.

### D4: `obd_stat` response extension

`camp/fridge_relay` is already written to memcache by `logfridge.py` on every cycle. `obd_stat` in `default.py` only needs to read it and include it in the return dict. No new memcache writes.

### D5: FRDRIVE tri-state colour logic

Priority: **green > orange > gray**.

```
if fridge_relay == 1:
    green   # fridge is powered — regardless of drive mode state
elif frdrivebtn == 1:
    orange  # drive mode on, fridge not powered (RPM or SOC condition unmet)
else:
    gray    # drive mode off, fridge not powered
```

The button label (✓/✗) continues to track `frdrivebtn` state independently of colour.

## Risks / Trade-offs

- **BMSTime timezone handling** → `BMSTime` is stored as an aware datetime string (UTC offset present). `datetime.fromisoformat()` on Python 3.7+ handles this. Comparison with `datetime.now(timezone.utc)` required; naive `datetime.now()` would raise `TypeError`. Mitigation: use `datetime.now(timezone.utc)` explicitly.
- **BMS data absent on first boot** → `BMSData` and `BMSTime` keys will be absent until batmon-ha writes them. `soc_ok = True` fallback handles this cleanly.
- **`camp/fridge_relay` absent on first `obd_stat` poll** → `logfridge.py` hasn't run yet. `obd_stat` should default to `0` when key is absent (`mc.get(...) or 0`), preventing JS `undefined` in the dashboard.
- **Button colour race** → Dashboard polls every ~5 s; `logfridge.py` runs every 10 s. There is a window where button state and relay state are briefly out of sync. Acceptable — consistent with existing indicator latency.
