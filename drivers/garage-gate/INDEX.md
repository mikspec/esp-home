# ESP Home - Garage Gate Controller

Complete ESPHome implementation for ESP8266-based garage gate controller.

## 📚 Documentation Index

### Getting Started
1. **[QUICKSTART.md](QUICKSTART.md)** - ⚡ 5-minute setup guide
   - Installation instructions
   - Basic configuration
   - First flash
   - Common commands

2. **[README.md](README.md)** - 📖 Main documentation
   - Project overview
   - Features
   - Hardware requirements
   - Setup instructions
   - Customization options

### Configuration Files
3. **[garage_gate.yaml](garage_gate.yaml)** - 🔧 Main configuration
   - Full-featured setup for NodeMCU v2
   - Includes all components (relay, button, contactron)
   - Cover integration for Home Assistant
   - Web server and diagnostics

4. **[garage_gate_simple.yaml](garage_gate_simple.yaml)** - 🔧 Minimal configuration
   - Simplified setup for ESP-01
   - Basic functionality only
   - Lower memory footprint

5. **[../config/secrets.yaml.template](../config/secrets.yaml.template)** - 🔐 Secrets template
   - Copy to `secrets.yaml`
   - Fill in WiFi credentials
   - API keys and passwords

### Hardware & Installation
6. **[WIRING.md](WIRING.md)** - 🔌 Wiring guide
   - Detailed pin assignments
   - Connection diagrams
   - Physical installation instructions
   - Safety considerations
   - Bill of materials

### Migration & Automation
7. **[MIGRATION.md](MIGRATION.md)** - 🔄 Migration guide
   - C to ESPHome migration
   - Feature comparison
   - Code examples
   - Performance notes

8. **[home-assistant-examples.yaml](home-assistant-examples.yaml)** - 🏠 HA automations
   - Auto-close automation
   - Notifications
   - Scripts
   - Dashboard cards
   - Advanced automations

### Support
9. **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** - 🔍 Troubleshooting
   - Installation issues
   - WiFi problems
   - Hardware debugging
   - Integration issues
   - Advanced diagnostics

## 🚀 Quick Links

| I want to... | Go to... |
|-------------|----------|
| Get started in 5 minutes | [QUICKSTART.md](QUICKSTART.md) |
| Understand the project | [README.md](README.md) |
| Wire up the hardware | [WIRING.md](WIRING.md) |
| Migrate from C code | [MIGRATION.md](MIGRATION.md) |
| Create automations | [home-assistant-examples.yaml](home-assistant-examples.yaml) |
| Fix a problem | [TROUBLESHOOTING.md](TROUBLESHOOTING.md) |
| Customize configuration | [garage_gate.yaml](garage_gate.yaml) |

## 📋 File Overview

```
esp-home/
├── compose.yaml                   # Docker Compose configuration
├── config/                        # ESPHome configurations
│   ├── garage_gate.yaml           # Main ESPHome configuration (NodeMCU)
│   ├── home-assistant-examples.yaml  # HA automation examples
│   └── secrets.yaml.template      # Template for credentials
├── drivers/                       # Driver documentation
│   └── garage-gate/               # Garage gate documentation
│       ├── gateEsp/               # Original C implementation
│       │   ├── gate/              # Arduino sketch
│       │   └── python/            # Python utilities
│       ├── INDEX.md               # This file
│       ├── MIGRATION.md           # C to ESPHome migration guide
│       ├── QUICKSTART.md          # Quick setup guide
│       ├── README.md              # Main documentation
│       ├── TROUBLESHOOTING.md     # Problem resolution
│       └── WIRING.md              # Hardware wiring guide
└── LICENSE                        # Project license
```

## 🎯 Common Tasks

### First Time Setup
```bash
# 1. Install ESPHome
pip3 install esphome

# 2. Create secrets file
cp ../config/secrets.yaml.template ../config/secrets.yaml
nano secrets.yaml

# 3. Flash ESP8266
esphome run garage_gate.yaml
```

### Update Configuration
```bash
# Edit configuration
nano garage_gate.yaml

# Upload wirelessly
esphome run garage_gate.yaml
```

### View Logs
```bash
esphome logs garage_gate.yaml
```

### Validate Configuration
```bash
esphome config garage_gate.yaml
```

## 🔧 Hardware Requirements

- **ESP8266** (NodeMCU v2 or ESP-01)
- **1x Relay module** (5V, optocoupler isolated)
- **1x Push button** (normally open)
- **1x Reed switch/contactron**
- **1x Magnet** (for contactron)
- Jumper wires and USB cable

## ✨ Features

✅ Wireless OTA updates  
✅ Home Assistant auto-discovery  
✅ Physical button control  
✅ Gate position sensing  
✅ Cover entity integration  
✅ Web interface  
✅ WiFi diagnostics  
✅ Secure API encryption  
✅ Fallback AP mode  
✅ Comprehensive logging  

## 🆘 Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| Can't upload | Hold FLASH button during upload |
| No WiFi | Connect to "Garage Gate Fallback" AP |
| Relay backwards | Add `inverted: true` to pin config |
| Wrong gate state | Swap OPEN/CLOSED in cover lambda |
| Not in HA | Manually add via IP address |

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for detailed solutions.

## 🔒 Security Notes

⚠️ **Important:**
- Never commit `secrets.yaml` (it's in .gitignore)
- Use strong passwords for OTA and AP
- Enable API encryption
- Use WPA2 for WiFi
- Add physical emergency stop button
- Test thoroughly before permanent installation

## 📞 Support & Resources

- **ESPHome Docs:** https://esphome.io
- **HA Community:** https://community.home-assistant.io
- **ESPHome Discord:** https://discord.gg/KhAMKrd
- **This Repo Issues:** GitHub Issues tab

## 📝 License

MIT License - See [LICENSE](LICENSE) file

## 🤝 Contributing

Contributions welcome! Please:
1. Test changes thoroughly
2. Update documentation
3. Follow existing code style
4. Submit PR with clear description

## 🎓 Learning Path

**Beginner:**
1. Read QUICKSTART.md
2. Follow setup steps
3. Test basic functionality

**Intermediate:**
1. Customize garage_gate.yaml
2. Add Home Assistant automations
3. Create dashboard cards

**Advanced:**
1. Add custom sensors
2. Create complex automations
3. Integrate with other systems
4. Contribute improvements

## 📊 Project Status

✅ Core functionality implemented  
✅ Documentation complete  
✅ Testing guidance provided  
✅ Migration path from C code  
✅ Home Assistant integration  
✅ Troubleshooting guide  

## 🎉 What's Next?

After successful garage gate implementation:

1. **Add more devices:**
   - Front door controller
   - Window sensors
   - Temperature monitoring

2. **Enhance automations:**
   - Voice control (Alexa/Google)
   - Geofencing
   - Scheduling

3. **Add security:**
   - Camera integration
   - Motion detection
   - Security alerts

4. **Monitor and optimize:**
   - Track usage patterns
   - Optimize battery (if applicable)
   - Add redundancy

---

**Ready to start?** Head to [QUICKSTART.md](QUICKSTART.md) for a 5-minute setup! 🚀
