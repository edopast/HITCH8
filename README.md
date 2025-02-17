![logo](https://raw.githubusercontent.com/edopast/HITCH8/master/hitch8graphic.png)

Project files for a 8-bit homebrew CPU

This repository holds a set of files used to design a homebrew CPU based on a Harward architecture, an 8-bit bus and 8-bit registers, just for educational purposes.

The name is an acronym paying homage to Douglas Adams' novel _The Hitchhiker's Guide to the Galaxy_:

**H**omebrew\
**I**C-based\
**T**actical\
**C**PU (for)\
**H**itchhikers (with)\
**8**-bit (architecture)

This project is intended as a purely recreational diversion for electronics and computer science enthusiasts. The author is aware that the choices made for this project are far from the efficiency of professional CPUs: he just wanted to put into practice all the stuff learned at University :slightly_smiling_face:

## Content of the repo

In this repo you will find:

- 2 logisim project, namely "HITCH8_w_buttons.circ", that is the first project in which every circuital block must be activated by hand, and "HITCH8.circ", the current project in which buttons are replaced by control unit signals;

- A directory "microcode" with the commented microcode (i.e. the memory to be stored in the microcode EEPROM, inside the control unit) and the uncommented microcode to be loaded;

- A directory "programs" with all the assembly programs created so far, to be loaded in program EEPROM;

- An .xlsx worksheet with important information regarding the instruction set and microcode memory.

## Blocks

A description of the functionalities of every functional block follows.

### 1. ALU

the Aritmetic and logic unit (ALU) is responsible for all the math performed in this CPU. It is a 8-bit ALU.

A and B are two 8-bit registers for storing the operands. The result is written directly on BUS (there is an appropriate 32-bit accumulator register that can be filled, if necessary). From this component 3 status flags are sent to the control unit through a 4-bit status register:

- **c<sub>out</sub>**, namely carry out. Used to tell the control unit whether an addition produces a carry-out bit, OR if a bit-wise shift discards a bit = 1.
- **is_zero**, (tells if the result is 00000000)
- **overflow**,

Moreover, an additional input coming from control unit is used as carry in (**c<sub>in</sub>**)

The operation is selected by 4 bits (**op_sel**):

0000 A+B+c<sub>in</sub>        (addition with **c<sub>in</sub>**=0)\
0001 A+not(B)+c<sub>in</sub>   (subtraction with **c<sub>in</sub>**=1)\
0010 A+c<sub>in</sub>          (increment A if **c<sub>in</sub>**=1, NOOP if **c<sub>in</sub>**=0)\
0011 A-1+c<sub>in</sub>        (decrement A if **c<sub>in</sub>**=0, NOOP if **c<sub>in</sub>**=1)\
0100 not(A)\
0101 A and B\
0110 A or B\
0111 A xor B\
1000 A>>1            (logic shift right. bit out --> c_out, c_in --> new bit in)\
1001 A>>1            (aritmetic shift right. bit out --> c_out)\
1010 A<<1            (logic/aritmetic shift left. bit out --> c_out, c_in --> new bit in)\
1011 ---\
1100 ---\
1101 ---\
1110 ---\
1111 ---

### 2. Registers

8 bits registers are used since the BUS is only 8 bit. These registers are written on rising front of **in** signal.

Beyond ALU's registers (A, B, status and Accumulator) there are:

- MAR: 2 8-bit registers for RAM memory address
- MDR: 1 8-bit register for RAM data in/out
- PC: 2 8-bit registers for program counter
- SP: 2 8-bit registers for stack pointer

Moreover, general purpose registers can be used:

- R0: 2 8-bit registers
- R1: 2 8-bit registers
- R2: 2 8-bit registers
- R3: 2 8-bit registers
- R4: 4 8-bit registers
- R5: 4 8-bit registers

### 3. Program Memory

The EEPROM memory containing the program is addressed by 13 bits (total 8192 assembly instructions). The returned instruction is 26 bits long.

### 4. RAM Memory

Addressed by MAR (actually two registers, one for LSB and one for MSB). The returned data are sent to MDR register.
This module is meant to store variables (the user/compiler must keep track of their address and size in memory, as a sort of "heap" area). Moreover, a register called "stack pointer" is used to point to the first 8-bit free address, forming the stack. In order to separate stack and heap, the former starts from 0x0000 and is filled incrementing the address, while the latter starts from 0xFFFF and is filled decrementing the address.
MAR is 16-bits, MDR is 8-bits. each combination of bits of MAR points to a different byte of RAM. 16-bits and 32-bits data are stored following the little-endian convention.

### 5. Control Unit

The instruction word is 26 bits long: the 6 most significant bytes are the OPCODE (reported in _Instruction set.xlsx_), followed by at most 3 operands that can be integers or general purpose registers.

The chosen control strategy is MICROCODE: an additional EEPROM memory addressed by the OPCODE, the status flags and 4 additional bit encoding for the clock step (allowing up to 16 steps per instruction). In particular, the status flags constitute the 3 most significant bits. For example, if an increment operations returns **c<sub>out</sub>**=1, the currently addressed microcode block changes, making it necessary to duplicate the instructions 8 times (2<sup>3</sup>).

The number of control signals is 32, hence each step of the microcode must return a string of 32 bits. To reduce the number of control signals, 3 signals (CU op1/CU op2/CU res reg) are introduced to choose which part of the instruction must select the current register: whether it is the part corresponding to the result register, the first operand register or the second one. Since the general purpose registers are selected by a 4 bit sequence, this solution spares 1 bit.
A similar strategy has been used to select which part of the instruction must be written on the BUS (because it holds an integer value to be sent to some other block).

The full list of control signals can be found in _Instruction set.xlsx_.

## Instruction Set

The complete instruction set and all related information can be found in _Instruction set.xlsx_.

Supported instructions are:

### General operations

- NOP: no operation, go to next instruction;
- HLT: stop execution;

### Bitwise operations

- AND: bitwise AND (8 or 16 bits) between two registers;
- OR: bitwise OR (8 or 16 bits) between two registers;
- SHL: bitwise logic shift to the left the content of a register (8, 16 or 32 bits);
- SHR: bitwise logic shift to the right the content of a register (8, 16 or 32 bits);
- SHRA: bitwise aritmetic shift to the right the content of a register (8, 16 or 32 bits);

### Basic arithmetic operations

- ADD: addition between two registers (8, 16 or 32 bits);
- SUB: subtraction between two registers (8, 16 or 32 bits);
- ADDI: register plus 8-bit immediate (8 bits reg);
- SUBI: register minus 8-bit immediate (8 bits reg);

### RAM management

- LDW: load from RAM memory to a register (8 or 16 bits);
- STW: store content of a register to RAM memory (8 or 16 bits);
- LDWSP: load from RAM memory to a register (8 or 16 bits). Address stored in Stack Pointer;
- STWSP: store content of a register to RAM memory (8 or 16 bits). Address stored in Stack Pointer;

### Branches and jumps

- BRZ: branch to program instruction if register (8, 16 or 32 bits) = 0;
- BRG: branch to program instruction if register (8, 16 or 32 bits) > 0;
- BRGE: branch to program instruction if register (8, 16 or 32 bits) >= 0;
- BRL: branch to program instruction if register (8, 16 or 32 bits) < 0;
- BRLE: branch to program instruction if register (8, 16 or 32 bits) <= 0;
- JMP: unconditional jump to instruction;

### Move and write registers

- MOV: move content of general-purpose register to another one, (8, 16 or 32 bits);
- MOVI: move 8bit or 16bit number to general-purpose register;

### Stack pointer management

- ADDSP: add 8-bit immediate to stack Pointer;
- SUBSP: subtract 8-bit immediate to stack Pointer;
- MOVSP: move value of stack pointer to another register;
- RESSP: reset stack pointer;

## Programs

So far, two samples programs have been created. They can be found in programs folder:
- MUL8 performs multiplications between two unsigned 8-bits integers
- MUL16 performs multiplications between two unsigned 16-bits integers
More examples in the future.

## Credits

 - [Ben Eater's 8-bit CPU](https://eater.net/) 
 - [Carl Burch's Logisim](https://sourceforge.net/projects/circuit/)
 - The "microcontrollers and DSPs" course I took during the Bachelor's degree course in Mechatronic Engineering, University of Padova