
# 🖥️ Nand2Tetris: Projects 1-5 Documentation

## Overview

This documentation covers the construction of the **Hack Computer**, starting from a single primitive logic gate (`Nand`) and building up to a fully functioning general-purpose computer.

* **Goal:** Build a modern computer from first principles.
* **Tools:** Hardware Description Language (HDL), Hardware Simulator, CPU Emulator.
* **Fundamental Block:** The `Nand` gate.

---

## Project 1: Boolean Logic

**Objective:** Construct elementary logic gates, 16-bit variants, and multi-way variants using only `Nand` gates.

### 1.1 Elementary Gates

All gates must be derived from `Nand`.

| Chip Name | Inputs | Output | Logic Description | Implementation Hint |
| --- | --- | --- | --- | --- |
| **Not** | `in` | `out` | Inverts the input. | `Nand(in, in)` |
| **And** | `a`, `b` | `out` | 1 if both a and b are 1. | `Not(Nand(a, b))` |
| **Or** | `a`, `b` | `out` | 1 if at least one input is 1. | De Morgan's Law: `Not(And(Not(a), Not(b)))` implies `Nand(Not(a), Not(b))` |
| **Xor** | `a`, `b` | `out` | 1 if inputs are different. | `Or(a,b) AND Not(And(a,b))` |

### 1.2 Multiplexers (Mux) & Demultiplexers (DMux)

These are control chips used for routing data.

* **Mux (Selector):**
* **Logic:** If `sel=0`, `out=a`. If `sel=1`, `out=b`.
* **Formula:** 


* **DMux (Distributor):**
* **Logic:** If `sel=0`, `{a=in, b=0}`. If `sel=1`, `{a=0, b=in}`.



### 1.3 16-Bit Variants

These apply boolean logic bit-wise to 16-bit buses (e.g., `And16`).

* **Implementation:** An array of 16 single-bit gates. No interaction between bits.

### 1.4 Multi-Way Variants

* **Or8Way:** Outputs 1 if *any* of the 8 bits are 1.
* *Tip:* Cascade standard `Or` gates in a tree structure.


* **Mux4Way16 / Mux8Way16:** Selects one 16-bit bus from 4 or 8 options.
* *Tip:* Use nested `Mux16` gates. For 4-way, use `sel[0]` to pick two winners, and `sel[1]` to pick the final winner.


* **DMux4Way / DMux8Way:** Channels input to one of 4 or 8 outputs.

---

## Project 2: Boolean Arithmetic

**Objective:** Build a chip capable of adding binary numbers and performing arithmetic/logic operations (The ALU).

### 2.1 Binary Addition

We use 2's complement for representing signed numbers.

* **Half Adder:** Adds 2 bits.
* `sum = a Xor b`
* `carry = a And b`


* **Full Adder:** Adds 3 bits (a, b, c).
* *Tip:* Use two Half Adders and an Or gate.


* **Add16:** Adds two 16-bit numbers.
* *Implementation:* A chain of 16 Full Adders. The carry output of bit  is the carry input of bit  (Ripple Carry Adder).


* **Inc16:** Adds 1 to a 16-bit number.
* *Tip:* Use `Add16` with the second input set to 1 (conceptually). Or hardwire a "1" into the carry of a specialized adder chain.



### 2.2 The Arithmetic Logic Unit (ALU)

The brain of the computer. It performs a function  based on 6 control bits.

**Inputs:** `x[16], y[16]`
**Control Bits:**

| Bit | Name | Effect | Logic |
| --- | --- | --- | --- |
| 1 | **zx** | Zero the x input | If `zx=1`, x becomes 0 |
| 2 | **nx** | Negate the x input | If `nx=1`, x becomes !x (bitwise not) |
| 3 | **zy** | Zero the y input | If `zy=1`, y becomes 0 |
| 4 | **ny** | Negate the y input | If `ny=1`, y becomes !y |
| 5 | **f** | Function code | If `f=1`, out = x + y. If `f=0`, out = x & y |
| 6 | **no** | Negate the output | If `no=1`, out = !out |

**Status Outputs:**

* **zr:** 1 if `out == 0`
* **ng:** 1 if `out < 0` (Check the Most Significant Bit).

---

## Project 3: Sequential Logic (Memory)

**Objective:** Build memory units that maintain state over time using the Clock.

### 3.1 The Flip-Flop

* **DFF (Data Flip-Flop):** Primitive gate provided by the simulator.
* **Behavior:** 
* This is the barrier between "current time" and "previous time".

### 3.2 1-Bit Register (Bit)

Stores 1 bit of data.

* **Inputs:** `in`, `load`
* **Logic:**
* If `load(t-1) == 1`, then `out(t) = in(t-1)` (Write)
* If `load(t-1) == 0`, then `out(t) = out(t-1)` (Read/Hold)


* **Implementation:** Use a Mux to choose between the new `in` and the old `out` (feedback loop), feeding into a DFF.

### 3.3 RAM Units (RAM8, RAM64, RAM512, RAM4K, RAM16K)

Recursive construction of memory.

* **RAM8:** 8 Registers. Uses `DMux8Way` to route the `load` signal to the correct register and `Mux8Way16` to select the correct output.
* **RAM64:** 8 RAM8 chips.
* *Address logic:* `address[0..2]` feeds the inner RAM chips. `address[3..5]` selects *which* RAM chip to activate.



### 3.4 Program Counter (PC)

Tracks which instruction to fetch next.

* **Capabilities:** Reset (to 0), Load (Jump to address), Increment (Next line).
* **Precedence:** `Reset > Load > Inc`.
* **Implementation:** A Register fed by a Mux chain logic handling the if-else precedence.

---

## Project 4: Machine Language

**Objective:** Write low-level Assembly programs for the Hack computer.

### 4.1 Hack Assembly Language

Two types of instructions:

1. **A-Instruction (@value):**
* Sets the A-Register to `value`.
* Used for data addresses (`@sum`) or jump targets (`@LOOP`).
* Binary: `0vvv vvvv vvvv vvvv`


2. **C-Instruction (dest = comp ; jump):**
* Performs computation using the ALU.
* **comp:** The calculation (e.g., `D+1`, `A-1`, `D+M`).
* **dest:** Where to store the result (`M`, `D`, `MD`, `A`, etc.).
* **jump:** Condition to jump (`JGT`, `JEQ`, `JMP`, etc.) based on previous ALU result.
* Binary: `111a cccc ccdd djjj`



### 4.2 Memory Map

* **RAM:** Data storage (0 - 16383). Accessed via `M`.
* **Screen:** Memory mapped I/O (16384 - 24575). Writing to these addresses turns pixels black/white.
* **Keyboard:** Memory mapped I/O (24576). Contains the scan code of the key currently pressed.

### 4.3 Key Programs

1. **Mult.asm:** Multiply `R0` and `R1`, store result in `R2`.
* *Strategy:* Repeated addition. Use a loop that runs `R1` times, adding `R0` to a sum accumulator.


2. **Fill.asm:** I/O interaction.
* *Strategy:* Infinite loop listening to `KBD`. If `KBD != 0`, fill Screen words with `-1` (all 1s, black). Else, fill with `0` (white).



---

## Project 5: Computer Architecture

**Objective:** Assemble the chips from Projects 1-3 into a working computer.

### 5.1 Memory Chip

Integrates Data RAM, Screen, and Keyboard into a single address space.

* **Logic:** Uses the high bits of the address to select between RAM16K, Screen, or Keyboard register.
* **Note:** Keyboard is read-only.

### 5.2 CPU (Central Processing Unit)

The heart of the project. It executes instructions.

* **Components:** ALU, Register A, Register D, Program Counter (PC).
* **Control Logic (Decoding):** You must look at the instruction bits to decide:
* Is it an A-instruction or C-instruction?
* Should the A-Register load the instruction value or the ALU output?
* Should the D-Register load?
* Should `M` (RAM) be written to (`writeM`)?
* **PC Logic:** Should we jump? (Compare ALU output flags `zr`, `ng` with instruction jump bits `j1, j2, j3`).



### 5.3 Computer

The top-level chip.

* **Wiring:** Connects `ROM32K` (Instruction Memory), `CPU`, and `Memory` (Data Memory).
* **Cycle:**
1. ROM sends instruction to CPU.
2. CPU executes, reads/writes to Memory, updates PC.
3. PC selects next instruction from ROM.



