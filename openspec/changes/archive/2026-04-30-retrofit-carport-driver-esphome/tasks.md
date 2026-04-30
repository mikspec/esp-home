## 1. ESPHome Carport Device Configuration

- [x] 1.1 Create ESPHome carport configuration for ESP8266 with fixed mapping: relay1 carport 230V plug, relay2 carport light
- [x] 1.2 Configure relay startup behavior by role (relay1 restore last state, relay2 safe off) and verify boot-time pin behavior on target board
- [x] 1.3 Expose stable, clearly named Home Assistant entities for carport plug and carport light

## 2. Home Assistant Integration Package

- [x] 2.1 Create Home Assistant package for carport integration under config/packages
- [x] 2.2 Add service/script wrappers for consistent plug and light control actions
- [x] 2.3 Add watchdog controls for carport light (enable/disable toggle and max-on minutes helper)
- [x] 2.4 Validate entity IDs and package wiring in configuration examples

## 3. Light Watchdog Runtime Safety

- [x] 3.1 Implement carport light watchdog logic that tracks continuous active runtime
- [x] 3.2 Enforce auto-off when watchdog is enabled and active runtime exceeds configured x minutes
- [x] 3.3 Ensure watchdog disabled state bypasses runtime auto-off behavior

## 4. Migration Documentation And Cutover

- [x] 4.1 Document mapping from legacy garage-branch C driver controls to new ESPHome entities
- [x] 4.2 Add migration and rollback steps for switching from legacy control path to esp-home control
- [x] 4.3 Add commissioning checklist for relay polarity, reboot safety, independent relay behavior, and watchdog controls

## 5. Validation

- [x] 5.1 Validate manual on/off operation for plug and light independently in Home Assistant
- [x] 5.2 Validate reboot behavior restores relay1 previous state and keeps relay2 off until explicit command
- [x] 5.3 Validate watchdog enabled mode turns light off after configured x minutes
- [x] 5.4 Validate watchdog disabled mode allows light runtime beyond x minutes without forced off
- [x] 5.5 Validate end-to-end behavior against user requirement: relay1 controls carport plug and relay2 controls carport light with optional watchdog protection for relay2
