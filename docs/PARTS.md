# Parts Shortlist (Phase 1)

Status key: **Chosen** = picked after checking manufacturer or distributor pages on 2026-11-16. **Candidate** = proposed, details still to verify in the datasheet before the schematic uses it.

| Block | Part | What was checked | Status |
|-------|------|------------------|--------|
| FPGA | AMD Spartan-7 XC7S25-1CSGA225C | 225-ball CSGA, 13 x 13 mm, 150 I/O, 23,360 logic cells, Active. See D-004 in the design log | Chosen |
| Vibration sensor | ST LSM6DSO | LGA-14L 2.5 x 3 x 0.83 mm, accelerometer +/-2/4/8/16 g, accelerometer output rate up to 6.66 kHz, SPI and I2C, -40 to +85 C, Active | Chosen |
| Current ADC | TI ADS131M04 | 4 simultaneously sampling differential inputs, 24-bit, programmable gain up to 128, TSSOP-20 or WQFN-20, -40 to +125 C, up to 64 kSPS in the current datasheet (older revision said 32 kSPS, check the revision) | Chosen (3 CT channels plus 1 spare) |
| Input buck converter | TI LMR36015 | 4.2 to 60 V input with transient tolerance to 66 V, 1.5 A, 12-pin VQFN-HR 2 x 3 mm, power-good and enable pins, about 90 percent efficiency for 24 V to 5 V at 1 A, quiescent current about 26 uA | Chosen, exact variant to pick |
| FPGA rails (1.0 V, 1.8 V, 3.3 V) | Three step-down converters from the 5 V rail, sequenced | TI reference design TIDA-050000 uses TPS65023, TPS568215 and TPS62067. TPS62067 class (3 MHz, 2 A) is the candidate for each rail | Candidate, verify enable and power-good pins and input range |
| RS-485 | TI THVD1410 (or THVD1450) | THVD14xx family: 3 to 5.5 V supply, +/-18 kV IEC ESD, +/-15 V common-mode range, half duplex, -40 to +125 C, SOIC-8, VSSOP-8 or VSON-8. THVD1410 is the low-EMI 500 kbps part | Chosen family, verify which variant is 500 kbps |
| CAN FD | TI TCAN1042GV family | 4.5 to 5.5 V supply, CAN FD at 2 Mbps (G variants 5 Mbps), V variants have a VIO pin for 3.3 V logic, bus fault protection +/-58 V, -55 to +125 C, SOIC-8 or VSON-8, Active. CAN controller is written in RTL on the FPGA | Chosen family, pick exact orderable part |
| Wireless | Espressif ESP32-C3-MINI-1 | Not re-checked | Candidate |
| Temperature | TI TMP117 | Not re-checked. A service assembles the board, so the small leadless package is no longer a problem | Candidate |
| FPGA configuration flash | 64 Mbit QSPI flash (for example W25Q64) | Not checked. Confirm the bitstream size fits | Candidate |
| Clock | 100 MHz oscillator | Not checked | Candidate |
| USB and programming | USB-C with onboard USB-JTAG and UART bridge (FT2232H class) | Not checked. See open decision O-1 | Candidate |
| Current sensors | Split-core CT clamps with voltage output, burden sized for fan current | Not selected | To pick |

## Notes

- Power flow: 9 to 36 V input, then LMR36015 to a 5 V rail, then three converters for the FPGA's 1.0 V core, 1.8 V auxiliary and 3.3 V I/O rails, enabled in sequence. The 5 V rail also feeds the CAN transceiver.
- The earlier LiPo battery, charger and low-power regulator ideas from the first concept no longer apply. The board is powered from the 9 to 36 V input.
- Demo machine: an AC box or pedestal fan with a deliberately added imbalance. See D-008.

## JLCPCB / LCSC sourcing check (2026-11-21)

Looked up each major part on LCSC and JLCPCB (same part numbers, JLCPCB assembles from the LCSC catalogue). Stock figures from search snapshots disagreed with each other (cached pages of different dates), so treat every number as "roughly" and re-check in the JLCPCB BOM tool before ordering. "Extended" means JLCPCB charges a one-time fee per extended part type on each assembly order.

| Ref | Part | LCSC / JLCPCB # | Class | Rough stock and price | Notes |
|-----|------|-----------------|-------|------------------------|-------|
| U3 | XC7S25-1CSGA225C | C1521699 | Extended | 10 to 62 units seen, about $13 to $20 each | Thin stock, buy early. Industrial-grade -I variant C1521710 has more stock (30 to 225) at similar price |
| U12 | ESP32-C3-MINI-1-N4 | C2838502 | Extended | 650 to 7,500 seen, about $2 to $3 | Mouser marks it not recommended for new designs. Newer N4X variant exists (needs chip revision 1.1 or later) |
| U13 | FT2232HL-REEL | C27882 | Extended | 300 to 2,500 seen, about $5 to $7.60 | LCSC marks it not recommended for new designs |
| U1 | LMR36015ARNXR | C1850345 | Not confirmed | Low at LCSC (27 in one warehouse), about $1.84 | DigiKey has thousands. Check JLCPCB stock |
| U9 | ADS131M04IPWR | C2862562 | Extended | Conflicting: 446 to 1,242, or out of stock | Check before ordering |
| U7 | LSM6DSOTR | C2655100 | Extended | 4,000 to 9,000, about $2.40 to $4.20 | Good |
| U8 | TMP117AIDRVR | C699536 | Extended | 6,000 to 15,000, about $0.93 | Good |
| U10 | THVD1410DR | C2671345 | Extended | Conflicting: 63 to 4,400, about $0.42 to $0.79 | Alternative in same family and package: THVD1400DR (C3235232), 3,000 to 7,000 in stock. Check pinout before switching |
| U11 | TCAN1042VDRQ1 | C485806 | Extended | 1,000 to 9,000, about $0.33 to $0.68 | Good. This is the orderable name of the part drawn as TCAN1042VDQ1 |
| U2, U4, U5 | TPS62067DSGR | C7098 | Not confirmed | 900 to 2,100, about $0.62 | Good |
| U6 | W25Q32JVSSIQ | C82344 or C179173 | Extended | Both have stock on some snapshots, C82344 out of stock on others | Pick the one in stock at order time |
| J11 | USB4105-GF-A | C3020560 | Extended | 0 at LCSC on one page; variant -120 (C5184243) shows 1,892 at JLCPCB | Verify variant and footprint match |
| U15 | AP2112K-3.3TRG1 | C51118 | Extended | Over 100,000 | Good |

Not yet checked: USBLC6-2SC6, 100 MHz oscillator, inductors, TVS, fuse, MOSFET, LEDs, switches, connectors and the passives (resistors and capacitors are usually basic parts).

Three things to decide from this check: the FPGA stock is the thinnest (consider the -I variant or ordering parts early), and the ESP32-C3-MINI-1-N4 and FT2232HL are both flagged as not recommended for new designs. They still work and are stocked, but note it in the write-up as a known sourcing risk.
