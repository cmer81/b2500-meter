# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- **Run application**: `pipenv run python main.py`
- **Install dependencies**: `pipenv install`
- **Format code**: `pipenv run format` (uses black)
- **Run linting**: `pipenv run flake8`
- **Run tests**: `pipenv run pytest`
- **Run single test**: `pipenv run pytest path/to/test_file.py`

### Docker
- **Build and run**: `docker-compose up -d`
- **View logs**: `docker-compose logs -f`

## Architecture

This is a Python-based smart meter emulator for Marstek storage systems (B2500, Jupiter, Venus). The system emulates multiple device types to bridge between real smart meters and Marstek energy storage systems.

### Core Components

1. **Device Emulators** (`ct001/`, `shelly/`):
   - `CT001`: UDP/TCP protocol emulator for CT001 devices
   - `Shelly`: UDP protocol emulator for various Shelly device types (Pro 3EM, EM Gen3, Pro EM50)

2. **Powermeter Integrations** (`powermeter/`):
   - Pluggable architecture with `base.Powermeter` abstract class
   - Implementations for: Shelly, Tasmota, Home Assistant, MQTT, Modbus, ESPHome, etc.
   - Each powermeter returns power values as list of watts (3-phase support)

3. **Configuration System** (`config/`):
   - INI-based configuration with multiple powermeter sections
   - Client filtering via NETMASK settings for multi-device deployments
   - Global and per-powermeter throttling support

4. **Main Application** (`main.py`):
   - Thread pool executor for running multiple device emulators concurrently
   - Configuration loading and device orchestration
   - Command-line argument parsing with config file fallbacks

### Key Patterns

- **Plugin Architecture**: Powermeters are loaded dynamically based on configuration sections
- **Client Filtering**: Multiple powermeters can serve different storage system IP ranges
- **Threaded Servers**: UDP discovery and TCP data streams run in separate threads
- **Power Value Convention**: Positive = grid import, Negative = grid export
- **Three-Phase Support**: All components handle 1-3 phase configurations

### Configuration Structure

- `config.ini` defines device types, powermeters, and network settings
- Multiple sections with same prefix (e.g., `[SHELLY_1]`, `[SHELLY_2]`) supported
- NETMASK filtering allows different powermeters for different client IPs
- Throttling prevents control instability in storage systems

### Testing

- Tests use `_test.py` suffix and are located alongside source files
- Powermeter implementations include unit tests for HTTP/API interactions
- Main test directory contains integration and compatibility tests