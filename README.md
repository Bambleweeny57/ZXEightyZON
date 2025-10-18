# ZXEightyZON 🎶

**ZXEightyZON** is a new sound interface for the ZX81, built around the **YM2149F** sound chip. Designed as a timing-accurate ZON-compatible sound card, this board offers clean integration with existing ZON-X81 demos and playback routines, while introducing flexible channel mixing options for stereo output.

---

## 🔧 Key Features

- **Sound Chip**: YM2149F (AY-3-8912 compatible, timing-accurate)
- **Address Decoding**:  
  Explicit 8-bit address decoding via GAL logic, supporting all 4 ZON-X register and data address combinations:
  - Latch: `0xDF`, Data: `0x0F` → Modified ZON-X  
  - Latch: `0xCF`, Data: `0x1F` → Original ZON-X  
  - Latch: `0xCF`, Data: `0x0F` → From ZON-X user manual  
  - Latch: `0xDF`, Data: `0x1F` → Additional combination
- **Stereo Output Configuration**:
  - Channel A: Always routed to **Left**
  - Mixed Channel: Selectable via jumper—either **Channel B** or **Channel C**
  - Right Channel: Assigned to the **non-mixed** channel  
    - If **B is mixed**, then **C is Right**  
    - If **C is mixed**, then **B is Right**
- **Noise Channel Nomination**:
  - Jumper-based selection of B or C as the mixed/noise channel
  - Supports regional playback preferences:
    - Western demos typically mix **Channel B**
    - Eastern demos often mix **Channel C**
- **Master Clock Selection**:
  - Jumper-based selection for either bus clock or divided-by-2 clock
  - `CLK_OUT`: Buffered ZX81 bus clock (YM2149 SEL = 0)
  - `CLK_DIV2`: Internally divided clock (YM2149 SEL = 1 or for AY-3-8912/8910)
- **PSG Clock Divider Control**:
  - Jumper-based selection for whether PSG divides master clock or receives pre-divided clock
  - **Closed**: Connect YM2149 SEL to GND (PSG divides clock)
  - **Open**: Disconnect SEL from GND (PSG uses divided clock)

---

## 📌 GAL16V8D Pin Assignment – ZXEightyZON Rev1.0

| **Pin** | **Signal Name** | **Direction** | **Description**                          |
|--------:|------------------|---------------|------------------------------------------|
| 01      | `CLK_IN`         | Input         | System clock from ZX81                   |
| 02–09   | `A0–A7`          | Input         | Address lines A0 to A7                   |
| 10      | `GND`            | —             | Ground                                   |
| 11–12   | `NC`             | —             | Not connected (output-only in complex mode) |
| 13      | `WR_N`           | Input         | Write strobe from ZX81                   |
| 14      | `RD_N`           | Input         | Read strobe from ZX81                    |
| 15      | `IORQ_N`         | Input         | I/O request from ZX81                    |
| 16      | `BDIR`           | Output        | PSG control line: bus direction          |
| 17      | `BC1`            | Output        | PSG control line: chip select            |
| 18      | `CLK_OUT`        | Output        | Buffered clock output                    |
| 19      | `CLK_DIV2`       | Output        | Divided clock output (1.65 MHz)          |
| 20      | `VCC`            | —             | +5V power supply                         |

> ⚠️ Pins 11 and 12 are output-only in complex mode and must not be used for inputs.

---

## 🧮 GAL16V8 CUPL Code — ZXEightyZON Rev1.0

```cupl
Name     ZXEightyZON;
PartNo   GAL16V8;
Date     2025-10-17;
Revision Rev1.0;
Designer Jonathan Gratton;
Company  Submeson Brain Corporation;
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

// -------- REGISTER SELECT (LATCH) --------
EQU latch_io =
    !IORQ_N & !WR_N & RD_N & (
      (Addr == 0xDF) #   // Modified ZON-X latch
      (Addr == 0xCF)     // Original + Manual ZON-X latch
    );

// -------- DATA WRITE --------
EQU data_io =
    !IORQ_N & !WR_N & RD_N & (
      (Addr == 0x0F) #   // Modified + Manual ZON-X data
      (Addr == 0x1F)     // Original + Additional ZON-X data
    );

/* ---------- CONTROL SIGNALS ---------- */
BDIR = data_io;
BC1  = latch_io;

/* ---------- OUTPUT ENABLE CONTROL ---------- */
BDIR.OE     = data_io;
BC1.OE      = latch_io;
CLK_OUT.OE  = 1;
CLK_DIV2.OE = 1;

/* ---------- CLOCK OUTPUTS ---------- */
CLK_OUT = CLK_IN;

CLK_DIV2.CLK = CLK_IN;
CLK_DIV2.D   = !CLK_DIV2.Q;
```

---

## 🧮 ZXEightyZON Truth Table – Bus Inputs vs Control & Clock Outputs

| `/IORQ` | `/WR` | `/RD` | `Addr` Match | Operation     | BDIR | BC1 | CLK_OUT | CLK_DIV2 |
|---------|-------|-------|--------------|---------------|------|-----|---------|----------|
| Low     | Low   | High  | `0xDF`        | Latch         | 1    | 1   | CLK     | ÷2       |
| Low     | Low   | High  | `0xCF`        | Latch         | 1    | 1   | CLK     | ÷2       |
| Low     | Low   | High  | `0x0F`        | Data Write    | 1    | 0   | CLK     | ÷2       |
| Low     | Low   | High  | `0x1F`        | Data Write    | 1    | 0   | CLK     | ÷2       |
| Low     | High  | Low   | `0x0F`        | Data Read     | 0    | 1   | CLK     | ÷2       |
| Low     | High  | Low   | `0x1F`        | Data Read     | 0    | 1   | CLK     | ÷2       |
| Any     | Any   | Any   | No match      | No operation  | 0    | 0   | CLK     | ÷2       |

---

## 🎛️ Stereo Jumper Logic

| Jumper Setting | Mixed Channel | Left Output | Right Output |
|----------------|----------------|-------------|--------------|
| Jumper = B     | Channel B      | Channel A   | Channel C    |
| Jumper = C     | Channel C      | Channel A   | Channel B    |

> Default jumper = B (Western demos)
