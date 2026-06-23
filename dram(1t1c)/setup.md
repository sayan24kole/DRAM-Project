# 1T1C Dram setup in microcap and hardware

## Description
This repository contains the design, Micro-Cap simulation, and physical breadboard implementation of a 1-Transistor 1-Capacitor (1T1C) DRAM cell. Online resources for physical, breadboard-level 1T1C DRAM implementations are scarce. This project provides a fully documented, hardware-verified design that can be scaled to multi-bit arrays (e.g., 32-bit or 64-bit).

## Repository Contents
- `[Name of your .cir file].cir` - Micro-Cap simulation file.
- `[Name of schematic image file]` - Micro-Cap circuit schematic.
- `[Name of graph image file]` - Simulation waveform outputs.
- `[Name of breadboard image file]` - Photographs of the physical breadboard build.
- `OPERATING_INSTRUCTIONS.md` - Detailed read and write instructions.

## Bill of Materials (Hardware Requirements)
To replicate the physical circuit, the following components are required:
- **1x Capacitor:** >200µF (sized large specifically for visual observation of charge/discharge)
- **1x MOSFET:** 2N7000 N-Channel
- **1x Tri-state Buffer:** 74HC244 or 74S244 (74HC244 is preferred for better rail-to-rail voltage)
- **1x Comparator:** LM339
- **1x Potentiometer:** To set the 0.7V - 1.3V reference voltage
- **Resistors:** Values ranging from 1kΩ to 10kΩ
- **Misc:** LEDs, manual switches, breadboard, and jumper wires

## Circuit Architecture & Design Considerations

### 1. Bitline Driver (Tri-state Buffer)
The circuit uses a 74x244 buffer to drive the bitline. 
*Note on IC Selection:* If using the 74S244, the maximum high output is approximately 3.6V, and the minimum low output is 0.4V. If using the 74HC244, the output ranges from 0.2V to 5V. The 74HC244 is recommended for better logic level margins, though the 74S244 is functional provided the comparator reference voltage is calibrated correctly.

### 2. Sense Amplifier (Comparator)
An LM339 comparator acts as the sense amplifier. The inverting input (-IN) must be set below the buffer's maximum high output (3.6V) but above its minimum low output (0.4V). A reference voltage of 0.8V (acceptable range: 0.7V to 1.3V) is established using a potentiometer. Input voltages below 0.7V evaluate as Logic 0, and voltages above evaluate as Logic 1. An LED is placed at the output of the LM339 (with a series resistor) for visual output.

## Simulation Setup (Micro-Cap)
Micro-Cap does not feature standard manual switches, so Piecewise Linear (PWL) voltage sources are used to simulate switch toggling. 

### Simulation Voltage Sources (V1 - V6):
- **V1 (Buffer Output Enable):** `PWL(0,5 1.99,5 2,0 5,0 5.01,5 12.99,5 13,0 16,0 16.01,5 25,5)`
- **V2 (Data Input):** `PWL(0,0 1.99,0 2,5 5,5 5.01,0 25,0)`
- **V3 (Word Line / MOSFET Gate):** `PWL(0,0 2.49,0 2.5,5 4.5,5 4.51,0 7.99,0 8,5 9,5 9.01,0 13.49,0 13.5,5 15.5,5 15.51,0 18.99,0 19,5 20,5 20.01,0 25,0)`
- **V4 (Sense Amp Reference):** `0.8V DC` (Represents the potentiometer setting)
- **V5 & V6 (Supply Voltage):** `5V DC`

### Graph Waveform Outputs:
- **Graph 1 `v(1)`:** Storage cell voltage.
- **Graph 2 `v(6)`:** Sense amplifier output (Main logic output driving the LED).
- **Graph 3 `v(2)`:** Word line voltage.
- **Graph 4 `v(3)`:** Bitline voltage.

## Hardware Operation Setup
When transitioning from the Micro-Cap simulation to the physical breadboard:
1. Replace **V1, V2, and V3** with manual mechanical switches. Configure them to toggle between 5V (ON) and GND (OFF).
2. Provide **5V DC** to the connections corresponding to **V5 and V6**.
3. Tune the potentiometer to provide **0.8V DC** at the connection corresponding to **V4**.
