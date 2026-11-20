# Sheet 1: Power input and 5 V rail

Goal: take 9 to 36 V DC from a screw terminal and make a clean 5 V rail that survives wrong polarity and voltage spikes.

All values below are starting points. Check each one against the datasheet or TI WEBENCH before you commit it to the schematic, and write what you checked in the design log.

## Signal flow

Screw terminal J1 -> fuse F1 -> reverse-polarity protection Q1 -> TVS D1 -> input capacitors -> LMR36015 buck U1 -> 5 V rail

## What to draw

| Ref | Part | Purpose | To verify |
|-----|------|---------|-----------|
| J1 | 2-pin screw terminal, 5 mm | 9 to 36 V in | Current rating above fuse |
| F1 | Resettable fuse or SMD fuse | Stops a short from burning the board | Hold current about 1.5 A, voltage rating above 36 V |
| Q1 | P-channel MOSFET, reverse-polarity guard | Blocks wrong polarity with low loss | Vds with clear margin over the TVS clamping voltage (start by looking at 80 to 100 V parts), check gate-source limit and add a Zener across it |
| D1 | Bidirectional TVS diode, 36 V standoff (SMBJ36CA family) | Clamps spikes in both directions. Bidirectional so a reversed supply does not short through it and blow the fuse | Check the maximum clamping voltage against the LMR36015 66 V transient limit and against Q1's Vds rating |
| C1 | Bulk input capacitor | Holds up the input | Voltage rating at least 50 V |
| U1 | LMR36015 | 36 V to 5 V buck | Use the WEBENCH design for inductor, output capacitors and feedback resistors |
| TP1, TP2, TP3 | Test points | VIN, 5 V, GND | |

Also add: power-good output of U1 brought out as a net named `PG_5V`. It will start the sequencing chain on the next sheet.

## Checks before you move on

- Run ERC in KiCad with zero errors.
- Every net has a name you would recognise on the scope.
- Write the chosen values and the reason in `DESIGN_LOG.md`.

Send me the sheet when done. I'll review it before we do sheet 2 (FPGA rails with sequencing).

## As drawn (2026-11-17)

Drawn in KiCad 10 and checked: ERC clean (0 errors, 0 warnings). Q1 has its drain on the fuse side and its source on the load side. D2 (12 V Zener) has its anode on the gate node and its cathode on VIN. R1 is 100 k from gate to ground, F1 is 2 A, C1 is 10 uF 50 V as a placeholder until the buck design sets the input capacitors.

Footprints: cleared on all symbols (2026-11-17). Real footprints are assigned in Phase 3 preparation.

## Buck converter as drawn (2026-11-19)

U1 LMR36015ARNXR (TI Ultra Librarian symbol, kept in hardware/ul_LMR36015ARNXR). Values follow the datasheet 5 V design table (SNVSB49D, Table 10-3): L1 10 uH, C2 and C3 22 uF, R2 100 k and R3 24.9 k feedback divider. C4 100 nF boot capacitor from BST to SW, C5 1 uF on VCC, EN tied to VIN, NC (pin 3) joined to SW as the datasheet asks, R4 100 k pull-up from PG to 5 V. Nets: `VIN`, `SW`, `5V`, `PG_5V`.

ERC: 0 errors, 4 warnings. The two errors the TI symbol caused (VCC and SW are typed as power inputs, but the chip drives them) are cleared with a PWR_FLAG on each net. The 4 warnings are expected: no footprint library yet (U1, L1), and unspecified pin types on BST and NC, which are joined to passive and power pins on purpose.

Open: input capacitor value near U1 VIN pins (datasheet section after Table 10-3), check the A, B and FB suffix variants (datasheet section 14), and confirm Vref in the electrical table.
