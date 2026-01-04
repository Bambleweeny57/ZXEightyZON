# ZXEightyZON GAL Logic – `zxeightyzon.cpl`

**Device:** GAL22V10  
**Revision:** 1.0  
**Designer:** Bambleweeny57  
**Company:** Submeson Brain Corporation  
**Date:** 04-01-2026

This GAL logic handles address decoding and control signal generation for the ZXEightyZON sound interface. 
It supports multiple ZON-X variants and provides clock division and PSG interfacing via BC1 and BDIR. 
All address combinations are genericaly decoded.

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
| 10  | !R_N       | Active-low write signal              |
| 11  | !IORQ_N    | Active-low I/O request               |
| 12  | GND        | Ground                               |
| 13  | !RD_N      | Active-low read signal               |
| 14  | BDIR       | PSG control: data direction          |
| 15  | BC1        | PSG control: register/data select    |
| 16  | CLK_DIV2   | Divided clock output (toggle logic)  |
| 17  | CLK_OUT    | Clock Buffered passthrough           |
| 18  | NC         | Not connected routed to header       |
| 19  | NC         | Not connected routed to header       |
| 20  | NC         | Not connected routed to header       |
| 21  | NC         | Not connected routed to header       |
| 22  | NC         | Not connected routed to header       |
| 23  | NC         | Not connected routed to header       |
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

### 🔹 Register Select (Latch) and Data Write
```cupl
VALID_IO = IORQ_N & WR_N & A0 & A1 & A2 & A3;
```

### 🔹 Control Signal Logic
```cupl
BDIR = VALID_IO;
BC1  = VALID_IO & A7;
```

### 🔹 Clock Buffered Passthough
```cupl
CLK_OUT = CLK_IN;
```

### 🔹 Clock Divider Logic
```cupl
CLK_DIV2.D = !CLK_DIV2;
```

### 🔹 Output Enables
```cupl
CLK_DIV2.OE = 'b'1;
BDIR.OE = 'b'1;
BC1.OE = 'b'1;
CLK_OUT.OE = 'b'1;
```

**Note on `RD` Signal Decoding:**  
> The `RD` line is not actively decoded in ZXEightyZON’s GAL logic for completeness and future compatibility. 
While no current ZON-X compatible software or demos utilize `RD`, its inclusion ensures accurate bus semantics and allows for potential future expansions (e.g., memory-mapped reads or refined timing control). 
Builders can treat it as provisioned but dormant — present in logic, unused in practice.

---

## 🔧 Build Instructions

1. Edit `gal/src/zxeightyzon.cpl` as needed.
2. Use VS Code task `"Compile ZXEightyZON GAL"` to generate `zxeightyzon.jed`.
3. Use task `"Copy JEDEC to Build"` to move the file to `gal/build/`.
4. Program the GAL22V10 using your programmer e.g.:
   - **T48** with XGPro

---

## 🧪 Testing Notes

- Use logic analyzer or LED rig to verify `BDIR` and `BC1` transitions.
- Confirm clock division on `CLK_DIV2` PIN 16 matches expected toggle rate.
- Confirm clock on `CLK_OUT` PIN 17 matches `CLK_IN` on PIN 1.
- Validate decoding against ZX81 bus activity and PSG response.

---

## 🌀 Integration Context

This GAL logic is part of the **ZXEightyZON** sound interface, enabling compatibility with multiple ZON-X variants and stereo expansion. 
It supports modular decoding and clock handling for PSG control, and is designed for remixable builder workflows.

---

## 🧬 Lore Tag