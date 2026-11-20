# Interview notes

Why each design choice was made, written so it can be said out loud in an interview. Added to as each sheet is finished. At the end of Phase 2 this becomes the written explanation of the schematic.

## Sheet 1: power input and 5 V rail

**Why a P-MOSFET for reverse-polarity protection, not a diode?**
A series diode drops about 0.5 to 0.7 V and dissipates that voltage times the current as heat. A MOSFET only drops current times Rds(on), which is milliohms, so the loss is far smaller. That matters on a 9 V input.

**Why a Zener (D2) and a 100 k resistor (R1) on the gate?**
R1 pulls the gate to ground, which turns the P-MOSFET on once power is applied. At 36 V in, that would put 36 V across gate and source, which exceeds most MOSFETs' gate rating (often 20 V). The 12 V Zener clamps the gate voltage.

**Why a bidirectional TVS (SMBJ36CA), and why before the MOSFET?**
The TVS absorbs spikes on the input. Bidirectional means a reversed supply cannot conduct through it and short out. It sits before the MOSFET so spikes are clamped before they reach anything fragile. The fuse is the last-resort protection.

**Why does Q1 need an 80 to 100 V rating on a 36 V input?**
The TVS clamps spikes to a voltage well above 36 V (roughly 58 V, check its datasheet). Q1 must survive the clamped spike, not just the normal supply.

**Why a buck to 5 V first, then three more converters?**
Going straight from 36 V to 1.0 V is inefficient and hard for one converter. Two stages are more efficient, and separate converters give independent rails and let each converter's power-good pin enable the next, which enforces the FPGA power-up order.

**Why 100 nF on BOOT and 1 uF on VCC?**
The LMR36015 datasheet requires them. The bootstrap capacitor powers the high-side switch's gate drive. The VCC capacitor decouples the chip's internal supply. Answer: I followed the manufacturer's required values.

**How do R2 (100 k) and R3 (24.9 k) set 5 V?**
They form a divider into the FB pin. The chip adjusts the output until FB reaches its internal reference, so Vout = Vref x (1 + R2/R3). With these values that is about 5 V if Vref is 1 V (confirm Vref in the datasheet electrical table).

**Why 10 uH and 2 x 22 uF?**
They come from TI's validated 5 V design table (24 V input example). Be ready to compute ripple: inductor ripple current = (Vin - Vout) x duty cycle / (L x switching frequency).

**Why is EN tied to VIN?**
The datasheet allows EN to connect directly to VIN, so the converter starts whenever power is applied. Simple, no extra parts.

**Why did ERC flag VCC and SW, and what did you do?**
TI's symbol types VCC and SW as power inputs, so ERC reported that nothing drives them. In reality the chip drives both (VCC is its internal supply, SW its switching output). I checked the datasheet to confirm, then put a PWR_FLAG on each net. A cleaner fix is to edit the symbol's pin types.

**What are the remaining ERC warnings?**
No footprint library yet (assigned in Phase 3), and unspecified pin types on BST and NC. NC is joined to SW on purpose because the datasheet asks for it. I reviewed each warning and left the ones that are expected.

## Sheet 2: FPGA rails (three TPS62067 converters)

**Why three separate converters instead of one?**
The Spartan-7 needs three supplies (1.0 V core, 1.8 V auxiliary, 3.3 V I/O) and an order to bring them up. Separate converters give independent rails, and each converter's power-good output can enable the next one.

**How is each output voltage set?**
A resistor divider into the FB pin. The chip adjusts the output until FB equals its 0.6 V reference, so Vout = 0.6 x (1 + R1/R2). For 1.0 V: 120k over 180k. For 1.8 V: 360k over 180k. For 3.3 V: 680k over 150k gives 3.32 V. R2 stays within the datasheet's 120 to 360 k range.

**What is the small capacitor across R1 (Cff) for?**
It adds a zero to the feedback loop and the datasheet requires it for good regulation. Value from Cff = 1 / (2 pi x R1 x 25 kHz), rounded to a standard value.

**Why 1 uH and 10 uF, and why 22 uF on the 3.3 V rail?**
The chip's internal compensation is tuned for 1 uH with 10 uF. Ceramic capacitors lose capacitance under DC bias, so the datasheet suggests 22 uF for outputs above 2 V.

**How does the power-up sequence work, and why do the PG pull-ups go to 5 V?**
PG_5V (from the first buck) enables the 1.0 V converter, whose PG enables the 1.8 V one, whose PG enables the 3.3 V one. PG is open drain, so it needs a pull-up. The next converter's EN needs at least 1 V to read high, which a 1.0 V pull-up would only just reach. The datasheet allows the pull-up on any rail at or below AVIN, so 5 V is used.

**Why does EN have to be connected?**
The datasheet says EN must be terminated and never left floating.

**How do signals cross between sheets?**
Net labels are local to a sheet. A hierarchical label inside the sub-sheet and a matching pin on the sheet box carry a signal across. The sub-sheet takes 5V and PG_5V in and gives 1V0, 1V8 and 3V3 out.

**Why PWR_FLAGs again?**
ERC only knows pin types. The TI symbol types SW as a power input, and the 5 V net has no pin typed as a power output, so ERC reports them as undriven. A PWR_FLAG marks the net as intentionally powered and adds nothing to the board.

**What went wrong and what did you learn?**
A first paste dropped a second copy of rail 1 exactly on top of the original. I found it from doubled reference text, proved it by moving a part, deleted the stack and rebuilt the rail, then checked the saved file for duplicate references. Lesson: paste once, and verify by reading the file, not only the drawing.

## Sheet 3: FPGA core (Spartan-7 XC7S25)

**Why does an FPGA need an external flash chip?**
The Spartan-7 stores its logic in volatile SRAM, so it forgets everything when power is removed. At every power-up it reloads itself from external flash over SPI. The flash holds the bitstream, the compiled file that makes my RISC-V core. SRAM-based FPGAs trade non-volatility for density and speed.

**What are the M pins?**
Three mode pins read at power-up that select the configuration method. 001 is Master SPI: the FPGA drives the clock and reads the flash itself. I tied M0 high and M1 and M2 low.

**Why pull-ups on PROGRAM_B, INIT_B and DONE?**
They are open-drain pins, so they can only pull low. A resistor to 3.3 V provides the high level.

**Why 10k pull-ups on the flash CS, WP and HOLD pins?**
Chip select must stay high until the FPGA takes control. WP# and HOLD# must not float low in single-bit mode or the flash stops responding.

**How did you decide how many decoupling capacitors?**
From AMD's UG483 table per supply. I skipped the large bulk capacitors because each TPS62067 converter allows only 22 uF effective on its output and its own output capacitor already acts as bulk. That is a deliberate deviation, recorded as D-019, with a plan to measure rail noise at bring-up.

**What went wrong and what did you learn?**
The Ultra Librarian symbol typed the ground pins as power outputs, so ERC reported ground pins driving each other. I changed them to power inputs in the library and the embedded copy. Later a file write of mine delivered an old copy over the new one and your sheet temporarily lost parts. I caught it from the ERC (labels with only one pin), restored from my copy and verified the file byte for byte. Lesson: verify a written file by reading it back.

**Why global labels for the signals and hierarchical pins for power?**
About 40 peripheral signals would need 40 sheet pins on the root page. Global labels connect by name across sheets. Power rails stay hierarchical so the power path is visible on the root page.

