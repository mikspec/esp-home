## 1. Configuration Surface

- [x] 1.1 Add a global weak-section pre-charge duration parameter to irrigation runtime configuration
- [x] 1.2 Add section-level weak/non-weak classification settings for all irrigation sections
- [x] 1.3 Define defaults and bounds for pre-charge duration and verify startup-safe behavior

## 2. Runtime State Machine

- [x] 2.1 Implement staged startup flow for weak sections: all sections off -> pump on -> delay X -> open weak section
- [x] 2.2 Ensure staged startup is applied for weak sections both from idle and during section-to-section transitions
- [x] 2.3 Implement latest-target-wins behavior when target selection changes during active pre-charge
- [x] 2.4 Preserve direct switching behavior for non-weak targets without forced pre-charge

## 3. Pump And Safety Semantics

- [x] 3.1 Update pump coupling logic so pump-off occurs only when no section is active and no pre-charge window is active
- [x] 3.2 Ensure all-off commands cancel pre-charge and force immediate safe-off for pump and sections
- [x] 3.3 Verify no additional notification pathways are introduced for pre-charge events

## 4. Home Assistant Integration And Validation

- [x] 4.1 Confirm existing schedule and manual control paths trigger identical weak-section behavior via device-level logic
- [x] 4.2 Validate weak-first startup scenario and non-weak-to-weak transition scenario on hardware
- [x] 4.3 Validate target switching during pre-charge and confirm stale pending targets are canceled
- [x] 4.4 Run regression checks for section interlock, runtime watchdog, and end-of-sequence all-off behavior
