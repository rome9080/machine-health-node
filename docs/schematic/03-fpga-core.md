# Sheet 3: FPGA core (hierarchical sheet `FPGA core`, fpga-core.kicad_sch)

Status: drawn 2026-11-20. Symbol and footprint from Ultra Librarian (XC7S25-1CSGA225C, 225-ball CSGA225), kept in hardware/XC7S25_1CSGA225C. Page size A2. ERC: only unconnected IO pins, the four JTAG hierarchical labels (no matching sheet pin until the programmer sheet exists) and the expected footprint-library warnings remain.

## What it does
Holds the Spartan-7 and everything it needs to start up and talk to the rest of the board: power pins, decoupling, boot flash, clock, configuration straps, status LEDs, buttons and two headers. The symbol is split into three units (A, B, C) on the sheet; all three are the one chip U3.

## Power pin connections (from AMD DS189, UG483, UG470)
| Pins | Net |
|---|---|
| VCCINT x8, VCCBRAM x2 | `1V0` |
| VCCAUX x2, VCCADC, VCCBATT | `1V8` |
| VCCO banks 0, 14, 15, 34 (11 pins), CFGBVS | `3V3` |
| GND x31, GNDADC, VP, VN, VREFP, VREFN | `GND` |

Notes: CFGBVS tied to 3V3 because VCCO_0 is 3.3 V. VCCADC is never tied to ground. VREFP/VREFN to ground is still to be checked against UG480. In the Ultra Librarian symbol the GND pins were typed power output; changed to power input in both the library file and the embedded copy so ERC does not report GND pins driving each other.

## Decoupling (D-019)
| Net | 4.7 uF | 0.47 uF |
|---|---|---|
| 1V0 (VCCINT + VCCBRAM) | 3 | 4 |
| 1V8 (VCCAUX) | 1 | 2 |
| 3V3 (VCCO, 4 banks) | 4 | 8 |
Bulk 47 and 100 uF capacitors from UG483 Table 2-1 are skipped because each converter's own output capacitor is the bulk, and the TPS62067 allows only 22 uF effective on its output.

## Configuration (Master SPI x4)
- Flash U6 W25Q32JVSS (32 Mbit, SOIC-8, 3.3 V), 100 nF on VCC. Lines `QSPI_CS`, `QSPI_CLK`, `QSPI_IO0..3` to CSO_B_14, CCLK_0, D00_MOSI, D01_DIN, D02, D03.
- Mode pins M[2:0] = 001 (Master SPI): M0 to 3V3, M1 and M2 to GND.
- Pull-ups to 3V3: 4.7k on `PROGRAM_B`, `INIT_B`, `DONE`; 10k on `QSPI_CS`, `QSPI_IO2` (WP#), `QSPI_IO3` (HOLD#).
- JTAG: `TCK`, `TMS`, `TDI`, `TDO` are hierarchical labels. Root-sheet pins are added when the programmer sheet is drawn. J3 (1x6: 3V3, GND, TCK, TMS, TDI, TDO) is a backup header.

## Clock
Y1 100 MHz oscillator (generic symbol Oscillator:ACO-xxxMHz, a real 3.3 V SMD part is chosen at the stock check), 100 nF on its supply, output `CLK_100M` to IO_L12P_T1_MRCC_34 (a clock-capable pin).

## User I/O
- 4 LEDs (D3 to D6) with 470 ohm resistors on `LED1..4`, to GND.
- 2 pushbuttons SW1, SW2 on `BTN1`, `BTN2` to GND (FPGA internal pull-ups to be set in the constraints file).
- J2 1x8 header: 3V3, GND, `GPIO0..5`.

## Pin assignment (global labels)
Peripheral signals use global labels so later sheets connect by name, with no sheet pins. Power rails stay hierarchical. Pins avoid the clock, VREF and special configuration pins. All 3.3 V banks.

| Signal | Direction | Ball | Pin name |
|---|---|---|---|
| `ACC_SCK` | output | A2 | IO_L2N_T0_34 |
| `ACC_MOSI` | output | A3 | IO_L1N_T0_34 |
| `ACC_MISO` | input | A4 | IO_L1P_T0_34 |
| `ACC_CS` | output | B1 | IO_L4N_T0_34 |
| `ACC_INT1` | input | B2 | IO_L2P_T0_34 |
| `ACC_INT2` | input | B3 | IO_L3N_T0_DQS_34 |
| `ADC_SCLK` | output | B4 | IO_L3P_T0_DQS_34 |
| `ADC_DIN` | output | B5 | IO_L5N_T0_34 |
| `ADC_DOUT` | input | C1 | IO_L4P_T0_34 |
| `ADC_CS` | output | C4 | IO_L7N_T1_34 |
| `ADC_DRDY` | input | C5 | IO_L5P_T0_34 |
| `ADC_RESET` | output | D2 | IO_L6P_T0_34 |
| `ADC_CLKIN` | output | D3 | IO_L9N_T1_DQS_34 |
| `TEMP_SCL` | bidirectional | D4 | IO_L7P_T1_34 |
| `TEMP_SDA` | bidirectional | E1 | IO_L8N_T1_34 |
| `TEMP_ALERT` | input | E2 | IO_L8P_T1_34 |
| `RS485_TX` | output | A5 | IO_L5P_T0_AD2P_15 |
| `RS485_RX` | input | A6 | IO_L5N_T0_AD2N_15 |
| `RS485_DE` | output | A7 | IO_L3N_T0_DQS_AD1N_15 |
| `CAN_TX` | output | A8 | IO_L6P_T0_15 |
| `CAN_RX` | input | A10 | IO_L9N_T1_DQS_AD11N_15 |
| `CAN_STB` | output | A11 | IO_L11P_T1_SRCC_AD12P_15 |
| `BLE_TX` | output | A12 | IO_L11N_T1_SRCC_AD12N_15 |
| `BLE_RX` | input | B6 | IO_L3P_T0_DQS_AD1P_15 |
| `BLE_EN` | output | B9 | IO_L9P_T1_DQS_AD11P_15 |
| `BLE_BOOT` | output | B10 | IO_L8N_T1_AD3N_15 |
| `USB_UART_TX` | output | B11 | IO_L10P_T1_AD4P_15 |
| `USB_UART_RX` | input | B12 | IO_L10N_T1_AD4N_15 |
| `LED1` | output | J11 | IO_0_14 |
| `LED2` | output | J15 | IO_L5P_T0_D06_14 |
| `LED3` | output | K14 | IO_L4P_T0_D04_14 |
| `LED4` | output | K15 | IO_L5N_T0_D07_14 |
| `BTN1` | input | L10 | IO_25_14 |
| `BTN2` | input | L13 | IO_L7P_T1_D09_14 |
| `GPIO0` | bidirectional | L14 | IO_L7N_T1_D10_14 |
| `GPIO1` | bidirectional | L15 | IO_L4N_T0_D05_14 |
| `GPIO2` | bidirectional | M7 | IO_L19P_T3_D26_14 |
| `GPIO3` | bidirectional | M13 | IO_L8P_T1_D11_14 |
| `GPIO4` | bidirectional | M14 | IO_L9P_T1_DQS_14 |
| `GPIO5` | bidirectional | M15 | IO_L9N_T1_DQS_D13_14 |

## Open items
- Verify VREFP/VREFN against UG480; confirm XC7S25 -C vs -I temperature grade.
- Choose a real 100 MHz oscillator (3.3 V, SMD, JLCPCB stock).
- Footprint fields for the generic parts are assigned in one pass at the end of Phase 2.
- Check total effective capacitance per rail (including these caps) against the TPS62067 4.5 to 22 uF range.
- Bank placement is a first pass; revisit during layout if routing is easier with swaps.
