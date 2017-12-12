# CS 246 Lecture 11
### Fix the linked list class
```c++
// list.h
class List
{
    struct Node; // private nested class - TYPE
    Node *theList = nullptr;
    
public:
    void addToFront(int n);
    int ith(int i);
    ~List();
    ...
};

/**********************************/

// list.cc
#include "list.h"

struct List::Node // Recall: Scope Resolution
{
    int data;
    Node *next;
    Node (int data, Node *next): ....{}
    ~Node(){ delete next; }
    ...
};

void List::addToFront(int n)
{
    theList = new Node{n, theList};
}

int List::ith(int i)
{
    int item;
    Node *cur = theList;
    for(int j = 0; j < i; ++i)
    {
        cur = cur->next;
    }
    return cur->data;
}

List::~List()
{
    delete theList;
}
```
- Only List can create/manipulate Node objects.
- Can guarantee the invariant. `next` is always `nullptr` or a heap ptr.

BUT! Now we can't traverse the list from node to node as we would with a linked list.
- Repeatedly calling ith to access the whole list is O(n^2).
- We can't expose the nodes or we lose encapsulation.

### SE Topic - Design Patterns
- Certain problems/scenarios arise frequently.
- Keep track of good solutions to these problems, use them in similar situations.
- This is the essence of design patterns.
    - The good solution "Hall of Fame"
- Design pattern
    - If you have a situation like THIS, then THIS technique may solve it.

#### Iterator Pattern
- Create a class that manages access to nodes
    - abstraction of a ptr
    - walk the list without exposing the actual ptrs.
- Insipiration: C

```c++
for(int *p = a; p!=a + n; ++p)
{
    cout << *p << endl;
}
```

- Implementation

```c++
class List
{
    struct Node;
    Node *theList;
    
public:
    ...
    class Iterator
    {
       Node *p;
       
    public:
        explicit Iterator(Node *p): p{p}{}
        int &operator*(){ return p->data; }
        Iterator &operator++()
        {
            p = p->next;
            return *this;
        }
        bool operator!=(const Iterator &other) const{ return p!= other.p; }
    }; // Iterator
    
    Iterator begin(){ return Iterator{thelist}; }
    Iterator end(){ return Iterator{nullptr}; }
}; // list

/**********************************/

// Client
int main()
{
    List l;
    l.addToFront(1); l.addToFront(2); l.addToFront(3);
    
    for(List::Iterator it = l.begin(); it!=l.end; ++i)
    {
        cout << *it << endl;
    }
    
} // O(n) traversal
```
- Shortcut: automatic type deduction
    - `auto x = y;`
    - Automatically gives `x` the same type as its initializer, `y`;
    
```c++
for(auto it = l.begin(); it != l.end(); ++it)
{
    cout << *it << endl;
}
```

- "Shorter" cut: range-based for loop:

```c++
for(auto n:l) // n is a copy of list item. Can mutate n, but list will not be mutated
{
    cout << n << endl;
}
```
- Available for any class with methods begin/end that produce iterators
- the iterator type must support unary*, !=, prefix ++;
- If you want to modify list items (or save copying)

```c++
for(auto &n: l)
{
    cout << n << endl;
    ++n; // will mutate list items.
}
```

### Encapsulation - Continued
- List client can create iterators directly:

```c++
auto it = List::Iterator{nullptr}; // violates encapsulation
```
- Client should be using begin/end;
- We could make Iterator's ctor private.
- Client can't call `List::Iterator(_);`
    - However, now List can't call it either
    
Solution: Give List privileged access to Iterator
- Make it a friend

```c++
class List
{
    ...
public:
    Class Iterator
    {
        Node *p;
        explicit Iterator(Node *p);
    public:
        ...
        friend class List; // List has access to all members of Iterator
    };
    ...
};
```
- Now List can still create iterators, but client can only create iterators by calling begin/end;
- Give your classes as few friends as possible because it weakens encapsulation.
- Once again, keep fields private, only make friends with classes that do something for you.

What if you want to provide access to fields?
- accessor and mutator methods.

```c++
class Vec
{
    int x, y;
public:
    ...
    int getX() const { return x; } // accessor
    void setY(int z) {y = z;} // mutator;
};
```