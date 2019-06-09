---
layout: post
title: "ez80 Processor Overview"
---

<img src="/assets/images/2015-07-02-ez80-processor-overview/cpu_block_diagram.png">

## Control Block

### Instruction Fetch

* Contains state machine controlling READs from memory.
* Fetches opcodes and operands.
* Keeps track of start and end of each instruction
* Stores opcodes during external memory READs and WRITEs
* Discards prefetched instructions:
    * Jumps
    * Interrupts
    * Other control transfer events

### Mode Control

* Controls current processor mode
    * HALT
    * SLEEP
    * Interrupt
    * Debug
    * ADL

### Opcode Decoder

* Decodes opcodes

## Data Block

### CPU Registers

* Contains all CPU registers
    * Data registers
    * Program Counter
    * Stack Pointer
    * Flags
    * CPU control registers

### ALU

* Performs arithmetic and logic functions on addresses and data
* Inputs:
    * Control Block
    * Registers

### Address Generator

* Creates addresses for all CPU memory READ and WRITE operations
* Contains the Z80 Memory Mode Base Address register (MBASE) for Z80
  compatibility mode.

### Data Selector

* Places appropriate data onto the data bus.
* Controls the data path based on the instruction being executed

## Pipeline Description

Each instruction must be fetched, decoded, and executed. This normally takes at
least three cycles. The pipeline can reduce the overall time to as little as
one by prefetching and decoding while executing other instructions.

<img src="/assets/images/2015-07-02-ez80-processor-overview/pipeline_overview.png">

The number of bytes prefetched is a function of the command currently executing
in the CPU. For example, if an **LD** instruction is executing, another **LD**
will only have it's opcode prefetched. The operand will not be fetched until
the current **LD** has finished executing, due to the bus WRITE.

When a control transfer takes place, the pipeline must be flushed. All
prefetched values are ignored. After the control transfer instruction executes,
the pipeline must start over.

Control transfer may occur from one of the following:

* Interrupt
* **JP**
* **CALL**
* **RET**
* **RST**
* Instruction similar to those listed above

## Memory Modes

The eZ80 CPU can operate in either the Z80 or ADL memory mode. While in Z80
mode, the processor operates with 16-bit addresses and 16-bit CPU registers.
Selection of the memory mode is controlled by the ADL mode bit. The Z80 and ADL
memory modes may be referred to as "ADL modes".

### Z80 MEMORY Mode, AKA non-ADL mode.

* Default operating mode on reset
* ADL bit cleared to 0
* Z80-compatible addressing
* 16-bit CPU registers
* 16-bit Stack Pointer Short (SPS) register used as stack pointer
* 8-bit MBASE register used as the MSB of all memory addresses.
* MBASE can only be changed while in ADL mode.

### ADL MEMORY Mode

* ADL bit set to 1
* 16MB address space
* 24-bit CPU registers ([see eZ80 CPU Registers in ADL
  Mode](#registers_in_adl))
* Full eZ80 instruction set
* MBASE does not affect memory addressing
* 24-bit Stack Pointer Long (SPL) register used as stack pointer
* All addresses and data are 24 bits
* All data READ and WRITE operations pass 3 bytes of data to and from the CPU
* Instructions in ADL mode may require more clock cycles to complete
* MBASE may be modified in this mode

## Registers and Bit Flags

### eZ80 CPU Working Registers

The CPU contains two banks of working registers -- the main register set and
the alternate register set. The main register set contains the 8-bit
accumulator register (A) and six 8-bit working registers (B, C, D, E, H, L).
The six 8-bit working registers can be combined to function as the multi-byte
register pairs BC, DE, and HL. The 8-bit Flag register F completes the main
register set.

The alternate (shadow) register set contains complements to the main registers
-- (A', B', C', D', E', H', L', F'). These can be swapped with the main
registers quickly using the **EX** and **EXX** instructions.

### eZ80 CPU Control Register Definitions

The CPU contains several registers that control CPU operation.

* Interrupt Page Address Register (I) -- the 16-bit I register stores the upper
  16 bits of the interrupt table address for Mode 2 vectored interrupts. *The
  16-bit I register is not supported on eZ80190, eZ80L92, or eZ80F92/F93
  devices.*
* Index Registers (IX and IY)
    * Allow standard addressing and relative displacement addressing in memory.
    * Many instructions use the IX and IY registers for relative addressing
      with an 8-bit signed displacement added to the contents of the register
    * Certain 8-bit opcodes address the High and Low bytes of these registers
      directly
    * The High bytes are indicated by IXH and IYH
    * The Low bytes are indicated by IXL and IYL
* Z80 Memory Mode Base Address (MBASE)
    * 8-bit
    * Determines the page of memory currently used while operating in Z80 mode
    * Only used during Z80 mode
    * Can only be altered from ADL mode
* Program Counter (PC) register
    * Stores the address of the current instruction being fetched from memory
    * Automatically incremented during program execution
    * What a program jump occurs, a new value overwrites the old
    * 16 bits in Z80 mode {MBASE, PC[15:0]}, 24 bits in ADL mode {PC[23:0]}
* Refresh Counter (R) register
    * Contains a count of executed instruction fetch cycles.
    * 7 least significant bits are automatically increment after each
      instruction fetch.
    * Most significant bit can only be changed by writing to the R register
    * Can be read from and written to with **LD A,R** and **LD R,A**
* Stack Pointer Long (SPL) register
    * 24-bit
    * Stores the address for the current top of the external stack in ADL mode
    * Stack is organized as last-in first-out (LIFO)
    * Modified with **PUSH** and **POP**
    * Interrupts, traps, calls, and returns also employ the stack
* Stack Pointer Short (SPS) register
    * 16-bit
    * Stores the address for the current top of the external stack in Z80 mode.
    * The 24-bit Stack Pointer address in Z80 mode is {MBASE, SPS}
    * Stack is organized as last-in first-out (LIFO)
    * Modified with **PUSH** and **POP**
    * Interrupts, traps, calls, and returns also employ the stack

### eZ80 CPU Control Bits

* Address and Data Long Mode Bit (ADL)
    * Indicates current memory mode of the CPU
    * ADL reset to 0 indicates 16-bit Z80 MEMORY mode
    * ADL set to 1 indicates 24-bit ADL mode
    * Reset to 0 by default
    * Can only be changed by instructions that allow persistent memory mode
      changes, interrupts, and traps
    * Cannot be directly written
* Mixed-ADL Bit (MADL)
    * Configures the CPU to execute programs containing code that uses both ADL
      and Z80 MEMORY modes
    * Explained in more detail in [Interrupts in Mixed Memory Mode
      Applications](#interrupts_in_mmas) and [Mixed-Memory Mode
      Applications](#mmas)
* Interrupt Enable Flags (IEF1 and IEF2)
    * Set or reset using **EI** and **DI**
    * When IEF1 = 0, a maskable interrupt cannot be accepted by the CPU.
    * Described in more detail in [Interrupts](#interrupts)

### eZ80 CPU Registers in Z80 Mode

* BC, DE, HL, IX, and IY function as 16-bit registers for multi-byte operations
  and indirect addressing.
* The active Stack Pointer is the 16-bit Stack Pointer Short (SPS)
* The Program Counter (PC) is 16 bits long
* Addresses composed as {MBASE, ADDR[15:0]}
* Upper byte (bits 23:16) of multi-byte registers are undefined
* Upper byte of the I register, bits [15:8], is unused

<img src="/assets/images/2015-07-02-ez80-processor-overview/cpu_regs_in_z80mode.png">

<img src="/assets/images/2015-07-02-ez80-processor-overview/cpu_control_in_z80mode.png">

<a name="registers_in_adl"></a>

### eZ80 CPU Registers in ADL Mode

* BC, DE, HL, IX, and IY are 24 bits long for multi-byte operations and indirect
  addressing.
* The most significant bytes (MSBs) are designated with a U for upper byte. For
  example, the upper byte of BC is designated BCU. Thus, the 24-bit BC register
  in ADL mode is composed of the three 8-bit registers {BCU, B, C}. Similarly,
  IX is {IXU, IXH, IXL}
    * *None of the upper bytes are individually accessible as standalone 8-bit
      registers*
* MBASE is writable but is not used for address generation
* PC is 24 bits long, as is SPL
* IEF1, IEF2, ADL, and MADL are single bit flags

<img src="/assets/images/2015-07-02-ez80-processor-overview/cpu_info_adlmode.png">

<img src="/assets/images/2015-07-02-ez80-processor-overview/cpu_flags_adlmode.png">

## eZ80 CPU Status Indicators (Flag Register)

The Flag register (F and F') contains status information for the CPU. The bit
position for each flag is indicated below.


Bit        | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 
-----------|---|---|---|---|---|---|---|---
 **Flag**  | S | Z | X | H | X |P/V| N | C 

* C = Carry Flag
* N = Add/Subtract Flag
* P/V = Parity/Overflow Flag
* H = Half-Carry Flag
* Z = Zero Flag
* S = Sign Flag
* X = Not used

### Carry Flag (C)

* Set by **ADD** or **SUBTRACT** instructions that carry/borrow
* Reset by **ADD** or **SUBTRACT** instructions that do not generate a
  carry/borrow
* Used by bit shift/rotation instructions to link between registers
* Can be set by **SCF** and complemented by **CCF**

### Add/Subtract Flag (N)

* Reset when an **ADD** executes, set when a **SUBTRACT** executes
* Used by **DAA**

### Parity/Overflow Flag (P/V)

* Indicates an overflow condition when the result in the accumulator is greater
  than +127 or less than -128
* Indicates parity of rotate instructions. If the number of 1 bits are odd, the
  P/V flag is set to 0. If it is even, the P/V flag is set to 1
* Monitors state of the byte count register during search and block transfer
  instructions. If BC = 0 the flag is reset to 0, otherwise it is set to 1
* Indicates parity of data read with **IN r,(C)**
* Set to contents of IEF2 when **LD A, I** and **LD A, R** are executed

### Half-Carry Flag (H)

* Set or reset depending on the carry and borrow status between bits 3 and 4 of
  an 8-bit arithmetic operation
* Used by **DAA** to correct the result of a packed BCD addition or subtraction
* Set if there was a carry, reset if there was no carry

### Zero Flag (Z)

* Set to 1 if the result generated by the execution of certain instructions is
  0
* Affected by
    * Arithmetic and logical (bit) operations
    * Compare instructions
    * Bit test instructions
    * I/O instructions

### Sign Flag (S)

* Stores the state of the most significant bit of the accumulator (bit 7).
* Also affected by **IN r,(C)**

## Memory Mode Switching

### ADL Mode and Z80 Mode

The CPU can easily switch between ADL and Z80 mode. There are two types of mode
changes: persistent and single-instruction. Persistent mode lasts until the
mode is explicitly changed again, while single-instruction mode allows a
certain instruction to operate using either addressing mode without making a
persistent change.

### Memory Mode Compiler Directives

These compiler directives indicate that either ADL or Z80 mode is the default
memory mode for the code being assembled/compiled.

* `.ASSUME ADL=1`
* `.ASSUME ADL=0`

## Opcode Suffixes for Memory Mode Control

Special opcode suffixes are added to the instruction set to assist with memory
mode switching operations. These are combined of 2 parts. The first part,
either **.S** or **.L** controls the data block while the second part, either
**.IS** or **.IL** controls the control block. For example, **LD HL, Mmn** in
**.IL** mode vs **LD HL, mn** in **.IS** mode

* **.S** - The CPU data block operates in Z80 mode using 16-bit registers, and
  all addresses use MBASE
* **.L** - The CPU data block operates in ADL mode using 24-bit registers.
  Addresses do not use MBASE
* **.IS** - The CPU control block operates in Z80 mode. For instructions with
  an ambiguous number of bytes, the .IS suffix indicates that only 2 bytes of
  immediate data or address must be fetched.
* **.IL** - The CPU control block operates in ADL mode. For instructions with
  an ambiguous number of bytes, the .IL suffix indicates that 3 bytes of
  immediate data or address must be fetched.

## Single-Instruction Memory Mode Changes

Certain CPU instructions can be appended with the memory opcode suffixes
**.SIS**, **.LIL**, **.LIS**, and **.SIL** to indicate that a particular memory
mode should be used for this instruction only.

### Example 1: LD HL, Mmn in Z80 Mode

```
.ASSUME ADL=0       ;Z80 mode operation is default
LD HL, 3456h        ;HL[23:0] = {00h, 3456h}.
LD HL, 123456h      ;Invalid - Z80 mode cannot load 24-bit values.

LD.SIS HL, 3456h    ;Exactly the same as LD HL, 3456h because ADL=0
                    ;.IS - fetch only 16 bits of data
                    ;.S  - force upper byte of HL to an undefined state

LD.LIL HL, 123456h  ;HL[23:0] = 123456h
                    ;.IL - fetch 24-bits of data
                    ;.L  - use all 3 bytes of HL register

LD.LIS HL, 3456h    ;HL[23:0] = {00h, 3456h}
                    ;.IS - fetch only 16 bits of data
                    ;.L  - use all 3 bytes of HL register

LD.SIL HL, 123456h  ;HL[23:0] = {00h, 3456h}
                    ;.IL - fetch 24 bits of data
                    ;.S  - force upper byte of HL to undefined because
                    ;      registers are defined to be only 16-bits
```

### Example 2: LD HL, Mmn in ADL Mode

```
.ASSUME ADL=1       ;ADL mode operation is default
LD HL, 3456h        ;HL[23:0] = 003456h.
LD HL, 123456h      ;HL[23:0] = 123456h.

LD.SIS HL, 3456h    ;HL[23:0] = {00h, 3456h}
                    ;.IS - fetch only 16 bits of data
                    ;.S  - force upper byte of HL to an undefined state

LD.LIL HL, 123456h  ;HL[23:0] = 123456h
                    ;.IL - fetch 24-bits of data
                    ;.L  - use all 3 bytes of HL register

LD.LIS HL, 3456h    ;HL[23:0] = {00h, 3456h}
                    ;.IS - fetch only 16 bits of data
                    ;.L  - use all 3 bytes of HL register

LD.SIL HL, 123456h  ;HL[23:0] = {00h, 3456h}
                    ;.IL - fetch 24 bits of data
                    ;.S  - force upper byte of HL to undefined because
                    ;      registers are defined to be only 16-bits
```

### Important Notes

While the comments in the code use the term "undefined state", the paragraphs
describing them state specifically that the upper bits loaded in **.IL** mode
are replaced with `00h`. If this is true on hardware, emulators should also do
this for consistency.

Memory WRITEs (and presumably READs) in **.S** mode will use MBASE, regardless
of whether **.IL** or **.IS** are used.

### Suffix Completion by Assembler

The assembler will convert partial suffixes to full suffixes based on the
current memory mode.

Partial Suffix |  ADL Mode Bit  |  Full Suffix
-------------- | -------------- | -------------
**.S**         |       0        |     .SIS
**.S**         |       1        |     .SIL
**.L**         |       0        |     .LIS
**.L**         |       1        |     .LIL
**.IS**        |       0        |     .SIS
**.IS**        |       1        |     .LIS
**.IL**        |       0        |     .SIL
**.IL**        |       1        |     .LIL

### Assembly of Opcode Suffixes

During assembly, the opcode suffixes become prefixes in the assembled code. The
processor must know what type of memory exceptions must be applied to the
instruction to follow. The prefix bytes replace Z80 instructions that do not
offer a useful function. If a CPU assembler encounters one of these replaced
instructions, it issues a warning message and assembled it as a NOP (`00h`).

CPU Code Suffix |  Prefix Byte | Replaced Instruction
--------------- | ------------ | ---------------------
**.SIS**        |     `40h`    |      **LD** B,B
**.LIS**        |     `49h`    |      **LD** C,C
**.SIL**        |     `52h`    |      **LD** D,D
**.LIL**        |     `5Bh`    |      **LD** E,E

For the traditional Z80 prefix bytes (`CBh`, `DDh`, `EDh`, and `FDh`), the CPU
does not allow an interrupt to occur in the time between fetching one of these
prefix bytes and fetching the following instruction. The MEMORY mode prefix
bytes (`40h`, `49h`, `52h`, `5Bh`) must precede the traditional Z80 prefix
byte.

### Persistent Memory Mode Changes in ADL and Z80 Modes

The CPU can only make persistent mode switches between ADL mode and Z80 mode as
part of a special control transfer instruction (**CALL**, **JP**, **RST**,
**RET**, **RETI**, or **RETN**), or as part of an interrupt or trap operation.
This prevents the Program Counter from making an uncontrolled jump. When the
memory mode is changed in any of these ways, it remains in its new state until
another of these operations changes the mode back.

User Code       |  ADL  |  Assembled Code  |  Operation
--------------- | ----- | ---------------- | ------------------------------------
**CALL mn**     |   0   |  `CD nn nm`      |  Normal Z80 CALL
**CALL Mmn**    |   1   |  `CD nn nm MM`   |  Normal eZ80 CALL
**CALL.IS mn**  |   0   |  `40 CD nn mm`   |  Push `02h` to SPL indicating call from Z80 mode. Proceed with normal Z80 CALL .
**CALL.IS mn**  |   1   |  `49 CD nn mm`   |  Push PC[15:0] to SPS stack     <br>
                |       |                  |  Push PC[23:16] to SPL stack    <br>
                |       |                  |  Push `03h` to SPL stack indicating
                |       |                  |  call from ADL mode             <br>
                |       |                  |  Reset ADL mode bit to 0        <br>
                |       |                  |  Set PC[15:0] = {mm, nn} 
**CALL.IL Mmn** |   0   |  `52 CD nn mm MM`|  Push PC[15:0] to SPL stack     <br>
                |       |                  |  Push `02h` to SPL stack indicating
                |       |                  |  call from Z80 mode             <br>
                |       |                  |  Set ADL mode bit to 1          <br>
                |       |                  |  Set PC[23:0] = {MM, mm, nn}
**CALL.IL Mmn** |   1   |  `5B CD nn mm MM`|  Push PC[23:0] to SPL stack     <br>
                |       |                  |  Push `03h` to SPL stack indicating
                |       |                  |  call from ADL mode             <br>
                |       |                  |  Proceed with normal eZ80 CALL

: CALL Mmn Instruction

<br/>

--------------------------------------------------------------------------------
User Code          ADL   Assembled Code     Operation
----------------  -----  ----------------   ------------------------------------
**JP mn**           0    `C3 nn mm`         PC[15:0] = {mm, nn} 

**JP.SIS mn**       0    `40 C3 nn mm`      Equivalent to **JP mn** 

**JP.LIL Mmn**      0    `5B C3 nn mm MM`   PC[23:0] = {MM, mm, nn}        <br/>
                                            Set ADL mode bit to 1 

**JP.SIL Mmn**      0    N/A                An illegal suffix for this
                                            instruction 

**JP.LIS mn**       0    N/A                An illegal suffix for this
                                            instruction 

**JP Mmn**          1    `C3 nn mm MM`      PC[23:0] = {MM, mm, nn} 

**JP.SIS mn**       1    `40 C3 nn mm`      PC[15:0] = {mm, nn}            <br/>
                                            Reset ADL mode bit to 0 

**JP.LIL Mmn**      1    `5B C3 nn mm MM`   Equivalent to **JP Mmn**

**JP.SIL Mmn**      1    N/A                An illegal suffix for this
                                            instruction 

**JP.LIS mn**       1    N/A                An illegal suffix for this
                                            instruction 
--------------------------------------------------------------------------------

: JP Mmn Instruction

<br/>

--------------------------------------------------------------------------------
User Code          ADL   Assembled Code     Operation
----------------  -----  ----------------   ------------------------------------
**JP (rr)**         0    `E9 or DD/FD E9`   PC[15:0] = rr[15:0]

**JP.S (rr)**       0    `40 E9 or          Equivalent to **JP (rr)**
                          40 DD/FD E9`

**JP.L (rr)**       0    `49 E9 or          PC[23:0] = rr[23:0]            <br/>
                          49 DD/FD E9`      Set ADL mode bit to 1 

**JP (rr)**         1    `E9 or DD/FD E9`   PC[23:0] = rr[23:0]

**JP.S (rr)**       1    `52 E9 or          PC[15:0] = rr[15:0]            <br/>
                          52 DD/FD E9`      Reset ADL mode bit to 0 

**JP.L (rr)**       1    `5B E9 or          Equivalent to **JP (rr)**
                          5B DD/FD E9`
--------------------------------------------------------------------------------

: JP (rr) Instruction

<br/>

--------------------------------------------------------------------------------
User Code          ADL   Assembled Code     Operation
----------------  -----  ----------------   ------------------------------------
**RST n**           0    `CD nn`            Push PC[15:0] to SPS stack     <br/>
                                            PC[15:0] = {`00h`, nn}

**RST n**           1    `CD nn`            Push PC[23:0] to SPL stack     <br/>
                                            PC[23:0] = {`0000h`, nn}

**RST.S n**         0    `40 CD nn`         Push PC[15:0] to SPS stack     <br/>
                                            Push `02h` to SPL stack, indicating
                                            an interrupt from Z80 mode     <br/>
                                            PC[15:0] = {`00h`, nn}

**RST.S n**         1    `52 CD nn`         Push PC[15:0] to SPS stack     <br/>
                                            Push PC[23:16] to SPL stack    <br/>
                                            Push `03h` to SPL stack, indicating
                                            an interrupt from ADL mode     <br/>
                                            Reset ADL mode bit to 0        <br/>
                                            PC[15:0] = {`00h`, nn}

**RST.L n**         0    `49 CD nn`         Push PC[15:0] to SPL stack     <br/>
                                            Push `02h` to SPL stack, indicating
                                            an interrupt from Z80 mode     <br/>
                                            Set ADL mode bit to 1          <br/>
                                            PC[23:0] = {`0000h`, nn}

**RST.L n**         1    `5B CD nn`         Push PC[23:0] to SPL stack     <br/>
                                            Push `03h` to SPL stack, indicating
                                            an interrupt from ADL mode     <br/>
                                            PC[23:0] = {`0000h`, nn}
--------------------------------------------------------------------------------

: RST n Instruction

<br/>

--------------------------------------------------------------------------------
User Code    ADL   Assembled Code   Operation
----------  -----  ---------------  --------------------------------------------
**RET**       0    `C9`             Pop 2-byte address from SPS to PC[15:0]

**RET**       1    `C9`             Pop 3-byte address from SPL to PC[23:0]

**RET.S**     0    N/A              An invalid suffix. **RET.L** must be used in
                                    all mixed-memory mode applications
                                    

**RET.S**     1    N/A              An invalid suffix. **RET.L** must be used in
                                    all mixed-memory mode applications

**RET.L**     0    `49 C9`          Pop a byte from SPL into ADL to set memory
                                    mode (`03h` = ADL, `02h` = Z80).       <br/>
                                    if ADL mode {                          <br/>
                                    \ \ \ \ pop upper byte from SPL
                                            to PC[23:16]                   <br/>
                                    \ \ \ \ pop 2-byte LS bytes from SPS
                                            to PC[15:0]                    <br/>
                                    } else Z80 mode {                      <br/>
                                    \ \ \ \ pop 2-byte address from SPS
                                            to PC[15:0]                    <br/>
                                    }

**RET.L**     1    `5B C9`          Pop a byte from SPL into ADL to set memory
                                    mode (`03h` = ADL, `02h` = Z80).       <br/>
                                    if ADL mode {                          <br/>
                                    \ \ \ \ pop 3-byte address from SPL
                                            to PC[23:0]                    <br/>
                                    } else Z80 mode {                      <br/>
                                    \ \ \ \ pop 2-byte address from SPL
                                            to PC[15:0]                    <br/>
                                    }


--------------------------------------------------------------------------------

: RET Instruction

<br/>

**RETI** acts like **RET** but it does **RETI** stuff

**RETN** acts like **RET** but it does **RETN** stuff

<a name="mmas"></a>

## Mixed-Memory Mode Applications

The Mixed-ADL (MADL) control bit affects operations of interrupts, illegal
instruction traps, and restart (**RST**) instructions. The MADL bit must be set
to 1 while executing code that runs in both Z80 and ADL mode. It can be reset
to 0 when executing code that only uses Z80 or ADL mode. The default state is
0.

Technically nothing can run exclusively in ADL mode, since the CPU defaults to
Z80 mode. If a single **JP.LIL** instruction is used at or neat the beginning
of the source code to permanently change to ADL mode, it is considered to use
ADL mode exclusively.

When the MADL control bit is set to 1, the CPU pushes a byte onto the (SPL?)
stack containing the current memory mode whenever an interrupt, trap, or
restart occurs. This happens even if the memory mode isn't changed by the
event.  A `02h` byte is pushed if the cpu is in Z80 mode. A `03h` byte is
pushed if the cpu is in ADL mode.

Additionally, when the MADL control bit is set to 1, all interrupts begin in
ADL mode.

The MADL control bit is set to 1 by **STMIX** and reset to 0 by **RSMIX**.

### Official MIXED MEMORY Mode Guidelines

Follow these rules when including legacy Z80 code with new ADL code.

* Include a **STMIX** in the initialization code.
* End interrupt service routines with **RETI.L** or **RETN.L**. *This may be
  irrelevant on the TI-84+CE.*
* Use a suffixed **CALL** to execute code in the memory mode in which it was
  assembled. Suffixed **JP** instructions can be used, but **CALL** is
  preferred.
* Any code than can be called from either Z80 or ADL mode must be called with a
  suffix. Similarly, this code must return with a suffixed **RET**
* If calling code operating in one mode must pass stack-based arguments to a
  routine assembled for a different mode, it must use suffixed instructions to
  set up the arguments. For **PUSH**, **.S** and **.L** control whether SPS or
  SPL is used, and whether the arguments are stored as 2- or 3-byte values.

<a name="interrupts"></a>

## Interrupts

### Terminology

* ISR: Interrupt Service Routine - executed when an interrupt occurs.
* NMI: Non-maskable Interrupt - an interrupt that cannot be disabled by the
  programmer.


### Interrupt Enable Flags (IEF1 and IEF2)

* Controlled with **EI** and **DI**.
* **EI** sets IEF1, **DI** reset IEF1.
* When IEF1 is set to 0, the CPU will be accept maskable interrupts.
* IEF2 is used as a temporary storage location for IEF1.
* IEF1 and IEF2 are reset to 0 when an interrupt is accepted, preventing
  further interrupts until a new **EI** is executed.

<a name="interrupts_in_mmas"></a>

### Interrupts in Mixed Memory Mode Applications

* When MADL is set to 1, all ISRs begin in ADL mode.
* ISRs should end with **RETI.L** or **RETN.L**.

### eZ80 CPU Response to a NMI

This section was just a giant table in the official documentation, go see it
yourself if you need to. It's pretty similar to how it works on the Z80, but
with memory mode management added in.

### eZ80 CPU Response to a Maskable Interrupt

There are three interrupt modes, set by **IM 0**, **IM 1**, and **IM 2**. Not
all eZ80 processors support all 3.

### Interrupt Mode 0

* Default interrupt mode on CPU reset
* Interrupting device places an **RST m** on the data bus with the value `C7h`,
  `CFh`, `D7h`, `E7h`, `EFh`, `E7h`, or `FFh`, or a **CALL Mmn** with the value
  `CDh`. Any other values are treated as a NOP.

### Interrupt Mode 1

* The CPU responds to an interrupt by executing **RST 38h**

### Interrupt Mode 2

* The interrupting device places the lower 8 bits of the interrupt vector on
  the data bus. Bit 0 of the byte must be 0. The middle byte is set by the
  CPU's Interrupt Vector Register, I.
* Applications running in Z80 mode exclusively use {MBASE, I[7:0], D[7:0]} as
  the full interrupt vector address. A 16-bit word is fetched and loaded into
  PC[15:0]
* Applications running in mixed-memory or ADL mode use {I[15:0], D[7:0]} as the
  full interrupt vector address. A 24-bit word is fetched and loaded into the
  Program Counter, PC[23:0].

All three interrupt modes have accompanying tables specifying exactly what
happens in different memory modes. Consult the official documentation for these
tables.

## Illegal Instruction Traps

When an eZ80 processor fetches an illegal instruction, it performs a TRAP
operation. This operates similarly to an **RST `00h`** instruction.

```
if ADL mode (ADL = 1) {
    (SPL) = PC[23:0]
    if MIXED MEMORY mode (MADL = 1) {
        (SPL) = 03h
    }
    PC[23:0] = 000000h
} else Z80 mode (ADL = 0) {
    SPS = PC[15:0]
    if MIXED MEMORY mode (MADL = 1) {
        (SPL) = 02h
    }
    PC[15:0] = 0000h
}
```

The memory mode suffixes (**.SIS**, **.SIL**, **.LIS**, and **.LIL**) do not
guarantee illegal instruction traps, even when used with instructions which have
no meaning. For example, **CCF.SIS** is allowed.

## I/O Space

The eZ80 CPU is capable of addressing a 64 KB I/O space with 16-bit addresses
using special I/O instructions. Whenever an I/O instruction is executed, the
upper byte of the 24-bit address bus is undefined.

## Addressing Modes

We know everything in this section. It covers different addressing modes like

* Operations that implicitly reference registers, such as the accumulator
* Indirection with normal and index registers
* RST
* Addressing with immediate values
* Relative addressing like with **JR**
