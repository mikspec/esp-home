# Copilot Instructions for ESP Home Repository

This document guides Copilot agents working in the ESP Home repository.

## Repository Overview

ESP Home is a collection of ESPHome configurations for home automation devices using ESP8266/ESP32 microcontrollers. The project provides reference implementations for several drivers (garage gate controller, carport relay controller, camper relay controller, and irrigation system) with Home Assistant integration.

## Build & Development

### Running ESPHome

The project uses Docker Compose for ESPHome development:

```bash
# Start ESPHome dashboard
docker-compose up -d

# Access at http://localhost:6052
# Upload one of: garage_gate.yaml, carport.yaml, irrigation.yaml, or camper.yaml from config/
```

No automated tests or linters exist in this repository. ESPHome validation occurs through the Docker container.

### Configuration Secrets

- Copy `config/secrets.yaml.template` to `config/secrets.yaml` (already exists, contains WiFi/API credentials)
- Secrets are referenced in YAML with `!secret key_name` syntax
- All GPIO assignments and timing parameters are documented in config files and driver READMEs

## Repository Structure & Architecture

```
config/                 # ESPHome YAML configurations
├── *.yaml             # Device configs: garage_gate.yaml, carport.yaml, irrigation.yaml, camper.yaml
├── packages/          # Home Assistant package integrations (*.yaml)
├── secrets.yaml       # Credentials and API keys (not committed)
└── .esphome/          # Build artifacts (gitignored)

drivers/               # Device documentation and legacy implementations
├── garage-gate/       # Garage gate controller docs, wiring guides, migration notes
├── carport/           # Carport relay controller documentation
├── camper/            # Camper relay controller documentation (ESP32)
└── irrigation/        # Irrigation system driver (legacy C implementation)
```

### Device Implementations

Each device follows a consistent pattern:

1. **Main ESPHome Config**: `config/{device_name}.yaml`
   - Defines hardware (board type: esp8266 or esp32, GPIO pins)
   - Configures sensors, switches, binary sensors (physical inputs/outputs)
   - Includes Home Assistant API service definitions
   - Uses substitutions for timing parameters and pin assignments

2. **Home Assistant Package**: `config/packages/{device_name}_ha.yaml`
   - Wraps raw entities in script aliases for user-friendly naming
   - Provides higher-level automations or helper entities
   - Loaded separately in Home Assistant, not deployed to device

3. **Device Documentation**: `drivers/{device_name}/README.md`
   - Hardware requirements and wiring diagrams
   - GPIO pin assignments and rationale
   - Setup instructions and commissioning checklists
   - Troubleshooting and customization guidance

### High-Level Architecture Patterns

#### Device Startup Behavior (`on_boot`)
- Devices define explicit startup policies per relay/output
- Common patterns: `ALWAYS_OFF` (safe default), `RESTORE_DEFAULT_OFF` (remember last state), `ALWAYS_ON` (required service)
- Garage gate initializes timestamp tracking on boot for auto-close logic
- Carport light explicitly turns off on boot (deterministic power recovery)

#### GPIO-Level Safety Mechanisms
- **Template switches** (e.g., inverter switch in camper config) wrap internal GPIO switches to enable safe shutdown sequences
- Template switch can execute scripts before GPIO changes
- Internal GPIO switches are hidden from Home Assistant to enforce safe shutdown flow
- Example: Camper inverter turns off N-G bond relay before de-energizing GPIO

#### Anti-Replay Protection
- Services like `garage_action` use monotonically increasing `request_id` integers
- Device compares current `request_id` against `last_request_timestamp` global
- Prevents replay attacks from duplicate/stale API calls

#### Watchdog Patterns
- Carport light watchdog: switch enable/disable + number for timeout (configurable max-on minutes)
- Script checks elapsed time and auto-off if threshold exceeded
- Watchdog state persists across reboots using preferences

#### State Persistence
- ESP8266/ESP32 can restore switch state from flash on boot
- Use `preferences: flash_write_interval: 1s` for frequent writes
- Some entities restore state, others always boot to safe defaults (per commissioning needs)

## Key Development Conventions

### Naming & Identifiers
- Device name (esphome.name): lowercase with hyphens (e.g., `garage-gate`, `carport`, `camper`)
- Entity internal IDs: lowercase with underscores (e.g., `gate_contactron`, `carport_light_relay`)
- Home Assistant entity names: descriptive, CamelCase or title case (e.g., "Carport: Plug On")
- Substitution names: snake_case, descriptive of purpose (e.g., `gate_relay_pulse_ms`, `carport_light_max_on_minutes`)

### ESPHome Config Structure
All configs follow this order:
1. `esphome:` block (name, friendly_name, on_boot)
2. Platform block (`esp8266:` or `esp32:`)
3. `wifi:` + fallback AP
4. `logger:` (DEBUG level for development)
5. `api:` + encryption
6. `ota:` 
7. `web_server:` (if applicable)
8. `globals:` (if state needed)
9. Component definitions (binary_sensor, switch, number, sensor, etc.)
10. Automations, scripts, and lambdas

### Lambda Usage
- Use C++ lambdas for logic (stored in service handlers, automations, on_change triggers)
- Log with `ESP_LOGW()`, `ESP_LOGI()`, `ESP_LOGD()` 
- Reference globals and internal entities using `id(entity_name)`
- Timestamps stored as `uint32_t` (millis()) or `int64_t` for request IDs

### Comments & Documentation
- Config files are self-documenting; minimal comments needed
- Complex lambda logic gets a brief inline comment (e.g., "anti-replay check", "safe shutdown sequence")
- Timing parameters include milliseconds units in substitution names
- Each service in api block documents its parameters (variable name, type, usage)

### Home Assistant Integration
- All devices configured for mDNS discovery (fallback AP for reconnection)
- API encryption key required for secure communication
- Services defined in ESPHome config are called as `esphome.{device_name}_{service_name}`
- Packages provide script layer for HA automation convenience

### Commissioning & Rollback
- Each driver README includes a commissioning checklist (verify pins, test state recovery, etc.)
- Rollback plan documented (e.g., carport specifies how to disable new config and revert to legacy)
- Migration notes provided for devices transitioning from C to ESPHome (garage-gate, carport)

## When Modifying Existing Configurations

1. **Before making changes**: Review the device README in `drivers/{device_name}/` for commissioning notes and any known limitations
2. **GPIO changes**: Update both the config and the README wiring diagram
3. **Startup behavior changes**: Update the commissioning checklist in the README
4. **New features**: Add Home Assistant package scripts to wrap new ESPHome services (don't expose raw services if user-friendly names are better)
5. **Testing**: No automated tests exist; document manual verification steps (e.g., "reboot and verify relay state restores")

## Helpful Commands

```bash
# Validate ESPHome config syntax (via Docker)
docker-compose exec esphome esphome config config/garage_gate.yaml

# View ESPHome logs
docker-compose logs -f esphome

# Stop services
docker-compose down

# Reconnect to device via OTA (after first USB upload)
# Edit the device name in dashboard or use: esphome run config/garage_gate.yaml --device garage-gate.local
```

Respond terse like smart caveman. All technical substance stay. Only fluff die.

Rules:
- Drop: articles (a/an/the), filler (just/really/basically), pleasantries, hedging
- Fragments OK. Short synonyms. Technical terms exact. Code unchanged.
- Pattern: [thing] [action] [reason]. [next step].
- Not: "Sure! I'd be happy to help you with that."
- Yes: "Bug in auth middleware. Fix:"

Switch level: /caveman lite|full|ultra|wenyan
Stop: "stop caveman" or "normal mode"

Auto-Clarity: drop caveman for security warnings, irreversible actions, user confused. Resume after.

Boundaries: code/commits/PRs written normal.
