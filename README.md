# HITCH8
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

- A directory "programs" with all the assembly programs so far created, to be loaded in program EEPROM;

- An .xlsx worksheet with important information regarding the instruction set and microcode memory.

## Blocks

A description of the functionalities of every functional block follows.

- ### ALU

the Aritmetic and logic unit (ALU) is responsible for all the math performed in this CPU. It is a 8-bit ALU.

A and B are two 8-bit registers for storing the operands. The result is written directly on BUS (there is an appropriate 32-bit accumulator register that can be filled, if necessary). From this component 3 status flags are sent to the control unit through a 4-bit status register:

- **c<sub>out</sub>**, namely carry out
- **is_zero**, (tells if the result is 00000000)
- **overflow**,

Moreover, an additional input coming from control unit is used as carry in (**c<sub>in</sub>**)

The operation is selected by 4 bits (**op_sel**):

0000 A+B+c<sub>in</sub>        (addition with **c<sub>in</sub>**=0)
0001 A+not(B)+c<sub>in</sub>   (subtraction with **c<sub>in</sub>**=1)
0010 A+c<sub>in</sub>          (increment A if **c<sub>in</sub>**=1, NOOP if **c<sub>in</sub>**=0)
0011 A-1+c<sub>in</sub>        (decrement A if **c<sub>in</sub>**=0, NOOP if **c<sub>in</sub>**=1)

0100 not(A)
0101 A and B
0110 A or B
0111 A xor B

1000 A>>1            (logic shift right)
1001 A>>1            (aritmetic shift right)
1010 A<<1            (logic/aritmetic shift left)

1011 ---
1100 ---
1101 ---
1110 ---
1111 ---

- ### Registers

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

- ### Program Memory

The EEPROM memory containing the program is addressed by 13 bits (total 8192 assembly instructions). The returned instruction is 26 bits long.

- ### RAM Memory

Addressed by MAR (two registers, one for LSB and one for MSB). The returned data are sent to MDR register.

- ### Control Unit

The instruction word is 26 bits long: the 6 most significant bytes are the OPCODE (reported in _Instruction set.xlsx_), followed by at most 3 operands that can be integers or general purpose registers.

The chosen control strategy is MICROCODE: an additional EEPROM memory addressed by the OPCODE, the status flags and 4 additional bit encoding for the clock step (allowing up to 16 steps per instruction). In particular, the status flags constitute the 3 most significant bits. For example, if an increment operations returns **c<sub>out</sub>**=1, the currently addressed microcode block changes, making it necessary to duplicate the instructions 8 times (2<sup>3</sup>).

The number of control signals is 32, hence each step of the microcode must return a string of 32 bits. To reduce the number of control signals, 3 signals (CU op1/CU op2/CU res reg) are introduced to choose which part of the instruction must select the current register: whether it is the part corresponding to the result register, the first operand register or the second one (since the general purpose registers are selected by a 4 bit sequence, this solution spares 1 bit).

## Instruction Set

The instruction set and all related information can be found in _Instruction set.xlsx_.

## Credits

 - [Ben Eater's 8-bit CPU](https://eater.net/) 
 - [Carl Burch's Logisim](https://sourceforge.net/projects/circuit/)
 - The "microcontrollers and DSPs" course I took during the Bachelor's degree course in Mechatronic Engineering, University of Padova


