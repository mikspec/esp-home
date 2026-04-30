# ESPHome Drivers Collection

A collection of ESPHome configurations for various home automation devices. Each driver is self-contained with its own documentation.

## Implemented Drivers

### 🏠 Garage Gate Controller
Complete ESPHome implementation for ESP8266-based garage gate controller with Home Assistant integration.

**Features:**
- Physical button control with long/short press actions
- Contactron position sensing
- Auto-close functionality
- Anti-replay protection for remote commands
- Runtime configuration via Home Assistant
- Cover entity integration

**Documentation:** [drivers/garage-gate/README.md](drivers/garage-gate/README.md)

### 💡 Carport Relay Controller
ESPHome implementation for ESP8266-based carport relay control.

**Features:**
- Relay1 control for 230V carport plug
- Relay2 control for carport light
- Power recovery policy by relay role (plug restore last state, light boot off)
- Optional light watchdog with configurable max-on minutes and enable/disable toggle
- Home Assistant package script wrappers for standard operations

**Documentation:** [drivers/carport/README.md](drivers/carport/README.md)

### 🚿 Irrigation Controller
ESPHome implementation for ESP8266-based irrigation relay control.

**Features:**
- Fixed relay mapping for pump and irrigation sections
- Single-active-section behavior with interlock protection
- Pump coupling to active sections with handoff-safe behavior
- Optional hot-period second daily run via Home Assistant package controls
- Configurable runtime safety watchdog with forced safe-off and phone notifications

 **ESPHome Config Reference:** [config/irrigation.yaml](config/irrigation.yaml)
 **Home Assistant Package Reference:** [config/packages/irrigation_ha.yaml](config/packages/irrigation_ha.yaml)

**Quick Start:**
```bash
# Start ESPHome dashboard
docker-compose up -d

# Access at http://localhost:6052
# Upload one of: garage_gate.yaml, carport.yaml, or irrigation.yaml from config/
```

## Project Structure

```
esp-home/
├── compose.yaml              # Docker Compose for ESPHome
├── LICENSE                   # Project license
├── README.md                 # This file
├── config/                   # ESPHome configurations
│   ├── carport.yaml          # Carport ESPHome config
│   ├── garage_gate.yaml      # Garage gate ESPHome config
│   ├── irrigation.yaml       # Irrigation ESPHome config
│   ├── home-assistant-examples.yaml  # HA automations & scripts
│   ├── packages/carport_ha.yaml  # Carport HA package
│   ├── packages/garage_gate.yaml  # Garage gate HA package
│   ├── packages/irrigation_ha.yaml  # Irrigation HA package
│   └── secrets.yaml.template # Secrets template
└── drivers/                  # Driver documentation
    ├── carport/              # Carport relay documentation
    │   └── README.md
    ├── irrigation/           # Irrigation legacy driver sources
    │   └── irrigationEsp/
    └── garage-gate/          # Garage gate documentation
        ├── gateEsp/          # Original C implementation
        │   ├── gate/         # Arduino sketch
        │   └── python/       # Python utilities
        ├── INDEX.md
        ├── MIGRATION.md
        ├── QUICKSTART.md
        ├── README.md
        ├── TROUBLESHOOTING.md
        └── WIRING.md
```

## Getting Started

### Prerequisites
- Docker and Docker Compose
- ESP8266/NodeMCU board
- Home Assistant (optional, for full integration)

### Setup
1. Clone this repository
2. Copy `config/secrets.yaml.template` to `config/secrets.yaml`
3. Edit `config/secrets.yaml` with your WiFi credentials and API keys
4. Start ESPHome: `docker-compose up -d`
5. Access dashboard at http://localhost:6052
6. Upload the desired configuration

### Development
```bash
# View logs
docker-compose logs -f esphome

# Stop service
docker-compose down

# Update ESPHome image
docker-compose pull
```

## Contributing

1. Create a new directory under `drivers/` for your device
2. Add ESPHome configuration to `config/`
3. Document wiring, setup, and features
4. Update this README with your new driver
5. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
