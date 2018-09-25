# CS 241 Lecture 06
## Formal Languages
- *alphabet* - finite sequence of symbols from `sigma`
- length of string `w`: `|w|`
- empty string `epsilon`(gonna call it `e`): `|e|`
- language set of strings   
    - **e.g.** {a<sup>2n</sup>b, n >= 0}

**Question**: How to automatically recognize if a given string belongs to a language?
- Depends on how complex the language is
    - {a<sup>2n</sup>b, n> =0 }
        - easy, `sigma` = {a,b}
    - {valid MIPS assembly progs}
        - fairly easy
    - {valid Java, or C/C++ progs}
        -  Harder
    - Som languages
        - Impossible
- We typically characterize languages according to their complexity to recognize
    - Finite - easy
    - Regular
    - Context-free - harder
    - Context-sensitive
    - Recursive - hard
    - Recursively enumerable
    - etc. (impossible)

### Finite languages 
- Only finite # of words/strings in Language Strings
- **e.g.**
    - L = {cat, cow, car}
    - `sigma` = {a,b,..,z}
- Given string `w` as input:
    1. read char by char
    2. can't go back once we've read char
    3. can't store chars, can't see previously seen chars

    ```
    => scan input left to right
        if first char is a 'c'
            if 2nd char is an 'a'
                if 3rd char is 'r'
                    if no more input, then accept
                    else reject
                else if 3rd char is 't'
                    ...
            if 2nd char is an 'o'
                ...
        else reject
    ```
- Configuration of a program can be represented by a trie, where each bubble is a "state"
- "Scanner" or *tokenizer*

### Regural Languages
Built from:
- finite languages
- union of finite languages
- concatenation
- repetition
    - Kleene Star *

**e.g.**: Concatenation
- L1 = {cat, dog}
- L2 = {fish, `epsilon`}
- L1*L2 = {xy | x in L1, y in L2}
    - L1.L2 = {catfish, dogfish, cat, dog}

**e.g.**: Repition
- L<sup>*</sup> = {`epsilon`} U {xy | x in L<sup>*</sup>, y in L}
    - L = {a,b}, `sigma` = {a,b,...,z} or possibly `sigma` = {a,b}
    - L<sup>*</sup> = {`epsilon`,a,b,aa,ab,ba,bb,aaa,aab,aba,abb,baa,bab,bba,bbb,...}

Show {a<sup>2n</sup>b, n >= 0} is regular, `sigma` = {a,b}
- ({aa})<sup>*</sup> * {b}

**Regular expressions**:
- Null
    - **Set notation**: {}
    - **Description**: Empty language
- `epsilon`
    - **Set notation**: {`epsilon`}
    - **Description**: language consisting of empty string
- a
    - **Set notation**: {a}
    - singleton language - language with 1 string
- E<sub>1</sub>|E<sub>2</sub> or  E<sub>1</sub>+E<sub>2</sub>
    - **Set notation**: L<sub>1</sub> U L<sub>2</sub>
    - **Description**: alternation
- E<sub>1</sub> * E<sub>2</sub> or E<sub>1</sub>E<sub>2</sub>
    - **Set notation**: L<sub>1</sub>L<sub>2</sub>
    - **Description**: concatenation
- E<sup>*</sup>
    - **Set notation**: L<sup>*</sup>
    - **Description**: repitition

**Precedence**: 
- * before <sup>*</sup>
    - aa* = a(a)<sup>*</sup>
- * before |
    - aa|b = (aa)|b

**Question:** Is C a regular language?
- A C program is a sequence of tokens, each of which comes from a regular language

C (valid C programs) is a subset of {tokens}<sup>*</sup>, {tokens} is finite

**Question**: How to automatically recognize Reg Langs?
- {a<sup>2n</sup>b, n >= 0}
- ab - "implicit error state"
- Every state has a transition for each character of alphabet

## Deterministic Finite Automata (DFA)
- **Formally**:
    - DFA M is a 5-tuple, M = {`sigma`, Q, q<sub>0</sub>, A, `delta`}
    - `sigma` = a finite non-empty alphabet
    - Q = a finite set of states
    - q<sub>0</sub> in Q - a start state
    - A subset of Q - set of accepting states
    - delta: (Q x `sigma`) -> Q<sup>Transition function</sup>
        - From a given state, on a given alphabet char, next state
        - Consumes a single char/symbol (extended transition function)

`delta`<sup>*</sup>(q, `sigma`)->q

`delta`<sup>*</sup>(q, cw)

= `delta`<sup>*</sup>(`delta`(q,c), w) 

c in `sigma`, w in `sigma`<sup>*</sup>

**DFA M**
- L(M) is the language of strings accepted by DFA M

**Regular Expression E**
- L(E) is the language of all strings represented by E

`sigma`<sup>*</sup>
- set of all possible strings over `sigma`

**Theorem** (Kleene)
- L is reuglar iff L=L(M) for some DFA M

**Implementation**
```
int state = q0
char ch

while NOT at EOF do
    read ch
    case state:
        qo:
            case ch:
                a: state...
                b: state...
                ...
        q1: case ch:
            ...
        ...
        qn: case ch:
            ...
    end case
end while

return state in A (boolean)
```