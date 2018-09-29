# CS 241 Lecture 07

DFA M = (`sigma`, Q, q<sub>0</sub>, A, `delta`)

`delta`: (Q x `sigma`) -> Q
- Q = state
- `sigma` is the input symbol
- Second Q is the next state

**Extended transition function**: `delta`<sup>*</sup>(q, `sigma`) = q

= `delta`<sup>*</sup>(q, cw)

= `delta`<sup>*</sup>(`delta`(q,c), w)

**Acceptance**: DFA M accepts w if `delta`<sup>*</sup>(q<sub>0</sub>, w) in A

L(M) - the "language of M" is the set of strings accepted by M

= {w | M accepts W} = {w | `delta`<sup>*</sup>(q<sub>0</sub>, w) in A}

**Theroem** (Kleene): L is regular iff L = L(M) for some DFA M

**DFA with actions**: can attach computations to the edges of a DFA
- **Possible actions**: could be "omit tokens" - as in a scanner

**Example**: 
- L = {binary #'s with no leading zeroes}
- `Sigma` = {0,1}
- **Reg expression**: 1(1|0)<sup>*</sup> | 0

Simultaneously compute value of a number N

**Example**:
- L = {w | w ends in abb}
- `sigma` = {a, b}

---

## Nondeterministic Finite Automata (NFA)
- More complex
- Allow more than 1 transition on a symbol from a state
- Machine accepts when at least one path leads to an accepting state
- Machine chooses one direction or the other
- Often simpler to design or draw
- Machine "guesses" which path to take

**Formally**: NFA M is a 5-tuple, (`sigma`, Q, q<sub>0</sub>, A, `delta`)
- `delta`: (Q x `sigma`) -> 2<sup>Q</sup> (set of states)
    - **e.g.** `delta`(1, a) -> {1, 2}

```
Trace baab

Read      NotRead      SetofStates
sigma     baabb        {1}
b         aabb         {1}
ba        abb          {1,2}
baa       bb           {1,2}
baab      b            {1,3}
baabb     e            {1,4} intersection with A not empty, accept
```

`delta`<sup>*</sup> for NFAs: set of states x `sigma` -> set of states

`delta`<sup>*</sup>(qs, `sigma`) = qs (set of states)

`delta`<sup>*</sup>(qs, cw)

= `delta`<sup>*</sup>(U(q from qs)`delta`(q,c), w)

Accept when `delta`<sup>*</sup>({q<sub>0</sub>}, w) intersection with A != empty

### Implementation:
```
states <- {q0}
while not EOF do
    ch <- read
    states <- U(q in states) d(q, ch)
end do
if states intersection with A is not empty then accept, else reject
```
- We can create a DFA where each set of states is a state of the DFA and add transitions for each character in `sigma`

**More complex example**:
- `Sigma` = {a,b,c}
- L = {cab} U {string with even # of a's}

```
Trace: caba

Read     NotRead     States
e        caba        {1}
c        aba         {2,6}
ca       ba          {3,5}
cab      a           {4,5}
caba     e           {6} - accepting
```
- Where can we go from {1} on a,b,c
- From {2,6} where can we go on a,b,c?
- Accepting states: any set that intersection with A is not null
    - Any srare with 1,4,6