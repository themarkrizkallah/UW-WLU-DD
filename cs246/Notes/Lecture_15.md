# CS 246 Lecture 15
Recall: `*pb1 = *pb2; // Book ptrs pointing at Text objects`
- If `operator=` non-virtual, partial assignment
- If `operator=` virtual, mixed assignment

Recommend: all superclasses abstract

```c++
class AbstractBook
{
    string title, author;
    int numPages;
    
protected:
    AbstractBook &operator=(const AbstractBook &other);
    /* - Prevents assignment through base class ptrs from compiling.
       - Implementations till available to subclasses */
    
public:
    AbstractBook(_);
    virtual ~AbstractBook() = 0;
    // Need at least 1 pure virtual method. 
    // If none exist, use the dtor.
};

class NormalBook: public AbstractBook
{
    ...
public:
    NormalBook(_);
    ~NormalBook();
    NormalBook &operator=(const NormalBook &other)
    {
        AbstractBook::operator=(other);
        return *this;
    }
}; // other classes are similar

AbstractBook::~AbstractBook(){}
```
Note:
- A virtual destructor must be implemented even if it is pure virtual.
- Step #3: subclass dtor WILL call it. So it must exist.

## Templates
HUGE Topic: just the highlights here

```c++
class List
{
    struct Node;
    Node *theList;
    ...
};

struct List:Node
{
    int data;
    Node *next;
};
```
- What if you want to store something else?
    - Whole new class?

OR, a Template: A class parameterized by a type

```c++
template <typename T> class List
{
    struct Node;
    Node *theList;
    
public:
    class Iterator
    {
        Node *p;
        
    public:
        ...
        T &operator*();
        ...
    };
    ...
    T ith(int i);
    void addToFront(T n);
    ...
};

template <typename T> struct List<T>::Node
{
    T data;
    Node *next;
};

template <typename T> class Stack
{
    int size, cap;
    T *theStack;
    
public:
    Stack(_);
    void push(T x);
    T top();
    void pop();
};

// Client:
List <int> l1;
List <List<int>> l2;
l1.addToFront(3);
l2.addToFront(l1);

// Looping:
for(List<int>::Iterator it = l1.begin(); it != l1.end(); ++it)
{
    cout << *it << endl;
}
```
- Compiler specializes templates at the source code level, before compilation (compiler writes the new classes for you)

## The Standard Template Library (STL)
Large # of useful templates

E.g.: dynamic length arrays: vectors

```c++
#include <vector>
using namespace std;

Vector<int> v{4,5}; // 4,5
v.emplace_back(6);
v.emplace_back(7); // 4,5,6,7

Vector<int> w(4,5); // 5,5,5,5 (4 fives)

// Looping:
for(int i = 0; i < v.size(); ++i) cout << v[i] << endl;

// lower-case i
for(vector<int>::iterator it = v.begin(); it != v.end(); ++it) 
{
    cout << *it << endl;
}

// OR
for(auto n:v)
{
    cout << n << endl;
}

// To iterate in reverse:
for(vector<int>::reverse_iterator it = v.rbegin(); 
    it != v.rend(); ++it) ...;

v.pop.back(); // remove last element
```

Use iterators to remove items from inside a vector

E.g.:

```c++
auto it = v.erase(v.begin()); // erase item 0
it = v.erase(v.begin() + 3); // erase item 3
it = v.erase(v.end() - 1); // last item
v[i] // ith element, unchecked, segmentation fault if u go out of bound
v.at(i) // checked version of v[i]
```

Problem:
- vector's code can detect the error, but doesn't know what to do about it
- client can respond, but can't detect the error

C: solution: functions return a status code or set the global variable `errno`
- leads to awkward programming
- encourages programmers to ignore error checks

C++: in error cases, the function raises an exception
- by default, execution stops (intentional crash)
- we can write handlers to catch exeptions and deal with them

`Vector<T>::at` throws `std::out_of_range` when it fails

```c++
#include <stdexcept>
...
try
{
    cout << v.at(1000) << endl;
}
catch (std::out_of_range theExn)
// type: std::out of range
// theExn object that was thrown
{
    cerr << "Range error" << endl;
}
cout << "Carry on" << endl;
```