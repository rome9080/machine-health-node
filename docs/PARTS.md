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

## Remaining parts (sourcing check part 2, 2026-11-21)

Same caveat as above: part numbers are real, but I could not confirm current stock for every one, so check the JLCPCB BOM tool before ordering. "Basic" parts carry no extra JLCPCB setup fee; "Extended" parts do.

| Ref | Part | LCSC / JLCPCB # | Class | Notes |
|-----|------|-----------------|-------|-------|
| Y1 | 100 MHz 3.3 V CMOS oscillator, 3.2x2.5 mm, 4-pin | C437136 (JGHC, 25 ppm) or C4353707 (YXC OT3225100MJBA4SL, 1.8 to 3.3 V) | Extended | Schematic still has a DIP-14 placeholder. Needs a 4-pin symbol (EN, GND, OUT, VDD) and a 3225 footprint. Avoid the LVDS/HCSL/LVPECL variants that came up in the search, since the FPGA clock pin expects single-ended CMOS |
| Y2 | 12 MHz crystal, 5032, 20 pF | C114436 (YXC X503212MSD2GI) or C6071568 (SST, 894 seen) | Not confirmed | The 27 pF load caps plus board stray capacitance give about 17 to 19 pF, close to the 20 pF the crystal wants. Re-check against the FT2232H datasheet |
| U14 | USBLC6-2SC6 | C7519 (ST original) | Extended | Cheap clones also exist (C2687116, C2827654) |
| D1 | SMBJ36CA TVS | C330067 (Jiejie) or C117600 | Extended | Many equivalent makers, all listed at fractions of a cent |
| D2 | BZT52C12 12 V zener | C353560 (6,450 seen) or C43491 | Not confirmed | Fine |
| D3-D6 | 0603 LEDs | Red C2286 (KT-0603R) | Basic | Yellow C2287 and blue C2288 are Extended. Colours not yet fixed in the schematic |
| Q1 | AO3401A P-channel MOSFET | C15127 (AOS original) | Basic | Gate limit is 12 V, so check that D2 clamps the gate on the real input range |
| L1 | 10 uH, 5x5 mm | C222949 (NRS5040T100MMGJ, 2.1 A) or C223066 (NR5040T100M) | Not confirmed | Footprint matches the NR50xx family. Saturation current still to check against the LMR36015 peak current |
| L2, L4, L5 | 1 uH, 3x3 mm | Candidates: C3221188 (TDK SPM3012T-1R0M-CA), C5997619 (TDK VLS3015CX-1R0M-H, 2.3 A), C22395445 (ABC, 2.1 A) | Extended | Most cheap 1 uH 3x3 parts saturate near 1.4 to 2.8 A. Pick against the TPS62067 current limit before ordering |
| F1 | 2 A resettable fuse, 1206 | C883135 (BHFUSE BSMD1206-200-12V) | Extended | **Open issue.** The common 2 A 1206 fuses (C883134, C2931955, C2892426) are rated for only 6 V. The input has a 36 V TVS, so the fuse needs a voltage rating that matches the real input range |
| SW1, SW2 | 6x6x5 mm tactile switch | C319409 (1TS009A-1800-5000-CT) or C255813 / C255814 / C319411 (other force options) | Not confirmed | Matches the 1TS009 footprint chosen in KiCad |
| C1 | 10 uF 50 V 1206 | C303950 (FH 1206B106K500NT, X7R) | Extended | Ceramic capacitance drops a lot under DC bias, so effective value at 12 V or more will be well below 10 uF |
| C 0.47 uF 0402 | 470 nF 6.3 V X5R | C364319 (Samsung) or C469626 (Yageo) | Not confirmed | 14 of these on the board |
| J11 | USB4105-GF-A | C3020560 (0 stock at LCSC on one page) | Extended | The -120 variant (C5184243) showed 1,892 at JLCPCB. JLCPCB flags assembly difficulty as high for it |
| J1-J10 | Screw terminal and pin headers | not looked up | n/a | Through-hole. JLCPCB's normal service does not place these, so plan to hand-solder them or pay extra |

Not checked: the other capacitor values (10 uF 0805, 22 uF 0805, 4.7 uF 0603, 3.3 uF 0603, 1 uF, 100 nF, 10 nF, 22/27/51/10 pF) and all resistor values. Standard values are usually in the JLCPCB basic library, so the BOM tool will confirm these in one upload at the end of Phase 4.

### New issues this pass raised (to settle in the design decisions step)
1. F1 voltage rating (see above).
2. Y1 needs a real symbol and footprint.
3. Inductor saturation currents for L1, L2, L4, L5.
4. Gate voltage limit of Q1 against the input range.

## Fixes applied to the schematic (2026-11-21)

Decisions: board input is 12 V nominal, designed to survive up to 24 V (the 36 V TVS stays).

| Ref | Change | Why |
|-----|--------|-----|
| F1 | Value now "2A 63V". Part: Walter 1206T2A63V (C354895), a standard (non-resettable) 1206 fuse rated 63 V DC, 50 A interrupt. Footprint unchanged | The resettable 2 A 1206 parts found earlier are 6 V devices |
| D2 | Value now "10V". Part: BZT52C10 (C2103, 9.4 to 10.6 V). Footprint unchanged | The 12 V zener spans 11.4 to 12.7 V, which can go above the 12 V gate limit I recall for the AO3401A (from memory, not checked against the datasheet in this pass). The 100 k gate resistor only passes about 0.14 mA at 24 V, so the zener is not stressed |
| Y1 | Symbol pins renumbered 1 (NC/EN), 2 GND, 3 OUT, 4 Vcc. Footprint now Oscillator_SMD_Abracon_ASE-4Pin_3.2x2.5mm. Part: C437136 or C4353707 | The old DIP-14 footprint could not be a 3.2x2.5 mm part. Pin 1 is left unconnected with its no-connect flag; check pin 1 in the chosen part's datasheet before ordering (many 3225 parts use it as an output-enable with an internal pull-up) |
| L2, L4, L5 | Value now "1uH 2.3A". Part: TDK VLS3015CX-1R0M-H (C5997619, Isat 2.3 A) | TPS62067 current limit is 2.3 A minimum, 2.75 A typical, 3.3 A maximum. TI says the conservative choice is to rate the inductor for the current limit. This part meets the minimum but not the maximum, which is acceptable only if each rail's load stays well under 1 A (not yet checked against a power budget) |
| L1 | Value now "10uH 2.1A". Part unchanged (NRS5040T100MMGJ, 2.1 A) | LMR36015 high-side current limit is 2.0 to 2.8 A. The inductor saturates below the typical limit, so this is acceptable only if the 5 V load stays around 1.2 A or less. Part of the open 5 V load decision |

Footprint of L2, L4, L5 is the 3x3 mm NR-30xx pattern, and the TDK part is 3.0 x 3.0 x 1.5 mm. Confirm pad fit when the layout starts.
