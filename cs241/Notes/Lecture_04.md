# CS 241 Lecture 04

- `mult $a, $b`
    - Product of two 32-bit #'s is 64 bits
    - Too big for one *register*
    - 2 special registers, `hi` and `lo`
    - `$a * $b => hi : lo`
- `mflo $a`
    - Move from `lo` to `$a`
- `mfhi $a`
    - Move from `hi` to `$a` 
- Division
    - `lo` stores quotient, `hi` stores remainder
  
### Implementing a procedure *f*
Two Problems:
1. Call and return
    1. How do we transfer control into and out of *f*?
    2. What if *f* calls another procedure *g*?
    3. How do we pass parameters?
2. Registers - What if a procedure we call overwrites a *register* we are using?
    1. **Example**: 
        1. Some critical data in `$3` 
        2. Call *f*
        3. *f* modifies `$3`
        4. On return, critical value is lost

**Call Stack is similar**
- **RAM**:
    - Choice, top or bottom?
    - Keep track of what is in use
- **MIPS**:
    - `$30` initialized (by loader) to store the memory address of the just passed last *word* of memory
    - Can use `$30` as a bookmark to separate used and unused **RAM**, if we allocate from the bottom
  
**Example**:
```
f calls g
    g calls h
        h returns
    g returns
f returns
```

**Stategy**
- Each procedure stores in **RAM** the *registers* it wants to use and restores the original values on returns

**RAM** used in *LIFO* - a *stack*
- `$30` a stack pointer, contains the address of top of *stack*

### Template for procedures:
```
f:
    sw $2, -4($30)   ; push regs f will modify onto stack
    sw $3, -8($30)
    lis $3           ; decrement $30
    .word 8
    sub $30, $30, $3
    ; body of f here, can manipulate $2 and $3
    add $30, $30, $3 ; increment $30
    lw $3, -8($30)   ; load regs
    lw $2, -4($30)
    __________       ; return 
```
1. What about call and return?

**Call**
```
main: 
    lis $5
    .word f ; address of line labelled f
    jr $5 ; jump to that line
    (HERE)
    ....

f: ... // f = address of this line in memory
```

**Return**?
- Need to set *PC* to line after `jr`, **i.e** (HERE)
- How do we know what address that is?
    - `jalr` "jump and link to reg"
    - Like `jr`, but sets `$31` to the address of the next instruction
    - Replace `jr $5` with `jalr $5`
    - Then return from f is `jr $31` => jumps back to (HERE)
    - `jalr` overwrites `$31`

```
main: 
    lis $5
    .word f
    sw $31, -4($30)   ; push $31 onto stack
    lis $31           ; decrement stack pointer $30
    .word -4
    add $30, $30, $31
    jalr $5           ; call f
    lis $31           ; increment stack pointer
    .word 4
    add $30, $30, $31
    lw $31, -4($30)
    ...
    jr $31 // return to loader

f: ...     ; push registers f will use onto stack
           ; decrement $30
           ; body of f
           ; increment $30
           ; pop the registers off the stack
    $jr 31 ; return to caller
```

**Parameter and Result passing**
- General use register (DOCUMENT!!!)
- If too many parameters, can push them onto stack
  
**Procedure Example**: sum1ToN

Add the numbers 1 to N
```
; sum1ToN: adds the numbers 1,..,N
; Registers
; $1 - working, save
; $2 - input (value of N), save
; $3 - output, do not save

sum1ToN:
    sw $1, -4($30) ; Push $1 and $2
    sw $2, -8($30)
    lis $1
    .word 8
    sub $30, $30, $1
    add $3, $0, $0
    lis $1
    .word -1

topOfLoop:
    add $3, $2, $3
    add $2, $2, $1
    bne $2, $0, topOfLoop
    list $1
    .word 8
    add $30, $30, $1
    lw $2, -8($30)
    lw $1, -4($30)
    jr $31
```

 **Recursion**: If you manage *registers*, params, *stack* properly, it will work

 **I/O**: 
 - Output: `sw` to 0xffff000c
    - Least-sig *byte* will be printed
- Input: `lw` from 0xffff0004
    - Next char from stdin will be least-sig byte

**Ex**: print "CS\n"
```
lis $1
.word 0xffff000c
lis $2
.word 67 ; C
sw $2, 0($1)

lis $2
.word 83 ; S
sw $2, 0($1)

lis $2
.word 10 ; \n
sw $2, 0($1)
jr $31
```

### The Assembler
**Input**: Assembly language program

**Output**: Machine code

Any translation process has 2 phases:
1. Analysis - understand what is meant by the source string
2. Synthesis - output the equivalent target string