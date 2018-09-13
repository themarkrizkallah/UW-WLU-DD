# CS 241 Lecture 01

### What is a sequential program?
- "ordinary": **NOT** _concurrent_ nor _parallel_ 
- _single-threaded_: one thing happens at a time

## Goal: Foundations
- Understand how sequential programs "work" from the ground up

### Starting Point: Bare hardware (how to speak)
#### At the end:
- Get programs written _C_-like language to run on _MIPS_
- What is "1010"?

### Binary and Hexidecimal Numbers
- **bit**: a _binary_ digit
    - Example:
        -  0 or 1 
        -  high or low voltage
        -  config in CD or magnetic tape
    - All the computer understands
    - Convenient to group bits together, usually in groups of 4
        - 1000 1011, for example
- **byte**: 8 _bits_
    - Example: 1100 1001
    - 2<sup>8</sup> = 256 different _bytes_
- **word**: _machine-specific_ grouping of _bytes_
    - Within this course, 32-_bit_ computer architecture
        - 1 _word_ = 32 _bits_ = 4 _bytes_
    - 64-_bits_ now common 

- **nibble**: 4 _bits_ (half a _byte_)

**Question**: Given a _byte_ (or a _word_) in a computer's memory, what does it mean? 1100 1001

**Answer**: Many things!

1. A number: what number?
   
1 | 1 | 0 | 0  | 1 | 0 | 0 | 1
--- | --- | --- | --- | --- | --- | --- | --- |
2<sup>7</sup> | 2<sup>6</sup> | 2<sup>5</sup> | 2<sup>4</sup>  | 2<sup>3</sup> | 2<sup>2</sup> | 2<sup>1</sup> | 2<sup>0</sup>
128 | 64 | 32 | 16  | 8 | 4 | 2 | 1

= 1 * 2<sup>7</sup> + 1 * 2<sup>6</sup> + 0 * 2<sup>5</sup> + ... + 1 * 2<sup>0</sup>

=  128 + 64 + 8 + 1

= 201

201:

201 - 128 = 73

73 - 64 = 9

9 - 8 = 1

11001001

What about negative numbers?

**Simple way**: first *bit* represents sign
- 0 is '+' and 1 is '-'
- 11001001 = -73, *signed-magnitude representation*

Why don't we like this?
- There are 2 zeroes:
    - 0000 0000 = 0
    - 1000 0000 = 0
- Wasteful
- Arithmetic is hard
    - 0000 0001 + 1000 0000 = ?

Better way: *2's complement representation*

1. Interpret the n-*bit* number as an *unsigned*
2. If the first *bit* is 0, done
3. Else (1st is 1) subtract 2<sup>n</sup>

**Example**:  n = 3 (*2's complement*)
000 | 001 | 010 | 011 | 100 | 101 | 110 | 111
--- | --- | --- | --- | --- | --- | --- | --- |
0 | 1 | 2 | 3 | 4-8= -4 | 5-8 = -3 | 6-8 = -2 | 7-8 = -1

- So n-*bits* represent -2<sup>n-1</sup> ... 2<sup>n-1</sup> - 1
- Addition is "clean" - just an arithmetic mod 2<sup>n</sup>
- **Convenience**: *hexadecimal* notation
    - base 16: 0, 1, ..., 9, A, B, ... F
    - A = 10, F = 15
    - More compact than binary
    - Each *hex* digit = 4 *bits* (*nibble*)
- 11001001<sub>2</sub> is *binary*
-  0xC9 is *hexadecimal* (0x means *hex*)

**Question**: Given a *byte* (1100 1001), how can we tell if it's *signed*, or *unsigned*?

**Answer**: We can't!
- Could be:
    - 201, -55, or -73
- We need to remember what we stored in the *byte*

What if it's not a number?

---

2. A character: which character?
    - We need a mapping between numbers and chars, a convention

- **ASCII**
    - Uses 7-*bits*
    - 8-*bits* can be used for additional chars

1100 1001 is not 7-*bit* ASCII
- 100 1001 = 73 decimal
    - Is ASCII for I
    - A-65: a-97
    - 10: newline '\n'

Other encodings: **EBCDIC** - Unicode
- We will use ASCII

3. An instruction (part of one)
    - For our machine, instructions are 32-*bits* = 4 *bytes*
    - 1100 1001 could be part of an instruction

4. Garbage (unused memory)

---

## Machine Language
- Computers operate on data
- Computer programs **ARE** data themselves
    - von Neumann architecture
        - Reside in the same memory space as the data they operate on
- Possible to write programs that manipulate other programs
    - Example: Operating systems, viruses

**Question**: How do we know which parts of memory contains instructions and which are data?

**Answer**: We don't ...necessarily...