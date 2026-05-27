# 8-Bit CPU Design

A fully functional 8-bit CPU implemented in Logisim, inspired by Ross McGowan's lecture series and J. Clark Scott's *But How Do It Know?*

## Overview

- ALU, 4 general-purpose registers, 256 bytes of RAM, clock, and program counter
- 32 opcodes including load, store, and jump instructions
- Extended architecture with an **escape bit** for opcode extension
- Verified with a hand-written assembly program manually converted and debugged into machine code

---

## Architecture

### Control Section
Drives the full fetch-execute cycle via a **Stepper** that manages set/enable signals across the CPU.

- **Steps 1–3**: FETCH
- **Steps 4–7**: EXECUTE

### Arithmetic Logic Unit (ALU)

| Code | Operation |
|------|-----------|
| `000` | ADD |
| `001` | SHL (Shift Left) |
| `010` | SHR (Shift Right) |
| `011` | NOT |
| `100` | AND |
| `101` | OR |
| `110` | XOR |
| `111` | COMPARE |

### Components

| Component | Description |
|-----------|-------------|
| **ACC** (Accumulator) | Stores the ALU result output |
| **R0–R3** | 4 general-purpose registers |
| **RAM** | 256 bytes of addressable memory |
| **TMP** (Temporary Register) | Stores the second ALU operand |
| **PC / IAR** | Stores the address of the next instruction |
| **IR** (Instruction Register) | Stores the current instruction being executed |
| **CLK** (Clock) | Master clock feeding into the control section |
| **Bus** | Increments the PC and enables TMP output |

---

## Register Encoding

| Bits | Register |
|------|----------|
| `00` | R0 |
| `01` | R1 |
| `10` | R2 |
| `11` | R3 |

---

## Instruction Set

### LOAD
```
0 0 0 0 RA RB
```
Load contents from the RAM address stored in RB into RA.

### STORE
```
0 0 0 1 RA RB
```
Store contents of RB into the RAM address stored in RA.

### DATA
```
0 0 1 0 - - RB
```
Load the next 8 bits in RAM into RB.

### JUMP REGISTER
```
0 0 1 1 - - RB
```
Jump to the address stored in register RB.

### JUMP ADDRESS
```
0 1 0 0 x x x x
```
Jump to the next byte in RAM.

### JUMP IF
```
0 1 0 1 C A E Z
```
Jump to the address in the next memory location if the specified flags are set.

| Bit | Flag |
|-----|------|
| C | Carry |
| A | A Greater |
| E | Equal |
| Z | Zero |

### CLEAR FLAGS
```
0 1 1 0 x x x x
```
Clears all flags.

### EXTENDED OPCODE
```
0 1 1 1 x x x x
```
Enables an additional opcode extension (escape bit).

### HALT *(Extended Opcode)*
```
0 0 0 0 0 0 0 0
```
Stops program execution. Must be preceded by the Extended Opcode instruction.

### ALU INSTRUCTION
```
1 ALU RA RB
```
Performs the specified ALU operation on RA and RB, storing the result in RB.

---

## Test Program

### Description
Loads the value `2` into R0, then repeatedly adds `2` (from R1) until R0 reaches or exceeds `0x14` (20), which is stored in R2. The program then halts.

### Machine Code

| Address | Binary | Hex | Description |
|---------|--------|-----|-------------|
| `0x00` | `0010 0000` | `0x20` | DATA — load 2 into R0 |
| `0x01` | `0000 0010` | `0x02` | Data value (2) |
| `0x02` | `0010 0001` | `0x21` | DATA — load 2 into R1 |
| `0x03` | `0000 0010` | `0x02` | Data value (2) |
| `0x04` | `0010 0010` | `0x22` | DATA — load 20 into R2 |
| `0x05` | `0001 0100` | `0x14` | Data value (20) |
| `0x06` | `1111 0010` | `0xF2` | COMPARE R0, R2 |
| `0x07` | `0101 0110` | `0x56` | JUMP IF R0 >= 20 |
| `0x08` | `0000 1100` | `0x0C` | Jump target address |
| `0x09` | `1000 0100` | `0x84` | ADD R1 to R0 |
| `0x0A` | `0100 0000` | `0x40` | JUMP ADDRESS |
| `0x0B` | `0000 0110` | `0x06` | Jump back to compare |
| `0x0C` | `0111 0000` | `0x70` | HALT |

---

## References

- J. Clark Scott — *But How Do It Know?*
- Ross McGowan's 8-bit CPU lecture series
