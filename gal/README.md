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
| 01  | CLK        | Input clock                          |
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
| 12  | NC         | Not connected                        |
| 13  | RD_N       | Active-low read signal               |
| 14  | BDIR       | PSG control: data direction          |
| 15  | BC1        | PSG control: register/data select    |
| 16  | NC         | Not connected                        |
| 17  | CLK_DIV2   | Divided clock output (toggle logic)  |
| 18  | CLK_OUT    | Clock Buffered passthrough           |             |
| 19  | NC         | Not connected                        |
| 20  | VCC        | Power                                |

---

## 🧩 ZON-X Variant Address Combinations

| Variant Name            | Latch Address | Data Address | Notes                                                  |
|-------------------------|---------------|--------------|--------------------------------------------------------|
| Modified ZON-X          | 0xDF          | 0x0F         | AY-ZONIC decoding logic for expanded compatibility     |
| Original ZON-X          | 0xCF          | 0x1F         | Matches legacy ZON-X hardware                          |
| Manual Variant          | 0xCF          | 0x0F         | Hybrid logic for manual PSG control                    |
| Additional Combination  | 0xDF          | 0x1F         | Alternate mapping for stereo or timing experiments     |

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
CLK_OUT = CLK;
```

### 🔹 Clock Divider Logic
```cupl
CLK_DIV2.D = !CLK_DIV2;
```

---

## 🔧 Build Instructions

1. Edit `gal/src/zxeightyzon.cpl` as needed.
2. Use VS Code task `"Compile ZXEightyZON GAL"` to generate `zxeightyzon.jed`.
3. Use task `"Copy JEDEC to Build"` to move the file to `gal/build/`.
4. Program the GAL22V10 using your programmer:
   - **T48** with XGPro
   - **Dataman Pro 40** with Dataman Control Software

---

## 🧪 Testing Notes

- Use logic analyzer or LED rig to verify `BDIR` and `BC1` transitions.
- Confirm clock division on `CLK_DIV2` matches expected toggle rate.
- Validate decoding against ZX81 bus activity and PSG response.

---

## 🌀 Integration Context

This GAL logic is part of the **ZXEightyZON** sound interface, enabling compatibility with multiple ZON-X variants and stereo expansion. It supports modular decoding and clock handling for PSG control, and is designed for remixable builder workflows.

---

## 🧬 Lore Tag

> “Designed by Jonathan Gratton for AY-ZONIC, where builder clarity meets retro fidelity.”
