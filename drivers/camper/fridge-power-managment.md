# Fridge Power Management System 

I would like to integrate esp camper driver with RasberryPi Zero W which is run in camper as well. RPi monitors:

## Existing camper management system

Existing car management system is run on RaspberryPi Zero W which is connected to few devices. 

- camper engine parameters reading engine parameters through BT from OBDII interface
- RV battery through BT connection to jkbms 
- RV and Truck battery using two channels A/D Atmega328P connected to RPi by I2C interface 
- current of heating system using converter connected to one channel A/D of Atmega328P
- current of PV panel/230V battery charger using converter connected to one channel A/D of Atmega328P    

Web2py project /home/cincin/workspace/web2pyp3/applications/camp serves UI to manage system, scripts under private directory manage to gather data from OBD, Atmega, GPS (run on android phone - which also plays router role), store data in memcache. logposition.py script is gathering data from GPS and memcache and make entry into sqlite3 database. Web2py camp application reads data from db and generate screens (main dashboard whith parameters, BMS parameters) - main screen has four buttons which allows to manage car battery charge (TBC), RV battery heating matt (HEAT), BMS charge switch (CHRG), pause OBD readings (OBD).

Having now espdriver which is able to manage inverted (switch inverter on/off) and manage power of fridge I would like to build logic around this topic. The idea is to power fridge during camper drive from inverter. I would like to add two additional flags (Grid Power) and (Inverter Outut Power), and three new buttons (INV) - Inverter, (FRIDGE) and FRDRIVE (Fridge during drive). Once clicked on Inverter - Rpi should call esp driver and switch on/off inverter relay. 

## Fridge management 

The fridge relay uses NC (normally-closed) wiring: when the relay is off the fridge is powered. Control logic is applied by `logfridge.py` on the RPi:

- When shore power is present, the fridge is always powered regardless of button state.
- When the inverter is on (`INV` button) and `FRDRIVE` is **off** — fridge is powered without conditions.
- When the inverter is on (`INV` button) and `FRDRIVE` is **on** — fridge is powered only while the engine is running (RPM > 0).
- When the inverter is off and shore power is absent — fridge is not powered.

`FRDRIVE` is **active by default**: on RPi restart, the drive-mode guard is on. The user must explicitly disable it to power the fridge unconditionally from the inverter.

```
power_present = shore_power OR inverter_output
soc_ok = True  # if BMS data absent or stale (> BMS_MAX_AGE_SECS)
         OR soc > SOC_DRIVE_THRESHOLD

fridge_on = power_present AND (
    shore_power                                              # grid connected — always power fridge
    OR (INV_BTN AND NOT FRDRIVE_BTN)                         # inverter, no drive mode — always power fridge
    OR (INV_BTN AND FRDRIVE_BTN AND RPM > 0 AND soc_ok)      # inverter, drive mode — engine running + SOC OK
)
```

Environment variables controlling the SOC guard (set in the `logfridge.py` process environment):

| Variable | Default | Description |
|---|---|---|
| `SOC_DRIVE_THRESHOLD` | `50.0` | RV battery SOC % required in drive mode |
| `BMS_MAX_AGE_SECS` | `300` | Seconds before BMSData is treated as stale/unavailable |

## Fridge management - battery SOC consideration

An additional guard condition is applied in drive mode: fridge is powered only when RPM > 0 **and** RV battery SOC > `SOC_DRIVE_THRESHOLD` (default 50%). The SOC is read from the `BMSData` memcache key (protobuf, decoded via `private/batmon-ha/bms_sample_v2_pb2.py`). If BMS data is absent or older than `BMS_MAX_AGE_SECS` seconds, the guard is bypassed (permissive fallback).

Implemented in `logfridge.py` via `_get_soc_ok()` helper.

## Fridge management - fridge power relay status

The FRDRIVE button uses tri-state background colour to show both drive-mode state and actual fridge relay status:

| Colour | Condition |
|---|---|
| Green (`#00aa00`) | `fridge_relay = 1` — fridge is powered (regardless of drive-mode state) |
| Orange (`#ff9600`) | Drive mode ON, fridge not powered (RPM or SOC condition not met) |
| Gray (`#555`) | Drive mode OFF, fridge not powered |

The button label (✓/✗) continues to reflect the drive-mode toggle independently of colour.

Implemented: `obd_stat` returns `fridge_relay` from `camp/fridge_relay` memcache key; `index.html` `update()` applies tri-state logic.


## Simplification

I have new idea - acctually we don't need FRIDGE button - just INV and FRDRIVE (active by default) - when inverter is on and FRDRIVE is on - fridge should be powered while engine is run. When INV is on and FRDRIVE is off - fridge should be powered without any conditions. Update this md and any related md's, make respective changes.

Implemented: FRIDGE button removed. See updated logic above.

Speed and engine RPM are stored under keys 'SPEED', 'RPM'. 

``` shell
(obd) pi@raspicamp:~/workspace/web2pyp3/applications/camp/private $ python getmemcache.py
{'INTAKE_TEMP': 37, 'camp/bat1_vol': '13.23', 'OBD_RESP': 0.715447, 'BMSTime': '2026-05-29 20:09:37.780496+00:00', 'RPM': 0, 'camp//sys/bus/w1/devices/28-0000061ffd8a': '22.75', 'camp/dht_humi': '35.1', 'OIL_TEMP': 76, 'camp//sys/bus/w1/devices/28-00000a07518e': '18.0', 'camp/temp_time': '2026-05-29 20:09:21.897261', 'BMSData': b'\n<\x08\xe5g(\x86\xfe\x0f0\x80\xc4\x138\xb0\xff\x04@\xc1\x9bGH\x03R\x08\x94\xa0\x01\xd0\xa5\x01\x00\x00X\xc4\xa9\x01b\x06\x08\x01\x10\x01\x18\x01h\x91\xcb\x83\rp\x81\xe5\xe7\xd0\x06\x88\x01\x01\x90\x01\x01\x12\x08\xfa\x19\xfa\x19\xf9\x19\xf9\x19', 'camp/dht_temp': '21.6', 'ENGINE_LOAD': 0, 'SPEED': 0}
```
## Connections

Propose the best solution for given requirements. Both Rpi and esp are in the same local lan given by Android phone. I was thinking about writing additional private script which will read data from memcache, connects to espdriver by ESPHome api to manage relays, and read power sensors, and serves information to camp web2py application. Check code base and propose. Ask for unknowns. 
