# Phase Sensors — Camper Driver

Two commercial optocoupler-based phase sensors detect AC phase presence at two measurement points: the shore power input (inverter input) and the inverter output.

## GPIO Assignment

| Sensor                  | GPIO  | Mode              |
|-------------------------|-------|-------------------|
| Camper Shore Power      | GPIO21 | Digital input, internal pull-down |
| Camper Inverter Output  | GPIO22 | Digital input, internal pull-down |

## Wiring Diagram

```
Shore power (L)                     Inverter output (L)
      │                                     │
      │                                     │
┌─────┴──────┐                     ┌────────┴───────┐
│  Phase     │                     │  Phase         │
│  Sensor    │                     │  Sensor        │
│  Module 1  │                     │  Module 2      │
│ (opto)     │                     │ (opto)         │
│            │                     │                │
│  OUT ──────┼──── GPIO21          │  OUT ──────────┼──── GPIO22
│  GND ──────┼──── GND             │  GND ──────────┼──── GND
│  VCC ──────┼──── 3.3V            │  VCC ──────────┼──── 3.3V
└────────────┘                     └────────────────┘
        ESP32 dev board
        ┌───────────────┐
        │ GPIO21        │◄── Shore power sensor OUT
        │ GPIO22        │◄── Inverter output sensor OUT
        │ GND           │◄── Both sensor GND
        │ 3V3           │◄── Both sensor VCC
        └───────────────┘
```

**Signal polarity**: Sensors output LOW when phase is present (active-low). GPIO internal pull-up resistors are used with `inverted: true` in the ESPHome config.

## 4-State AC Source Logic

| Shore Power (GPIO21) | Inverter Output (GPIO22) | Meaning                        |
|----------------------|--------------------------|--------------------------------|
| ON                   | ON                       | Grid pass-through              |
| OFF                  | ON                       | Inverter generating (off-grid) |
| OFF                  | OFF                      | No AC available                |
| ON                   | OFF                      | ⚠ Unusual — check inverter     |