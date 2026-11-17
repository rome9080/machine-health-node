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
| Q1 | P-channel MOSFET, reverse-polarity guard | Blocks wrong polarity with low loss | Vds at least 60 V, check gate-source limit and add a Zener across it |
| D1 | TVS diode, 36 V standoff class (SMBJ36A family) | Clamps spikes | Clamping voltage must stay below the LMR36015 66 V transient limit |
| C1 | Bulk input capacitor | Holds up the input | Voltage rating at least 50 V |
| U1 | LMR36015 | 36 V to 5 V buck | Use the WEBENCH design for inductor, output capacitors and feedback resistors |
| TP1, TP2, TP3 | Test points | VIN, 5 V, GND | |

Also add: power-good output of U1 brought out as a net named `PG_5V`. It will start the sequencing chain on the next sheet.

## Checks before you move on

- Run ERC in KiCad with zero errors.
- Every net has a name you would recognise on the scope.
- Write the chosen values and the reason in `DESIGN_LOG.md`.

Send me the sheet when done. I'll review it before we do sheet 2 (FPGA rails with sequencing).
