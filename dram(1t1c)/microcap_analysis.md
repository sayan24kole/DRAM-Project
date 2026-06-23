# Circuit Architecture and Simulation Analysis

This document provides a detailed breakdown of the Micro-Cap schematic, the simulation methodology using Piecewise Linear (PWL) sources, and an in-depth analysis of the resulting waveform graphs.

## 1. Micro-Cap Circuit Breakdown

The schematic is divided into three main functional blocks: the storage cell, the bitline driver, and the sense amplifier.

### The Storage Cell (1T1C)
* **C1 (250µF):** The storage capacitor. It holds the charge representing a logic '1' or '0'. A large value is used here to slow down the charge/discharge times, making it easy to observe the process visually on a breadboard or in simulation.
* **M1 (2N7000):** The access transistor (MOSFET). It acts as a valve between the capacitor and the bitline. The gate is controlled by the Word Line.
* **Node `v(1)`:** The voltage directly at the storage capacitor.

### The Bitline Driver
* **X1 (74x244 Buffer):** Drives the Bitline during WRITE operations. 
  * `1GBAR` is the Output Enable pin (Active Low). When low (0V), the buffer drives the bitline. When high (5V), the buffer is in a high-impedance (tri-state) mode, effectively disconnecting it from the circuit so READ operations can occur.
  * `1A1` is the Data Input pin.
* **Node `v(3)`:** The Bitline. This is the common data bus connecting the buffer, the access transistor, and the sense amplifier.
* **R1 (1kΩ):** A pull-down resistor on the bitline. It ensures the bitline rests at 0V and provides a path for the capacitor to discharge during a READ operation.

### The Sense Amplifier
* **X2 (LM339 Comparator):** Reads the minute voltage changes on the bitline.
  * The non-inverting input (`+`) is tied to the Bitline `v(3)`.
  * The inverting input (`-`) is tied to a 0.8V reference (Node `X2` / V4 in simulation). 
* **R5 (10kΩ):** The LM339 has an open-collector output. This pull-up resistor is required to pull the output to 5V when the comparator evaluates to True (Bitline > 0.8V).
* **Node `v(6)`:** The final Sense Amp output (where an LED is connected in hardware).

---

## 2. Piecewise Linear (PWL) Sources Explained

Micro-Cap does not feature interactive, real-time mechanical switches. To simulate a user physically flipping switches over time, Piecewise Linear (PWL) voltage sources are used. 

A PWL source defines voltage at specific points in time. The syntax is `PWL(Time1, Voltage1 Time2, Voltage2 ...)`. The software draws a straight line between these defined points. 

### PWL Breakdown in this Simulation:
By programming sharp transitions (e.g., changing from 5V to 0V in 0.01 seconds), the PWL acts like a digital switch. 
* **V1 (Enable Buffer):** Drops to 0V at 2s (start of Write 1) and returns to 5V at 5s. It drops to 0V again at 13s (start of Write 0) and returns to 5V at 16s.
* **V2 (Data In):** Goes to 5V at 2s, holds until 5s (writing a '1'). Remains at 0V for the rest of the simulation (writing a '0').
* **V3 (Word Line):** Fires 5V pulses to open the MOSFET gate. It pulses from 2.5s-4.5s (Write 1), 8s-9s (Read 1), 13.5s-15.5s (Write 0), and 19s-20s (Read 0).

---

## 3. Waveform Graph Analysis

The simulation graph maps a 25-second sequence demonstrating a full Write 1, Read 1, Write 0, and Read 0 cycle. 

**Graph Legend:**
* **Graph 1 `v(1)` (Blue):** Voltage inside the storage capacitor.
* **Graph 2 `v(6)` (Red):** Sense Amp Output (Logic Output / LED).
* **Graph 3 `v(2)` (Green):** Word Line (MOSFET Gate).
* **Graph 4 `v(3)` (Red):** Bitline Voltage.

### Chronological Event Breakdown:

**Time 0.00s - 2.00s: Initialization**
All lines are idle. The capacitor `v(1)` is empty. The buffer is disabled.

**Time 2.00s - 5.00s: WRITE '1'**
1. At 2.00s, V1 enables the buffer, and V2 sets the input to 5V. The bitline `v(3)` jumps to ~3.6V (the max high output of the 74x244 buffer). The Sense Amp `v(6)` immediately evaluates this as high.
2. At 2.50s, the Word Line `v(2)` goes high, opening the MOSFET. 
3. The bitline voltage rushes into the capacitor. You can see `v(1)` climb to ~3.3V (Bitline voltage minus the MOSFET threshold voltage drop). 
4. At 4.50s, the Word Line drops, trapping the ~3.3V inside `v(1)`.
5. At 5.00s, the buffer is disabled. The bitline `v(3)` drops back to 0V. The Sense Amp `v(6)` drops to 0V. 
*Result: A '1' is securely stored in `v(1)`.*

**Time 8.00s - 9.00s: READ '1'**
1. The buffer is disabled.
2. At 8.00s, the Word Line `v(2)` goes high. The MOSFET opens.
3. The stored charge in `v(1)` dumps onto the empty bitline. Look at `v(3)`: it spikes well above the 0.8V threshold.
4. Because `v(3)` > 0.8V, the Sense Amp `v(6)` snaps to 5V (LED ON). 
5. Notice how `v(1)` and `v(3)` decay curve downwards. This is the capacitor discharging through the bitline pull-down resistor (`R1`). Once `v(3)` falls below 0.8V, the Sense Amp `v(6)` cuts off. 
*Result: A '1' was successfully read. (Note: The read is destructive; the capacitor is now empty).*

**Time 13.00s - 16.00s: WRITE '0'**
1. At 13.00s, the buffer is enabled, but Data In (V2) is 0V. The bitline `v(3)` is driven firmly to 0V.
2. At 13.50s, the Word Line `v(2)` goes high, opening the MOSFET.
3. Because the bitline is 0V, any residual charge in `v(1)` is drained immediately to ground. `v(1)` becomes completely flat at 0V.
4. At 15.50s, the Word Line closes.
*Result: A '0' is securely stored in `v(1)`.*

**Time 19.00s - 20.00s: READ '0'**
1. At 19.00s, the Word Line `v(2)` goes high.
2. Because the capacitor is entirely empty, no charge is dumped onto the bitline. `v(3)` remains completely flat at 0V.
3. Because `v(3)` is below 0.8V, the Sense Amp `v(6)` remains at 0V (LED OFF).
*Result: A '0' was successfully read.*
