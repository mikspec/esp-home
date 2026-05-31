## 1. ESP32 Firmware

- [x] 1.1 Add `inverted: true` to fridge relay GPIO pin (GPIO26) in `config/camper.yaml`
- [x] 1.2 Change fridge relay `restore_mode` from `ALWAYS_OFF` to `ALWAYS_ON` in `config/camper.yaml`

## 2. RPi Bridge Script

- [x] 2.1 Create `logfridge.py` in `web2pyp3/applications/camp/private/` — 10s polling loop reading memcache and ESP REST, applying inverter and fridge logic, writing state back to memcache

## 3. web2py Dashboard

- [x] 3.1 Extend `obd_stat` in `controllers/default.py` to read and return `INV_BTN`, `FRDRIVE_BTN`, `camp/shore_power`, `camp/inverter_output` from memcache
- [x] 3.2 Extend `obd_save` in `controllers/default.py` to write `INV_BTN`, `FRDRIVE_BTN` to memcache based on request vars
- [x] 3.3 Add INV and FRDRIVE buttons and GRID/INV_OUT indicators to `views/default/index.html` with matching JS state, onclick handlers, save/update functions

## 4. Documentation

- [x] 4.1 Update `drivers/camper/README.md` to document fridge relay inversion, `ALWAYS_ON` restore mode, and reference to `logfridge.py`
