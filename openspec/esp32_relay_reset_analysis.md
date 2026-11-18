# ESP32 Camper Relay Board Reset Analysis

## Problem Statement

ESP32 relay board (4 relays) resets consistently after a couple of minutes of operation. The board is powered by a LiFePO4 camper battery. The issue occurs on **both** power inputs:
- **5V DC input**
- **7-30V DC input** (via onboard linear regulator)

Logs gathered via OTA show stable operation during active monitoring, but resets occur when not actively watched.

---

## Hardware Configuration

| Component | Details |
|-----------|---------|
| Board | ESP32-DevKit style with 4-channel relay module |
| Power source | LiFePO4 battery (~12.8V nominal) |
| Relay 1 (GPIO25) | Camper Inverter — `ALWAYS_OFF` |
| Relay 2 (GPIO26) | Camper Fridge — `ALWAYS_ON` (inverted) |
| Relay 3 (GPIO33) | Camper N-G Bond — `ALWAYS_OFF` |
| Relay 4 (GPIO32) | Camper Spare Relay 2 — `ALWAYS_OFF` |
| Input 1 (GPIO21) | Shore Power detect (inverted, pull-up) |
| Input 2 (GPIO22) | Inverter Output detect (inverted, pull-up) |

### Key Software Behaviours
- **On boot**: 3 relays forced OFF; N-G bond auto-evaluation script runs
- **N-G Bond Auto**: Enabled via template switch; evaluates on input state changes
- **Inverter shutdown**: Safe-shutdown script with 5s delay before opening inverter relay
- **Input triggers**: Both binary sensors trigger `ng_bond_evaluate` script on every state change

---

## Critical Observations

### Observation 1: Reset occurs on BOTH 5V and 12V inputs
- **Ruling out**: Onboard 5V regulator thermal shutdown as the *sole* cause (would only affect 12V input)
- **Pointing to**: Shared power path issue, relay coil noise, or software watchdog

### Observation 2: OTA logs show stable operation during monitoring
- **Ruling out**: Pure software infinite loop or immediate crash
- **Pointing to**: Intermittent condition (noise, transient, thermal, or specific state combination)

### Observation 3: Reset timing is consistent (~couple of minutes)
- **Pointing to**: Thermal buildup, script accumulation, or a specific automation trigger sequence

---

## Root Cause Hypotheses (Ranked)

### 1. Relay Coil Ground Bounce / Power Rail Collapse (Most Likely)

Cheap 4-channel relay modules often share the relay coil ground with the logic ground. When relays energize/de-energize, inductive kickback or current surge causes ground bounce or 5V rail sag.

**Evidence:**
- Reset occurs regardless of input voltage (5V or 12V)
- Fridge relay is ON continuously (constant load + switching transients from other relays)
- `ng_bond_evaluate` can trigger relay state changes, causing inrush
- OTA logging may alter timing enough to mask the issue occasionally

**Confirmation Test:**
- Disconnect relay **loads** (AC/DC side) but keep control wires connected
- If resets stop → back-EMF from loads is coupling back
- If resets continue → relay coil current itself is collapsing the logic rail

**Fix:**
- **Remove JD-VCC jumper** on relay module
- Power relay coils from a **separate 5V supply** (or separate thick wires from the same buck converter) via JD-VCC and GND
- Power logic side (VCC/GND/INx) from ESP32's clean 5V rail
- Ensure common ground reference between both supplies

---

### 2. Noisy Input Pins Thrashing the Script (High Likelihood)

GPIO21 (Shore Power) and GPIO22 (Inverter Output) use internal pull-ups (~30-50kΩ) with long wires in an electrically noisy camper environment. EMI from inverters, compressors, and DC-DC converters can cause rapid false toggling.

The `ng_bond_evaluate` script uses `mode: restart`. If inputs toggle at high frequency, the script restarts constantly, consuming CPU and potentially starving the task watchdog.

**Evidence:**
- Camper environment is electrically noisy (inverters, motors, long wires)
- Internal pull-ups are weak for long wire runs
- Script restart mode is sensitive to rapid re-triggering
- Consistent "couple of minutes" timing matches gradual heap/CPU exhaustion

**Confirmation Test:**
- Add `delayed_on: 100ms` and `delayed_off: 100ms` filters to both inputs
- Monitor logs for input chatter before the filter is applied

**Fix:**
```yaml
binary_sensor:
  - platform: gpio
    id: camper_shore_power
    # ... existing config ...
    filters:
      - delayed_on: 100ms
      - delayed_off: 100ms
    on_state:
      - script.execute: ng_bond_evaluate

  - platform: gpio
    id: camper_inverter_output
    # ... existing config ...
    filters:
      - delayed_on: 100ms
      - delayed_off: 100ms
    on_state:
      - script.execute: ng_bond_evaluate
```

Additionally, add **external hardware pull-ups** (4.7kΩ–10kΩ) and **100nF ceramic capacitors** from each input pin to GND at the board for EMI suppression.

---

### 3. Inadequate 5V Supply / Wiring Voltage Drop (Likely)

The LiFePO4 battery is stable, but the 5V generation or wiring may not handle the combined load:
- ESP32 + WiFi: ~200mA average, 400mA+ peak
- 4 relay coils: ~70–90mA each = ~280–360mA total
- **Total peak load: 600–800mA**

If the buck converter is undersized, or wires are thin/long, voltage sags below the ESP32's brownout threshold (~2.7V on the 3.3V rail) when relays switch.

**Evidence:**
- Reset occurs on both 5V and 12V inputs (suggests the 5V rail itself is the weak point)
- Fridge relay is always ON (constant base load)
- Additional relay switching creates transient sag

**Confirmation Test:**
- Measure 5V rail at the ESP32 board with a multimeter during relay switching
- Look for dips below ~4.5V (which would cause 3.3V LDO dropout)

**Fix:**
- Use a **5V buck converter rated for 3A+** (not 1A or 2A)
- Use **short, thick wires** (18–20 AWG) from converter to board
- Add **1000µF–2200µF electrolytic capacitor** across 5V and GND at the board
- Add **100nF ceramic capacitor** in parallel for high-frequency noise

---

### 4. Back-EMF from Inductive Loads (Moderate Likelihood)

If the relay contacts switch inductive loads (fridge compressor, inverter transformer), the load-side back-EMF can arc across contacts and couple EMI back into the control circuitry.

**Evidence:**
- Fridge is a compressor (highly inductive)
- Inverter has a transformer (inductive)
- Reset timing correlates with relay switching events

**Confirmation Test:**
- Disconnect AC/load wires from relay contacts
- If resets stop, back-EMF is the cause

**Fix:**
- Add **RC snubbers** or **MOV varistors** across the relay contacts (load side)
- Ensure load wiring is separated from low-voltage control wiring
- Consider using relays with higher contact ratings or dedicated AC-rated SSRs for high-inrush loads

---

### 5. Onboard 3.3V Regulator Weakness (Less Likely but Possible)

Some cheap ESP32 boards use undersized 3.3V LDOs (e.g., XC6206 rated for 200mA) that cannot handle ESP32 WiFi peaks + relay noise transients.

**Evidence:**
- OTA logs (which keep WiFi active) seem stable, which argues against this
- But combined with relay noise, the regulator may drop out

**Fix:**
- If isolating relay power and adding capacitors does not resolve, consider replacing the board with a known-quality ESP32-DevKitC v4 or WROOM-32D module with a proper AMS1117-3.3 regulator.

---

## Diagnostic Action Plan

### Step 1: Capture Reset Reason (Priority: CRITICAL)
Start OTA logs, let the system reset, then reconnect immediately and record the **first boot message**.

| Message | Interpretation |
|---------|---------------|
| `Brownout detector was triggered` | Power rail sag — focus on supply, wiring, capacitors, relay isolation |
| `rst:0x8 (TG1WDT_SYS_RESET)` | CPU watchdog — focus on noisy inputs thrashing scripts |
| `rst:0x10 (RTCWDT_RTC_RESET)` | RTC watchdog — same as above, or deep sleep issue |
| `rst:0x1 (POWERON_RESET)` | Hard power loss — regulator thermal, loose connection, or severe brownout |

### Step 2: Input Filtering (Priority: HIGH)
Add 100ms software debounce filters to GPIO21 and GPIO22 immediately. This is a zero-cost, high-impact fix.

### Step 3: Relay Power Isolation (Priority: HIGH)
- Remove JD-VCC jumper
- Separate relay coil power from logic power
- Re-test with both 5V and 12V inputs

### Step 4: Load Disconnection Test (Priority: MEDIUM)
- Disconnect AC wires from relay contacts
- If stable, add snubbers/varistors to load side

### Step 5: Power Rail Measurement (Priority: MEDIUM)
- Monitor 5V rail with multimeter during relay switching
- Verify buck converter is rated for 3A+ continuous

### Step 6: Hardware Pull-ups (Priority: LOW)
- Add 4.7kΩ external pull-ups to GPIO21/22
- Add 100nF ceramic caps from pins to GND

---

## Recommended Config Changes

```yaml
# Add to existing config

esp32:
  board: esp32dev
  framework:
    type: esp-idf
    advanced:
      minimum_chip_revision: "3.1"
      sram1_as_iram: true

binary_sensor:
  - platform: gpio
    id: camper_shore_power
    name: "Camper Shore Power"
    pin:
      number: GPIO21
      inverted: true
      mode:
        input: true
        pullup: true
    device_class: power
    filters:
      - delayed_on: 100ms
      - delayed_off: 100ms
    on_state:
      - script.execute: ng_bond_evaluate

  - platform: gpio
    id: camper_inverter_output
    name: "Camper Inverter Output"
    pin:
      number: GPIO22
      inverted: true
      mode:
        input: true
        pullup: true
    device_class: power
    filters:
      - delayed_on: 100ms
      - delayed_off: 100ms
    on_state:
      - script.execute: ng_bond_evaluate
```

**Hardware additions:**
- **1000µF–2200µF electrolytic capacitor** across 5V and GND at the ESP32 board
- **100nF ceramic capacitor** in parallel with the above
- **External 4.7kΩ pull-ups** on GPIO21 and GPIO22 (if long wire runs)
- **100nF ceramic capacitors** from GPIO21 and GPIO22 to GND at the board

---

## Current Assessment

> **Most likely cause:** Relay coil current and/or ground bounce from the 4-channel relay module is collapsing the ESP32's logic power rail or triggering brownout. The fact that resets occur on both 5V and 12V inputs, combined with OTA stability (which may reflect timing differences), points to a **shared power/ground path issue** rather than a pure software bug.
>
> **Recommended immediate actions:**
> 1. Add input debounce filters (100ms) to GPIO21/22
> 2. Isolate relay coil power via JD-VCC removal
> 3. Add 1000µF+ capacitor across 5V/GND
> 4. Capture the reset reason from boot logs

---

*Analysis generated for ESPHome camper configuration.*
*Date: 2026-08-11*
