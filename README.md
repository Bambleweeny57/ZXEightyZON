# ZXEightyZON 🎶

**ZXEightyZON** is a new sound interface for the ZX81, built around the **YM2149F** or **AY-3-8910** sound chips. 
Designed as a timing-accurate ZON-compatible sound card, this board offers clean integration with existing ZON-X81 and ZON-X demos and playback routines, while introducing flexible channel mixing options for stereo output.

---

## 🔧 Key Features

- **Sound Chip**: YM2149F or AY-3-8910
- **Address Decoding**:  
  Generic 8-bit address decoding via GAL logic, supporting all 4 known ZON-X register and data address combinations:
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
  - `CLK_OUT`: Buffered ZX81 bus clock (YM2149 SEL = GND)
  - `CLK_DIV2`: Internally divided clock (YM2149 SEL = !GND or AY-3-8910)
- **PSG Clock Divider Control**:
  - Jumper-based selection for whether PSG divides master clock (YM2149) or receives pre-divided clock
  - **Closed**: Connect YM2149 SEL to GND (PSG divides clock - use with YM2149 only)
  - **Open**: Disconnect SEL from GND (PSG uses divided clock - use with YM2149 or AY-3-8910)

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
| 10  | !WR_N      | Active-low write signal              |
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

### 🧮 ZXEightyZON Truth Table – ZON-X Address Variants vs PSG Control Signals

This table maps decoded ZON-X addresses and I/O control signals to PSG control outputs. 
Each variant is listed explicitly to reflect all known ZON-X mappings.

| Variant                  | /IORQ | /WR  | Address | Operation     | BDIR | BC1 |
|--------------------------|:-----:|:----:|:-------:|:--------------|:----:|:---:|
| Modified ZON-X           | Low   | Low  | 0xDF    | Latch         | 1    | 1   |
| Modified ZON-X           | Low   | Low  | 0x0F    | Data Write    | 1    | 0   |
| Original ZON-X           | Low   | Low  | 0xCF    | Latch         | 1    | 1   |
| Original ZON-X           | Low   | Low  | 0x1F    | Data Write    | 1    | 0   |
| Manual Variant           | Low   | Low  | 0xCF    | Latch         | 1    | 1   |
| Manual Variant           | Low   | Low  | 0x0F    | Data Write    | 1    | 0   |
| Additional Combination   | Low   | Low  | 0xDF    | Latch         | 1    | 1   |
| Additional Combination   | Low   | Low  | 0x1F    | Data Write    | 1    | 0   |
| —                        | Any   | Any  | —       | No Operation  | 0    | 0   |

**Notes:**
- `BDIR` and `BC1` follow PSG truth table conventions for register and data access.
- Address variants are duplicated intentionally to reflect ZON-X lineage and contributor clarity.
- This logic ensures compatibility with Sinclair-era demos and hardware, while supporting AY-ZONIC extensions.

 **Note on `RD` Signal Decoding:**  
> The `RD` line is actively decoded in ZXEightyZON’s GAL logic for completeness and future compatibility. While no current ZON-X compatible software or demos utilize `RD`, its inclusion ensures accurate bus semantics and allows for potential future expansions (e.g., memory-mapped reads or refined timing control). Builders can treat it as provisioned but dormant — present in logic, unused in practice.

---

## 🎚️ AY Channel Assignment Jumper Block

This 2×2 jumper block allows manual swapping of **AY channels B and C** between:

- **CH_B** → Mixed (Left + Right)
- **CH_C** → Right-only

**CH_A is always fixed to Left-only.**

---

### 📐 Pinout Grid

```
╭────────────┬────────────╮
│ [1] B_OUT  │ [2] CH_C   │
├────────────┼────────────┤
│ [3] CH_B   │ [4] C_OUT  │
╰────────────┴────────────╯
```

---

### 🔁 Jumper Combinations

- **Vertical jumpers**  
  - B_OUT → CH_B → Mixed (L+R)  
  - C_OUT → CH_C → Right-only  

- **Horizontal jumpers**  
  - B_OUT → CH_C → Right-only  
  - C_OUT → CH_B → Mixed (L+R)  

---

### 🧠 Notes

- CH_A is always Left-only  
- CH_B feeds Left + Right  
- CH_C feeds Right-only

> Vertical jumpers = B mixed, C right (Western demos)
> Horizontal jumpers = C mixed, B right (Eastern demos)
