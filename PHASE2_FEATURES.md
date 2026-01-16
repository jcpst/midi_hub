# Phase 2 Features - Arbitrary MIDI Message Sequences

## Overview

Phase 2 extends the MIDI Hub beyond simple channel filtering and remapping to support programmable MIDI message sequences. When a Program Change (PC) message is received, the device can send an arbitrary sequence of MIDI messages in addition to (or instead of) passing through the original PC message.

This enables complex MIDI routing scenarios such as:
- Sending multiple CC messages to configure synth parameters alongside a PC change
- Triggering note messages for gate/trigger outputs
- Broadcasting PC changes to multiple channels
- Creating custom MIDI macros triggered by incoming PC messages

## Feature Specifications

### Message Sequence Support

Each of the 128 program numbers (0-127) can be associated with a sequence of up to **16 MIDI messages** that will be sent when that PC message is received.

#### Supported Message Types

1. **Control Change (CC)**
   - Controller number: 0-127
   - Value: 0-127
   - Channel: 1-16

2. **Note On/Off**
   - Note number: 0-127
   - Velocity: 0-127 (Note Off can use velocity 0 or explicit Note Off message)
   - Channel: 1-16

3. **Program Change (PC)**
   - Program number: 0-127
   - Channel: 1-16

4. **Additional MIDI Messages (Future Expansion)**
   - Pitch Bend
   - Aftertouch (Channel and Polyphonic)
   - System Exclusive (SysEx) - with length limitations

### Per-Output Configuration

Each of the 4 MIDI outputs can be configured independently:

1. **Original Message Passthrough**: Enable/disable passing the original PC message
2. **Sequence Playback**: Enable/disable playing the programmed sequence
3. **Channel Filtering**: Existing channel filter functionality retained
4. **Channel Remapping**: Existing channel remap functionality retained

This allows fine-grained control such as:
- Output 1: Pass through original PC + play sequence
- Output 2: Play sequence only (suppress original PC)
- Output 3: Pass through original PC only (no sequence)
- Output 4: Disabled (no output)

### Timing and Playback

- **Sequence Timing**: Messages sent as fast as possible (minimal inter-message delay)
- **Order**: Messages sent in programmed order (0-15)
- **Non-blocking**: MIDI processing continues during sequence playback
- **Buffer Management**: Output buffer must accommodate burst transmission

**Future Enhancement**: Add configurable inter-message delay (0-100ms per message) for devices requiring spacing between MIDI commands.

## Data Structures

### Message Sequence Entry

Each MIDI message in a sequence requires 4 bytes:

```
Byte 0: Message Type & Channel
  Bits 7-4: Message type
    0000 = Empty/unused slot
    0001 = Note Off
    0010 = Note On
    0011 = Control Change (CC)
    0100 = Program Change (PC)
    0101-1111 = Reserved for future message types
  Bits 3-0: MIDI channel (0-15 = channels 1-16)

Byte 1: Data byte 1
  - Note Off/On: Note number (0-127)
  - CC: Controller number (0-127)
  - PC: Program number (0-127)

Byte 2: Data byte 2
  - Note Off/On: Velocity (0-127)
  - CC: Controller value (0-127)
  - PC: Unused (0)

Byte 3: Reserved
  - Future use: delay timing, flags, etc.
  - Currently set to 0
```

### Sequence Storage Layout

Each program (0-127) has an associated message sequence:

```
Per Program Sequence: 16 messages × 4 bytes = 64 bytes
Total for 128 programs: 128 × 64 = 8,192 bytes (8KB)
```

### Output Configuration Extension

Current output configuration (16 bytes total) extended to:

```
Per Output Configuration (8 bytes):
  Byte 0-1: Input channel filter (16-bit bitmask, existing)
  Byte 2: Output channel remap (0-15 or 255 for none, existing)
  Byte 3: Flags (new)
    Bit 0: Passthrough original PC (1 = yes, 0 = no)
    Bit 1: Play sequence (1 = yes, 0 = no)
    Bits 2-7: Reserved
  Bytes 4-7: Reserved for future use

Total: 4 outputs × 8 bytes = 32 bytes
```

## EEPROM Storage Map (Updated)

### Current Usage (Phase 1)
```
0x0000 - 0x0FFF: Preset names (128 × 32 bytes = 4KB)
0x1000 - 0x100F: Output configuration (4 × 4 bytes = 16 bytes)
0x1010 - 0x7FFF: Unused (28KB available)
```

### Phase 2 Usage
```
0x0000 - 0x0FFF: Preset names (128 × 32 bytes = 4KB)
0x1000 - 0x101F: Output configuration (4 × 8 bytes = 32 bytes)
0x1020 - 0x102F: Reserved/alignment (16 bytes)
0x1030 - 0x302F: Message sequences (128 × 64 bytes = 8KB)
0x3030 - 0x7FFF: Unused (20KB available)
```

**Total Phase 2 Storage**: 12KB (4KB preset names + 32B config + 8KB sequences)  
**Remaining Available**: 20KB for future enhancements

## WebMIDI Configuration Interface

### New UI Components

#### 1. Sequence Editor (per program)

For each program (0-127), a sequence editor allowing:

- **Message List**: Display up to 16 message slots
- **Add Message Button**: Opens message configuration dialog
- **Message Configuration**:
  - Type dropdown: Note On, Note Off, CC, PC
  - Channel selector: 1-16
  - Data fields: Specific to message type
    - Note: Number (0-127), Velocity (0-127)
    - CC: Controller (0-127), Value (0-127)
    - PC: Program (0-127)
- **Reorder**: Drag-and-drop to change message order
- **Delete**: Remove message from sequence
- **Clear All**: Empty entire sequence

#### 2. Output Behavior Configuration (per output)

For each output (1-4):

- **Passthrough Original PC**: Checkbox (default: enabled)
- **Play Sequence**: Checkbox (default: enabled)
- **Channel Filter**: Existing 16-channel bitmask UI (retained)
- **Channel Remap**: Existing dropdown 1-16 or "None" (retained)

#### 3. Bulk Operations

- **Copy Sequence**: Copy sequence from one program to another
- **Clear All Sequences**: Reset all 128 sequences to empty
- **Import/Export**: JSON format for backup and sharing configurations

### UI Layout Proposal

```
┌─────────────────────────────────────────────────┐
│ MIDI Hub Configuration                          │
├─────────────────────────────────────────────────┤
│                                                 │
│ [Preset Names] [Sequences] [Outputs] [Settings] │ ← Tabs
│                                                 │
│ ┌─────────────────────────────────────────────┐ │
│ │ Sequences Tab                               │ │
│ │                                             │ │
│ │ Program: [▼ 42 - "Piano Patch"]            │ │
│ │                                             │ │
│ │ Message Sequence (6/16 slots used):        │ │
│ │                                             │ │
│ │ 1. [≡] CC #1 (Mod) = 64 on Ch 1    [×] [⚙] │ │
│ │ 2. [≡] CC #7 (Vol) = 100 on Ch 1   [×] [⚙] │ │
│ │ 3. [≡] CC #10 (Pan) = 64 on Ch 1   [×] [⚙] │ │
│ │ 4. [≡] PC #12 on Ch 1              [×] [⚙] │ │
│ │ 5. [≡] Note On C3 Vel 127 on Ch 10 [×] [⚙] │ │
│ │ 6. [≡] Note Off C3 on Ch 10        [×] [⚙] │ │
│ │                                             │ │
│ │ [+ Add Message]  [Clear All]  [Copy From…] │ │
│ │                                             │ │
│ └─────────────────────────────────────────────┘ │
│                                                 │
│ ┌─────────────────────────────────────────────┐ │
│ │ Outputs Tab                                 │ │
│ │                                             │ │
│ │ Output 1 (5-pin DIN):                       │ │
│ │   ☑ Passthrough original PC                 │ │
│ │   ☑ Play programmed sequence                │ │
│ │   Channel Filter: [All] ▼                   │ │
│ │   Channel Remap: [None] ▼                   │ │
│ │                                             │ │
│ │ Output 2 (⅛" TRS):                          │ │
│ │   ☑ Passthrough original PC                 │ │
│ │   ☑ Play programmed sequence                │ │
│ │   Channel Filter: [Custom...] ▼             │ │
│ │   Channel Remap: [Channel 2] ▼              │ │
│ │                                             │ │
│ └─────────────────────────────────────────────┘ │
│                                                 │
│ [Save to Device]  [Export Config]  [Import]    │
└─────────────────────────────────────────────────┘
```

## Implementation Considerations

### Software Architecture Changes

#### MIDI Processing Flow

**Phase 1 (Current)**:
```
MIDI IN → Parse → Channel Filter → Channel Remap → MIDI OUT
                → Display Update (if PC)
```

**Phase 2 (Enhanced)**:
```
MIDI IN → Parse ─┬→ Display Update (if PC)
                 │
                 ├→ Original Message ──→ Channel Filter → Channel Remap → MIDI OUT
                 │                       (if passthrough enabled)
                 │
                 └→ Sequence Lookup ───→ Sequence Playback → MIDI OUT
                    (if PC message)      (if sequence enabled)
```

#### New Software Modules

1. **Sequence Manager**
   - Load sequence from EEPROM for given program number
   - Parse sequence data into MIDI messages
   - Queue messages for transmission

2. **Message Encoder**
   - Convert sequence data structure to MIDI byte stream
   - Apply per-output channel remapping if configured
   - Handle MIDI running status (optional optimization)

3. **Output Buffer**
   - Manage MIDI TX buffer for burst transmission
   - Handle timing between messages (if delays implemented)
   - Prevent buffer overflow

### Performance Considerations

#### Timing Analysis

At 31.25 kbaud MIDI baud rate:
- 1 MIDI byte = 320 µs (10 bits including start/stop)
- 3-byte MIDI message = 960 µs (~1 ms)
- 16-message sequence = ~16 ms maximum

**Impact**: Sequence playback completes in <20ms, acceptable for most use cases.

#### Memory Requirements

**Program Memory (Flash)**:
- Sequence manager code: ~2KB estimated
- Message encoder: ~1KB estimated
- Buffer management: ~0.5KB estimated
- **Total new code**: ~3.5KB

**RAM**:
- Active sequence buffer: 64 bytes (one program's sequence)
- Output buffers: 4 × 32 bytes = 128 bytes
- Message processing state: ~32 bytes
- **Total new RAM**: ~224 bytes

ATmega32U4 has 32KB flash and 2.5KB RAM, sufficient for this feature.

#### EEPROM Wear Leveling

With 1,000,000 write cycles:
- Sequence updates are infrequent (configuration changes)
- No wear leveling needed if updates are <1000 per sequence lifetime
- Monitor: If editing sequences frequently in live use, implement wear leveling

### Backward Compatibility

**Configuration Migration**:
- Devices with Phase 1 firmware: Use first 4KB + 16B of EEPROM
- Phase 2 firmware: Expand configuration area, preserve existing data
- Migration: On first boot, detect old format and migrate:
  - Copy output config from 0x1000-0x100F to 0x1000-0x101F (expanded)
  - Initialize new flags: Passthrough=1, Play Sequence=0 (disabled by default)
  - Initialize all sequences to empty

**Default Behavior**:
- Fresh install: All sequences empty, passthrough enabled
- Upgraded device: Sequences empty, passthrough enabled (same as Phase 1)
- User experience: No change until sequences are programmed

## Use Cases and Examples

### Use Case 1: Synth Parameter Recall

**Scenario**: Load synth patch and set specific CC values

```
Program 42 → "Analog Pad"
Sequence:
  1. PC #12 on Ch 1       (Load synth preset 12)
  2. CC #1 = 80 on Ch 1   (Modulation wheel)
  3. CC #74 = 64 on Ch 1  (Filter cutoff)
  4. CC #71 = 40 on Ch 1  (Filter resonance)
  5. CC #7 = 100 on Ch 1  (Volume)
```

**Result**: Pressing one button on MIDI controller loads preset and sets all parameters instantly.

### Use Case 2: Multi-Channel Setup

**Scenario**: One PC change controls multiple devices

```
Program 7 → "Song Intro"
Sequence:
  1. PC #1 on Ch 1   (Piano sound)
  2. PC #25 on Ch 2  (Bass sound)
  3. PC #48 on Ch 3  (Pad sound)
  4. PC #0 on Ch 10  (Drum kit)
```

**Result**: Entire band setup recalled with one PC message.

### Use Case 3: Trigger Sequence

**Scenario**: Gate/trigger outputs for modular synths

```
Program 99 → "Trigger Pattern"
Sequence:
  1. Note On C3, Vel 127 on Ch 10
  2. Note On D3, Vel 127 on Ch 10
  3. Note On E3, Vel 127 on Ch 10
  4. Note Off C3 on Ch 10
  5. Note Off D3 on Ch 10
  6. Note Off E3 on Ch 10
```

**Result**: Trigger three gates in sequence when PC 99 is received.

### Use Case 4: Output-Specific Behavior

**Configuration**:
- Output 1: Passthrough original PC, play sequence (main synth gets everything)
- Output 2: Sequence only (secondary device gets custom messages)
- Output 3: Passthrough only (simple MIDI thru)
- Output 4: Disabled

**Result**: Complex routing where different outputs receive different MIDI data from the same input.

## Testing Requirements

### Unit Tests

1. **Message Encoding**
   - Verify correct MIDI byte generation for each message type
   - Test all channels (1-16)
   - Test boundary values (0, 127)

2. **Sequence Parsing**
   - Load sequence from EEPROM
   - Parse message entries correctly
   - Handle empty slots
   - Handle corrupted data gracefully

3. **Output Configuration**
   - Read/write output flags
   - Apply passthrough setting
   - Apply sequence playback setting
   - Combine with existing channel filter/remap

### Integration Tests

1. **Sequence Playback**
   - Receive PC message
   - Verify correct sequence loaded
   - Verify messages sent in order
   - Verify timing between messages

2. **Per-Output Behavior**
   - Configure outputs differently
   - Verify each output behaves independently
   - Verify channel filtering still works
   - Verify channel remapping still works

3. **EEPROM Persistence**
   - Write sequences to EEPROM
   - Power cycle
   - Verify sequences restored correctly

### System Tests

1. **Live Performance**
   - Send PC messages at realistic rates (1-5 per second)
   - Verify no dropped messages
   - Verify no buffer overflows
   - Verify display still updates correctly

2. **Stress Testing**
   - Send rapid PC messages (10+ per second)
   - All sequences full (16 messages each)
   - All outputs enabled
   - Verify system stability

3. **WebMIDI Interface**
   - Edit sequences via browser
   - Save to device
   - Verify changes reflected immediately
   - Test bulk operations (copy, clear, export, import)

## Migration Path

### Phase 1 → Phase 2 Firmware Update

**Steps**:
1. User downloads Phase 2 firmware
2. User uploads via USB (standard PlatformIO upload)
3. On first boot, firmware detects Phase 1 data:
   - Output config at 0x1000-0x100F (16 bytes)
   - Preset names at 0x0000-0x0FFF (4KB)
4. Firmware migrates:
   - Expand output config to 32 bytes
   - Initialize new flags (passthrough=1, sequence=0)
   - Initialize all sequences to empty
   - Set migration flag at 0x7FFE to prevent re-migration
5. Device operates normally with backward-compatible defaults

**Rollback**:
- User can downgrade to Phase 1 firmware
- Phase 1 firmware ignores additional EEPROM data
- Original preset names and channel settings preserved

### WebMIDI Interface Update

**Changes**:
- Add "Sequences" tab
- Add "Outputs" tab enhancements (passthrough, sequence flags)
- Maintain compatibility with Phase 1 devices (hide new tabs if unsupported)

**Feature Detection**:
- Send firmware version query via SysEx
- Enable/disable UI features based on firmware capabilities
- Graceful degradation for older firmware

## Future Enhancements (Beyond Phase 2)

### Inter-Message Timing

Add configurable delay per message (0-255ms):
- Byte 3 of message entry: Delay in milliseconds
- Useful for devices requiring time between commands
- Example: SysEx parameter changes

### Advanced Message Types

1. **Pitch Bend**
   - 14-bit value (0-16383)
   - Requires 2 data bytes

2. **SysEx Messages**
   - Variable length (challenge for fixed 4-byte structure)
   - Possible solution: Chain multiple entries for long SysEx

3. **Real-Time Messages**
   - Start, Stop, Continue
   - MIDI Clock

### Conditional Logic

- **If/Then Rules**: "If PC on channel X, then send sequence Y"
- **Value Mapping**: "Map incoming CC value to outgoing CC with curve"
- **Note Transpose**: Add configurable note offset

### Templates and Presets

- **Template Library**: Pre-built sequences for common scenarios
- **Device Profiles**: Synth-specific parameter mappings
- **Share Community**: Export/import sequences online

### Live Editing

- **MIDI Learn**: Record incoming MIDI to build sequences
- **Real-Time Preview**: Hear sequence without saving
- **Undo/Redo**: Edit history for sequence changes

## Conclusion

Phase 2 transforms the MIDI Hub from a passive thru box into a programmable MIDI processor. The design maintains backward compatibility, uses available EEPROM efficiently, and provides a clear path for future enhancements.

**Key Benefits**:
- ✅ Maintains all Phase 1 functionality
- ✅ Adds powerful MIDI sequencing capabilities
- ✅ Configurable via WebMIDI interface
- ✅ Independent per-output control
- ✅ Room for future expansion (20KB EEPROM remaining)

**Implementation Complexity**: Moderate
- Firmware: ~3.5KB new code, well within flash budget
- WebMIDI: New UI components, standard web development
- EEPROM: Clear data structure, efficient storage

**Timeline Estimate**: 40-60 hours development + testing
- Firmware core: 20h
- WebMIDI interface: 15h
- Testing and debugging: 10h
- Documentation updates: 5h
