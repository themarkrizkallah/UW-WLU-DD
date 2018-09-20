# CS 241 Lecture 05

### The Assembler
- **Input**: Assembly Code
- **Output**: Machine Code

**Translation** - 2 phases:
1. Analysis - understand the meaning of the string
2. Synthesis - output equivalent target string

- Assembly file - stream of characters
    - Group characters into meaningful *tokens*
    - **e.g.** *label*, *hex* #, *reg* #, `.word`, etc.

**Your Job**:
1. Group *tokens* into instructions (if possible) - *Analysis*
    1. *Scanner*
2. Output equivalent machine code - *Synthesis*

### Basic Strategy
- Focus on finding valid assembly language sequences
- Anything else is an *ERROR* - output to `stderr`

**Big Problem**
- Labels might not come until later in the sequence
- *Semantic* error - doesn't have a meaning
```
bne $0, $2, here ; we don't know what 'here' is yet
.
.
.
here: ...
```

#### 2 pass assembler
1. **Pass 1**
    - Group *tokens* into instructions
    - Record the address of all labelled instructions
    - "*Symbol* table" - a list of (*label*, *address*) pairs
        - Hash table/map

**Note**: A line of *assembly* may have multiple labels for the same memory address
```
f:
g:
    mult $1, $2
```
- Valid, both *labels* have the same value

2. **Pass 2**
    - Translate each instruction to machine language
    - lookup *labels* when necessary

**Your Assmbler** - IMPORTANT
- Output assembled *MIPS* to `stdout`
- Output the "*symbol* table" to `stderr`

**Example**
```
main:   
    lis $2          ;       0
    .word 13        ;       4
    add $3, $0, $0  ;       8

top:    
    add $3, $3, $2  ;       C
    lis $2          ;      10
    .word -1        ;      14  
    add $2, $2, $1  ;      18
    bne $2, $0, top ;      1C
    jr $31          ;      20

beyond:             ;      24
```
We examine what happens
1. **Pass 1**
    - Group *tokens* into *instructions*
    - Build *symbol table*
        - First label `(main, 0x0)`
        - Second label `(top, 0xC)`
        - Third label `(beyond, 0x24)`
2. **Pass 2**
    - Translate each *instruction*
    - **e.g.** 
        - `lis` => 0000 0000 0000 0000 0001 0000 0001 0100
            - hex: 0x00001010
        - `.word 13` => 0000 0000 0000 0000 0000 0000 0000 1101
            - hex: 0x0000000D
        - `bne $2, $0, top`
            - lookup `top` `(0x0C)`
            - calculate: (top-pc)/4 = (c-20)/4 = -5
            - assembly: `0x1440FFFB`

**Note** To negate 2's complement #, flip bits and add 1

**Example** 
- 5<sub>10</sub> = 00000000 00000101
- -5<sub>10</sub> = 11111111 11111010 + 1
- -5<sub>10</sub> = 1111 1111 1111 1011 => `0xFFFB`

#### Bit-level operations
|- - - - - -| - - - - - | - - - - - | - - - - - - - - - - - - - - - - |

6 bits | 5 bits | 5 bits | 16-bits
- First 6 bits
    - opcode
    - bne = `000101` = 5<sub>10</sub>
- Next 5 bits
    -  reg #s
    -  $2 = 2<sub>10</sub>
- Next 5 bits
    - reg t
    - $0 = 0<sub>10</sub>
- Next 16 bits
    -  Offset i

To put `0001010` into first 6 bits, we need to append 26 zeroes
1. left-shift by 26 bits (arithmetic-shift 5-26) - `5<<26`
    - We get `0001010...0` with 26 zeroes

2. Move `$2` 21 bits to the left: `(arithmetic-shift 2-21)` ` 2<<21`

3. Move `$0` 16 bits to the left: `(arithmetic-shift 0-16)` ` 0<<16`

4. -5<sub>10</sub> = `0xFFFFFFF`<sub>16</sub> (32 *bits*, 8 *bytes*) Only want last 16 bits
- bitwise-AND with `0xFFFF` => `(bitwise-and -5 #xFFFF)` `5&0xFFFF`

**Put together** - bitwise OR all pieces
- `(bitwise-or (1) (2) (3) (4))` or in C/C++ `(1)|(2)(3)|(4)`

**Binary Output**: Assembled instruction: `339804155`<sub>10</sub>
- `cout << 339804155` - Prints out as 9 *bytes*

To send raw bytes to `stdout`, print as *chars*

Suppose we have:
```c++
int instr = 339804155;

char c = instr >> 24;
std::cout << c;

c = instr >> 16;
std::cout << c;

c = instr >> 8;
std::cout << c;

c = instr;
std::cout << c;
```

### Formal Languages
- **Assembly Language**
    - **Structure**: Simple
    - **Recognition**: Easy
    - **Translation to ML**: 1-to-1, unambiguous
- **High-level Language**
    - **Structure**: More complex
    - **Recognition**: Harder
    - **Translation to ML**: usually no single translation

**Question**: How do we handle complexity?
- Want a formal theory of string recoginition
- General principles that work in the context of any programming language

**Definitions**
- *alphabet* - finite set of symbols
    - **e.g.** {a,b,c}
    - Denote by *Sigma* = {a,b,c}
- *string* (or *word*)
    - Finite sequence of symbols from *Sigma*
    - **e.g.** a, aba, ab, abccbbac, ...
- length of a string |w| = # chars in w 
    - **e.g.** |aba| = 3
- empty *string*
    - Empty sequence of symbols
- *language*
    - Set of strings
    - **e.g.** {a<sup>2n</sup>b, n >= 0}
        - Language of strings with even # of a's followed by b

**Note** 