# MIDI Hub - MIDI Thru + Display

A compact MIDI thru box with OLED display for showing program change (PC) messages and preset names. Features configurable channel filtering and remapping per output, with WebMIDI browser configuration.

## Features

### Current (Phase 1)

- **MIDI Thru Distribution**: 4 MIDI outputs (1 DIN, 2 ⅛" TRS Type A, 1 ¼" TRS Type A)
- **OLED Display**: 1.3" SSD1306 I2C display shows program names on PC messages
- **Preset Storage**: 128 programs × 32 characters stored in external EEPROM
- **Display Modes**: 3-digit PC number or bank/preset format (4 presets per bank, 32 banks)
- **Per-Output Configuration**: Input channel filter + output channel remap for each output
- **WebMIDI Interface**: Browser-based configuration for preset names and settings
- **Screensaver**: Power-up animation until first MIDI message received

### Planned (Phase 2)

- **Programmable MIDI Sequences**: Send up to 16 custom MIDI messages (CC, Note, PC) per program
- **Advanced Output Control**: Independent passthrough and sequence playback settings per output
- **Enhanced WebMIDI Editor**: Sequence editor with drag-and-drop, copy/paste, and import/export

See [PHASE2_FEATURES.md](PHASE2_FEATURES.md) for complete Phase 2 specifications.

## Hardware

- **MCU**: ATmega32U4 (Pro Micro breakout) - native USB for WebMIDI
- **Display**: 1.3" SSD1306 OLED (I2C)
- **Storage**: 24LC256 external I2C EEPROM (32KB)
- **MIDI Input**: H11L1M optocoupler
- **MIDI Distribution**: 74HC14 hex Schmitt-trigger buffer
- **Power**: 9V center-negative barrel jack with 5V regulation
- **Enclosure**: Hammond 1590BS (112×60×38mm)
- **Construction**: Through-hole/perfboard

See [BOM.md](BOM.md) for complete parts list.

## Architecture

The system uses an ATmega32U4 for MIDI processing and USB communication, with an external EEPROM for storing preset names and configuration. The 74HC14 buffer distributes the MIDI signal to 4 outputs with configurable channel filtering and remapping.

See [ARCHITECTURE.md](ARCHITECTURE.md) for detailed technical information.

### Phase 2 Features (Planned)

A comprehensive plan for Phase 2 enhancements is available in [PHASE2_FEATURES.md](PHASE2_FEATURES.md). Phase 2 will add the ability to send programmable MIDI message sequences (CC, Note, PC) in response to Program Change messages, enabling complex MIDI routing and automation scenarios. This feature will be fully configurable via the WebMIDI interface.

## Getting Started

### Prerequisites

- [PlatformIO](https://platformio.org/) installed
- USB cable for programming Pro Micro
- Completed hardware build (see BOM.md)

### Building and Uploading

```bash
# Build the project
pio run

# Upload to device
pio run --target upload

# Monitor serial output (optional)
pio device monitor
```

### Configuration

1. Connect the device via USB
2. Open the WebMIDI configuration interface in a compatible browser (Chrome, Edge, Opera)
3. Configure preset names and output channel mappings
4. Settings are automatically saved to EEPROM

## Usage

- Power on the device - screensaver animation will display
- Send MIDI to the input - screensaver stops
- Send Program Change messages - display shows preset name or number
- Configure up to 128 preset names via WebMIDI interface
- Each output can filter/remap MIDI channels independently

## License

[Add your license here]

## Author

[Add author information here]
