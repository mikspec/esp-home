## 1. ESPHome Irrigation Device Setup

- [x] 1.1 Create ESPHome irrigation device configuration for ESP8266 with relay mapping: relay1 pump, relay2 left, relay3 right, relay4 back, relay5 lines
- [x] 1.2 Configure all irrigation relays with safe startup behavior (default off) and verify boot-safe pin behavior
- [x] 1.3 Expose logical entities for four sections with clear names and IDs used by automations
- [x] 1.4 Expose pump state as a read-only Home Assistant entity while keeping pump relay control internal to automation logic

## 2. Section Interlock And Pump Coupling

- [x] 2.1 Implement mutual exclusion so only one section (left/right/back/lines) can be active at a time
- [x] 2.2 Implement section transition behavior where enabling one section forces all other sections off
- [x] 2.3 Implement pump coupling so any active section turns pump on and no active sections turn pump off

## 3. Runtime Safety Watchdog

- [x] 3.1 Add configurable max runtime parameter for section operation with default 120 minutes and bounded range
- [x] 3.2 Implement watchdog checks that detect section runtime limit violations
- [x] 3.3 Implement forced safe-off action that turns off all sections and pump when runtime limit is exceeded

## 4. Home Assistant Sequence Scheduling

- [x] 4.1 Create Home Assistant irrigation package and define a daily 05:00 trigger automation
- [x] 4.2 Implement sequence script: right 15 minutes -> left 15 minutes -> back 15 minutes -> lines 30 minutes
- [x] 4.3 Add explicit end-of-sequence all-off cleanup for sections and pump
- [x] 4.4 Add manual hot-period toggle and optional evening trigger controls with default evening time set to 22:00 that execute the same sequence

## 5. Validation And Cutover

- [x] 5.1 Validate relay mapping and exclusivity with manual control tests in Home Assistant
- [x] 5.2 Validate schedule timing and pump behavior against expected timeline from legacy cron flow
- [x] 5.3 Validate runtime watchdog by simulating overrun and confirming forced safe-off
- [x] 5.4 Validate watchdog notification delivery to `notify.all_phones`
- [x] 5.5 Disable legacy cron relay script after successful end-to-end validation and document rollback path
