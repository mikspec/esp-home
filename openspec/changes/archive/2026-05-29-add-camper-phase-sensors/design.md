## Context

The camper ESP32 relay driver currently exposes four relay switches but has no awareness of AC power source state. Two commercial optocoupler-based phase sensor modules will be wired to the ESP32 to detect phase presence at two points: the inverter input (shore power) and the inverter output. The inverter operates in pass-through mode when grid is present, so measuring both points allows unambiguous determination of the current AC source.

Existing relay GPIO assignments are GPIO25, GPIO26, GPIO32, GPIO33. GPIO21 and GPIO22 are unused, adjacent on the header, and both support internal pull resistors — making them the natural choice for the two new digital inputs.

## Goals / Non-Goals

**Goals:**
- Add two `binary_sensor` GPIO entities to `config/camper.yaml` for shore power and inverter output phase detection.
- Assign GPIO21 to shore power sensor and GPIO22 to inverter output sensor.
- Expose stable, role-oriented Home Assistant entities: `Camper Shore Power` and `Camper Inverter Output`.
- Document wiring and pin rationale in driver documentation.

**Non-Goals:**
- Implementing any automation, control logic, or load management based on sensor state — deferred to a follow-up change.
- Modifying existing relay behavior or GPIO assignments.
- Voltage measurement or waveform analysis — binary presence detection only.

## Decisions

1. GPIO21 for shore power, GPIO22 for inverter output
   - Decision: Use GPIO21 (shore power) and GPIO22 (inverter output).
   - Rationale: Both are bidirectional with internal pull resistors, adjacent on the header, and not used by any existing relay or system function. GPIO34 (originally considered) is input-only with no internal pull resistor, requiring an external component.
   - Alternative considered: GPIO34 for one sensor. Rejected due to missing internal pull resistor.

2. `binary_sensor` platform with `gpio` component
   - Decision: Use ESPHome `binary_sensor` with `platform: gpio` and `device_class: power`.
   - Rationale: Phase presence is a discrete on/off signal from the optocoupler. No analog measurement is needed. `device_class: power` maps cleanly to HA power presence semantics.
   - Alternative considered: `sensor` with ADC. Rejected — optocoupler output is digital.

3. No automation logic in this change
   - Decision: Sensors are passive observers only; no relay interactions or automations added here.
   - Rationale: Keeps scope minimal and testable independently. Load management strategy (e.g., fridge behavior based on AC source) is a separate, more complex concern.

## Risks / Trade-offs

- [Optocoupler module signal polarity varies by vendor] → Mitigation: Document that `inverted:` flag in ESPHome config may need toggling depending on the specific module used; include verification step in commissioning checklist.
- [GPIO21 is used as I2C SDA on many ESP32 pinouts] → Mitigation: No I2C devices are present in this driver; no conflict. Noted for future reference if I2C sensors are ever added.
- [Phase sensor detects presence but not quality] → Mitigation: Acceptable for current scope; voltage/frequency measurement deferred to a future change if needed.

## Open Questions

- Confirm active-high vs active-low output polarity for the specific optocoupler module in use — determines whether `inverted: true` is needed in the config.
