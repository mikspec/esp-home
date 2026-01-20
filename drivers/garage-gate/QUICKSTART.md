# Quick Start Guide

## 5-Minute Setup

### Prerequisites
- ESP8266 board (NodeMCU v2 recommended)
- Micro USB cable
- Python 3.7+ installed
- WiFi network credentials

### Step 1: Install ESPHome (2 minutes)

```bash
# Install using pip
pip3 install esphome

# Verify installation
esphome version
```

### Step 2: Configure Secrets (1 minute)

```bash
# Copy template
cp ../config/secrets.yaml.template ../config/secrets.yaml

# Edit with your credentials (use any text editor)
nano secrets.yaml
```

Required values in `secrets.yaml`:
```yaml
wifi_ssid: "YourWiFiName"
wifi_password: "YourWiFiPassword"
fallback_password: "Setup123"  # At least 8 characters
api_encryption_key: "generate-with-command-below"
ota_password: "Update123"
```

Generate API key:
```bash
openssl rand -base64 32
```

### Step 3: Flash ESP8266 (2 minutes)

```bash
# Connect ESP8266 via USB
# Run esphome
esphome run garage_gate.yaml

# Select option: 1 (for USB serial port)
# Wait for compilation and upload
```

**Done!** Your garage gate controller is now running ESPHome.

---

## Adding to Home Assistant

### Option 1: Automatic Discovery (Easiest)

1. Go to **Settings** → **Devices & Services**
2. Look for "Discovered" section
3. Click **Configure** on "Garage Gate"
4. Enter your API encryption key from secrets.yaml
5. Click **Submit**

### Option 2: Manual Addition

1. Go to **Settings** → **Devices & Services**
2. Click **Add Integration**
3. Search for "ESPHome"
4. Enter IP address or hostname: `garage-gate.local`
5. Enter API encryption key
6. Click **Submit**

---

## Common Commands

### View Logs
```bash
esphome logs garage_gate.yaml
```

### Update OTA (Wireless)
```bash
esphome run garage_gate.yaml
# Select option: 2 (OTA)
```

### Compile Only (No Upload)
```bash
esphome compile garage_gate.yaml
```

### Clean Build Files
```bash
esphome clean garage_gate.yaml
```

---

## Troubleshooting Quick Fixes

### Can't Find Device
```bash
# Find IP address
ping garage-gate.local

# Or check router's DHCP leases
```

### WiFi Connection Failed
- Device creates fallback AP: "Garage Gate Fallback"
- Connect to it with password from secrets.yaml
- Configure WiFi through web interface at http://192.168.4.1

### Upload Failed
- Hold FLASH button on ESP8266 during upload
- Use different USB cable or port
- Try lower baud rate: `esphome run garage_gate.yaml --device /dev/ttyUSB0 --baud-rate 115200`

### Relay Inverted
Edit garage_gate.yaml:
```yaml
switch:
  - platform: gpio
    pin:
         number: GPIO5
      inverted: true  # Add this line
```

---

## Testing Components

### Test Relay
Open Home Assistant → Go to device → Toggle "Garage Gate Relay"
- Should hear relay click
- LED on relay module should light up

### Test Button
Press physical button → Check logs:
```bash
esphome logs garage_gate.yaml
# Should see: "Button pressed"
```

### Test Contactron
Move magnet near reed switch → Check logs:
```bash
# Should see: "Gate is CLOSED" or "Gate is OPEN"
```

---

## Pin Reference (NodeMCU v2)

```
D5 (GPIO14) -> Contactron
D2 (GPIO4) -> Button
D1 (GPIO5) -> Relay
```

---

## Safety Checklist

- [ ] Tested relay activation before connecting to motor
- [ ] Verified contactron detects gate position correctly
- [ ] Original emergency stop still works
- [ ] Gate safety sensors still functional
- [ ] Proper electrical isolation for relay
- [ ] Enclosure is weatherproof
- [ ] Device within WiFi range

---

## Next Steps After Setup

1. **Create Automation in Home Assistant:**
   - Auto-close gate at night
   - Notifications when gate opens
   - Reminders if gate left open

2. **Add More Sensors:**
   - Temperature sensor in garage
   - Motion detector
   - Door contact sensor

3. **Voice Control:**
   - "Alexa, open garage door"
   - "Hey Google, close garage door"

4. **Dashboard Card:**
   Add to Lovelace UI for quick access

---

## Support Resources

- 📖 Full documentation: `README.md`
- 🔌 Wiring guide: `WIRING.md`
- 🔄 Migration guide: `MIGRATION.md`
- 🌐 ESPHome docs: https://esphome.io
- 💬 Community: https://community.home-assistant.io

---

## Update Checklist

When updating configuration:

1. Edit `garage_gate.yaml`
2. Save changes
3. Run: `esphome run garage_gate.yaml`
4. Select OTA update
5. Wait for device to reboot
6. Verify in Home Assistant

---

**That's it! You now have a modern, maintainable garage gate controller.**
