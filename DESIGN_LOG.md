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

## Open decisions

- **O-1:** onboard USB-JTAG and UART bridge (more parts, but you only need a USB-C cable) or a JTAG header with an external programmer (simpler board, but you must buy a programmer). Recommended: onboard.

## Change history

| Date | Change |
|------|--------|
| 2026-11-16 | New log started for the 4-layer scope, D-001 to D-007. Old log archived. |
| 2026-11-16 | Phase 1 part selection: D-008 to D-011 and docs/PARTS.md added. |
