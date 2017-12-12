# CS 246 Lecture 10

Recall: Move ctor

```c++
struct Node
{
    ...
    Node(Node &&other): data{other.data}, next{other.next}
    {
        other.next = nullptr;
    }
};
```

Similarly, 

```c++
Node m;
m = plusOne(n);
```

Move assignment operator:

```c++
struct Node
{
    ...
    Node &operator=(Node &&other) // Steal other's data
    {
        using std::swap;
        swap(data, other.data); // Destroy my old data
        swap(next, other.next); // Easy: swap without the copy
        return *this; // temp will be destroyed, take old data with it
    }
};
```

- If you don't write move ctor/assignment, the copy versions will be used.
- If the move ctor/assignment is defined, it will replace all calls to the copy ctor/assignment when the arg "`other`" is an rvalue

## Copy/Move Elision
```c++
Vec makeAVec()
{
    return {0,0}; // invoke basic ctor
}

Vec v = makeAVec(); // What runs? 
// Copy ctor? (if Vec doesn't have a move ctor)
// Move ctor? (if exists a move ctor)
```

- In some circumstances, the compiler is allowed to skip calling copy/move ctor (but doesn't have to).
- In the example above: `makeAVec`writes its result, `{0, 0}`, direclty into the space occupied by v in the caller, rather than copy it later.

Ex:

```c++
void doSomething(Vec v){ ... } // pass-by-value - copy or move ctor
doSomething(makeAVec()); // result of makeAVec() written directly into the param, no copy or move;
```
- This is allowed, even if dropping ctor calls would change the behaviour of the program (e.g. if ctors print something).
    - We are not expected to know exactly when copy/move elision is allowed, just that its possible.
- If you need all the ctors to run:
    - `g++ -fno-elide-constructors file.cc`
    - This can slow the program down considerably.

### Rule of 5 (Big 5)

If you need to customize any one of
- copy ctor
- copy assignment
- dtor
- move ctor
- move assignment

then you usually need to customize all 5.

## Arrays of Objects
```c++
struct Vec
{
    int x, y;
    Vec(int x, int y): x{x}, y{y}{}
};

Vec *vp = new Vec[15]; // Doesn't compile
Vec moreVecs[10]; // Doesn't compile
```
- C++ doesn't let an object go unconstructed
- These want to call the default ctor on each item - no default ctor for Vec.
- Options:
    - Provide a default ctor
        - Not a good idea
    - For stack arrays:
        - `Vec moreVecs[] = {Vec{0,0}, Vec{1,1}, Vec{2,2}};`
        - For heap arrays: - create an array of ptrs:
            - `Vec **vp = new Vec*[3]; V[0] = new Vec{0,0}; ...`
            - Don't forget to delete the elements and the array.

## Invariants + Encapsulation
Consider:

```c++
struct Node
{
    int data;
    Node *next;
    Node(int data, Node *next);
    ...
    ~Node(){delete next;}
};

Node n1 {1, new Node{2, nullptr}};
Node n2 {3, nullptr};
Node n3 {4, &n2};
```
- What happens when these go out of scope?
    - n1: dtor runs, deletes the whole list.
    - n2, n3: n3's dtor tries to delete n2, but n2 is on the stack, not the heap!
        - Undefined behaviour!
- `Node` relies on assumption for its proper operation
    - `next` is either `nullptr` or a valid ptr to the heap, allocated by `new`
- This is the essence of an invariant
    - a statement that always holds true
    - the invariant is that `next` is either `nullptr` or allocated by `new`, upon which `Node` relies
    - We can't guarantee the invariant, can't trust user to use `Node` properly
- Right now, can't enforce any invariants; thus, the user can interfere with our data.

Eg. stack:
- Invariant: last item pushed is the first item popped
    - but not if the client can rearrange the underlying data
- Hard to reason about programs without invariants.

To enforce variants, introduce ENCAPSULATION
- Want clients to treat our objects as black boxes, or capsules.
- Implementation details sealed away
- Can only interact via provided methods and or functions
- Creates an abstraction, regains control over our objects

Eg.

```c++
struct Vec
{
    Vec(int x, int y);
private:
    int x, y;
public:
    Vec operator+(const Vec &other);
    ...
};
```
- Default visibility in structs is public

In general, fields should be private, only methods should be public.
- Better to have default visibility = private
- Switch from `struct` to `class`

```c++
cass Vec
{
    int x,y;
public:
    Vec(int x, int y);
    Vec operator+(const Vec &v);
    ...
};
```
- Only difference between class & struct is default visibility
    - public in `struct`, `private` in class.

Fix the linked list:
```c++
// list.h
class List
{
    struct Node;
    Node *theList;
}
```