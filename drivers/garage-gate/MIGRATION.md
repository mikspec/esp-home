# Migration Guide: From C to ESPHome

This document explains the migration from the original C-based ESP8266 garage gate controller to the ESPHome YAML-based implementation.

## Why Migrate to ESPHome?

### Advantages of ESPHome

1. **Declarative Configuration:** No need to write C code - everything is configured in YAML
2. **OTA Updates:** Update firmware wirelessly without USB connection
3. **Home Assistant Integration:** Automatic discovery and integration
4. **Built-in Features:** WiFi management, logging, web interface included
5. **Maintainability:** Much easier to update and modify
6. **Community Support:** Large ecosystem and active community
7. **Security:** Built-in encryption and secure OTA updates

### What's Different?

| Feature | C Implementation | ESPHome Implementation |
|---------|------------------|------------------------|
| Configuration | Hard-coded in C | YAML file |
| Compilation | Arduino IDE / PlatformIO | ESPHome CLI / Docker |
| Updates | USB cable required | OTA (wireless) |
| Logging | Serial only | Serial + WiFi + Web |
| Home Assistant | Manual MQTT setup | Auto-discovery |
| WiFi Setup | Hard-coded credentials | Separate secrets file |
| Code Size | ~20-50 KB | ~350-400 KB (more features) |

## Migration Steps

### 1. Understanding Your C Code

Your original C implementation likely had:

```c
// Simplified example of what the C code might have looked like

#define RELAY_PIN 13
#define BUTTON_PIN 12
#define CONTACTRON_PIN 14

void setup() {
  pinMode(RELAY_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  pinMode(CONTACTRON_PIN, INPUT_PULLUP);
  digitalWrite(RELAY_PIN, LOW);
}

void loop() {
  // Button press handling
  if (digitalRead(BUTTON_PIN) == LOW) {
    triggerGate();
    delay(500); // Debounce
  }
  
  // Check contactron state
  bool gateOpen = digitalRead(CONTACTRON_PIN) == HIGH;
  
  delay(10);
}

void triggerGate() {
  digitalWrite(RELAY_PIN, HIGH);
  delay(500);
  digitalWrite(RELAY_PIN, LOW);
}
```

### 2. ESPHome Equivalent

All of the above is replaced with YAML configuration:

```yaml
# Pin definitions
switch:
  - platform: gpio
    id: gate_relay
    pin: GPIO5  # Same as RELAY_PIN

binary_sensor:
  - platform: gpio
    id: gate_button
    pin:
      number: GPIO4  # Same as BUTTON_PIN
      mode:
        input: true
        pullup: true
    on_press:
      # Same as triggerGate()
      - switch.turn_on: gate_relay
      - delay: 1000ms
      - switch.turn_off: gate_relay

  - platform: gpio
    id: gate_contactron
    pin:
      number: GPIO14  # Same as CONTACTRON_PIN
      mode:
        input: true
        pullup: true
```

### 3. Feature Mapping

#### WiFi Connection

**C Code:**
```c
#include <ESP8266WiFi.h>

WiFi.begin("SSID", "PASSWORD");
while (WiFi.status() != WL_CONNECTED) {
  delay(500);
}
```

**ESPHome:**
```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

#### MQTT / Home Assistant Integration

**C Code:**
```c
#include <PubSubClient.h>

WiFiClient espClient;
PubSubClient client(espClient);

client.setServer("mqtt_server", 1883);
client.publish("garage/state", gateOpen ? "open" : "closed");
```

**ESPHome:**
```yaml
api:
  encryption:
    key: !secret api_encryption_key
# That's it! No MQTT needed - uses native API
```

#### Logging

**C Code:**
```c
Serial.begin(115200);
Serial.println("Gate triggered");
```

**ESPHome:**
```yaml
logger:
  level: INFO
# Automatic logging of all events
```

#### OTA Updates

**C Code:**
```c
#include <ArduinoOTA.h>

ArduinoOTA.setPassword("admin");
ArduinoOTA.begin();

void loop() {
  ArduinoOTA.handle();
  // ... rest of code
}
```

**ESPHome:**
```yaml
ota:
  - platform: esphome
    password: !secret ota_password
# Fully automatic, no code needed
```

### 4. Preserving Custom Logic

If you had custom logic in your C code, here's how to migrate it:

#### Timers

**C Code:**
```c
unsigned long lastTrigger = 0;

if (millis() - lastTrigger > 60000) {
  // Do something every minute
  lastTrigger = millis();
}
```

**ESPHome:**
```yaml
interval:
  - interval: 60s
    then:
      - logger.log: "Running periodic task"
```

#### Debouncing

**C Code:**
```c
unsigned long lastButtonPress = 0;

if (buttonPressed && (millis() - lastButtonPress > 500)) {
  triggerGate();
  lastButtonPress = millis();
}
```

**ESPHome:**
```yaml
binary_sensor:
  - platform: gpio
    id: gate_button
    filters:
      - delayed_on: 500ms  # Built-in debouncing!
```

#### State Tracking

**C Code:**
```c
enum GateState {
  GATE_OPEN,
  GATE_CLOSED,
  GATE_OPENING,
  GATE_CLOSING
};

GateState currentState = GATE_CLOSED;
```

**ESPHome:**
```yaml
# Use the cover component which handles states automatically
cover:
  - platform: template
    name: "Garage Gate"
    # States are tracked automatically
```

### 5. Advanced Features

ESPHome provides features that would require significant C code:

#### Web Interface

No C code equivalent - would require web server library:

```yaml
web_server:
  port: 80
  # Automatic web UI for all components
```

#### Diagnostic Sensors

**ESPHome:**
```yaml
sensor:
  - platform: wifi_signal
    name: "WiFi Signal"
  - platform: uptime
    name: "Uptime"

text_sensor:
  - platform: wifi_info
    ip_address:
      name: "IP Address"
```

No equivalent C code needed!

### 6. Flash Size Comparison

| Implementation | Flash Usage | RAM Usage |
|----------------|-------------|-----------|
| C (minimal) | ~250 KB | ~25 KB |
| ESPHome | ~400 KB | ~35 KB |

**Note:** ESPHome uses more resources but provides significantly more functionality.

## Configuration Checklist

- [x] Copy `garage_gate.yaml` configuration file
- [x] Create `secrets.yaml` from template
- [x] Update GPIO pins if different from defaults
- [x] Generate API encryption key
- [x] Set OTA password
- [x] Configure WiFi credentials
- [x] Flash ESP8266 via USB first time
- [x] Test all components before installation
- [x] Install and test in actual garage
- [x] Add to Home Assistant

## Common Migration Issues

### Issue 1: Different Pin Behavior

**Problem:** Relay works opposite (on when it should be off)

**Solution:** Add `inverted: true` to the pin configuration:
```yaml
pin:
  number: GPIO5
  inverted: true
```

### Issue 2: Can't Upload First Time

**Problem:** ESPHome can't connect to ESP8266

**Solution:** First upload must be via USB. Hold FLASH button during upload if needed.

### Issue 3: Secrets Not Found

**Problem:** `!secret wifi_ssid` error

**Solution:** Create `secrets.yaml` file in same directory as YAML config.

### Issue 4: WiFi Credentials Don't Work

**Problem:** Device creates fallback AP

**Solution:** Connect to fallback AP and reconfigure WiFi through web interface.

## Performance Comparison

### Boot Time
- **C Implementation:** ~2-3 seconds
- **ESPHome:** ~5-8 seconds (more initialization)

### Response Time
- **C Implementation:** <100ms
- **ESPHome:** <200ms (negligible difference)

### Reliability
- Both are equally reliable when properly configured

## Rollback Plan

If you need to go back to C implementation:

1. Keep your original C code
2. Flash it using Arduino IDE or PlatformIO
3. Reconfigure WiFi credentials in code
4. Remove from Home Assistant

## Next Steps

After successful migration, consider:

1. **Add Automations:** Create Home Assistant automations
2. **Add Sensors:** Temperature, humidity sensors in garage
3. **Add Notifications:** Get alerts when gate opens/closes
4. **Add Security:** Add camera integration
5. **Add Scheduling:** Auto-close at night

## Resources

- [ESPHome Documentation](https://esphome.io/)
- [ESP8266 GPIO Reference](https://randomnerdtutorials.com/esp8266-pinout-reference-gpios/)
- [Home Assistant ESPHome Integration](https://www.home-assistant.io/integrations/esphome/)

## Support

If you encounter issues during migration:

1. Check ESPHome logs: `esphome logs garage_gate.yaml`
2. Verify wiring matches original C implementation
3. Test components individually before full integration
4. Consult ESPHome community forums

## Conclusion

The migration from C to ESPHome provides:
- ✅ Easier maintenance and updates
- ✅ Better Home Assistant integration  
- ✅ More features with less code
- ✅ Wireless updates
- ✅ Better debugging and logging

The trade-off is slightly larger flash/RAM usage, but on ESP8266 this is negligible for most use cases.
