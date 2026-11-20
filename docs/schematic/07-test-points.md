# Sheet 7: test points

**File:** `hardware/testpoints.kicad_sch` (root page 7)

Bare pads for a scope or multimeter: TP1-TP4 rails (5V, 3V3, 1V8, 1V0), TP5-TP7 ground, TP8-TP15 signals (ADC_DRDY, ADC_CLKIN, ACC_INT1, TEMP_ALERT, USB_UART_TX, USB_UART_RX, RS485_DE, CAN_TX).

Rails come in as hierarchical pins on the root sheet (joined to the rail wires with TP_ local labels); signals use global labels. Footprints (1 mm pads) are assigned at the end of Phase 2.
