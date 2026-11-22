# Design Log: Industrial Machine Health Node

One entry per non-obvious decision: options, choice, reason, status. Figures marked "verify" must be checked against a datasheet before use.

## D-001: Change of scope from a 2-layer sensor node to a 4-layer machine-health board
- **Options:** keep the small BLE sensor node, or build a larger board with a real purpose
- **Choice:** larger board. The small node (archived in `docs/archive/`) did not show enough depth.
- **Reason:** a 4-layer FPGA board with multi-rail power, mixed-signal inputs and industrial interfaces exercises the skills that hardware employers screen for.
- **Status:** Decided 2026-11-16

## D-002: Problem statement
- **Choice:** predict machine failure from vibration and motor current, on the device.
- **Reason:** unplanned downtime is a real problem, and the sensing, analysis and communication blocks map to hardware, firmware and software roles.
- **Status:** Decided 2026-11-16, assumed from the earlier "FPGA condition monitor" option. Change if a different problem is preferred.

## D-003: Why 4 layers
- **Reason:** BGA escape routing, three FPGA supply rails on planes, controlled-impedance USB, clean return paths for mixed-signal inputs.
- **Stackup:** signal / ground / power / signal (verify against the fabricator's stackup before layout).
- **Status:** Decided

## D-004: FPGA family
- **Choice:** AMD Spartan-7 XC7S25-1CSGA225C (commercial, 0 to 85 C) for the first prototype. An industrial-grade -1CSGA225I exists in the same 225-ball package.
- **Fallback:** Lattice iCE40UP5K in QFN-48, used only if the Spartan-7 cannot be sourced.
- **Checked 2026-11-16 against distributor listings:** 225-ball CSGA, 13 x 13 mm, 150 I/O, 23,360 logic cells, status Active. VCCINT 0.95 to 1.05 V for -1 devices, VCCAUX 1.71 to 1.89 V. Each rail must ramp from 0 V to 90 percent in 0.2 to 50 ms. In stock at Digi-Key and LCSC, but stock and price vary by region and day (about $16 at LCSC to about 36 to 41 EUR at Digi-Key).
- **Still to verify:** 0.8 mm ball pitch (package guide UG475), recommended power-on order (DS189), and whether the part is in the assembler's parts library or must be consigned.
- **Power design references:** TI reference design TIDUE63 and the Renesas Spartan-7 power application note PM-096.
- **Status:** Part chosen, verification items open

## D-005: Assembly route
- **Options:** hand or home reflow, or a PCB assembly service
- **Choice:** assembly service (JLCPCB or PCBWay). Hand-soldering is not possible for this build.
- **Reason:** both services advertise BGA assembly and accept consigned parts. Final pick between them is made at part selection, based on FPGA availability and total cost.
- **Status:** Decided 2026-11-16

## D-006: Add-on strategy
- **Choice:** hardware add-ons (power input, current channel, buses, test access) all go on the first board revision. Software and process add-ons are built in tiers.
- **Reason:** a board cannot gain hardware after fabrication, but software can grow at any time.
- **Tiers:** see `docs/PROJECT_PLAN.md`.
- **Status:** Decided 2026-11-16

## D-007: Safety rule
- **Choice:** the board never connects to mains. Current is measured with split-core CT clamps only.
- **Status:** Decided 2026-11-16

## D-008: Demo machine
- **Choice:** an AC box or pedestal fan, with a small weight taped on one blade to create imbalance on demand.
- **Reason:** cheap, safe, easy to fault on purpose, and low current, so a small clamp sensor works. It gives healthy and faulty data for the analysis and ML add-ons. Input ranges are sized so a pump can be monitored later.
- **Safety:** measure current with a commercial line-splitter cord and a split-core clamp so no mains wire is cut or exposed.
- **Status:** Decided 2026-11-16

## D-009: Sensing parts
- **Vibration:** LSM6DSO. Accelerometer output rate up to 6.66 kHz covers fan vibration.
- **Current:** ADS131M04, four simultaneous channels, three used for CT clamps and one spare.
- **Status:** Decided 2026-11-16. See `docs/PARTS.md` for the facts checked.

## D-010: Power architecture
- **Choice:** 9 to 36 V in, LMR36015 buck to 5 V, then three step-down converters for the FPGA's 1.0 V, 1.8 V and 3.3 V rails with enable chaining for sequencing.
- **Reason:** LMR36015 accepts up to 60 V with transient tolerance, so the 9 to 36 V input has margin. Discrete converters make the sequencing visible and measurable, which is the point of add-on H2.
- **Open:** exact converter for the three rails, and the recommended power-on order from the FPGA datasheet.
- **Status:** Decided, with open items

## D-011: Industrial interfaces
- **Choice:** RS-485 with THVD14xx and CAN FD with TCAN1042 family. The CAN controller is built in RTL on the FPGA.
- **Status:** Decided 2026-11-16, exact variants to pick in Phase 2

## D-012: Onboard USB programmer (was O-1)
- **Choice:** USB-C with an onboard USB-JTAG and UART bridge.
- **Reason:** more parts, but only a USB-C cable is needed, no separate programmer to buy.
- **Status:** Decided 2026-11-16

## D-013: Verification of the FPGA (closes the open items in D-004)
- **Ball pitch:** CSGA225 is 13 x 13 mm at 0.8 mm pitch, per the 7 series selection guide.
- **Power-on order:** VCCINT, VCCBRAM, VCCAUX, then VCCO, with power-off in reverse (DS189). VCCINT and VCCBRAM share the 1.0 V rail. VCCAUX (1.8 V) and VCCO (3.3 V) differ, so they stay separate rails.
- **Assembler:** JLCPCB. LCSC lists XC7S25-1CSGA225C as C1521699 and marks it a JLCPCB extended part. Stock figures conflict between pages, so confirm at order time and keep the consign route as a fallback.
- **Open:** the industrial XC7S25-1CSGA225I is listed at a similar price. Decide in Phase 2 whether to use it.
- **Status:** Decided 2026-11-16, stock to recheck at order time

## D-014: Rail converters
- **Choice:** TI TPS62067 for each of the 1.0 V, 1.8 V and 3.3 V rails, fed from the 5 V rail. Each has an enable pin and an open-drain power-good output, so the power-good of one rail enables the next (1.0 V, then 1.8 V, then 3.3 V), matching the order in D-013.
- **Checked:** input range 2.9 to 6 V, 2 A, 3 MHz, power-good output, LCSC part C7098.
- **Open:** confirm 1.0 V and 1.8 V load currents once the FPGA design size is known (power estimator, Phase 5).
- **Status:** Decided 2026-11-16

## D-015: Bus transceiver variants
- **RS-485:** THVD1410, the 500 kbps low-EMI speed grade. Modbus RTU runs far below that. THVD1450 (50 Mbps) is the fallback.
- **CAN FD:** TCAN1042VDRQ1 (SOIC-8, 2 Mbps CAN FD, VIO pin so it takes 3.3 V logic from the FPGA, 5 V supply). LCSC C485806, a JLCPCB extended part.
- **ADC:** ADS131M04 stays. Its sample rate is far above the few kSPS needed, so the datasheet revision question no longer matters.
- **Status:** Decided 2026-11-16

## D-016: Sheet 1 power input front end
- **Choice:** fuse F1 (2 A), bidirectional TVS D1 (SMBJ36CA class), P-channel MOSFET Q1 for reverse-polarity protection, gate resistor R1 100 k to ground, 12 V Zener D2 between gate and source, 10 uF 50 V input capacitor.
- **Reason:** the TVS is bidirectional so a reversed supply cannot short through it. The Zener keeps the gate-source voltage inside the MOSFET's limit at 36 V in. Q1's drain faces the input and its source faces the load, so the body diode conducts only when polarity is right.
- **Checked:** ERC clean. Connectivity read from the saved schematic file.
- **Open:** pick Q1 with a Vds rating that clears the TVS clamping voltage, confirm its on-resistance at the gate voltage the circuit gives at 9 V in. Footprint fields cleared 2026-11-17; real ones are assigned in Phase 3.
- **Status:** Drawn 2026-11-17, review items open

## D-017: Sheet 1 buck converter
- **Choice:** LMR36015ARNXR (A variant) making 5 V. L1 10 uH, C2 and C3 22 uF, R2 100 k and R3 24.9 k feedback, C4 100 nF boot, C5 1 uF on VCC, R4 100 k pull-up on PG (net PG_5V), EN tied to VIN.
- **Reason:** values taken from TI's validated 5 V design table (SNVSB49D). PG_5V will enable the next converter in the 1.0 V, 1.8 V, 3.3 V sequence.
- **Checked:** ERC 0 errors, 4 expected warnings. PWR_FLAGs added on VCC and SW because the TI symbol types those pins as power inputs.
- **Open:** input capacitor near VIN, A/B/FB variant check, Vref from the electrical table.
- **Status:** Drawn 2026-11-19

## D-018: Sheet 2 FPGA rails
- **Choice:** three TPS62067DSGR (2 A, 3 MHz) from the 5 V rail: U2 1.0 V, U4 1.8 V, U5 3.3 V. 1 uH inductors, 10 uF input caps, 10 uF output on 1.0 V and 1.8 V, 22 uF on 3.3 V. Dividers from Vref 0.6 V: 120k/180k, 360k/180k, 680k/150k (3.32 V). Cff set for a 25 kHz zero: 51 pF, 22 pF, 10 pF.
- **Reason:** values from the TPS62065/67 datasheet (SLVS833E): 1 uH and 10 uF are what the internal compensation is tuned for; 22 uF above 2 V because of DC-bias loss; R2 inside 120 to 360 k.
- **Sequencing:** PG_5V enables 1.0 V, its PG enables 1.8 V, its PG enables 3.3 V. PG pull-ups go to 5 V, not the rail itself, because EN needs at least 1 V to read high.
- **Checked:** ERC leaves only the 3 expected unconnected sheet outputs. PWR_FLAGs on 5V and the three SW nets.
- **Open:** total effective capacitance per rail including FPGA bypass caps must stay within 4.5 to 22 uF; inductor Isat and part choice; input cap voltage rating (10 V or more); 5 V load budget against the LMR36015 1.5 A limit; E24 resistor stock at JLCPCB.
- **Status:** Drawn 2026-11-19

## D-019: Sheet 3 FPGA decoupling
- **Choice:** AMD UG483 small capacitors only. 1V0: 3 x 4.7 uF + 4 x 0.47 uF. 1V8: 1 x 4.7 uF + 2 x 0.47 uF. 3V3: 4 x 4.7 uF + 8 x 0.47 uF (1 x 4.7 uF and 2 x 0.47 uF per bank). The 47 and 100 uF bulk capacitors in the table are skipped.
- **Reason:** each TPS62067 allows 4.5 to 22 uF effective on its output, and its own 10 or 22 uF output capacitor already acts as the bulk. UG483's full per-bank count (2 x 4.7 uF + 4 x 0.47 uF) on the 3.3 V rail would pass that limit. This is thinner than AMD's table.
- **Open:** check effective capacitance per rail with DC-bias derating; revisit if bring-up shows noise on the rails.
- **Status:** Drawn 2026-11-20

## D-020: Sheet 3 configuration, clock, I/O plan
- **Choice:** Master SPI x4 boot (M[2:0] = 001) from a W25Q32JVSS 32 Mbit flash on 3.3 V. Pull-ups: 4.7k on PROGRAM_B, INIT_B, DONE; 10k on QSPI_CS, QSPI_IO2, QSPI_IO3. 100 MHz oscillator to a clock-capable (MRCC) pin. 4 LEDs, 2 buttons, a 1x8 GPIO header and a 1x6 JTAG backup header. 40 signal pins assigned in banks 14, 15 and 34, connected by global labels; power rails stay hierarchical.
- **Reason:** the Spartan-7 is SRAM-based and loads its bitstream from external flash at every power-up. 32 Mbit holds the roughly 10 Mbit bitstream with room for RISC-V firmware. Global labels avoid roughly 40 sheet pins on the root page.
- **Checked:** ERC leaves only unwired spare IO pins, four JTAG labels waiting on the programmer sheet and footprint-library warnings. GND pins retyped from power output to power input in the symbol (library and embedded copy).
- **Open:** verify VREFP/VREFN against UG480; -C vs -I grade; real 100 MHz oscillator part; bank swaps at layout; no-connect flags on spare pins at the end of Phase 2.
- **Status:** Drawn 2026-11-20

## D-021: Sheet 4 sensors
- **Choice:** LSM6DSO accelerometer on SPI (mode: SDx/SCx grounded, VDD and VDDIO on 3V3, 100 nF + 10 uF on VDD, 100 nF on VDDIO). TMP117 on I2C (ADD0 to GND, 100 nF, 4.7k pull-ups on SDA, SCL, ALERT). ADS131M04 on SPI (AVDD and DVDD 1 uF each, CAP pin 220 nF to GND, SYNC/RESET pulled up 10k and driven by the FPGA, unused AIN3 pair to GND, CLKIN 8.192 MHz from the FPGA). Three current-clamp inputs: 2-pin header, 33 ohm burden across the leads, 1k in series on each lead, 10 nF across the filtered pair. Signals connect by global labels; 3V3 enters through one hierarchical pin.
- **Reason:** the 33 ohm burden turns a 50 mA CT secondary into about 1.65 V peak for the ADC; the 1k + 10 nF pair is the anti-alias filter. The ADS131M04 samples all channels at the same instant, which is what real-power measurement needs.
- **Open:** CT clamp part (100 A : 50 mA assumed) and burden value; ESD/TVS on the clamp inputs; decoupling values against the final datasheets at the review.
- **Status:** Drawn 2026-11-20

## D-022: Sheet 5 communications
- **Choice:** RS-485 THVD1410 on 3V3 with half-duplex direction pin (RE and DE tied), 10 ohm series resistors, 120 ohm termination behind a jumper, 3-pin screw terminal (A, B, GND). CAN FD TCAN1042V on 5V with VIO on 3V3, STB pulled low by 10k (normal mode by default), 120 ohm termination behind a jumper, 3-pin terminal. BLE: ESP32-C3-MINI-1 with EN RC (10k, 1 uF), IO2, IO8, IO9 pulled high, UART to the FPGA, FPGA control of EN and BOOT for reflashing. Signals by global labels; 3V3 and 5V by hierarchical pins.
- **Reason:** the THVD1410 has built-in fail-safe biasing so no bias resistors are needed; the jumper lets the node sit mid-bus without termination. TCAN1042V takes 3.3 V logic through VIO, so the FPGA connects directly. The ESP32 handles the BLE stack, which would be impractical in RTL.
- **Open:** RS-485 TVS (SM712 class) not fitted on the sheet yet; CAN ESD/common-mode choke not added (transceiver is rated for 58 V bus faults and 15 kV IEC ESD); ESP32-C3-MINI-1 pin and strapping details to be checked against the Espressif datasheet; ESP32 footprint library to be added at footprint assignment; JLCPCB stock for the -N4 variant.
- **Status:** Drawn 2026-11-20

## Open decisions

- None blocking Phase 2. Still candidates, to confirm while drawing the schematic: ESP32-C3-MINI-1, TMP117, QSPI flash, clock, USB bridge chip, CT clamps.

## Change history

| Date | Change |
|------|--------|
| 2026-11-16 | New log started for the 4-layer scope, D-001 to D-007. Old log archived. |
| 2026-11-16 | Phase 1 part selection: D-008 to D-011 and docs/PARTS.md added. |
| 2026-11-16 | D-012 to D-015: programmer, FPGA verification, assembler, rails, bus parts. Phase 1 closed. |
| 2026-11-17 | D-016: schematic sheet 1 (power input front end) drawn and committed. |
| 2026-11-19 | D-017: sheet 1 buck converter drawn, TI symbol and footprint added to hardware/. |
| 2026-11-19 | D-018: sheet 2 FPGA rails (three TPS62067 converters) drawn, TI symbol and footprint added to hardware/. |
| 2026-11-20 | D-019, D-020: sheet 3 FPGA core drawn (power pins, decoupling, flash boot, clock, I/O plan); XC7S25 symbol and footprint added to hardware/. |
| 2026-11-20 | D-021: sheet 4 sensors drawn (LSM6DSO, TMP117, ADS131M04, three CT inputs); Ultra Librarian symbols and footprints added to hardware/. |
| 2026-11-20 | D-022: sheet 5 communications drawn (RS-485, CAN FD, ESP32-C3 BLE); THVD1410, TCAN1042V and Espressif symbol libraries added to hardware/. |
| 2026-11-20 | D-023: sheet 6 programmer drawn (USB-C, ESD, AP2112K LDO, FT2232HL with JTAG on channel A and UART on channel B); JTAG now wired to FPGA core at root. |
| 2026-11-20 | D-024: sheet 7 test points drawn (rails, grounds, eight key signals). |
| 2026-11-20 | D-025: first full ERC (131 errors to 5): no-connect flags on spare FPGA/ESP32 pins, FPGA mode pins M2/M1 tied low (M0 high = master SPI), PWR_FLAGs on rails and VBUS, duplicate refs fixed. Remaining: 4 harmless GND pin-type errors on U7/U9 (to exclude) plus footprint warnings. |
| 2026-11-21 | D-026: footprints assigned to all parts (0402 R and small C, 0603/0805/1206 for larger C, 0603 LEDs, SOIC-8 flash, GCT USB4105 USB-C, test pads, placeholder Taiyo-Yuden NR inductors); vendor footprint folders registered in fp-lib-table. Still open: ESP32-C3 footprint library, Y1 100 MHz oscillator (DIP-14 symbol vs SMD part), real inductor choice, USB-C shield pad name check. |
| 2026-11-21 | D-027: ESP32-C3-MINI-1 footprint added from Espressif's kicad-libraries (hardware/Espressif.pretty, registered as PCM_Espressif); switches SW1/SW2 use the 6x6x5 mm SMD tactile footprint (LCSC-linked). ERC: 0 errors, remaining warnings are symbol-library mismatches and vendor pin-type quirks. |
| 2026-11-21 | D-028: first JLCPCB/LCSC sourcing check of the main parts. FPGA stock is thin; ESP32-C3-MINI-1-N4 and FT2232HL are flagged not recommended for new designs but are stocked. Details in docs/PARTS.md. |
| 2026-11-21 | D-029: second sourcing pass covering oscillators, crystal, ESD/TVS, zener, MOSFET, LEDs, inductors, fuse, switches. Raised four issues for the design decisions step: F1 voltage rating (common 2 A 1206 fuses are 6 V parts), Y1 placeholder, inductor saturation current, Q1 gate limit. Details in docs/PARTS.md. |
| 2026-11-21 | D-030: schematic fixes from the sourcing pass. F1 standard 2 A 63 V fuse, D2 10 V zener, Y1 4-pin SMD footprint with renumbered pins, inductor values updated with saturation current. Input range set as 12 V nominal, 24 V survivable. Details and caveats in docs/PARTS.md. |
| 2026-11-21 | D-031: power budget (estimates). 1V0 about 0.3 A, 1V8 about 0.1 A, 3V3 about 0.8 A and 5V about 0.9 A with margin. L1 and L2/L4/L5 stay as chosen (about 3x margin on the 3V3 inductor, L1 under the 1.2 A threshold). FPGA load to be replaced with the AMD power estimator figure after synthesis. |
| 2026-11-21 | D-032: CT clamp and pre-layout checks. Corrected an earlier error: the board already has a 33 ohm burden per channel, so the clamp must be current-output (SCT-013-000, 100 A : 50 mA), not voltage-output (SCT-013-005). 50 mA is RMS, so full 100 A would give about 2.3 V peak and clip the ADC (+/-1.2 V); channel is usable to about 51 A RMS, fine for the fan. ADS131M04 uses an internal 1.2 V reference with no external reference pins, so there is no reference wiring to check. ESP32-C3 strapping checked against Espressif's table: IO2 and IO8 pulled high and IO9 pulled high with FPGA able to pull it low gives SPI boot normally and download boot on demand. Firmware must not hold BLE_BOOT low at reset unless entering download mode. |
| 2026-11-21 | D-033: first Update PCB from Schematic raised 3 errors on Q1: the KiCad Q_PMOS symbol numbers its pins G, S, D but the SOT-23 footprint has pads 1, 2, 3. Renumbered the symbol to G=1, S=2, D=3 (standard SOT-23 MOSFET order; to confirm against the AO3401A datasheet pinout). |
| 2026-11-21 | D-034: Update PCB from Schematic gave 6 warnings: the TPS62067 footprint (DSG0008A) has two thermal vias named V that no symbol pin matches. Renamed them to 9 so they join the exposed pad net (PAD, tied to ground). Via-in-pad: check JLCPCB via-in-pad rules (0.2 mm drill) at the DFM review. |
| 2026-11-21 | D-035: Phase 3 rough placement done (100x80 mm, 4 layers, connectors on edges, FPGA central, decoupling under FPGA). Design rules set from JLCPCB 4-layer limits (0.127 mm min track/space, 0.45/0.20 via). Copper-to-hole clearance 0.15 (USB-C J11 footprint is 0.194). Malformed-courtyard check downgraded to warning (U7, U10, U11 library courtyards are broken). J4/J6 moved to 7 mm pitch. DRC: 0 errors besides 499 unrouted connections. Test points TP1-TP15 not yet placed. |
| 2026-11-21 | D-036: Placement finalised. FT2232 decoupling caps moved next to U13; crystal load caps next to Y2; test points TP1-TP15 placed as a block at top middle; CT analog front end moved next to ADC U9 and CT connectors (burden resistors at connectors, 1k + 10 nF filters above the ADC, ADC supply caps along its top). Reason: keep the noise-sensitive input path short. DRC 0 errors besides unrouted. |
