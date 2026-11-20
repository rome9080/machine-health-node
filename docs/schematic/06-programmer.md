# Sheet 6: USB-C and onboard programmer

**File:** `hardware/programmer.kicad_sch` (root page 6)

One USB-C cable gives the board a JTAG programmer (to load the FPGA) and a UART console (to talk to the RISC-V core), with no external dongle.

## Parts
| Ref | Part | Job |
|---|---|---|
| J11 | USB-C receptacle (USB 2.0, 16 pin) | Cable connector, VBUS, D+/D-, CC |
| U14 | USBLC6-2SC6 | ESD protection on D+/D- |
| U15 | AP2112K-3.3 | Makes a clean 3V3_USB from VBUS |
| U13 | FT2232HL | USB to dual serial chip: channel A = JTAG, channel B = UART |
| Y2 | 12 MHz crystal, 2 x 27 pF | FT2232H clock |
| R46 | 12k 1% | REF pin bias (datasheet value) |
| R47, R48 | 5.1k | CC1/CC2 pull-downs so a USB-C host supplies 5 V |

## Connections
- **3V3_USB** (from VBUS through the AP2112K) feeds VREGIN, VPHY, VPLL and RESET#. VCCIO pins use the board **3V3** rail, so the FT pins talk 3.3 V to the FPGA.
- **VCORE_1V8** is the FT2232H's own regulator output (VREGOUT) wired to all VCORE pins, with 3.3 uF + 100 nF.
- **JTAG (channel A):** ADBUS0 TCK, ADBUS1 TDI, ADBUS2 TDO, ADBUS3 TMS, via hierarchical labels to the FPGA core sheet.
- **UART (channel B):** BDBUS0 -> USB_UART_RX (FPGA receives), BDBUS1 <- USB_UART_TX (FPGA sends).
- EEPROM pins (61-63) and all unused I/O have no-connect flags. EEPROM is optional; default USB IDs are fine.

## Open items
VPLL LC filter (datasheet suggests one), VBUS fuse or TVS, FT2232H EEPROM, USB-C footprint choice, footprint properties on the new parts.
