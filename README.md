# ZXEightyZON – Coming Soon 🎶

**ZXEightyZON** is a new sound interface for the ZX81, built around the **YM2149F** sound chip. Designed as a timing-accurate, minimalist alternative to the AY-ZONIC-Core, this board offers clean integration with existing ZON-X81 demos and playback routines, while introducing flexible channel mixing options for stereo output.

## 🔧 Key Features

- **Sound Chip**: YM2149F (AY-3-8912 compatible, timing-accurate)
- **Address Decoding**: Simple A4 decoding, mirroring ZON-X81 behavior
- **GAL-coded address decoding**: Supports all 4 main ZON-X latch and data address combinations, with option to add others without schematic changes
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
- **Master Clock Selection**:
  - ZX81 bus clock to allow PSG to divide the clock (YM2149 SEL = 0)
  - ZX81 bus clock ÷ 2 to supply the PSG with ready divided clock (YM2149 SEL = 1 or AY-3-8912 etc)
- **PSG Master Clock Divider**:
  - **Closed**: Connect YM2149 SEL to GND (let PSG divide the clock)
  - **Open**: Disconnect YM2149 SEL from GND (PSG uses divided clock)

---

## 🧪 Development Status

This project is currently in development. Stay tuned for schematics, build logs, and demo recordings. ZXEightyZON is designed to be simple, flexible, and fun—perfect for bridging the gap while AY-ZONIC-Core continues its evolution.

---

## 📌 GAL16V8D Pin Assignment – ZXEightyZON Rev1.0 (Complex Mode)

| **Pin** | **Signal Name** | **Direction** | **Description**                          |
|--------:|------------------|---------------|------------------------------------------|
| 01      | `CLK_IN`         | Input         | System clock from ZX81                   |
| 02      | `A0`             | Input         | Address line A0                          |
| 03      | `A1`             | Input         | Address line A1                          |
| 04      | `A2`             | Input         | Address line A2                          |
| 05      | `A3`             | Input         | Address line A3                          |
| 06      | `A4`             | Input         | Address line A4                          |
| 07      | `A5`             | Input         | Address line A5                          |
| 08      | `A6`             | Input         | Address line A6                          |
| 09      | `A7`             | Input         | Address line A7                          |
| 10      | `GND`            | —             | Ground                                   |
| 11      | `NC`             | —             | Not connected (output-only in complex mode) |
| 12      | `NC`             | —             | Not connected (output-only in complex mode) |
| 13      | `WR_N`           | Input         | Write strobe from ZX81                   |
| 14      | `RD_N`           | Input         | Read strobe from ZX81                    |
| 15      | `IORQ_N`         | Input         | I/O request from ZX81                    |
| 16      | `BDIR`           | Output        | PSG control line: bus direction          |
| 17      | `BC1`            | Output        | PSG control line: chip select            |
| 18      | `CLK_OUT`        | Output        | Buffered clock output                    |
| 19      | `CLK_DIV2`       | Output        | Divided clock output (1.75 MHz)          |
| 20      | `VCC`            | —             | +5V power supply                         |

> ⚠️ Pins 11 and 12 are output-only in complex mode and must not be used for inputs.
> ✅ Pin 19 is output-only and safely used for `CLK_DIV2`.
---

## GAL16V8 CUPL Code — ZXEightyZON

### YM2149 Configuration
```cupl
Name     ZXEightyZON;
PartNo   GAL16V8;
Date     2025-10-17;
Revision Rev1.0;
Designer Jonathan Gratton;
Company  RetroCore;
Device   g16v8;

/* ---------- PIN DEFINITIONS ---------- */
PIN 01 = CLK_IN;
PIN 02 = A0;
PIN 03 = A1;
PIN 04 = A2;
PIN 05 = A3;
PIN 06 = A4;
PIN 07 = A5;
PIN 08 = A6;
PIN 09 = A7;
PIN 10 = GND;
PIN 11 = NC;
PIN 12 = NC;
PIN 13 = WR_N;
PIN 14 = RD_N;
PIN 15 = IORQ_N;
PIN 16 = BDIR;
PIN 17 = BC1;
PIN 18 = CLK_OUT;
PIN 19 = CLK_DIV2;
PIN 20 = VCC;

/* ---------- ADDRESS FIELD ---------- */
FIELD Addr = [A7..A0];

/* ---------- OPERATION DECODING ---------- */
EQU latch_io =
    !IORQ_N & !WR_N & RD_N & (
      (Addr & 0xDF == 0x0F) #
      (Addr & 0xCF == 0x1F) #
      (Addr & 0xCF == 0x0F) #
      (Addr & 0xDF == 0x1F)
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

/* ---------- OUTPUT ENABLE CONTROL ---------- */
BDIR.OE     = latch_io # data_io;
BC1.OE      = latch_io # read_io;
CLK_OUT.OE  = 1;
CLK_DIV2.OE = 1;

/* ---------- CLOCK OUTPUTS ---------- */
CLK_OUT = CLK_IN;

CLK_DIV2.CLK = CLK_IN;
CLK_DIV2.D   = !CLK_DIV2.Q;
```
---

## 🎛️ ZXEightyZON Control Signal Decoding Table

> Address match logic uses masked comparisons: `(Addr & Mask) == Value`

| Operation Type | `/IORQ` | `/WR` | `/RD` | Address Match     | BDIR | BC1 | Description                          |
|----------------|---------|-------|-------|--------------------|------|-----|--------------------------------------|
| **Latch**      | Low     | Low   | High  | `0xDF & 0x0F`      | 1    | 1   | Modified ZON-X register select       |
|                |         |       |       | `0xCF & 0x0F`      | 1    | 1   | ZON-X manual register select         |
|                |         |       |       | `0xCF & 0x1F`      | 1    | 1   | Original ZON-X (used for latch)      |
|                |         |       |       | `0xDF & 0x1F`      | 1    | 1   | Additional combo (used for latch)    |
| **Data Write** | Low     | Low   | High  | Same as above      | 1    | 0   | Data write to selected register      |
| **Data Read**  | Low     | High  | Low   | Same as above      | 0    | 1   | Read from selected register          |

### AY/YM Control Mode Summary

| BDIR | BC1 | Mode        |
|------|-----|-------------|
| 1    | 1   | Latch       |
| 1    | 0   | Data Write  |
| 0    | 1   | Data Read   |

## 🧮 ZXEightyZON Truth Table – Bus Inputs vs Control & Clock Outputs

| `/IORQ` | `/WR` | `/RD` | Address Match     | Operation     | BDIR | BC1 | CLK_OUT | CLK_DIV2 |
|---------|-------|-------|--------------------|---------------|------|-----|---------|---------|
| Low     | Low   | High  | `0xDF & 0x0F`      | Latch         | 1    | 1   | CLK     | Divided |
| Low     | Low   | High  | `0xCF & 0x0F`      | Latch         | 1    | 1   | CLK     | Divided |
| Low     | Low   | High  | `0xCF & 0x1F`      | Latch         | 1    | 1   | CLK     | Divided |
| Low     | Low   | High  | `0xDF & 0x1F`      | Latch         | 1    | 1   | CLK     | Divided |
| Low     | Low   | High  | (same as above)    | Data Write    | 1    | 0   | CLK     | Divided |
| Low     | High  | Low   | (same as above)    | Data Read     | 0    | 1   | CLK     | Divided |
| Any     | Any   | Any   | No match           | No operation  | 0    | 0   | CLK     | Divided |

> Notes:
> - `CLK_OUT` is always a buffered pass-through of `CLK_IN` from the ZX81 bus.
> - `CLK_DIV2` is generated internally by toggling on each rising edge of `CLK_IN`.
> - Address match logic uses masked comparisons: `(Addr & Mask) == Value`
> - BDIR and BC1 are only active when valid I/O cycles and address matches occur.
---

## 🔀 Stereo Jumper Logic

| Jumper Setting | Mixed Channel | Left Output | Right Output |
|----------------|----------------|-------------|--------------|
| Jumper = B     | Channel B      | Channel A   | Channel C    |
| Jumper = C     | Channel C      | Channel A   | Channel B    |

> Default jumper = B (Western demos)
