# ZXEightyZON – Coming Soon 🎶

**ZXEightyZON** is a new sound interface for the ZX81, built around the **YM2149F** sound chip. Designed as a timing-accurate, minimalist alternative to the AY-ZONIC-Core, this board offers clean integration with existing ZON-X81 demos and playback routines, while introducing flexible channel mixing options for stereo output.

## 🔧 Key Features

- **Sound Chip**: YM2149F (AY-3-8912 compatible, timing-accurate)
- **Address Decoding**: Simple A4 (with optional A14,A15 for other PSG interfaces) decoding, mirroring ZON-X81 behavior
- **GAL coded address decoding**: Support for all 4 main ZON X latch and data address combinations with option to add others without schematic changes
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
- **Master Clock selection**:
  - ZX81 bus clock to allow PSG to divide the clock (YM2149 SEL = 0)
  - ZX81 bus clock / 2 to supply the PSG with ready divided clock (YM2149 SEL = 1 or AY-3-8912 etc)
- **PSG Master Clock divider**:
  - Close to connect YM2149 SEL to GND (let PSG divide the clock)
  - Open to disconnect YM2149 SEL from GND (PSG using divvided clock)

## 🧪 Development Status

This project is currently in development. Stay tuned for schematics, build logs, and demo recordings. ZXEightyZON is designed to be simple, flexible, and fun—perfect for bridging the gap while AY-ZONIC-Core continues its evolution.

---
## GAL22V10 Pinout – ZXEightyZON

Pin 01 – CLK_IN      ; System clock input from ZX81  
Pin 02 – A0          ; Address bit 0  
Pin 03 – A1          ; Address bit 1  
Pin 04 – A2          ; Address bit 2  
Pin 05 – A3          ; Address bit 3  
Pin 06 – A4          ; Address bit 4  
Pin 07 – A5          ; Address bit 5  
Pin 08 – A6          ; Address bit 6  
Pin 09 – A7          ; Address bit 7  
Pin 10 – IORQ_N      ; IO request (active low)  
Pin 11 – WR_N        ; Write strobe (active low)  
Pin 12 – GND         ; Ground  
Pin 13 – RD_N        ; Read strobe (active low)  
Pin 14 – BDIR        ; AY control output  
Pin 15 – BC1         ; AY control output  
Pin 16 – A14         ; Address bit 14  
Pin 17 – A15         ; Address bit 15  
Pin 18 – M1          ; Opcode fetch indicator  
Pin 19 – CLK_DIV2    ; Divided clock output (toggle flip-flop)  
Pin 20 – CLK_OUT     ; Pass-through clock output  
Pin 21 – IOA         ; Expansion header I/O  
Pin 22 – IOB         ; Expansion header I/O  
Pin 23 – IOC         ; Expansion header I/O  
Pin 24 – VCC         ; +5V supply
---
## GAL16V8 CUPL Code — ZXEightyZON

### YM2149 Configuration
```cupl
Name     ZXEightyZON;
PartNo   GAL22V10;
Date     2025-10-17;
Revision Rev1.0;
Designer Jonathan Gratton;
Company  RetroCore;
Device   g22v10;

/* ---------- PIN DEFINITIONS ---------- */
PIN 01 = CLK_IN;     /* System clock from ZX81 */
PIN 02 = A0;
PIN 03 = A1;
PIN 04 = A2;
PIN 05 = A3;
PIN 06 = A4;
PIN 07 = A5;
PIN 08 = A6;
PIN 09 = A7;
PIN 10 = IORQ_N;
PIN 11 = WR_N;
PIN 12 = GND;
PIN 13 = RD_N;
PIN 14 = BDIR;       /* Output to YM2149 */
PIN 15 = BC1;        /* Output to YM2149 */
PIN 16 = A14;
PIN 17 = A15;
PIN 18 = M1;
PIN 19 = CLK_DIV2;   /* Divided clock output */
PIN 20 = CLK_OUT;    /* Pass-through clock output */
PIN 21 = IOA;        /* Expansion I/O */
PIN 22 = IOB;        /* Expansion I/O */
PIN 23 = IOC;        /* Expansion I/O */
PIN 24 = VCC;

/* ---------- ADDRESS FIELD ---------- */
FIELD Addr = [A7..A0];

/* ---------- OPERATION DECODING ---------- */
/*
  YM2149 Control Modes:
  - Latch: BDIR=1, BC1=1
  - Data Write: BDIR=1, BC1=0
  - Data Read: BDIR=0, BC1=1
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
      (Addr & 0xDF == 0x0F) #
      (Addr & 0xCF == 0x1F) #
      (Addr & 0xCF == 0x0F) #
      (Addr & 0xDF == 0x1F)
    );

EQU read_io =
    !IORQ_N & !RD_N & WR_N & (
      (Addr & 0xDF == 0x0F) #
      (Addr & 0xCF == 0x1F) #
      (Addr & 0xCF == 0x0F) #
      (Addr & 0xDF == 0x1F)
    );

/* ---------- CONTROL SIGNALS ---------- */
BDIR = latch_io # data_io;
BC1  = latch_io # read_io;

/* ---------- CLOCK OUTPUTS ---------- */
CLK_OUT = CLK_IN;

CLK_DIV2.CLK = CLK_IN;
CLK_DIV2.D   = !CLK_DIV2.Q;

/* ---------- EXPANSION I/O ---------- */
/* IOA, IOB, IOC are reserved for future use */
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
| **Data Read**  | Low     | High  | Low   | `0xDF & 0x0F`      | 0    | 1   | Modified ZON-X data read             |
|                |         |       |       | `0xCF & 0x0F`      | 0    | 1   | ZON-X manual data read               |
|                |         |       |       | `0xCF & 0x1F`      | 0    | 1   | Original ZON-X data read             |
|                |         |       |       | `0xDF & 0x1F`      | 0    | 1   | Additional combo data read           |

### 🧠 Notes

- All address combinations are decoded during **I/O write cycles** (`/IORQ = 0`, `/WR = 0`, `/RD = 1`) and **I/O read cycles** (`/IORQ = 0`, `/WR = 1`, `/RD = 0`) 
- The same address may appear in both latch and data blocks, but **control line logic determines the operation**
- `BDIR` and `BC1` are synthesized by the GAL to match AY/YM expectations
- This decoding ensures compatibility with all known ZON-X derivatives
