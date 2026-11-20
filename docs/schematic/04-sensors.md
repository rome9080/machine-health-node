# Sheet 4: Sensors (hierarchical sheet `Sensors`, sensors.kicad_sch)

Status: drawn 2026-11-20. Symbols and footprints from Ultra Librarian, kept in hardware/ul_ADS131M04IPWR, hardware/ul_LSM6DSOTR, hardware/ul_TMP117AIDRVR.

## What it does
Measures vibration (LSM6DSO), board temperature (TMP117) and motor current on three phases (ADS131M04 with three current-clamp inputs). All signals reach the FPGA by global labels; the only hierarchical pin is `3V3` (root sheet pin wired to the 3V3 rail).

## Parts
| Ref | Part | Package | Interface |
|---|---|---|---|
| U7 | LSM6DSO | LGA-14 | SPI |
| U8 | TMP117 | WSON-6 (DRV) | I2C |
| U9 | ADS131M04 | TSSOP-20 (PW) | SPI |

## Connections
- **LSM6DSO:** VDD and VDDIO on 3V3; SDx and SCx to GND; OCS_Aux to 3V3; SDO_Aux not connected. Labels `ACC_SCK`, `ACC_MOSI`, `ACC_MISO`, `ACC_CS`, `ACC_INT1`, `ACC_INT2`. Caps: 100 nF + 10 uF on VDD, 100 nF on VDDIO.
- **TMP117:** V+ on 3V3 with 100 nF; ADD0 to GND; `TEMP_SDA`, `TEMP_SCL`, `TEMP_ALERT` each pulled to 3V3 by 4.7k.
- **ADS131M04:** AVDD and DVDD on 3V3 with 1 uF each; CAP pin 220 nF to GND; SYNC/RESET to `ADC_RESET` with 10k pull-up to 3V3; labels `ADC_SCLK`, `ADC_DIN`, `ADC_DOUT`, `ADC_CS`, `ADC_DRDY`, `ADC_CLKIN`; AIN3P and AIN3N to GND (spare channel).
- **Current inputs (x3, J4, J6, J5):** 2-pin header, 33 ohm burden across the leads, 1k in series on each lead, 10 nF across the filtered pair. Filtered nets `CTn_AF` and `CTn_BF` go to AIN(n-1)P and AIN(n-1)N: channel 1 on AIN0, 2 on AIN1, 3 on AIN2 (J4 = ch 1, J6 = ch 2, J5 = ch 3).

## Safety
The board never touches mains. Current is measured only with split-core clamp CTs, which isolate the board from the conductor.

## Open items
CT clamp part and burden value (100 A : 50 mA assumed); ESD/TVS on the clamp inputs; ADC clock source frequency confirmed at firmware stage.
