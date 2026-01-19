# Wiring Guide for Garage Gate Controller

## Bill of Materials

### Electronic Components
- 1x ESP8266 NodeMCU v2 (or compatible board)
- 1x 5V Relay Module (optocoupler isolated recommended)
- 1x Reed switch (contactron/magnetic sensor)
- 1x Push button (normally open)
- 1x Magnet (for reed switch)
- Jumper wires
- Micro USB cable for programming
- Power supply (5V 1A minimum)

### Optional Components
- Enclosure/case for electronics
- Terminal blocks for secure connections
- Pull-down resistors (if not using internal pull-ups)

## Detailed Wiring Instructions

### NodeMCU v2 Pin Layout

```
                     +-----------------+
                     |    NodeMCU v2   |
                     |                 |
                     |  [USB]          |
                     |                 |
              3.3V --|RST          TX |-- GPIO1
              GND  --|A0           RX |-- GPIO3
Contactron ---|  --|D5 (GPIO14) D1  |-- GPIO5
              |  --|D6 (GPIO12) D2  |-- GPIO4
              |  --|D7 (GPIO13) D3  |-- GPIO0
Button -------|  --|D8 (GPIO15) D4  |-- GPIO2 (LED)
Relay --------|  --|3V3         GND |-- GND
              +-----|5V          5V  |-------+-- 5V Power
                     |                 |
                     +-----------------+
```

### Connection Details

#### 1. Contactron (Reed Switch) Wiring
```
Contactron Pin 1 -----> GPIO14 (D5)
Contactron Pin 2 -----> GND

Note: Internal pull-up resistor is enabled in software
      Reed switch closes when magnet is near (gate closed)
      Reed switch opens when magnet is far (gate open)
```

#### 2. Button Wiring
```
Button Pin 1 -----> GPIO12 (D6)
Button Pin 2 -----> GND

Note: Internal pull-up resistor is enabled in software
      Button is normally open, closes when pressed
```

#### 3. Relay Module Wiring

**Control Connections:**
```
Relay VCC -----> 5V (from NodeMCU 5V pin or external 5V)
Relay GND -----> GND
Relay IN  -----> GPIO13 (D7)
```

**Load Connections (Motor Side):**
```
Relay COM (Common) -----> Garage Door Motor Terminal 1
Relay NO (Normally Open) -> Garage Door Motor Terminal 2

Important: Verify your garage door motor wiring!
Most garage doors use a momentary contact between two terminals.
```

#### 4. Power Supply

**Option A: USB Power**
```
5V USB Power Supply -----> NodeMCU Micro USB Port
```

**Option B: External 5V Supply**
```
5V Power Supply + -----> NodeMCU VIN (or 5V pin)
5V Power Supply - -----> NodeMCU GND
```

**Important:** Ensure adequate current capacity (minimum 500mA, recommended 1A+)

## Physical Installation

### 1. Mounting the Contactron

The contactron (reed switch) detects when the gate is closed:

```
[Garage Gate]
     |
     |  <- Magnet attached to moving gate
     |
[Reed Switch] <- Fixed to garage door frame
```

**Installation Steps:**
1. Mount the reed switch on the door frame (fixed position)
2. Mount the magnet on the moving gate door
3. Align so they are very close when gate is fully closed
4. Test alignment by moving gate and checking sensor readings

**Proper Alignment:**
- Distance when aligned: < 10mm for reliable operation
- The reed switch will close when magnet is near
- The reed switch will open when magnet moves away

### 2. Mounting the Button

Install the button in an accessible location:
- Inside the garage for convenient manual operation
- Protected from weather if mounted outside
- At a comfortable height (around 1.2-1.5m)

### 3. Mounting the Electronics

**Safety First:**
1. Use a weatherproof enclosure if installed in garage
2. Ensure proper ventilation to prevent overheating
3. Mount securely to prevent vibration damage
4. Keep away from moisture and extreme temperatures

**Recommended Location:**
- Near the garage door motor unit
- Within WiFi range
- Protected from direct sunlight and rain
- Accessible for maintenance

### 4. Connecting to Gate Motor

**Before connecting:**
1. Identify the correct motor terminals (usually labeled "WALL" or "BUTTON")
2. Disconnect existing wall button temporarily
3. Verify voltage (should be low voltage, typically 12-24V)

**Connection:**
1. Connect relay COM to one motor terminal
2. Connect relay NO to the other motor terminal
3. You can wire the original button in parallel with the relay

**Parallel Wiring Diagram:**
```
Motor Terminal 1 ----+-----> Relay COM
                     |
                     +-----> Original Button Pin 1

Motor Terminal 2 ----+-----> Relay NO
                     |
                     +-----> Original Button Pin 2
```

This allows both the ESP8266 and the original button to work.

## Testing Procedure

### 1. Initial Bench Test (Before Installation)

**Step 1:** Upload configuration to ESP8266
**Step 2:** Power on and verify WiFi connection
**Step 3:** Test relay by toggling switch in Home Assistant
**Step 4:** Test button - should trigger relay pulse
**Step 5:** Test contactron with magnet - should report state changes

### 2. Installation Test

**Step 1:** Connect everything EXCEPT motor
**Step 2:** Verify all sensors and button work
**Step 3:** Connect to motor (with gate disconnected from motor if possible)
**Step 4:** Test relay activation (listen for relay click)
**Step 5:** Reconnect motor to gate
**Step 6:** Test full operation with gate movement

### 3. Safety Test

**Important Safety Checks:**
- ✓ Emergency stop button still works
- ✓ Safety sensors (if any) still function
- ✓ Gate reverses on obstruction
- ✓ Manual disconnect still works
- ✓ Relay returns to OFF state after power cycle

## Troubleshooting

### Relay Doesn't Click
- Check 5V power supply to relay module
- Verify GPIO13 connection
- Test relay manually by connecting IN to GND

### Contactron Always Shows Same State
- Check magnet proximity (should be < 10mm when closed)
- Verify reed switch is working (test with multimeter)
- Try inverting the sensor in YAML config

### Button Doesn't Trigger
- Check GND connection
- Verify button is normally open type
- Test button with multimeter in continuity mode

### Gate Doesn't Respond
- Verify motor terminal connections
- Check if relay is rated for motor voltage/current
- Test with original wall button to confirm motor works

## Alternative ESP8266 Boards

### ESP-01 Module

For minimal installations, use `garage_gate_simple.yaml`:

```
Pin Assignments:
GPIO0 -> Contactron
GPIO2 -> Button  
GPIO3 -> Relay (RX pin - disables serial logging!)

Note: ESP-01 has limited pins. Not recommended for beginners.
```

### D1 Mini

```
Pin Assignments:
D1 (GPIO5)  -> Contactron
D2 (GPIO4)  -> Button
D5 (GPIO14) -> Relay

Note: D1 Mini is compact and breadboard-friendly
```

## Security Considerations

1. **WiFi Security:** Use WPA2 encryption on your WiFi network
2. **API Encryption:** Always enable API encryption key
3. **OTA Password:** Set a strong OTA password
4. **Physical Security:** Secure the enclosure to prevent tampering
5. **Network Isolation:** Consider a separate VLAN for IoT devices

## Maintenance

- **Weekly:** Check WiFi connection and responsiveness
- **Monthly:** Verify contactron alignment
- **Quarterly:** Check relay for wear (normally closed relays wear out)
- **Annually:** Replace relay module if showing signs of degradation

## Support

For additional help:
- Refer to ESPHome documentation: https://esphome.io/
- Check garage door opener manual for terminal identification
- Consult with an electrician for safety concerns
