# CS 246 lecture 5

Recall:

```c++
struct Node
{
    int data;
    Node *next;
},

Node n1 = {5,nullptr};
const Node n2 = n1;
const Node *p1 = &1 // Can change n1, but only directly; not through p1
Node *p2 = &n2; // Illegal
```
- Can have ptr-to-const pointing to non-const.
- Cannot have ptr-to-non-const pointing to const.

## Parameter Passing

Recall:
```c++
void inc(int n){n = n + 1;}
...
int x = 5;
inc(x);
cout << x; // 5 - Why?
```
- Pass-by-value: `inc` gets a copy of x, increments the copy
- Original unchanged

Solution
- If a f'n needs to modify a param, then pass a ptr.

```c++
void inc(int *n){*n = *n + 1;}
...
int x = 5;
inc(&x); 
/*  x's address passed by value 
    Inc changes the data at that address 
    Visible to caller */
cout << x; // 6
```
- Q: Why `cin >> x;` and not `cin >> (&x);` ?
- A: C++ has another ptr-like type: references

## References - IMPORTANT!
```c++
int y = 10;
int &z = y; 
/* z is an lvalue reference to int (to y)
   like a const ptr
   similar to int *const z = &y, always going to point at y. */
z = 12; // NOT *z = 12; Now, y == 12
int *p = &z; // Gives the address of y
```
- References are like constant ptrs with automatic dereferencing.
- In all cases, z behaves exactly like y.
- z is an ALIAS for y.

### Things you CAN'T do with lvalue references:
- Leave them uninitialized. ex: `int &x;`
    - Must be initialized with something that has an address (ex: an lvalue) since refs are ptrs.
    - `int &x = 4;` WRONG!
    - `int &x = y + z;` WRONG!
    - `int &x = y;` Ok!
- Create a ptr to a reference.
    - `int &*x = ___;` WRONG!
    - A reference to a pointer is fine. 
        - `int *&x = ___;` Ok!
- Create a reference to a reference.
    - `int &&r = ___;` WRONG!
        - Will compile.
        - Means something different (Rvalue, will get to it later...)
- Create an array of references.
    - `int &r[3] = {__, __, __};`

### Things you CAN do with lvalue references
- Pass them as f'n arguments
```c++
// Const ptr to the arg (x). Changes to n affect x
void inc(int &n){n = n + 1;}
...
int x = 5;
inc(x);
cout << x; // 6
```
- Why does `cin >> x` work? - B/c it takes x by reference.
    - `istream & operator >> (istream &in, int &x);` // Can't copy stream

Pass-by value: 
- Eg. `int f(int n){...}` copies the argument
- If the argument is big, copy is expensive.

Eg.

```c++
struct ReallyBig{...};
int f(ReallyBig rb){...} // copy - slow
int g(ReallyBig &rb){...} // alias - fast. But could change rb in the caller.
int h(const ReallyBig &rb){...} // alias - fast. Param is immutable
```
Advice: prefer pass-by-const-ref over pass-by-value for anything larger than a *ptr, uness the f'n needs to make a copy anyway. THEN use pass-by-value.

Also:

```c++
int f(int &n){...}
int g(const int &n){...}
f(5); // Illegal. Can't initialize an lvalue ref (n) to a literal value.
g(5); // Ok! Since n can never be changed, the compiler allows this.
// How? Compiler creates a temporary location in memory to hold the 5, so the reference n has something to point to.
```
## Dynamize Memory Allocation:
C: 
```c
int *p = malloc(___ * sizeof(int));
...
free p;
```
- Don't use in C++.

C++:
- Use: `new`/`delete` - type aware, less error-prone
```c++
Node *np = new Node;
```
- Allocated memory resides on the heap.
- Remains allocated until delete is called.
- If you don't delete all allocated memory, that is called a memory leak.

Eg.
```c++
// Expensive* - n is copied to caller's stack frame on return.
Node getMeANode()
{
    Node n;
    return n;
}
```
- Return a ref or a ptr instead?

```c++
// Literally the worst thing ever - returns a ptr to stack allocated data, which is dead on return.
Node &getMeANode()
{
    Node n;
    return n;
}
```

```c++
// Ok! Return ptr to the heap, still alive
Node *getMeANode()
{
    return new Node; // Don't forget to delete when done.
}
```

## Operator Overloading
Give meanings to c++ for new types.

Eg.
```c++
struct Vec
{
    int x,y;
};

Vec operator+(const Vec&v1, const Vec&v2)
{
    Vec v{v1.x + v2.x, v1.y + v2.y};
    return v;
}

Vec operator*(const int k, const Vec &v1)
{
    return {k*v1.x, k*v1.y};
}

// Overloading
Vec operator*(const Vec &v1, const int k)
{
    return k*v1;
}
```

Allows us to do:
```c++
Vec v,w,x;
v = w + x;
```