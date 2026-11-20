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

