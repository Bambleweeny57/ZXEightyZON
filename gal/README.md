# ZXEightyZON GAL Logic – `zxeightyzon.cpl`

**Device:** GAL22V10  
**Revision:** 1.0  
**Designer:** Bambleweeny57  
**Company:** Submeson Brain Corporation  
**Date:** 2025-10-22

This GAL logic handles address decoding and control signal generation for the ZXEightyZON sound interface. It supports multiple ZON-X variants and provides clock division and PSG interfacing via BC1 and BDIR. All address combinations are explicitly decoded for builder clarity and remixability.

---

## 📌 Pin Assignments (GAL22V10)

| Pin | Signal     | Description                          |
|-----|------------|--------------------------------------|
| 01  | CLK_IN     | Input clock                          |
| 02  | A0         | Address bit 0                        |
| 03  | A1         | Address bit 1                        |
| 04  | A2         | Address bit 2                        |
| 05  | A3         | Address bit 3                        |
| 06  | A4         | Address bit 4                        |
| 07  | A5         | Address bit 5                        |
| 08  | A6         | Address bit 6                        |
| 09  | A7         | Address bit 7                        |
| 10  | WR_N       | Active-low write signal              |
| 11  | IORQ_N     | Active-low I/O request               |
| 12  | GND        | Ground                               |
| 13  | RD_N       | Active-low read signal               |
| 14  | BDIR       | PSG control: data direction          |
| 15  | BC1        | PSG control: register/data select    |
| 16  | CLK_DIV2   | Divided clock output (toggle logic)  |
| 17  | CLK_OUT    | Clock Buffered passthrough           |
| 18  | NC         | Not connected                        |
| 19  | NC         | Not connected                        |
| 20  | NC         | Not connected                        |
| 21  | NC         | Not connected                        |
| 22  | NC         | Not connected                        |
| 23  | NC         | Not connected                        |
| 24  | VCC        | Power                                |

---

## 🧩 ZON-X Variant Address Combinations

| Variant Name            | Latch Address | Data Address |
|-------------------------|---------------|--------------|
| Modified ZON-X          | 0xDF          | 0x0F         |
| Original ZON-X          | 0xCF          | 0x1F         |
| Manual Variant          | 0xCF          | 0x0F         |
| Additional Combination  | 0xDF          | 0x1F         |

---

## 🧠 Logic Overview

### 🔹 Register Select (Latch)
```cupl
ADDR_LATCH1 = A7 & A6 & !A5 & A4 & A3 & !A2 & !A1 & !A0;  // H'DF'
ADDR_LATCH2 = A7 & A6 & !A5 & !A4 & A3 & !A2 & !A1 & !A0; // H'CF'
ADDR_LATCH3 = A7 & A6 & !A5 & !A4 & A3 & !A2 & !A1 & !A0; // H'CF'
ADDR_LATCH4 = A7 & A6 & !A5 & A4 & A3 & !A2 & !A1 & !A0;  // H'DF'

latch_io = !IORQ_N & !WR_N & RD_N & (
  ADDR_LATCH1 # ADDR_LATCH2 # ADDR_LATCH3 # ADDR_LATCH4
);
```

### 🔹 Data Write
```cupl
ADDR_DATA1 = !A7 & !A6 & !A5 & !A4 & A3 & A2 & A1 & A0;   // H'0F'
ADDR_DATA2 = !A7 & A6 & !A5 & !A4 & A3 & A2 & A1 & A0;    // H'1F'
ADDR_DATA3 = !A7 & !A6 & !A5 & !A4 & A3 & A2 & A1 & A0;   // H'0F'
ADDR_DATA4 = !A7 & A6 & !A5 & !A4 & A3 & A2 & A1 & A0;    // H'1F'

data_io = !IORQ_N & !WR_N & RD_N & (
  ADDR_DATA1 # ADDR_DATA2 # ADDR_DATA3 # ADDR_DATA4
);
```

### 🔹 Data Read
```cupl
ADDR_READ1 = !A7 & !A6 & !A5 & !A4 & A3 & A2 & A1 & A0;   // H'0F'
ADDR_READ2 = !A7 & A6 & !A5 & !A4 & A3 & A2 & A1 & A0;    // H'1F'
ADDR_READ3 = !A7 & !A6 & !A5 & !A4 & A3 & A2 & A1 & A0;   // H'0F'
ADDR_READ4 = !A7 & A6 & !A5 & !A4 & A3 & A2 & A1 & A0;    // H'1F'

read_io = !IORQ_N & !RD_N & WR_N & (
  ADDR_READ1 # ADDR_READ2 # ADDR_READ3 # ADDR_READ4
);
```

### 🔹 Control Signal Logic
```cupl
BDIR = data_io & !read_io;
BC1  = latch_io # read_io;
```

### 🔹 Clock Buffered Passthough
```cupl
CLK_OUT = CLK_IN;
```

### 🔹 Clock Divider Logic
```cupl
CLK_DIV2.D = !CLK_DIV2;
```

**Note on `RD` Signal Decoding:**  
> The `RD` line is actively decoded in ZXEightyZON’s GAL logic for completeness and future compatibility. While no current ZON-X compatible software or demos utilize `RD`, its inclusion ensures accurate bus semantics and allows for potential future expansions (e.g., memory-mapped reads or refined timing control). Builders can treat it as provisioned but dormant — present in logic, unused in practice.

---

## 🔧 Build Instructions

1. Edit `gal/src/zxeightyzon.cpl` as needed.
2. Use VS Code task `"Compile ZXEightyZON GAL"` to generate `zxeightyzon.jed`.
3. Use task `"Copy JEDEC to Build"` to move the file to `gal/build/`.
4. Program the GAL22V10 using your programmer e.g.:
   - **T48** with XGPro
   - **Dataman Pro 40** with Dataman Control Software

---

## 🧪 Testing Notes

- Use logic analyzer or LED rig to verify `BDIR` and `BC1` transitions.
- Confirm clock division on `CLK_DIV2` PIN 16 matches expected toggle rate.
- Confirm clock on `CLK_OUT` PIN 17 matches `CLK_IN` on PIN 1.
- Validate decoding against ZX81 bus activity and PSG response.

---

## 🌀 Integration Context

This GAL logic is part of the **ZXEightyZON** sound interface, enabling compatibility with multiple ZON-X variants and stereo expansion. It supports modular decoding and clock handling for PSG control, and is designed for remixable builder workflows.

---

## 🧬 Lore Tag

> “Designed by Bambleweeny57 for Submeson Brain Company, where builder clarity meets retro fidelity.”




Name     ZXEightyZON;
PartNo   GAL22V10C;
Date     22/10/2025;
Revision 1.3;
Designer Bambleweeny57;
Company  AY-ZONIC;
Assembly ZXEightyZON;
Location U2;
Device   g22v10;

/*
   AY‑ZONIC PSG Decoder — ZON X / ZON X‑81 Compatible
   ---------------------------------------------------
   Hardware (this board):
     • A0–A4, A7 connected to GAL
     • A5, A6 connected but ignored in decode
     • A4 is always wired to the ZX81 bus

   Software-visible behaviour:
     • ZON X‑81 style:
         Latch: 0xCF = 1100 1111 (A7=1, A4=0, A3–A0=1111)
         Data:  0x0F = 0000 1111 (A7=0, A4=0, A3–A0=1111)

     • ZON X style:
         Latch: 0xDF = 1101 1111 (A7=1, A4=1, A3–A0=1111)
         Data:  0x1F = 0001 1111 (A7=0, A4=1, A3–A0=1111)

   AY Truth Table:
     Latch (select register): BDIR=1, BC1=1
     Data write:              BDIR=1, BC1=0
     Idle:                    BDIR=0, BC1=0
*/

/* ---------- INPUTS ---------- */

PIN 1  = CLK_IN;
PIN 2  = A0;
PIN 3  = A1;
PIN 4  = A2;
PIN 5  = A3;
PIN 6  = A4;
PIN 7  = A5;   /* Present, ignored in decode */
PIN 8  = A6;   /* Present, ignored in decode */
PIN 9  = A7;

PIN 10 = WR_N;
PIN 11 = IORQ_N;
PIN 13 = RD_N;

/* ---------- OUTPUTS ---------- */

PIN 14 = BDIR;
PIN 15 = BC1;

PIN 16 = CLK_DIV2;
PIN 17 = CLK_OUT;

/* ---------- ADDRESS DECODE (A5/A6 ignored) ---------- */

/* Common low‑bits pattern: A0–A3 = 1 */
BASE = A0 & A1 & A2 & A3;

/* ---- ZON X‑81 style (A4 = 0) ---- */
/* 0xCF = 1100 1111 (A7=1, A4=0) */
ADDR_LATCH_CF = BASE & !A4 & A7;

/* 0x0F = 0000 1111 (A7=0, A4=0) */
ADDR_DATA_0F  = BASE & !A4 & !A7;

/* ---- ZON X style (A4 = 1) ---- */
/* 0xDF = 1101 1111 (A7=1, A4=1) */
ADDR_LATCH_DF = BASE &  A4 & A7;

/* 0x1F = 0001 1111 (A7=0, A4=1) */
ADDR_DATA_1F  = BASE &  A4 & !A7;

/* Combined latch and data decodes */
/* All four ports are valid; software chooses which pair it uses */
LATCH_ADDR = ADDR_LATCH_CF # ADDR_LATCH_DF;
DATA_ADDR  = ADDR_DATA_0F  # ADDR_DATA_1F;

/* ---------- I/O STROBES ---------- */

IO_WRITE = !IORQ_N & !WR_N & RD_N;

latch_io = IO_WRITE & LATCH_ADDR;
data_io  = IO_WRITE & DATA_ADDR;

/* ---------- AY CONTROL OUTPUTS ---------- */

BDIR = latch_io # data_io;
BC1  = latch_io;

/* ---------- CLOCK DIVIDE-BY-2 ---------- */

CLK_DIV2.D = !CLK_DIV2;

/* ---------- CLOCK PASS THROUGH ---------- */

CLK_OUT = CLK_IN;

