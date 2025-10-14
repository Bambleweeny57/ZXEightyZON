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
| 20  | VCC      | —         | +5V power                           | +5V 

---
## GAL16V8 CUPL Code — ZXEightyZON

### YM2149 Configuration
```cupl
Name     ZXEightyZON_YM2149;
PartNo   001;
Date     2025-10-14;
Revision Rev1.4;
Designer Jonathan Gratton;
Company  RetroCore;
Device   g16v8;

/* ---------- PIN DEFINITIONS ---------- */
PIN 1   = CLK_IN;
PIN 2   = A0;
PIN 3   = A1;
PIN 4   = A2;
PIN 5   = A3;
PIN 6   = A4;
PIN 7   = A5;
PIN 8   = A6;
PIN 9   = A7;
PIN 10  = GND;
PIN 11  = WR_N;
PIN 12  = IORQ_N;
PIN 13  = RD_N;
PIN 14  = BDIR;       /* Output to YM2149 */
PIN 15  = BC1;        /* Output to YM2149 */
PIN 16  = SEL;        /* Unused by YM, but set to 0 */
PIN 17  = CLK_OUT;    /* Pass-through clock output */
PIN 18  = NC;
PIN 19  = NC;
PIN 20  = VCC;

/* ---------- ADDRESS FIELD ---------- */
FIELD Addr = [A7..A0];

/* ---------- LATCH AND DATA DECODING ---------- */
/*
  All four ZON-X address/control combinations are valid.
  Operation type is determined by control line state.
  - Latch: BDIR=1, BC1=1
  - Data:  BDIR=1, BC1=0
*/

EQU latch_io =
    !IORQ_N & !WR_N & RD_N & (
      (Addr & 0xDF == 0x0F) #  // Modified ZON-X
      (Addr & 0xCF == 0x1F) #  // Original ZON-X
      (Addr & 0xCF == 0x0F) #  // ZON-X manual
      (Addr & 0xDF == 0x1F)    // Additional combo
    );

EQU data_io =
    !IORQ_N & !WR_N & RD_N & (
      (Addr & 0xDF == 0x0F) #  // Modified ZON-X
      (Addr & 0xCF == 0x1F) #  // Original ZON-X
      (Addr & 0xCF == 0x0F) #  // ZON-X manual
      (Addr & 0xDF == 0x1F)    // Additional combo
    );

/* ---------- CONTROL SIGNALS ---------- */
BDIR = latch_io # data_io;
BC1  = latch_io;

/* ---------- OTHER OUTPUTS ---------- */
SEL      = 0;
CLK_OUT  = CLK_IN;
```

### AY-3-8912 Configuration 
```cupl
Name     ZXEightyZON_AY8912;
PartNo   002;
Date     2025-10-14;
Revision Rev1.4;
Designer Jonathan Gratton;
Company  RetroCore;
Device   g16v8;

/* ---------- PIN DEFINITIONS ---------- */
PIN 1   = CLK_IN;     /* System clock from ZX81 */
PIN 2   = A0;
PIN 3   = A1;
PIN 4   = A2;
PIN 5   = A3;
PIN 6   = A4;
PIN 7   = A5;
PIN 8   = A6;
PIN 9   = A7;
PIN 10  = GND;
PIN 11  = WR_N;
PIN 12  = IORQ_N;
PIN 13  = RD_N;
PIN 14  = BDIR;       /* Output to AY8912 */
PIN 15  = BC1;        /* Output to AY8912 */
PIN 16  = SEL;        /* Unused by AY, but set to 0 */
PIN 17  = CLK_OUT;    /* ÷2 clock output for AY */
PIN 18  = NC;
PIN 19  = NC;
PIN 20  = VCC;

/* ---------- ADDRESS FIELD ---------- */
FIELD Addr = [A7..A0];

/* ---------- LATCH AND DATA DECODING ---------- */
/*
  All four ZON-X address/control combinations are valid.
  Operation type is determined by control line state.
  - Latch: BDIR=1, BC1=1
  - Data:  BDIR=1, BC1=0
*/

EQU latch_io =
    !IORQ_N & !WR_N & RD_N & (
      (Addr & 0xDF == 0x0F) #  // Modified ZON-X
      (Addr & 0xCF == 0x1F) #  // Original ZON-X
      (Addr & 0xCF == 0x0F) #  // ZON-X manual
      (Addr & 0xDF == 0x1F)    // Additional combo
    );

EQU data_io =
    !IORQ_N & !WR_N & RD_N & (
      (Addr & 0xDF == 0x0F) #  // Modified ZON-X
      (Addr & 0xCF == 0x1F) #  // Original ZON-X
      (Addr & 0xCF == 0x0F) #  // ZON-X manual
      (Addr & 0xDF == 0x1F)    // Additional combo
    );

/* ---------- CONTROL SIGNALS ---------- */
BDIR = latch_io # data_io;
BC1  = latch_io;

/* ---------- CLOCK DIVIDER ---------- */
CLK_OUT.d  = !CLK_OUT;
CLK_OUT.ck = CLK_IN;

/* ---------- OTHER OUTPUTS ---------- */
SEL = 0;
```
---
### 🎛️ ZXEightyZON Control Signal Decoding Table

| Operation Type | `/IORQ` | `/WR` | `/RD` | Address Match     | BDIR | BC1 | Description                          |
|----------------|---------|-------|-------|--------------------|------|-----|--------------------------------------|
| **Latch**      | Low     | Low   | High  | `0xDF & 0x0F`      | 1    | 1   | Modified ZON-X register select       |
|                |         |       |       | `0xCF & 0x0F`      | 1    | 1   | ZON-X manual register select         |
|                |         |       |       | `0xCF & 0x1F`      | 1    | 1   | Original ZON-X (used for latch)      |
|                |         |       |       | `0xDF & 0x1F`      | 1    | 1   | Additional combo (used for latch)    |
| **Data Write** | Low     | Low   | High  | `0xDF & 0x0F`      | 1    | 0   | Modified ZON-X data write            |
|                |         |       |       | `0xCF & 0x0F`      | 1    | 0   | ZON-X manual data write              |
|                |         |       |       | `0xCF & 0x1F`      | 1    | 0   | Original ZON-X data write            |
|                |         |       |       | `0xDF & 0x1F`      | 1    | 0   | Additional combo data write          |

### 🧠 Notes

- All address combinations are decoded during **I/O write cycles only** (`/IORQ = 0`, `/WR = 0`, `/RD = 1`)
- The same address may appear in both latch and data blocks, but **control line logic determines the operation**
- `BDIR` and `BC1` are synthesized by the GAL to match AY/YM expectations
- This decoding ensures compatibility with all known ZON-X derivatives
