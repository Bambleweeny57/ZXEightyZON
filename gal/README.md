\# ZXEightyZON GAL Logic – `zxeightyzon.cpl`



\*\*Device:\*\* GAL16V8D  

\*\*Revision:\*\* 1.0  

\*\*Designer:\*\* Bambleweeny57  

\*\*Company:\*\* Submeson Brain Corporation  

\*\*Date:\*\* 2025-10-19



This GAL logic handles address decoding and control signal generation for the ZXEightyZON sound interface. It supports multiple ZON-X variants and provides clock division and PSG interfacing via BC1 and BDIR.



---



\## 📌 Pin Assignments



| Pin | Signal     | Description                          |

|-----|------------|--------------------------------------|

| 01  | `CLK\_IN`   | Input clock                          |

| 02  | `A0`       | Address bit 0                        |

| 03  | `A1`       | Address bit 1                        |

| 04  | `A2`       | Address bit 2                        |

| 05  | `A3`       | Address bit 3                        |

| 06  | `A4`       | Address bit 4                        |

| 07  | `A5`       | Address bit 5                        |

| 08  | `A6`       | Address bit 6                        |

| 09  | `A7`       | Address bit 7                        |

| 10  | `GND`      | Ground                               |

| 11  | `NC`       | Not connected                        |

| 12  | `NC`       | Not connected                        |

| 13  | `WR\_N`     | Active-low write signal              |

| 14  | `RD\_N`     | Active-low read signal               |

| 15  | `IORQ\_N`   | Active-low I/O request               |

| 16  | `BDIR`     | PSG control: data direction          |

| 17  | `BC1`      | PSG control: register/data select    |

| 18  | `CLK\_OUT`  | Buffered clock output                |

| 19  | `CLK\_DIV2` | Divided clock output (toggle logic) |

| 20  | `VCC`      | Power                                |



---



\## 🧩 ZON-X Variant Address Combinations



| Variant Name            | Latch Address | Data Address | Notes |

|-------------------------|---------------|--------------|-------|

| \*\*Modified ZON-X\*\*      | `0xDF`        | `0x0F`       | Custom decoding logic for expanded compatibility |

| \*\*Original ZON-X\*\*      | `0xCF`        | `0x1F`       | Matches legacy ZON-X hardware |

| \*\*Manual Variant\*\*      | `0xCF`        | `0x0F`       | Hybrid logic for manual PSG control |

| \*\*Additional Combination\*\* | `0xDF`     | `0x1F`       | Alternate mapping for stereo or timing experiments |



> All combinations are explicitly decoded in CUPL to ensure builder clarity and avoid ambiguity.



---



\## 🧠 Logic Overview



\### 🔹 Address Field

```cupl

FIELD Addr = \[A7..A0];

```



\### 🔹 Register Select (Latch)

```cupl

EQU latch\_io =

&nbsp;   !IORQ\_N \& !WR\_N \& RD\_N \& (

&nbsp;     (Addr == 0xDF) #   // Modified ZON-X latch

&nbsp;     (Addr == 0xCF) #   // Original ZON-X latch

&nbsp;     (Addr == 0xCF) #   // Manual variant latch

&nbsp;     (Addr == 0xDF)     // Additional combination latch

&nbsp;   );

```



\### 🔹 Data Write

```cupl

EQU data\_io =

&nbsp;   !IORQ\_N \& !WR\_N \& RD\_N \& (

&nbsp;     (Addr == 0x0F) #   // Modified ZON-X data

&nbsp;     (Addr == 0x1F) #   // Original ZON-X data

&nbsp;     (Addr == 0x0F) #   // Manual variant data

&nbsp;     (Addr == 0x1F)     // Additional combination data

&nbsp;   );

```



\### 🔹 Data Read

```cupl

EQU read\_io =

&nbsp;   !IORQ\_N \& !RD\_N \& WR\_N \& (

&nbsp;     (Addr == 0x0F) #   // Modified ZON-X data read

&nbsp;     (Addr == 0x1F) #   // Original ZON-X data read

&nbsp;     (Addr == 0x0F) #   // Manual variant data read

&nbsp;     (Addr == 0x1F)     // Additional combination data read

&nbsp;   );

```



\### 🔹 Control Signal Logic

```cupl

BDIR = data\_io \& !read\_io;     // Write = 1, Read = 0

BC1  = latch\_io # read\_io;     // Latch or Read = 1

```



\### 🔹 Output Enable

```cupl

BDIR.OE     = data\_io # read\_io;

BC1.OE      = latch\_io # read\_io;

CLK\_OUT.OE  = 1;

CLK\_DIV2.OE = 1;

```



\### 🔹 Clock Logic

```cupl

CLK\_OUT = CLK\_IN;

CLK\_DIV2.CLK = CLK\_IN;

CLK\_DIV2.D   = !CLK\_DIV2.Q;

```



---



\## 🔧 Build Instructions



1\. Edit `gal/src/zxeightyzon.cpl` as needed.

2\. Use VS Code task `"Compile ZXEightyZON GAL"` to generate `zxeightyzon.jed`.

3\. Use task `"Copy JEDEC to Build"` to move the file to `gal/build/`.

4\. Program the GAL16V8D using you programmer:

&nbsp;  - \*\*T48\*\* with XGPro

&nbsp;  - \*\*Dataman Pro 40\*\* with Dataman Control Software



---



\## 🧪 Testing Notes



\- Use logic analyzer or LED rig to verify `BDIR` and `BC1` transitions.

\- Confirm clock division on `CLK\_DIV2` matches expected toggle rate.

\- Validate decoding against ZX81 bus activity and PSG response.



---



\## 🌀 Integration Context



This GAL logic is part of the \*\*ZXEightyZON\*\* sound interface, enabling compatibility with multiple ZON-X variants and stereo expansion. It supports modular decoding and clock handling for PSG control, and is designed for remixable builder workflows.



---



\## 🧬 Lore Tag



> “Designed by Bambleweeny57 for Submeson Brain Corporation, where logic meets lunacy.”



