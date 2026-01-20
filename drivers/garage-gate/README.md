# ESP Home Implementation

ESPHome configurations for various ESP8266/ESP32 based home automation devices.

## Garage Gate Controller

This project provides an ESPHome implementation for controlling a garage gate using ESP8266. It replaces the previous C-based implementation with a declarative YAML configuration.

### Hardware Requirements

- **ESP8266 board** (e.g., NodeMCU v2)
- **1x Relay module** - Controls the garage gate motor
- **1x Push button** - Manual gate control
- **1x Contactron** (reed switch/magnetic sensor) - Detects gate position (open/closed)

### Wiring Diagram

#### Default GPIO Pin Assignments (NodeMCU v2)

| Component | GPIO Pin | NodeMCU Pin | Description |
|-----------|----------|-------------|-------------|
| Relay | GPIO5 | D1 | Controls gate motor |
| Button | GPIO4 | D2 | Manual trigger button |
| Contactron | GPIO14 | D5 | Gate position sensor |
| Status LED | GPIO2 | D4 (built-in) | Status indicator |

**Important:** Ensure proper electrical isolation when connecting the relay to the gate motor. Use optocoupler-based relay modules for safety.

#### Contactron Wiring
- One wire to GPIO14 (D5)
- Other wire to GND
- Internal pull-up resistor is enabled in software

#### Button Wiring
- One wire to GPIO4 (D2)
- Other wire to GND
- Internal pull-up resistor is enabled in software

#### Relay Wiring
- VCC to 5V or 3.3V (depending on relay module)
- GND to GND
- IN to GPIO5 (D1)

### Setup Instructions

#### 1. Install ESPHome

```bash
# Install ESPHome via pip
pip install esphome

# Or using Docker
docker pull esphome/esphome
```

#### 2. Configure Secrets

Copy the template file and add your credentials:

```bash
cp ../config/secrets.yaml.template ../config/secrets.yaml
```

Edit `secrets.yaml` with your actual values:
- WiFi SSID and password
- API encryption key (generate with: `openssl rand -base64 32`)
- OTA password
- Fallback AP password

#### 3. Compile and Upload

**First time upload (via USB):**
```bash
esphome run garage_gate.yaml
```

**Subsequent uploads (via OTA):**
```bash
esphome run garage_gate.yaml --device garage-gate.local
```

**Using Docker:**
```bash
docker run --rm -v "${PWD}":/config -it esphome/esphome run garage_gate.yaml
```

#### 4. Add to Home Assistant

Once the device is running, it will automatically be discovered by Home Assistant if you have the ESPHome integration installed.

1. Go to **Settings** → **Devices & Services**
2. Look for the discovered "Garage Gate" device
3. Click **Configure** and enter your API encryption key

### Features

#### Entities Exposed to Home Assistant

1. **Cover: Garage Gate**
   - Open/Close/Stop controls
   - Current state (open/closed)
   
2. **Binary Sensor: Garage Gate Position Sensor**
   - Shows if gate is open or closed
   
3. **Switch: Garage Gate Relay**
   - Direct relay control (for advanced automation)
   
4. **Sensor: Garage Gate WiFi Signal**
   - WiFi signal strength in dB
   
5. **Text Sensors:**
   - IP Address
   - Connected SSID
   - ESPHome Version

#### Automation Features

- **Button Control:** Long-press behavior based on configured delays (default: action 1 after 500ms, action 2 after 2000ms)
- **Smart Cover Control:** Prevents unnecessary operations (e.g., won't try to open an already open gate)
- **Debouncing:** Button and contactron inputs are debounced to prevent false triggers
- **Logging:** Comprehensive logging for debugging
- **Failsafe:** Relay defaults to OFF state on boot

#### Secure Triggering (Anti-Replay)

The ESPHome API includes a custom `gate_action` service that ignores duplicate `request_id` values, preventing replay of the same request.

Service parameters:
- `request_id` (int, must be unique per request)
- `action` (string: `open`, `close`, `stop`, `toggle`)

### Customization

#### Changing GPIO Pins

Edit `garage_gate.yaml` and modify the pin numbers in the respective sections:

```yaml
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO14  # Change this to your desired pin
```

#### Adjusting Relay Pulse Duration

Change the `gate_relay_pulse_ms` substitution in garage_gate.yaml:

```yaml
substitutions:
   gate_relay_pulse_ms: "1000"  # Adjust this duration (milliseconds)
```

#### Disabling Web Server

If you don't need the standalone web interface, comment out:

```yaml
# web_server:
#   port: 80
```

### Troubleshooting

#### Device Won't Connect to WiFi

1. Check if credentials in `secrets.yaml` are correct
2. Look for the fallback AP "Garage Gate Fallback"
3. Connect to it and reconfigure WiFi settings

#### Relay Not Working

1. Verify GPIO pin assignment
2. Check if relay needs inverted logic (set `inverted: true`)
3. Ensure proper power supply to relay module

#### Contactron Reporting Wrong State

- Try inverting the logic by changing `inverted: true` to `inverted: false` or vice versa

#### Enable Debug Logging

Change logger level to DEBUG:

```yaml
logger:
  level: DEBUG
```

### Safety Considerations

⚠️ **Important Safety Notes:**

1. Always disconnect power before wiring
2. Use proper electrical isolation for relay connections
3. Ensure the relay module can handle the motor current
4. Add physical safety mechanisms (e.g., emergency stop button)
5. Test thoroughly before permanent installation
6. Comply with local electrical codes

### License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

### Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

### Support

For issues and questions:
- Open an issue on GitHub
- Consult [ESPHome documentation](https://esphome.io/)

### Acknowledgments

- ESPHome project for the excellent framework
- Original C implementation that this replaces
