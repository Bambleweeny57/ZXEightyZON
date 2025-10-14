# ZXEightyZON – Coming Soon 🎶

**ZXEightyZON** is a new sound interface for the ZX81, built around the **YM2149F** sound chip. Designed as a timing-accurate, minimalist alternative to the AY-ZONIC-Core, this board offers clean integration with existing ZON-X81 demos and playback routines, while introducing flexible channel mixing options for stereo output.

## 🔧 Key Features

- **Sound Chip**: YM2149F (AY-3-8912 compatible, timing-accurate)
- **Address Decoding**: Simple A4 decoding, mirroring ZON-X81 behavior
- **A4 Jumper**: Optional jumper to connect A4 to ZX81 bus for **ZONX compatibility**
- **Stereo Output Configuration**:
  - Channel A: Always routed to **Left**
  - Mixed Channel: Selectable via jumper—either **Channel B** or **Channel C**
  - Right Channel: Automatically assigned to the **non-mixed** channel
    - If **B is mixed**, then **C is Right**
    - If **C is mixed**, then **B is Right**
- **Noise Channel Nomination**:
  - Jumper-based selection of B or C as the mixed/noise channel
  - Supports regional playback preferences:
    - Western demos typically mix **Channel B**
    - Eastern demos often mix **Channel C**
- **Compatibility**:
  - Fully compatible with ZON-X81 demos and playback logic
  - Designed for real hardware playback and experimentation

## 🧪 Development Status

This project is currently in development. Stay tuned for schematics, build logs, and demo recordings. ZXEightyZON is designed to be simple, flexible, and fun—perfect for bridging the gap while AY-ZONIC-Core continues its evolution.

---
## GAL16V8 Pinout — ZXEightyZON

| Pin | Signal   | Direction | YM2149 Mode Description             | AY-3-8912 Mode Description         |
|-----|----------|-----------|-------------------------------------|------------------------------------|
| 1   | CLK_IN   | Input     | System clock from ZX81              | System clock from ZX81             |
| 2   | A0       | Input     | Address bit 0                       | Address bit 0                      |
| 3   | A1       | Input     | Address bit 1                       | Address bit 1                      |
| 4   | A2       | Input     | Address bit 2                       | Address bit 2                      |
| 5   | A3       | Input     | Address bit 3                       | Address bit 3                      |
| 6   | A4       | Input     | Address bit 4                       | Address bit 4                      |
| 7   | A5       | Input     | Address bit 5                       | Address bit 5                      |
| 8   | A6       | Input     | Address bit 6                       | Address bit 6                      |
| 9   | A7       | Input     | Address bit 7                       | Address bit 7                      |
| 10  | GND      | —         | Ground                              | Ground                             |
| 11  | WR_N     | Input     | Active-low write                    | Active-low write                   |
| 12  | IORQ_N   | Input     | Active-low I/O request              | Active-low I/O request             |
| 13  | RD_N     | Input     | Active-low read                     | Active-low read                    |
| 14  | BDIR     | Output    | Controls YM register/data write     | Controls AY register/data write    |
| 15  | BC1      | Output    | Controls YM register select         | Controls AY register select        |
| 16  | SEL      | Output    | Tied low to select YM2149           | Tied low (not used by AY)          |
| 17  | CLK_OUT  | Output    | Pass-through clock to YM2149        | ÷2 clock output to AY              |
| 18  | NC       | —         | Not connected                       | Not connected                      |
| 19  | NC       | —         | Not connected                       | Not connected                      |
| 20  | VCC      | —         | +5V power                           | +5V power                          |

---
