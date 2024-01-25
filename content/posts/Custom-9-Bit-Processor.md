---
date: "2024-01-24T15:47:02-08:00"
title: "Custom 9-Bit Processor"
authors: ['michaelye']
categories:
  - Computer Architecture
tags:
  - SystemVerilog
  - Python
draft: false
---

## Overview
In this project, we had to design and implement our own hardware, instruction set architecture (ISA) and assembler for a 9-bit processor. Additionally, our processor needs to handle specific Forward Error Correction tasks which we need to program using our instruction set.

Given our limited instructions (9 bits), we had to finely balance the number of operations we could define with the number of register operands we could use. We ended up settling with a 3-bit opcode, and two 3-bit operand registers, totaling to 8 distinct operations and registers.

### Specifications
- Instruction memory: 9-bits wide, cannot exceed {{< katex inline >}}2^{11}{{< /katex >}} entries.
- Data memory: 8-bits wide, cannot exceed {{< katex inline >}}2^{9}{{< /katex >}} entries.
- Lookup tables limited to 32 elements.
- ALU instructions is a subset of ARMsim

### Hardware Architecture
Our final architecture model:
![png](../../images/architecture.png)

Our top-level interface only required three 1-bit I/O ports: clock input, start input, done output


### Instruction Formats
|Type| Format| Instructions|
| ------- | ------ | ------ |
|R|3-bit opcodes, two 3-bit operand registers| MOV, NAND, ADD, LDR, STR |
|I|3-bit opcode, 3 bit operand register, 3 bit immediate| SET, ROR |
|B| 3-bit opcodes, two 3-bit operand registers|BNE|

### Operations
|Name|Type|Explanation|
| ------- | ------ | ------ |
|MOV|R|Write the value from r2 to r1 while not changing r2.|
|NAND|R|NAND the contents of the two registers and store it in register TMP.|
|ADD|R|Add two registers and set the results to the first register. **Operation ignores carry-out**.|
|LDR|R|Load a value from the memory address specified from the second register to the first register.|
|STR|R|Store a value from the second register to the memory address specified in the first register.|
|SET|I|Set the register to the specified immediate value.|
|ROR|I|Rotate the values in the register by n shifts to the right.|
|BNE|B|- Case 1 (Relative): Branch to RES when the values in the registers are not equal. - Case 2 (Absolute): Branch to location in LUT when programmer counter equals LUT entry.|

### Pseudo-Operations built into Assembler
| Name | Type | Explanation |
| ------- | ------ | ------ |
|AND|R|AND of two eight-bit wide registers. **Implemented using NAND**|
|ORR|R|OR of two eight-bit wide registers. **Implemented using NAND**|
|EOR|R|Exclusive Or of two eight-bit wide registers. **Implemented using NAND**|

### Internal Operands
We support up to 8 registers. We have two special registers - parity (r6) and RES (r7) and six general purpose registers (r0, r1, r2, r3, r4, r5). In order to maximize what we can do in 9 bits, we have an implicit destination register RES for certain operations to allow up to 2 operand registers. 

For example, when branching we would need to compare two operand registers, but we still need a branch destination, which we will use the value stored in RES. If we do a branch, our program counter (PC) will be set to RES. 

R0 is considered a general purpose register with the caveat that it is used by our pseudo instructions to store temporary values. This means that junk values can be written to it as a side effect of other functions, which the programmer needs to be aware of. We use r0 along with RES to implement our Pseudo ORR, EOR,  and AND operations where we need to designate temporary registers in order to keep track of all the NAND outputs we will receive.

### Control Flow - Branches
We support a “not equal to” branch that will only branch when the two operand registers are not equal to each other. The target address is the contents from the implicit RES register. Since the register holds 8 bits, we can branch to addresses from 0 to {{< katex inline >}}2^{8}{{< /katex >}} - 1. To accommodate large jumps, we implemented a Look-Up Table (LUT) that will move our program counter to a pre-specified location whenever we have a BNE instruction on a lookup table entry. Since our LUT can be as wide as our PC, we can jump to anywhere in our program (locations within {{< katex inline >}}2^{11}{{< /katex >}} instructions from 0). For example, if our instruction starts with 111 (BNE) at PC == 10 and we have an entry in LUT at 10, then we know we can do an absolute jump to the value specified in the table.

### Addressing Modes
We have load/store operations for our memory addressing modes. To calculate the addresses, we will use our SET, ADD, and ROR operations to build the memory address we want to access. For example, if we want to access `mem[128]` we will set r1 to 1 then rotate it (ROR) 1 time so that the 1 bit now occupies the most significant bit. With our memory location calculated, we can now use another register’s values, let’s say r2, to store into the location in `mem[128]`. So what this looks like in Assembly is the following:

```
SET r1 #1	// r1 = 0b00000001
ROR r1 #1	// r1 = 0b10000000
STR r1 r2	// contents of r2 now in mem[128]
```

### Programs
We can coded three forward error correction programs on our processor. Here are their prompts:
- Given a series of fifteen 11-bit message blocks in `datamem[0:29]`, generate the corresponding 16-bit encoded versions and store these in `datamem[30:59]`.
- Given a series of 15 two-byte encoded data values – possibly corrupted – in `datamem[30:59]`, recover the original message and write into `datamem[0:29]`. 
- Given a continuous message string in `datamem[0:31]` and a 5-bit pattern in bits [7:3] of `datamem[32]`:
  - Enter the total number of occurrences of the given 5-bit pattern in any byte into `datamem[33]`. Do not cross byte boundaries for this count.
  - Write the number of bytes within which the pattern occurs into `datamem[34]`.
  - Write the total number of times it occurs anywhere in the string into `datamem[35]`. For this total count, consider the 32 bytes to comprise one continuous 256-bit message, such that the 5-bit pattern could span adjacent portions of two consecutive bytes.

This project was mainly based on a CSE 141L (Project in Computer Architecture) quarter assignment taught by Professor John Eldon.