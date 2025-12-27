
# 💾 Nand2Tetris: Projects 6-12 Documentation

## Overview: The Software Stack

While Projects 1-5 built the *computer*, Projects 6-12 build the *compiler* and *OS* required to run high-level code on that computer.

* **Goal:** Build a compiler for a high-level object-oriented language (Jack) that runs on the Hack computer.
* **Approach:** Two-tier compilation.
1. **Jack** (High Level)  **VM Code** (Intermediate)
2. **VM Code** (Intermediate)  **Hack Assembly** (Low Level)



---

## Project 6: The Assembler

**Objective:** Write a program (in Python, Java, etc.) that translates Hack Assembly (`.asm`) into binary machine code (`.hack`).

### 6.1 Architecture

The assembler typically consists of three modules:

1. **Parser:** Reads the file line-by-line, strips whitespace/comments, and identifies the instruction type (`A_COMMAND`, `C_COMMAND`, `L_COMMAND` (Pseudo-labels)).
2. **Code:** Translates the mnemonic parts of the assembly (e.g., `D+M`, `JMP`) into their corresponding binary bits.
3. **SymbolTable:** Manages the mapping of symbolic labels (e.g., `@LOOP`, `@sum`) to specific RAM addresses.

### 6.2 The Translation Process (Two Passes)

Because you can jump to a label declared *later* in the code (forward reference), you need two passes over the source file.

1. **First Pass (Symbol Extraction):**
* Scan code for labels `(LOOP)`.
* Add the label and the current ROM address (instruction number) to the `SymbolTable`.
* *Do not* output any code.


2. **Second Pass (Code Generation):**
* Scan code again.
* If it's an **A-Instruction** (`@xxx`):
* If `xxx` is a number: convert to binary.
* If `xxx` is a symbol: check `SymbolTable`. If present, use the address. If not present (and not a label), it is a **variable**. Assign it the next available RAM address (starting at 16) and add to the table.


* If it's a **C-Instruction**: Use the `Code` module to concatenate the binary strings for `comp`, `dest`, and `jump`.



---

## Projects 7 & 8: The Virtual Machine (VM)

**Objective:** Build the backend of the compiler. Translates high-level VM commands into Hack Assembly. The VM is a **Stack Machine**.

### 7.1 Stack Arithmetic (Project 7)

The VM uses a global stack to perform calculations.

* **Push:** Move data from a memory segment onto the stack top.
* **Pop:** Move data from the stack top into a memory segment.
* **Arithmetic:** `add`, `sub`, `neg`, `eq`, `gt`, `lt`, `and`, `or`, `not`.
* *Implementation:* Pop operands from stack to CPU registers (D/A), perform ALU operation, push result back.
* *Tricky part:* Comparison commands (`eq`, `gt`, `lt`) require branching logic in Assembly (Jumps) to set true (-1) or false (0).



### 7.2 Memory Access (Project 7)

The VM abstracts RAM into 8 virtual segments. Your translator must map these to Hack RAM locations.

| VM Segment | Purpose | Hack Implementation |
| --- | --- | --- |
| `constant` | Virtual numbers (0..32767) | No RAM. Just loading `D=i`. |
| `local` | Function local variables | Dynamic. Base address stored in `LCL` (RAM[1]). |
| `argument` | Function arguments | Dynamic. Base address stored in `ARG` (RAM[2]). |
| `this` / `that` | Object fields / Array entries | Dynamic. Base addresses in `THIS`/`THAT`. |
| `temp` | Scratchpad | Fixed. RAM[5] to RAM[12]. |
| `pointer` | Accessing THIS/THAT pointers | `pointer 0` maps to `THIS`, `1` to `THAT`. |
| `static` | Class-level variables | Static. Mapped to `Filename.i` labels in Assembly. |

### 7.3 Program Flow & Function Calling (Project 8)

* **Branching:** `label`, `goto`, `if-goto`.
* `if-goto` pops the top value of the stack; if it's not zero, it jumps.


* **Functions:**
* `call f n`: Push the return address and save the caller's state (`LCL`, `ARG`, `THIS`, `THAT`) onto the stack. Reposition `ARG` and `LCL` for the new function. Jump to function `f`.
* `function f k`: Declare function `f` with `k` local variables. Initialize those k locals to 0.
* `return`: Copy the return value to the caller's stack. Restore the caller's state (pointers). Jump to return address.



---

## Project 9: High-Level Language (Jack)

**Objective:** Learn the Jack language by writing a simple game (e.g., Snake, Pong, Tetris).

* **Syntax:** Java-like but simpler. Strong typing.
* **Structure:**
* Files: `.jack` extension. One class per file.
* Entry point: `Main.main()`.


* **Key Concept:** Understanding how to use the OS standard library (Screen class for graphics, Output for text) which you will build in Project 12.

---

## Projects 10 & 11: The Compiler

**Objective:** Translate Jack code (`.jack`) into VM code (`.vm`).

### 10.1 Syntax Analysis (Tokenizer & Parser)

* **Tokenizer (Lexer):** Reads stream of characters, groups them into tokens (keywords, symbols, identifiers, integer constants, string constants).
* **Parser:** Analyzes the stream of tokens against the Jack grammar rules.
* Output: An XML file representing the **Parse Tree** (hierarchy of the code structure).
* *Recursive Descent:* A standard parsing algorithm where every grammar rule (e.g., `compileClass`, `compileLet`) is a method in your code.



### 10.2 Code Generation

The parser is modified to output VM code instead of XML.

* **Symbol Table:**
* You need two tables: **Class-Scope** (fields/statics) and **Subroutine-Scope** (args/locals).
* Tracks: Name, Type (int, boolean, ClassName), Kind (field, static, var, arg), and Index (0, 1, 2...).


* **Handling Expressions:** Convert infix notation (`x + y`) to postfix/RPN for the stack (`push x`, `push y`, `add`).
* **Handling Objects:**
* **Constructors:** Allocate memory (`Memory.alloc`) based on the number of fields. Returns `this`.
* **Methods:** The first argument is implicitly `this` (pointer 0).


* **Handling Arrays:** To access `arr[i]`, you calculate `addr = arr + i`, then use `that` segment to access the memory at `addr`.

---

## Project 12: The Operating System

**Objective:** Implement the standard library in Jack. These are the built-in system classes.

### 12.1 Mathematical Operations (Math.jack)

Since the Hack hardware only does `+` and `-`, you must implement:

* **Multiplication:** Efficient bit-shift algorithm (), not repeated addition.
* **Division:** Recursive shift-subtract algorithm.
* **Sqrt:** Binary search or bit-wise approximation.

### 12.2 Memory Management (Memory.jack)

* **Peek/Poke:** Direct memory access.
* **Alloc/DeAlloc:** Heap management.
* *Free List:* A linked list embedded in the free memory blocks keeping track of available segments.
* `alloc(size)`: Searches the free list (First-fit or Best-fit) for a block large enough.



### 12.3 String & Array

* **String:** Handles length, appending characters, and converting Integer  String.
* **Array:** Just a wrapper for memory allocation.

### 12.4 I/O Drivers

* **Screen.jack:**
* `drawPixel(x, y)`: Calculates the specific RAM address and bit mask to flip.
* `drawLine`: Bresenham's line algorithm (uses only integer addition/subtraction for speed).


* **Keyboard.jack:** Reads the RAM[24576] register.
* **Output.jack:** Handles font rendering. Uses a bitmap font map (provided) to draw characters on screen pixels.

---

### 💡 General Tips for Software Projects

1. **Test the Test:** When debugging the Compiler, stick to the provided test files (`Square`, `Average`, `ComplexArrays`) strictly in order.
2. **WhiteSpace:** In Project 10, ensure your XML output matches the "Compare" files exactly, including whitespace. The comparison tool is strict.
3. **Variable Mapping:** In Project 11, confusing `local`, `argument`, `this`, and `field` scopes is the most common bug. Draw a table of what variable lives in what segment.
4. **Efficiency:** In Project 12 (OS), efficiency matters. A slow `Math.multiply` or `Screen.drawPixel` will make games unplayable. Avoid recursion where a loop suffices, except for specific algorithms like QuickSort or Division.

