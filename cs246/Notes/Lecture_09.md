# CS 246 Lecture 9

Old version

```c++
struct Node
{
    ...
    Node &operator=(const Node &other)
    {
        data = other.data;
        delete next;
        next = other.next ? new Node {other.next} : nullptr;
        return *this;
    } // DANGEROUS: n = n??   
};
```

Ok version, but not optimal

```c++
struct Node
{
    ...
    Node &operator=(const Node &other)
    {
        if(this == &other) return *this;
        data = other.data;
        delete next;
        next = other.next ? new Node {other.next} : nullptr;
        return *this;
    } // What if new fails? - the f'n (operators) will abort;
};
```

Better version

```c++
Node & operator=(const Node &other)
{  
   if(this == other) return *this; 
   Node *tmp = next;
   data = other.data;
   next = other.next ? new Node{other.next} : nullptr; // next points at undeleted data
   delete tmp; // never happens if new fails 
   return *this;
}
```

### Aternative: Copy-and-swap idiom
```c++
#include <utility>

struct Node
{
    ...
    void swap(Node &other)
    {
        using std::swap;
        swap(data, other.data);
        swap(next, other.next);
    }
    
    Node &operator=(const Node &other)
    {
        Node tmp = other; // Copy
        swap(tmp); // Swap
        return *this;
    }
};
```

Notice: `operator=` is a member f'n, not a standalone f'n
- When an operator is declared as a member f'n, `*this` plays the role of the first operand

Example:

```c++
struct Vec
{
    int x, y;
    ...
    Vec operator+(const Vec &other){return {x + other.x, y + other.y};}
    Vec operator*(const int k){return {x * k, y * k};}
};

Vec operator*(const int k, const Vec &V){return v*k;}
```

What about I/O operators?

```c++
struct Vec
{
    ...
    ostream &operator<<(ostream &out)
    {
        return out << x << ' ' << y;
    } // would have to write it as v << cout
}
```

Certain operators MUST be members:
- `operator=`
- `operator[]`
- `operator->`
- `operator()`
- `operator T` (where T is a type)

## Seperate Compilation for classes
```c++
// Node.h
#ifndef NODE_H
#define NODE_H
struct Node
{
    int data;
    Node *next;
    explicit Node(int data, Node *next = nullptr);
    bool isSingleton();
};
#endif

// Node.cc
#include "Node.h"
Node::Node(int data, Node *next): data{data}, next{next}
bool Node::isSingleton(){return next==nullptr;}
```
- `::` scope resolution operator

## Const Objects
`int f(const Node &n){...}`
- const objects arise often, especially as params.
- A const object is an object where the fields are immutable.
- Can we call methods on a const object?
 - Issue: the method may modify fields, violate const.
 - A: Yes - we can call methods that promise not to modify fields.
 
Example:

```c++
struct Student
{
    int assns, mt, final;
    float grade() const; // this method doesn't modify fields, so declare it const.
};

float Student::grade() const{_}
```

## Rvalues + Rvalue References
Recall:
- An lvalue is anything with an address.
- An lvalue reference (&) is like a const ptr with auto deref 
    - always initialized to an lvalue

Now consider:

```c++
Node n {1, new Node{2, nullptr}};
Node m = n; // copy ctor
Node m2;
m2 = n; // copy assignment operator
Node plusOne(Node n)
{
    for(Node *p = &n; p; p=p->next) ++p->data;
    return n;
}
Node m3 = plusOne(n);
```
- Compiler creates a temp obj to hold the result of plus one.
- Other is a ref to this temporary
- copy ctor deep copies the data from this temporary

BUT:
- The temporary is just going to be discarded anyway, as soon as the struct `Node m3 = plusOne(n)` is done
- Wasteful to deep copy - Why not steal the data?
- How can we tell whether other is a ref to a temp or a standalone obj?

C++ rvalue ref: `Node &&` is a ref to a temporary obj (rvalue) of type Node.
- Version of the ctor that takes a `Node &&`

```c++
struct Node
{
    // Node(Node &&other) move ctor 
    // Steals data because it's faster than copying
    Node(Node &&other): data{other.data}, next{other.next}
    {
        other.next = nullptr;
    }
}
```