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

**Quick Start:**
```bash
# Start ESPHome dashboard
docker-compose up -d

# Access at http://localhost:6052
# Upload garage_gate.yaml from config/ directory
```

## Project Structure

```
esp-home/
├── compose.yaml              # Docker Compose for ESPHome
├── LICENSE                   # Project license
├── README.md                 # This file
├── config/                   # ESPHome configurations
│   ├── garage_gate.yaml      # Garage gate ESPHome config
│   ├── home-assistant-examples.yaml  # HA automations & scripts
│   └── secrets.yaml.template # Secrets template
└── drivers/                  # Driver documentation
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
