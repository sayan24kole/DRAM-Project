# Read and Write Operations

# This file contains how to verify the circuit stores 0 and 1

## How to WRITE a '1' (Store High Voltage)
1. **Set V1 switch to GND.** This enables the buffer, allowing the chip to drive the Bit Line.
2. **Set V2 switch to 5V.** This sets the input data to '1'.
3. **Press and hold the V3 switch for 2 seconds.** This applies voltage to the MOSFET gate, turning it on. The 2-second hold provides sufficient time for the large 250µF capacitor to charge.
4. **Release the V3 switch.** This removes voltage from the gate, turning the MOSFET off and trapping the charge inside the capacitor.
5. **Set V1 switch to 5V.** This disables the buffer, disconnecting the chip from the Bit Line so it returns to a resting state.

## How to READ a '1'
1. **Ensure the V1 switch is set to 5V.** The buffer must be OFF for a read operation.
2. **Observe the output LED.** It should currently be OFF.
3. **Press the V3 switch.** 
4. **Result:** The MOSFET opens, allowing the capacitor to discharge its stored voltage onto the Bit Line. The Bit Line voltage spikes past the 0.8V reference threshold of the LM339. The output LED will turn ON and remain lit momentarily while the charge drains through the circuit's resistors.

## How to WRITE a '0' (Erase to Low Voltage)
1. **Set V1 switch to GND.** This enables the buffer.
2. **Set V2 switch to GND.** This sets the input data to '0'.
3. **Press and hold the V3 switch for 2 seconds.** This opens the MOSFET gate. Because the Bit Line is held at 0V, any existing charge in the capacitor is drained to ground.
4. **Release the V3 switch.** This closes the gate, isolating the empty capacitor and storing the '0' state.
5. **Set V1 switch to 5V.** This disables the buffer, returning the Bit Line to its resting state.

## How to READ a '0'
1. **Ensure the V1 switch is set to 5V.** The buffer must be OFF.
2. **Observe the output LED.** It should currently be OFF.
3. **Press the V3 switch.**
4. **Result:** The MOSFET opens. Because the capacitor holds no charge, no voltage is pushed onto the Bit Line (it may pull the voltage down slightly). The LM339 detects that the voltage remains well below the 0.8V threshold, and the output LED remains OFF.
