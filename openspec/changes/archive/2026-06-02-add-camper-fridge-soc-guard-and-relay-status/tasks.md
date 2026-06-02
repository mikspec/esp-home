## 1. logfridge.py — BMS import and env vars

- [x] 1.1 Add `sys.path` injection at module top to include `private/batmon-ha/` and import `BmsData` from `bms_sample_v2_pb2`
- [x] 1.2 Add `_SOC_DRIVE_THRESHOLD` env var constant (default `50.0`, float)
- [x] 1.3 Add `_BMS_MAX_AGE_SECS` env var constant (default `300`, int)

## 2. logfridge.py — SOC guard helper

- [x] 2.1 Implement `_get_soc_ok()` helper: reads `BMSData` and `BMSTime` from memcache, checks freshness against `_BMS_MAX_AGE_SECS`, decodes protobuf SOC, returns `True` when data unavailable/stale or `soc > _SOC_DRIVE_THRESHOLD`
- [x] 2.2 Use `datetime.now(timezone.utc)` for freshness comparison (handle aware datetime from `BMSTime`)

## 3. logfridge.py — fridge logic update

- [x] 3.1 Call `_get_soc_ok()` in `_poll()` and add `soc_ok` to the drive-mode condition: `inv_btn and frdrive_btn and rpm > 0 and soc_ok`
- [x] 3.2 Add `soc_ok` to the debug log line

## 4. default.py — obd_stat extension

- [x] 4.1 In `obd_stat`, read `camp/fridge_relay` from memcache (`mc.get('camp/fridge_relay') or 0`) when `CAR_DEPLOYMENT` is true
- [x] 4.2 Add `fridge_relay` to the `obd_stat` return dict

## 5. index.html — dashboard tri-state button

- [x] 5.1 Add `fridge_relay: 0` to the initial `state` object
- [x] 5.2 Update `update()` function: replace two-state FRDRIVE colour logic with tri-state — green when `fridge_relay == 1`, orange when `frdrivebtn == 1 && fridge_relay == 0`, gray otherwise

## 6. Spec and docs update

- [x] 6.1 Update `fridge-power-managment.md` — add SOC guard to the fridge logic pseudocode block, document new env vars, update drive-mode scenario table
- [x] 6.2 Update `drivers/camper/README.md` — update FRDRIVE button colour description in the RPi Integration section to reflect tri-state (green/orange/gray) logic
