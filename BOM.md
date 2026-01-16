# Bill of Materials (BOM)

## MIDI Hub - MIDI Thru + Display

This is a complete parts list for building the MIDI Hub. All components are through-hole for easy perfboard construction.

## Main Components

| Qty | Part Number / Description | Description | Notes |
|-----|---------------------------|-------------|-------|
| 1 | Pro Micro (ATmega32U4) | Microcontroller breakout board | 5V/16MHz version required |
| 1 | H11L1M | Optocoupler (6-pin DIP) | For MIDI input isolation |
| 1 | 74HC14 | Hex Schmitt-trigger inverter (14-pin DIP) | For MIDI thru buffering |
| 1 | SSD1306 OLED Display | 1.3" I2C OLED display | 128×64 pixels, I2C interface |
| 1 | 24LC256 | 32KB I2C EEPROM (8-pin DIP) | I2C serial EEPROM |

## Power Supply

| Qty | Part Number / Description | Description | Notes |
|-----|---------------------------|-------------|-------|
| 1 | 7805 or LM7805 | 5V voltage regulator (TO-220) | 1A+ rated |
| 1 | Barrel Jack | DC power jack | 2.1mm, center-negative |
| 1 | 100µF | Electrolytic capacitor | Input filter (16V+) |
| 1 | 10µF | Electrolytic capacitor | Output filter (16V+) |
| 2 | 0.1µF (100nF) | Ceramic capacitor | Bypass capacitors |
| 1 | 1N4001 or 1N4007 | Rectifier diode | Reverse polarity protection (optional) |

## MIDI Input Circuit

| Qty | Part Number / Description | Description | Notes |
|-----|---------------------------|-------------|-------|
| 1 | 5-pin DIN connector | Female panel-mount MIDI connector | 180° DIN connector |
| 1 | 220Ω | Resistor (¼W) | MIDI input current limiting |
| 1 | 10kΩ | Resistor (¼W) | Pull-up resistor |
| 1 | 1N4148 | Switching diode | MIDI input protection |

## MIDI Output Circuits (×4)

### Output 1: 5-pin DIN

| Qty | Part Number / Description | Description | Notes |
|-----|---------------------------|-------------|-------|
| 1 | 5-pin DIN connector | Male panel-mount MIDI connector | 180° DIN connector |
| 2 | 220Ω | Resistor (¼W) | MIDI current limiting (pins 4 & 5) |

### Output 2 & 3: ⅛" (3.5mm) TRS Type A

| Qty | Part Number / Description | Description | Notes |
|-----|---------------------------|-------------|-------|
| 2 | ⅛" TRS jack | 3.5mm stereo panel-mount jack | TRS (3-conductor) |
| 4 | 220Ω | Resistor (¼W) | MIDI current limiting (2 per output) |

### Output 4: ¼" (6.35mm) TRS Type A

| Qty | Part Number / Description | Description | Notes |
|-----|---------------------------|-------------|-------|
| 1 | ¼" TRS jack | 6.35mm stereo panel-mount jack | TRS (3-conductor) |
| 2 | 220Ω | Resistor (¼W) | MIDI current limiting |

## Enclosure & Hardware

| Qty | Part Number / Description | Description | Notes |
|-----|---------------------------|-------------|-------|
| 1 | Hammond 1590BS | Die-cast aluminum enclosure | 112×60×38mm |
| 1 | Perfboard | Prototype PCB | Sized to fit enclosure (~100×50mm) |
| 1 | IC socket | 14-pin DIP socket | For 74HC14 |
| 1 | IC socket | 8-pin DIP socket | For 24LC256 |
| 1 | IC socket | 6-pin DIP socket | For H11L1M |
| 4 | M3 standoffs | 10-15mm threaded standoffs | For mounting perfboard |
| 8 | M3 screws | Machine screws | For standoff mounting |
| - | 22 AWG solid wire | Hookup wire | Various colors for wiring |
| - | Heat shrink tubing | Assorted sizes | For wire insulation |

## Hardware Summary

### Integrated Circuits
- 1× ATmega32U4 (Pro Micro breakout)
- 1× H11L1M optocoupler
- 1× 74HC14 hex Schmitt-trigger
- 1× 24LC256 EEPROM

### Semiconductors
- 1× 7805 voltage regulator
- 1× 1N4001/1N4007 diode (reverse protection)
- 1× 1N4148 diode (MIDI protection)

### Resistors (¼W)
- 11× 220Ω (MIDI current limiting)
- 1× 10kΩ (pull-up)

### Capacitors
- 1× 100µF electrolytic
- 1× 10µF electrolytic
- 2× 0.1µF ceramic

### Connectors
- 2× 5-pin DIN (1 female input, 1 male output)
- 2× ⅛" (3.5mm) TRS jack
- 1× ¼" (6.35mm) TRS jack
- 1× DC barrel jack (2.1mm)

### Display
- 1× 1.3" SSD1306 OLED (I2C)

## TRS MIDI Wiring (Type A)

For the TRS outputs, use **Type A** wiring standard:

| Pin | Function |
|-----|----------|
| Tip | Current Sink (MIDI pin 5 equivalent) |
| Ring | Current Source (MIDI pin 4 equivalent) |
| Sleeve | Ground |

**Note**: Type A is the most common TRS MIDI standard used by Korg, Make Noise, and others. Type B (used by Arturia) has Tip and Ring swapped.

## MIDI DIN-5 Pinout

Standard 5-pin DIN MIDI connectors:

### MIDI OUT (Male connector, looking at pins)
```
    5     4
  3   2   1
```

| Pin | Function |
|-----|----------|
| 1 | Not connected |
| 2 | Ground (cable shield) |
| 3 | Not connected |
| 4 | Current Source (+5V via 220Ω) |
| 5 | Current Sink (via 220Ω) |

### MIDI IN (Female connector)
Same pinout, but receives signal through optocoupler.

## I2C Addresses

| Device | I2C Address (7-bit) |
|--------|---------------------|
| SSD1306 OLED | 0x3C (typical) |
| 24LC256 EEPROM | 0x50 (default, A0-A2 grounded) |

**Note**: Verify I2C addresses of your specific modules, as some may vary.

## Power Supply Specifications

- **Input**: 9V DC (7-12V acceptable range)
- **Polarity**: Center-negative
- **Current**: 500mA minimum power supply capacity (~85mA typical device draw)
- **Regulated Output**: 5V DC
- **Regulator**: 7805 (TO-220 package, 1A+ rating)

## Optional Components

| Qty | Part Number / Description | Description | Notes |
|-----|---------------------------|-------------|-------|
| 1 | LED | 3mm or 5mm LED | Power indicator (optional) |
| 1 | 1kΩ | Resistor (¼W) | LED current limiting |
| 1 | SPST switch | Panel-mount toggle | Power switch (optional) |

## Tools Required

- Soldering iron and solder
- Wire strippers
- Flush cutters
- Drill and bits (for enclosure mounting holes)
- Step drill bit or chassis punch (for connector holes)
- Multimeter (for testing)
- Hot glue gun (for strain relief, optional)

## Vendor Suggestions

Most components are available from:
- **Electronics distributors**: Digi-Key, Mouser, Newark
- **Online retailers**: Amazon, eBay, AliExpress
- **Specialized suppliers**: 
  - Pro Micro: SparkFun, Adafruit, or clones
  - OLED Display: Adafruit, Amazon
  - MIDI connectors: Thonk, Mouser, Digi-Key

## Cost Estimate

Approximate component costs (USD, as of 2026):

| Category | Cost Range |
|----------|------------|
| Microcontroller (Pro Micro) | $10-20 |
| Display (SSD1306 OLED) | $5-10 |
| ICs & semiconductors | $5-10 |
| Passive components | $5-10 |
| Connectors | $10-15 |
| Enclosure & hardware | $10-15 |
| Power supply components | $5-10 |
| **Total** | **$50-90** |

Prices vary by vendor, quantity, and whether you purchase OEM or clone components.

## Build Notes

1. **IC sockets recommended**: Makes troubleshooting easier and protects ICs during soldering
2. **Test power supply first**: Verify 5V regulation before connecting sensitive components
3. **MIDI polarity**: Double-check MIDI current source/sink connections
4. **I2C pullups**: Most OLED modules have built-in pullups; additional pullups usually not needed
5. **Grounding**: Connect all grounds together, including MIDI cable shields
6. **Cable strain relief**: Use hot glue or zip ties to prevent connector damage
7. **Verify I2C addresses**: Use I2C scanner sketch if devices don't respond

## Alternatives & Substitutions

### Microcontroller
- Arduino Leonardo (same ATmega32U4 chip)
- Arduino Micro (same chip, smaller form factor)

### Optocoupler
- 6N138 (faster, but different pinout)
- PC900V (similar to H11L1M)

### Buffer
- 74HCT14 (TTL-compatible version)
- CD40106 (CMOS alternative, lower drive strength)

### Voltage Regulator
- LM7805 (common substitute for 7805)
- L7805CV (another common variant)
- LM2940 (low dropout alternative for lower input voltages)

### EEPROM
- 24LC512 (64KB, larger capacity)
- AT24C256 (Atmel equivalent)
- 24LC128 (16KB, smaller but sufficient for 128 presets)

## Safety Notes

- **Reverse polarity protection**: The 1N4001 diode is optional but recommended to protect against reversed power supply
- **Heatsink**: 7805 may require a heatsink if input voltage >9V or current draw >100mA
- **Capacitor polarity**: Ensure electrolytic capacitors are oriented correctly
- **Avoid shorts**: Check for solder bridges and bare wire contacts before powering on
