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
- **Candidate:** AMD Spartan-7 XC7S25 in CSGA225 (0.8 mm pitch). verify package and pitch.
- **Fallback:** Lattice iCE40UP5K in QFN-48, which is hand-solderable with hot air.
- **Open:** depends on the assembly route (D-005).
- **Status:** Open

## D-005: Assembly route
- **Options:** reflow with stencil and hot plate or hot air, pay a service for the BGA and QFN parts only, or use the iCE40 fallback and hand-solder
- **Status:** Open, needs user input on budget and tools

## D-006: Add-on strategy
- **Choice:** hardware add-ons (power input, current channel, buses, test access) all go on the first board revision. Software and process add-ons are built in tiers.
- **Reason:** a board cannot gain hardware after fabrication, but software can grow at any time.
- **Tiers:** see `docs/PROJECT_PLAN.md`.
- **Status:** Decided 2026-11-16

## D-007: Safety rule
- **Choice:** the board never connects to mains. Current is measured with split-core CT clamps only.
- **Status:** Decided 2026-11-16

## Change history

| Date | Change |
|------|--------|
| 2026-11-16 | New log started for the 4-layer scope, D-001 to D-007. Old log archived. |
