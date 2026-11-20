# Sheet 2: FPGA rails (hierarchical sheet `FPGA rails`, rails.kicad_sch)

Status: drawn 2026-11-19 from the TPS62065/TPS62067 datasheet (SLVS833E, Rev. E, Oct 2020). ERC: only the 3 expected unconnected sheet pins remain (1V0, 1V8, 3V3), which the FPGA sheet will use.

## What it does
Three TPS62067DSGR step-down converters (2 A each) turn the 5 V rail from sheet 1 into the three Spartan-7 supplies:
1.0 V (VCCINT/VCCBRAM), 1.8 V (VCCAUX), 3.3 V (VCCO). Power-good (PG) of each stage enables the next, so the rails come up in order.

## Datasheet facts used
- VIN 2.9 to 6 V (abs max 7 V): 5 V is fine. AVIN and PVIN tied together.
- Vref = 0.6 V. Vout = 0.6 x (1 + R1/R2). R2 should be 120 k to 360 k; R1 + R2 should stay under about 1 M.
- Cff across R1: Cff = 1 / (2 pi R1 25 kHz).
- Switching frequency 3 MHz, optimized for 1 uH inductor and 10 uF output capacitor. Effective L 0.7 to 1.6 uH, effective COUT 4.5 to 22 uF.
- Use 22 uF output cap for rails above 2 V (DC-bias loss).
- Input cap 10 uF ceramic recommended.
- EN must be terminated (never floating). EN high threshold min 1 V, max 6 V.
- PG is open drain: pull-up to a rail at or below AVIN, sink current max 1 mA. PG is pulled low immediately when EN is low.
- PowerPAD is used as GND. PGND (1), AGND (3) and pad (9) all go to GND.
- 5 V rail current limit and 2.75 A typical switch current limit (ILIMF min 2.3 A, max 3.3 A).

## Per-rail values
| Rail | R1 (VOUT to FB) | R2 (FB to GND) | Vout actual | Cff across R1 | COUT | CIN | L |
|---|---|---|---|---|---|---|---|
| 1V0 | 120 k | 180 k | 1.000 V | 51 pF | 10 uF | 10 uF | 1 uH |
| 1V8 | 360 k | 180 k | 1.800 V | 22 pF | 10 uF | 10 uF | 1 uH |
| 3V3 | 680 k | 150 k | 3.320 V | 10 pF | 22 uF | 10 uF | 1 uH |

Per rail also: PG pull-up 100 k to 5 V.

## Enable chain
- 1V0 stage EN <- PG_5V (buck converter power-good from sheet 1)
- 1V8 stage EN <- PG of 1V0 stage
- 3V3 stage EN <- PG of 1V8 stage
- PG pull-ups go to 5 V, not to the stage's own output: a 1.0 V pull-up would sit right at the EN high threshold (min 1 V).

## Interface (hierarchical labels)
In: `5V`, `PG_5V`. Out: `1V0`, `1V8`, `3V3` (and `PG_3V3` if useful later). GND is a global power symbol.

## Open items
- Total effective capacitance on each rail includes the FPGA bypass caps; keep it inside 4.5 to 22 uF effective (check when FPGA sheet is drawn).
- Inductor: 1 uH, shielded, saturation current to pick against the 2.75 A typical limit; part choice at JLCPCB stock check.
- Input cap voltage rating: pick 10 V or higher (6.3 V parts lose too much capacitance at 5 V).
- Sum of loads on the 5 V rail must stay within the LMR36015 1.5 A limit (budget to do once all sheets are drawn).
- Resistor values (360 k, 680 k, 180 k, 150 k, 120 k) are E24; check JLCPCB stock.

## As drawn (2026-11-19)
File is `rails.kicad_sch` (not fpga-rails). TI symbol and footprint (DSG0008A, 8-pin WSON) kept in hardware/ul_TPS62067DSGR.

| Rail | Chip | Inductor | R1 / R2 | Cff | COUT | CIN | PG pull-up |
|---|---|---|---|---|---|---|---|
| 1V0 | U2 | L2 | R5 120k / R6 180k | C8 51 pF | C7 10 uF | C6 10 uF | R7 100 k |
| 1V8 | U4 | L4 | R11 360k / R12 180k | C14 22 pF | C13 10 uF | C12 10 uF | R13 100 k |
| 3V3 | U5 | L5 | R14 680k / R15 150k | C17 10 pF | C16 22 uF | C15 10 uF | R16 100 k |

Nets: inputs `5V`, `PG_5V`; outputs `1V0`, `1V8`, `3V3`; per-rail `FB_*` and `PG_*`. Enable chain as specified: U2 EN = PG_5V, U4 EN = PG of U2 (`PG_1V0`), U5 EN = PG of U4 (`PG_1V8`). Sheet pins on the root page: `5V` and `PG_5V` wired to sheet 1 nets, with a PWR_FLAG on 5V. PWR_FLAG also on each SW net (the TI symbol types SW as a power input).

Note: a first copy and paste left a hidden duplicate copy of rail 1 under the original. Found from doubled reference text, deleted and rebuilt; verified no duplicate references in the saved file.
