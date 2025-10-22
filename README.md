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

## 📌 GAL22V10D Pin Assignment – ZXEightyZON Rev1.0

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
| 10      | `WR_N`           | Input         | Write strobe from ZX81                   |
| 11      | `IORQ_N`         | Input         | I/O request from ZX81                    |
| 12      | —                | —             | Unused (do not assign)                   |
| 13      | `RD_N`           | Input         | Read strobe from ZX81                    |
| 14      | `BDIR`           | Output        | PSG control line: bus direction          |
| 15      | `BC1`            | Output        | PSG control line: chip select            |
| 16      | —                | —             | Unused (available for future use)        |
| 17      | `CLK_DIV2`       | Output        | Divided clock output (1.65 MHz)          |
| 18–22   | —                | —             | Unused (left floating, compiler-safe)    |
| 23      | —                | —             | Not used (internal logic only)           |
| 24      | `VCC`            | —             | +5V power supply                         |

---

## 🧮 GAL22V10 CUPL Code — ZXEightyZON Rev1.0

```cupl
Name     ZXEightyZON;
PartNo   01;
Date     22/10/2025;
Revision 1.0;
Designer Jonathan Gratton;
Company  AY-ZONIC;
Assembly ZXEightyZON;
Location U2;
Device   G22V10;

/* ---------- PIN DEFINITIONS (Mode 3) ---------- */
PIN 1  = CLK_IN;
PIN 2  = A0;
PIN 3  = A1;
PIN 4  = A2;
PIN 5  = A3;
PIN 6  = A4;
PIN 7  = A5;
PIN 8  = A6;
PIN 9  = A7;
PIN 10 = WR_N;
PIN 11 = IORQ_N;
PIN 13 = RD_N;

PIN 14 = BDIR;
PIN 15 = BC1;
PIN 17 = CLK_DIV2;

/* ---------- INTERNAL LOGIC SIGNALS ---------- */
BOOLEAN latch_io;
BOOLEAN data_io;
BOOLEAN read_io;

/* ---------- ZONX ADDRESS DECODING ---------- */

// -------- REGISTER SELECT (LATCH) --------
// Modified ZONX latch (AY-ZONIC variant)
ADDR_LATCH1 = A7 & A6 & !A5 & A4 & A3 & !A2 & !A1 & !A0;  // H'DF'
// Original ZONX latch (Sinclair-era mapping)
ADDR_LATCH2 = A7 & A6 & !A5 & !A4 & A3 & !A2 & !A1 & !A0; // H'CF'
// Manual variant latch (custom override)
ADDR_LATCH3 = A7 & A6 & !A5 & !A4 & A3 & !A2 & !A1 & !A0; // H'CF'
// Additional combination latch (AY-ZONIC fallback)
ADDR_LATCH4 = A7 & A6 & !A5 & A4 & A3 & !A2 & !A1 & !A0;  // H'DF'

latch_io = !IORQ_N & !WR_N & RD_N & (
  ADDR_LATCH1 # ADDR_LATCH2 # ADDR_LATCH3 # ADDR_LATCH4
);

// -------- DATA WRITE --------
// Modified ZONX data write (AY-ZONIC variant)
ADDR_DATA1 = !A7 & !A6 & !A5 & !A4 & A3 & A2 & A1 & A0;   // H'0F'
// Original ZONX data write (Sinclair-era mapping)
ADDR_DATA2 = !A7 & A6 & !A5 & !A4 & A3 & A2 & A1 & A0;    // H'1F'
// Manual variant data write (custom override)
ADDR_DATA3 = !A7 & !A6 & !A5 & !A4 & A3 & A2 & A1 & A0;   // H'0F'
// Additional combination data write (AY-ZONIC fallback)
ADDR_DATA4 = !A7 & A6 & !A5 & !A4 & A3 & A2 & A1 & A0;    // H'1F'

data_io = !IORQ_N & !WR_N & RD_N & (
  ADDR_DATA1 # ADDR_DATA2 # ADDR_DATA3 # ADDR_DATA4
);

// -------- DATA READ --------
// Modified ZONX data read (AY-ZONIC variant)
ADDR_READ1 = !A7 & !A6 & !A5 & !A4 & A3 & A2 & A1 & A0;   // H'0F'
// Original ZONX data read (Sinclair-era mapping)
ADDR_READ2 = !A7 & A6 & !A5 & !A4 & A3 & A2 & A1 & A0;    // H'1F'
// Manual variant data read (custom override)
ADDR_READ3 = !A7 & !A6 & !A5 & !A4 & A3 & A2 & A1 & A0;   // H'0F'
// Additional combination data read (AY-ZONIC fallback)
ADDR_READ4 = !A7 & A6 & !A5 & !A4 & A3 & A2 & A1 & A0;    // H'1F'

read_io = !IORQ_N & !RD_N & WR_N & (
  ADDR_READ1 # ADDR_READ2 # ADDR_READ3 # ADDR_READ4
);

/* ---------- CONTROL SIGNALS ---------- */
BDIR => data_io & !read_io;
BC1  => latch_io # read_io;

/* ---------- OUTPUT ENABLE CONTROL ---------- */
BDIR.OE     = data_io # read_io;
BC1.OE      = latch_io # read_io;
CLK_DIV2.OE = 1;

/* ---------- CLOCK DIVIDE-BY-2 ---------- */
CLK_DIV2.CLK = CLK_IN;
CLK_DIV2     => !CLK_DIV2.Q;
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
