# Troubleshooting Guide

## Table of Contents
1. [Installation Issues](#installation-issues)
2. [WiFi Connection Issues](#wifi-connection-issues)
3. [Hardware Issues](#hardware-issues)
4. [Home Assistant Integration Issues](#home-assistant-integration-issues)
5. [Performance Issues](#performance-issues)
6. [Advanced Diagnostics](#advanced-diagnostics)

---

## Installation Issues

### ESPHome Won't Install

**Error: `pip: command not found`**
```bash
# Install pip first
sudo apt-get update
sudo apt-get install python3-pip

# Then install ESPHome
pip3 install esphome
```

**Error: `Permission denied`**
```bash
# Use --user flag
pip3 install --user esphome

# Or use sudo (not recommended)
sudo pip3 install esphome
```

**Error: `No module named 'esphome'`**
```bash
# Check Python path
python3 -m esphome version

# Or add to PATH
export PATH=$PATH:~/.local/bin
```

### Upload Fails - Can't Find Serial Port

**Linux:**
```bash
# List available ports
ls /dev/ttyUSB* /dev/ttyACM*

# Add user to dialout group
sudo usermod -a -G dialout $USER
# Log out and back in

# Grant permissions
sudo chmod 666 /dev/ttyUSB0
```

**macOS:**
```bash
# List ports
ls /dev/cu.*

# Install CH340 driver if needed (for cheap NodeMCU clones)
# Download from: https://github.com/adrianmihalko/ch340g-ch34g-ch34x-mac-os-x-driver
```

**Windows:**
```bash
# Install CP210x or CH340 driver
# Check Device Manager for COM port number
# Use: --device COM3 (replace with your COM port)
```

### Upload Hangs at "Connecting..."

**Solution 1: Hold FLASH button**
```
1. Start upload process
2. When you see "Connecting...", hold FLASH button
3. Release when upload starts
```

**Solution 2: Lower baud rate**
```bash
esphome run garage_gate.yaml --device /dev/ttyUSB0 --baud-rate 115200
```

**Solution 3: Try esptool directly**
```bash
pip install esptool
esptool.py --port /dev/ttyUSB0 write_flash 0x0 .esphome/build/garage-gate/.pioenvs/garage-gate/firmware.bin
```

---

## WiFi Connection Issues

### Device Creates Fallback AP

**Symptoms:**
- New WiFi network appears: "Garage Gate Fallback"
- Device doesn't connect to your WiFi

**Solution:**
```
1. Connect to "Garage Gate Fallback" WiFi
2. Use password from secrets.yaml (fallback_password)
3. Navigate to: http://192.168.4.1
4. Enter correct WiFi credentials
5. Click Save
```

### WiFi Credentials Are Correct But Won't Connect

**Check SSID:**
```yaml
# Common issue: Hidden SSID
wifi:
  ssid: "YourSSID"
  password: "YourPassword"
  fast_connect: true  # Add this for hidden networks
```

**Check WiFi Band:**
```
ESP8266 only supports 2.4GHz WiFi
- Ensure router has 2.4GHz enabled
- Check if SSID is on 2.4GHz band (not 5GHz)
```

**Check Special Characters:**
```yaml
# Use quotes for passwords with special characters
wifi_password: "P@ssw0rd!123"
```

### Frequent Disconnections

**Increase WiFi Power:**
```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  power_save_mode: none  # Disable power saving
  output_power: 20.0     # Maximum power (20.5 dBm)
```

**Add Static IP:**
```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  manual_ip:
    static_ip: 192.168.1.100
    gateway: 192.168.1.1
    subnet: 255.255.255.0
    dns1: 192.168.1.1
```

**Poor Signal Strength:**
```bash
# Check WiFi signal from logs
esphome logs garage_gate.yaml

# Look for WiFi signal strength (should be > -70 dBm)
# If too weak:
# - Move router closer
# - Use WiFi extender
# - Add external antenna to ESP8266
```

---

## Hardware Issues

### Relay Doesn't Click

**Check Power:**
```
Relay Module VCC -> 5V (not 3.3V)
Relay Module GND -> GND
```

**Check GPIO:**
```yaml
# Verify correct pin number
switch:
  - platform: gpio
    pin: GPIO13  # D7 on NodeMCU
```

**Test Relay Manually:**
```bash
# Connect IN to GND - relay should click
# If it does, GPIO is the issue
# If it doesn't, relay module is faulty
```

**Try Inverted Logic:**
```yaml
switch:
  - platform: gpio
    pin:
      number: GPIO13
      inverted: true  # Some relays are active-low
```

### Relay Works Backwards (On When Should Be Off)

**Solution:**
```yaml
switch:
  - platform: gpio
    pin:
      number: GPIO13
      inverted: true  # Flip the logic
```

### Button Doesn't Respond

**Check Pull-up:**
```yaml
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO12
      mode:
        input: true
        pullup: true  # Enable internal pull-up
```

**Check Inverted:**
```yaml
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO12
      inverted: true  # Active-low button
```

**Test Button:**
```bash
# View logs while pressing button
esphome logs garage_gate.yaml

# Should see state changes
# If no changes: wiring issue
# If constant changes: add debouncing
```

**Increase Debouncing:**
```yaml
binary_sensor:
  - platform: gpio
    filters:
      - delayed_on: 100ms   # Increase these values
      - delayed_off: 100ms
```

### Contactron Shows Wrong State

**Symptoms:**
- Shows "open" when gate is closed
- Shows "closed" when gate is open

**Solution 1: Invert Logic**
```yaml
binary_sensor:
  - platform: gpio
    pin:
      inverted: false  # Change to true or vice versa
```

**Solution 2: Swap Lambda in Cover**
```yaml
cover:
  - platform: template
    lambda: |-
      if (id(gate_contactron).state) {
        return COVER_CLOSED;  # Swap OPEN and CLOSED
      } else {
        return COVER_OPEN;
      }
```

**Test Contactron:**
```
1. View logs: esphome logs garage_gate.yaml
2. Move magnet near/away from reed switch
3. Should see state changes
4. If no changes: check wiring or faulty sensor
```

### Contactron Always Shows Same State

**Check Magnet Proximity:**
```
- Magnet should be within 10mm when active
- Try stronger magnet
- Ensure correct alignment
```

**Test Reed Switch:**
```bash
# Use multimeter in continuity mode
# Should beep when magnet is near
# No beep when magnet is far
```

---

## Home Assistant Integration Issues

### Device Not Auto-Discovered

**Manual Addition:**
```
1. Settings -> Devices & Services
2. Add Integration -> ESPHome
3. Enter: garage-gate.local
4. Enter API encryption key
```

**Check mDNS:**
```bash
# Ping by hostname
ping garage-gate.local

# If fails, use IP address instead
ping 192.168.1.xxx
```

**Find IP Address:**
```bash
# From logs
esphome logs garage_gate.yaml
# Look for "WiFi connected" message with IP

# Or from router DHCP leases
# Or use network scanner
```

### "Invalid Encryption Key" Error

**Symptoms:**
- Can't connect to device
- "Authentication failed" error

**Solution:**
```
1. Check encryption key in secrets.yaml
2. Ensure no extra spaces/quotes
3. Generate new key: openssl rand -base64 32
4. Update both secrets.yaml and Home Assistant
```

### Entities Not Showing Up

**Check Configuration:**
```yaml
# Ensure entities have names
switch:
  - platform: gpio
    id: gate_relay
    name: "Garage Gate Relay"  # This is required for HA
```

**Check ESPHome Logs:**
```bash
esphome logs garage_gate.yaml
# Look for connection to Home Assistant
# Should see: "Client connected"
```

**Reload Integration:**
```
1. Settings -> Devices & Services
2. Find ESPHome integration
3. Click "..." -> Reload
```

### Cover Shows "Unknown" State

**Check Contactron:**
```yaml
# Ensure contactron is returning state
binary_sensor:
  - platform: gpio
    id: gate_contactron
    # Must have ID for lambda reference
```

**Add Logging:**
```yaml
cover:
  - platform: template
    lambda: |-
      if (id(gate_contactron).state) {
        ESP_LOGD("cover", "Contactron is ON - Gate OPEN");
        return COVER_OPEN;
      } else {
        ESP_LOGD("cover", "Contactron is OFF - Gate CLOSED");
        return COVER_CLOSED;
      }
```

---

## Performance Issues

### Slow Response Time

**Reduce Logger Level:**
```yaml
logger:
  level: WARN  # Change from DEBUG/INFO
```

**Disable Web Server:**
```yaml
# Comment out if not needed
# web_server:
#   port: 80
```

**Optimize WiFi:**
```yaml
wifi:
  fast_connect: true
  power_save_mode: none
```

### Random Reboots

**Check Power Supply:**
```
- Use quality 5V 1A+ power supply
- Avoid USB ports (often under-powered)
- Check voltage with multimeter (should be 4.75-5.25V)
```

**Add Capacitor:**
```
- 100-470µF capacitor between 5V and GND
- Place close to ESP8266
- Helps with voltage spikes during WiFi transmission
```

**Enable Watchdog:**
```yaml
# ESPHome has built-in watchdog
# If not working, check power supply first
```

### Memory Issues

**Symptoms:**
- Random crashes
- Failed uploads
- "Out of memory" errors

**Reduce Memory Usage:**
```yaml
logger:
  level: WARN
  baud_rate: 0  # Disable serial logging

# Remove unused features
# web_server:
#   port: 80
```

**Check Memory:**
```yaml
# Add debug sensor
sensor:
  - platform: template
    name: "Free Heap"
    lambda: return ESP.getFreeHeap();
```

---

## Advanced Diagnostics

### Enable Verbose Logging

```yaml
logger:
  level: VERBOSE
  logs:
    wifi: VERBOSE
    api: VERBOSE
    sensor: DEBUG
```

### Monitor Boot Process

```bash
# Watch serial output during boot
esphome logs garage_gate.yaml --device /dev/ttyUSB0
```

### Check Component Status

```yaml
# Add status sensor
binary_sensor:
  - platform: status
    name: "Garage Gate Status"
```

### Validate Configuration

```bash
# Check for errors without uploading
esphome config garage_gate.yaml
```

### Clean Build

```bash
# Remove cached build files
esphome clean garage_gate.yaml

# Force full recompile
esphome compile garage_gate.yaml
```

### Export Compiled Firmware

```bash
# Compile and export binary
esphome compile garage_gate.yaml

# Binary location:
# .esphome/build/garage-gate/.pioenvs/garage-gate/firmware.bin
```

### Use Different Upload Method

```bash
# Use esptool.py directly
esptool.py --port /dev/ttyUSB0 write_flash 0x0 firmware.bin

# Use OTA with specific IP
esphome run garage_gate.yaml --device 192.168.1.100
```

---

## Getting Help

If issues persist:

1. **Collect Logs:**
```bash
esphome logs garage_gate.yaml > garage_logs.txt
```

2. **Check Configuration:**
```bash
esphome config garage_gate.yaml
```

3. **Test Components Individually:**
- Test relay only
- Test button only
- Test contactron only

4. **Ask for Help:**
- ESPHome Discord: https://discord.gg/KhAMKrd
- Home Assistant Forum: https://community.home-assistant.io
- GitHub Issues: Include logs and configuration

5. **Provide Information:**
- ESP8266 board model
- ESPHome version
- Configuration file (remove secrets!)
- Logs showing the issue
- What you've already tried

---

## Emergency Recovery

### Factory Reset ESP8266

```bash
# Erase flash completely
esptool.py --port /dev/ttyUSB0 erase_flash

# Reflash firmware
esphome run garage_gate.yaml
```

### Restore Backup Configuration

```bash
# Keep backups of working configs
cp garage_gate.yaml garage_gate.yaml.backup

# Restore if needed
cp garage_gate.yaml.backup garage_gate.yaml
```

### Return to Original C Code

If you need to revert to C implementation:
1. Keep original C code safe
2. Flash using Arduino IDE
3. Reconfigure WiFi in code
4. Remove from Home Assistant

---

**Remember:** Most issues are wiring or configuration related. Double-check connections and review configuration carefully before assuming hardware failure.
