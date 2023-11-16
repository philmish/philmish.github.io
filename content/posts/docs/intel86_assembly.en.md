---
weight: 4
title: "Intel x86 Assembly Overview"
description: "A overview over the basics of Intel x86 Assembly and its most relevant instructions"
date: "2023-11-13"
draft: false
author: "philmish"
images: []
tags: ["Intel", "Assembly", "32-bit", "x86"]
categories: ["Docs"]
lightgallery: true
---

This is an overview over the relevant basics of Intel's x86 Assembly language.

## Architecture

- follows `Van Neuman` architecture
- Hardware components: `CPU`, `RAM`, `I/O`
- bitness: `little endian`

![architecture](images/intel_x86_architecture.png "Figure 1. Simplified Hardware Layout")

## Main Memory Layout

- Data: also called `data section`, put in place when the program is initially loaded, the values contained in data may not be changed (`static`) and are available to every part of the program (`global`)
- Code: includes the instructions fetched by the `CPU` to execute the program, controls how the program's tasks are orchestrated
- Heap: is used for dynamic memory during execution, changes frequently while the program is running
- Stack: is used for local variables and function parameters, helps control flow


![memory](images/intel_x86_mem_layout.png "Figure 2. Simplified memory layout")

## Syntax

- Most instructions are invoked in this format: `Mnemonic Destination Source`
```asm
mov ecx 0x42
```
- each instruction corresponds to `opcodes`

## Operands

- `operands` are used to identify data used by instructions
- there are 3 types of `operands`:
	1. Immediate operands: fixed values such as 0x42 in the Syntax example
	2. Register operands: refer to registers, for example `ecx`
	3. Memory address operands: refer to a memory address which contains value of interest, typically denoted with a value, register or equation in bracktes, like `[eax]`
## Registers

- small amounts of data storage available to the CPU
- can be accessed more quickly than storage available elsewhere
- registers on x86 fall into four categories:
	1. General registers: used by the CPU during execution
	2. Segment registers: used to track sections of memory
	3. Status Flags: used to make decisions
	4. Instruction Pointers: are used to keep track of the next instruction to execute

### General Registers

- EAX (AX AH AL)
- EBX (BX BH BL)
- ECX (CX CH CL)
- EDX (DX DH DL)
- EBP (BP)
- ESP (SP)
- ESI (SI)

- general registers can either be accessed by their full name to retrieve the full 32-bit of data, but they can also be accessed in 16-bit mode, for example use EDX for all 32-bits in the register and DX to reference the lower 16-bits of the register
- four of the general registers (`EAX`, `EBX`, `ECX` and `EDX`) can also be referenced as 8-bit values, for example `AL` is used to reference the lowest 8-bits of `EAX` and `AH` is used to reference the second set of 8-bits, contained in the 16-bit register `AX`
- some instructions in x86 use specific registers, for example multiplication and division always use `EAX` and `EDX`
- if general registers are used in a consistent fashion across a program this is known as a `convention`, knowing different conventions used by different compilers can be helpful during analysis, for example `EAX` is commonly used to store the return value of a function, so if you see an operation on the `EAX` register after a call you can assume the return value is manipulated/used further in the code

### Segment Registers

- CS
- SS
- DS
- ES
- FS
- GS

### Status Registers

- EFLAGS

- the `EFLAGS` register is a 32-bit status register
- every bit in it is a flag, it can be set (1) or cleared (0)

![flags](images/intel_x86_eflags.png "Figure 3. Layout of the EFLAGS register")


- some important flags:
	- `ZF`: the Zero Flag is set if the result of an operation is equal to zero, otherwise it is cleared
	- `CF`: the Carry Flag is set if the result of an operation is too large or too small for the destination operand, otherwise it is cleared
	- `SF`: the Sign Flag is set when the result of an operation is negative or cleared when the result is positive; is also set when the most significant bit is set after an arithmetic operation
	- `TF`: the Trap Flag is used for debugging, if the flag is set the x86 processor will only execute one instruction at a time


### Instruction Pointer

- EIP

- also known as `instruction pointer` or `program counter`
- in x86 it is used to store the memory address of the next instruction to be executed
- tells the processor what to do next

## Instructions

A list of some of the most important instructions, what they do and their syntax.

### mov

- used to move data from one location to another

| Instruction | Description |
|-------------|--------------|
| `mov eax, ebx` | Copies the content of `EBX` into the `EAX` register|
| `mov eax, 0x42` | Copies the the value `0x42` into the `EAX` register|
| `mov eax, [0x4037C4]` | Copies the 4 bytes at the memory location `0x4037C4` into the `EAX` register|
| `mov eax, [ebx]` | Copies the 4 bytes at the memory location specified by the `EBX` register into the `EAX` register|
| `mov eax, [ebx+esi*4]` | Copies the 4 bytes at the memory location specified by the result of the equation `ebx+esi*4` into the `EAX` register|

### lea

- `load effective address`
- format: `lea destination, source`
- used to put a memory address into the destination
- in contrast to `mov` which moves the content at memory location into the destination
- example: `lea eax, [ebx+8]` will put `ebx+8` into `EAX`, while `mov eax, [ebx+8]` loads the data at the memory address specified by `ebx+8` into `EAX`

![lea](images/intel_x86_mov_vs_lea.png "Figure 4. Difference between mov and lea")

### Arithmetic


#### Addition and Subtraction


| Instruction | Description |
|-------------|--------------|
| `sub eax, 0x10` | Subtracts `0x10` from `EAX` |
| `add eax, ebx` | Adds `EBX` to `EAX` and stores the result in `EAX` |
| `inc edx` | Increments `EDX` by 1 |
| `dec edx` | Decrements `ECX` by 1 |


#### Multiplication and Division

- both always act on predefined registers
- this requires correct setup of the related registers before the us of  the `mul` or `div` instructions
- the syntax for the instruction is `mul/div value`, as the registers used are already predefined
- `mul` and `div` operate on unsigned value, `imul` and `idiv` are their equivalents for operations on signed values

| Instruction | Description |
|-------------|--------------|
| `mul 0x50` | Multiplies `EAX` by `0x50` and stores the result in `EDX:EAX` |
| `div 0x75` | Divides `EDX:EAX` by `0x75` and stores the result in `EAX` and the remainder in `EDX` |
##### Multiplication

- `mul` always multiplies `EAX` with the provided `value`, so `EAX` needs to be setup correctly before the instruction
- the result is store as a 64-bit value across `EDX` and `EAX`
- `EDX` stores the 32 most significant bits and `EAX` the least significant bits of the return value

##### Division

- `div` divides the 64-bits stored across `EDX` and `EAX` by `value`
- both `EDX` and `EAX` must be setup correctly for the use of the instruction
- the result of the division is stored in `EAX` and the remainder in `EDX`
- the remainder can be accessed with the `modulo` operator, which will access `EDX` after running `div`

#### Logical Operators

- `AND`, `OR`, `XOR`, `SHL`, `SHR`, `ROR`, `ROL` for example
- work like `add` and `sub`, perform operation between specified `source` and `destination` and storing the result in `destination`
- `xor` can be used as an optimization by the compiler to set a register to 0 by `xor` it with itself, like `xor eax, eax` to set `EAX` to 0, as the instruction only requires 2 bytes instead of 5 for an equivalent `mov` instruction
- `shl` and `shr` are used to shift registers, their format is `shr/shl destination, count`
- they shift the bits in `destination` left (`shl`) or right (`shr`) by the number of bits specified by `count`
- bits shifted beyond the boundary are first shifted into `CF` (Carry Flag Bit), zero bits in the `destination` are filled in, so after the instruction `CF` contains the last bit shifted beyond the boundary
- in comparison `ror` and `rol` shift the `destination` and append the bits going over the boundary are rotated to the other site

| Instruction | Description |
|-------------|--------------|
| `xor eax, eax` | Clears the `EAX` register |
| `or eax, 0x7575` | Perform the logical or operation on `EAX` with `0x7575` |
| `shl eax, 2` | Shifts `EAX` register to the left 2 bits |
| `ror bl, 2` | Rotates the BL register to the right 2 bits, fall-off bits will be rotated in to the right |

