# MIDI Hub Architecture

## System Overview

The MIDI Hub is a microcontroller-based MIDI thru box with display functionality. It receives MIDI input, displays program change information on an OLED screen, and distributes the MIDI signal to 4 outputs with configurable channel filtering and remapping.

## Block Diagram

```
                                          ┌─────────────────┐
                                          │   ATmega32U4    │
                                          │   (Pro Micro)   │
                                          │                 │
MIDI IN ──┬─── H11L1M ───────────────────▶│ MIDI RX (Pin)  │
          │   (Optocoupler)                │                 │
          │                                │ I2C SDA ────────┼──┬─── SSD1306 OLED
          │                                │ I2C SCL ────────┼──┤
          │                                │                 │  └─── 24LC256 EEPROM
          │                                │                 │
          │                                │ USB ────────────┼──── WebMIDI Config
          │                                │                 │
          │                                │ MIDI TX (Pin)  │
          │                                └─────────┬───────┘
          │                                          │
          └───────────────────────────┬──────────────┘
                                      │
                              ┌───────▼───────┐
                              │   74HC14      │
                              │ Hex Schmitt   │
                              │   Trigger     │
                              └───┬───┬───┬───┬──┘
                                  │   │   │   │
                                  │   │   │   └──▶ MIDI OUT 4 (¼" TRS Type A)
                                  │   │   └──────▶ MIDI OUT 3 (⅛" TRS Type A)
                                  │   └──────────▶ MIDI OUT 2 (⅛" TRS Type A)
                                  └──────────────▶ MIDI OUT 1 (5-pin DIN)
```

## Hardware Components

### Microcontroller

**ATmega32U4** (via Pro Micro breakout board)
- Native USB support for WebMIDI configuration interface
- Hardware UART for MIDI communication
- I2C master for display and EEPROM
- Operating voltage: 5V
- Clock: 16MHz

### MIDI Input

**H11L1M Optocoupler**
- Provides galvanic isolation between MIDI input and MCU
- Standard MIDI input circuit implementation
- Current limiting resistor: 220Ω
- Supports MIDI 1.0 specification (31.25 kbaud)

### MIDI Distribution

**74HC14 Hex Schmitt-trigger Inverter**
- Buffers and distributes MIDI signal to 4 outputs
- Schmitt-trigger inputs provide noise immunity
- Clean signal edges for reliable MIDI transmission
- Each output independently driven

### Display

**SSD1306 OLED Display** (1.3", I2C)
- Resolution: 128×64 pixels
- I2C address: 0x3C (typical)
- Shows 2 rows of text for program names (up to 32 characters)
- Low power consumption

### Storage

**24LC256 EEPROM** (32KB, I2C)
- I2C address: 0x50 (typical)
- Stores 128 preset names (32 bytes each) = 4KB
- Stores output configuration (channel filters, remaps)
- Additional space available for future features
- Data retention: 200 years (typical)
- Endurance: 1,000,000 erase/write cycles

### Power Supply

- Input: 9V DC center-negative barrel jack
- Regulation: 5V linear regulator (7805 or equivalent)
- Current requirements: ~85mA typical
- Bypass capacitors for stable operation

### Outputs

1. **5-pin DIN** - Standard MIDI connector
2. **⅛" TRS Type A** - Compact TRS MIDI (Tip=Current Sink, Ring=Current Source)
3. **⅛" TRS Type A** - Compact TRS MIDI
4. **¼" TRS Type A** - Standard TRS MIDI

## Software Architecture

### Main Components

#### MIDI Processing
- Receives MIDI via hardware UART
- Parses MIDI messages in real-time
- Detects Program Change (PC) messages
- Applies channel filtering per output
- Remaps MIDI channels per output configuration

#### Display Controller
- Updates OLED on PC message reception
- Displays in two modes:
  - **3-digit mode**: Shows PC number (0-127)
  - **Bank/Preset mode**: Shows bank (1-32) and preset (1-4)
- Manages screensaver animation until first MIDI received

#### EEPROM Manager
- Reads/writes preset names (128 × 32 bytes)
- Stores output configuration:
  - Input channel filter (16 bits per output)
  - Output channel remap (0-15 per output)
- Implements wear-leveling for configuration updates

#### WebMIDI Interface
- USB MIDI device class implementation
- Bidirectional communication with browser
- Allows editing preset names
- Configures per-output channel filtering/remapping
- Changes saved directly to EEPROM

### Data Structures

#### Preset Name Storage
```
Address Range: 0x0000 - 0x0FFF (4096 bytes)
Layout: 128 presets × 32 bytes = 4KB
Each preset: null-terminated ASCII string
```

#### Configuration Storage
```
Address Range: 0x1000 - 0x10FF (256 bytes)
Per Output (4 outputs × 4 bytes = 16 bytes):
  - Input channel filter: 2 bytes (bitmask for channels 1-16)
  - Output channel remap: 1 byte (0-15, or 255 for no remap)
  - Reserved: 1 byte
```

### Display Format

#### Mode 1: 3-Digit PC Number
```
┌──────────────┐
│   PC: 042    │  ← Program Change number (0-127)
│ Preset Name  │  ← Up to 32 chars, 2 rows
└──────────────┘
```

#### Mode 2: Bank/Preset
```
┌──────────────┐
│   B11 P3     │  ← Bank 11, Preset 3 (4 presets per bank)
│ Preset Name  │  ← Up to 32 chars, 2 rows
└──────────────┘
```

### Bank/Preset Calculation
- Program Change 0-127 maps to:
  - Bank = (PC / 4) + 1 (Banks 1-32)
  - Preset = (PC % 4) + 1 (Presets 1-4)
- Example: PC 42 → Bank 11, Preset 3

## Pin Assignments (Pro Micro)

| Pin | Function | Description |
|-----|----------|-------------|
| RX1 | MIDI IN | Hardware UART receive (via H11L1M) |
| TX1 | MIDI OUT | Hardware UART transmit (to 74HC14) |
| SDA | I2C Data | Display and EEPROM data line |
| SCL | I2C Clock | Display and EEPROM clock line |
| VCC | Power | 5V power output |
| GND | Ground | Common ground |

## Communication Protocols

### MIDI (31.25 kbaud, 8-N-1)
- Standard MIDI 1.0 specification
- 8 data bits, no parity, 1 stop bit
- Asynchronous serial communication

### I2C (100 kHz or 400 kHz)
- Standard or Fast mode
- 7-bit addressing
- SSD1306: 0x3C
- 24LC256: 0x50

### USB MIDI
- USB 2.0 Full Speed (12 Mbps)
- MIDI device class
- WebMIDI API compatible

## Configuration Interface

The WebMIDI browser interface provides:
1. **Preset Name Editor**: Edit all 128 preset names
2. **Output Configuration**: Per-output settings for each of 4 outputs
   - Channel filter: Enable/disable specific MIDI channels (1-16)
   - Channel remap: Remap to different channel (1-16) or disable
3. **Display Mode**: Toggle between PC number and Bank/Preset modes
4. **Live Preview**: See changes reflected on display

## Power Budget

| Component | Current Draw (typical) |
|-----------|----------------------|
| ATmega32U4 | 30 mA |
| SSD1306 OLED | 20 mA |
| 24LC256 EEPROM | 3 mA (active), <1 µA (standby) |
| 74HC14 | 10 mA |
| MIDI outputs | 5 mA × 4 = 20 mA |
| **Total** | ~85 mA |
| **Regulated (headroom)** | 150-200 mA |

## Enclosure Specifications

**Hammond 1590BS**
- Dimensions: 112 × 60 × 38 mm (L × W × H)
- Material: Die-cast aluminum
- Finish: Powder coat (various colors available)
- Mounting: Through-hole components on perfboard

### Panel Layout
- Front: OLED display window
- Rear: 4× MIDI outputs, 1× MIDI input, power jack
- Side: USB port for WebMIDI configuration

## Future Expansion Possibilities

With 28KB of unused EEPROM space and available GPIO pins, potential features include:
- MIDI merge functionality
- Additional MIDI outputs
- Velocity curves
- Note filtering/transpose
- MIDI clock generation/sync
- Footswitch input for preset changes
