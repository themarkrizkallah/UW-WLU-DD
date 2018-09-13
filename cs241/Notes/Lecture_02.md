# CS 241 Lecture 02

## Overview
**High Level Program**
- **i.e.** C/C++ program
- After it goes through the *compiler*, we get:

**Assembly Language Program**
- Low level English-like Instructions
- After it goes through the *assmbler*, we get:

**Machine Language**
- 1's and 0's version (*binary*)
- CPU understands this!
- We must then run it on our hardware using a *loader*
    - **Memory** (*RAM*)
    - **CPU**

## Computer Programs
- Operate on data
- Are data themselves
- **von Neumann architecture**
- Reside in the same memory as the data they operate on
- Possible to write programs that not only manipulate data, but also manipulate other programs
    - **i.e.** OS, viruses

**Question:** How do we know whether memory contains *machine instructions*?
- We don't (necessarily!)

**Question:** What does a *machine instruction* look like?
- Specific *binary* sequence
- Many different Machine languages
- Processor specific

### MIPS (Simplified)
- **CPU**: central processing unit
    - *CU*: control unit
        - Decodes instructions
        - Dispatches to other parts of the computer to carry them out
    - *ALU*: arithmetic/logic unit
        - Does math

- **Memory**
    - **CPU** *cache* 
        - FAST
    - **RAM**
        -  "Main memory"
    - *Disk*
        - **i.e.** Hardrive, SSD
    - *Network*
        - Slow
        - Cheap

On the **CPU**: small amount of very fast memory
- Called *registers*
- *MIPS*
    - 32 "general purpose" *registers*
- Each *register* holds 32 *bits*
    - **i.e.** one *word*
- **CPU** can only operate on data that is stored in *registers*
- Called $0, $1, ..., $31
- $0 is ALWAYS 0, immutable
- $31 is also special
- $30 is sort of special
- **Ex:** *reg* operation:
    - "Add contents of *registers* s and t and put the result in *register* d"

**Question:** How many *bits* are needed to encode a register #?
- \# reg = 32 = 2<sup>5</sup> => 5 *bits*

**Question:** How much space for our operation parameters?
- 3 *regs* * 5 *bits* = 15 *bits*
- 32 *bits* in a register - 15 = 17 *bits* left to encode the operation

**RAM**
- Large amount of memory stored away from the **CPU**
- Data travels between **RAM** and the **CPU** via the *bus*
- Big array of n *bytes*, n is large
- Each cell has an address 0, 1, ..., n-1
- Each 4 *byte* block 4k, ..., 4k+3 is a *word*
- *Words* have address 0, 4, 8, ..., C, 10, 14, 18, 1C
- **RAM** access is much slower than *register* access
- Data in **RAM** must be transferred to *registers* before it can be used by **CPU**

Communicating with **RAM**: 2 operations:
- *Load*
    - Transfer a *word* from an address to a *register*
    -  Desired address goes into *Memory Address Register* (*MAR*)
    -  This goes out on the *bus*
    -  Data at that location comes back on the *bus*
        -  Goes into the *Memory Data Register* (*MDR*)
    - Value in *MDR* is moved to the destination *reg*
- *Store*
    - Does the reverse

**Question:** How does a *machine instruction* execute?
- Special *register* called *PC* (*Program Counter*)
    - Holds the address of the next instruction to execute
- By convention, we guarantee that a specific address (**i.e.** 0) contains code
    - Initialize *PC* to 0
- Computer runs the *Fetch-Execute* cycle
    ```
    PC <- 0
    loop
        // IR: Instruction Register holds current instruction
        IR <- MEM[PC] 
        PC <- PC + 4
        decode and execute instruction in IR
    ```
- Only program the machine really runs
- Note: *PC* holds address of the *next* instruction when the *current* instruction is executing

**Question:** How does a program get executed?
- Program called the *Loader* puts the program in memory and sets the *PC* to the address of the *first* instruction

**Question:** What happens when a program ends?