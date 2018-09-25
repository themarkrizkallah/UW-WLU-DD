# CS 241 Lecture 03
**Question:** What happens when a program ends?
- *Recall:* 
    - A *loader* program, loaded your program into *RAM*, then your program completed => return control to *loader*
        - Set *PC* to address of next *instruction* in the loader, which memory address is that?
        - $31 will contain the right address
        - Need to set the *PC* to $31 => `jr $31`

**Example:** Add the value in reg 5 ($5) to the value in reg 7 ($7), store result in reg 3 ($3) and return
```
; Assembly
add $3, $5, $7 ; add $d, $s, $t
jr $31         ; jr $s

; Binary
; add $3, $5, $7 | address: 0
; 0000 00ss ssst tttt dddd d000 0010 0000
; d = 00011, s = 00101, t = 00111
0000 0000 1010 0111 0011 1000 0010 0000

; jr $31 | address: 4
; 0000 00ss sss0 0000 0000 0000 0000 1000
; s = 11111
0000 0011 1110 0000 0000 0000 0000 1000
```

- `lis $d`
    - "load immediate and skip"
    - Treat next *word* as an *immediate value* and load it into $d
    - Then, skip to the following *instruction*
        - `PC = PC + 4`
    - Immediate value probably not an *instruction*


**Example:** Add 42 to 52, store sum in $3 and return
```
; Assembly
lis $5         ; address: 0
.word 42       ; address : 4
list $7        ; address: 8
.word 52       ; address: C
add $3, $5, $7 ; address: 10
jr $31         ; address: 14
```

**Assembly Language**
- Replace *binary* with easier to use mnemonies
- Less chance of errors
- Translation to binary, can be automated
- One line of assembly = one machine *instruction* (*word*)

**Example:** Compute the absolute value of $1, store in $1 and return
- Some *instructions* modify *PC*
    - "Branches and jumps"
        - **i.e.** `jr`
- `beq`
    - Branch if two *registers* have equal contents
    - Increment *PC* by given # of *words*
    - Can branch backwards

- `bne` 
    - Branch if two *registers* don't have equal contents
    - 2 cases: negative or non-negative
        - `slt` "set less than"

```
slt $2, $1, $0 ; Compare $1 < 0 | address: 0
beq $2, $0, 1  ; If false, skip next *instruction* | address: 4
sub $1, $0, $1 ; $1 <- 0 - $1 | address: 8
jr $31
```

**Example:** Looping: Sum the integers 1,2,...,13, store in $3 and return
```
lis $2         ; address: 0
.word 13       ; address: 4
add $3, $0, $0 ; address: 8
add $3, $3, $2 ; address: C
lis $1         ; address: 10
.word -1       ; address: 14
add $2, $2, $1 ; address: 18
bne $0, $2, -5 ; address: 1C
jr $31         ; address: 20
```

**More efficient answer**
```
lis $2
.word 13
lis $1
.word -1
add $3, $0, $0
add $3, $3, $2
add $2, $2, $1
bne $0, $2, -3 ; Careful, updating the offset
jr $31
```
- May have to update offsets if you add, remove, or  move code around
- Instead, the assembler provides *labelled instructions*
- **i.e.** `label: operation operands`
```
foo: add $3, $3, $2 ; address: 14
.
.
.
bne $0, $2, foo     ; address: 1C
jr $31              ; address: 20
```
- `foo` is the memory address of the next *instruction*
    - **i.e.** 14
- Assembler computes:

(foo-PC)/4
= (14-20)/4
= -3

*RAM*
- `lw` "load word" from RAM into register
- `lw $a, i($b)` load word at MEM[$b+i]

**Example:** 
$1 = address of an array, $2 = # of elements of array, Place element 5 in $3

```
lw $3 20($1) ; 20 = 5x4, 4 bytes per word
jr $31
```

General:
```
lis $5
.word 5
lis $4
.word 4
mult $5, $4
mflo $5
add $5, $5, $1
lw $3, 0($5)
jr $31
```
