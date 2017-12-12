# CS Lecture 19
Recall:

```c++
class Book
{
public:
    ...
    virtual void accept(Book Visitor &v){ v.visit(*this); }
    ...
};

class Text: public Book
{
public:
    ...
    void accept(BookVisitor &v) override { v.visit(*this); }
};

// Comic - similar

class BookVisitor
{
public:
    virtual void visit(Book &b) = 0;
    virtual void visit(Text &t) = 0;
    virtual void visit(Comic &c) = 0;
};
```

But it won't compile! Why?
- book.h includes BookVisitor.h includes text.h includes book.h (won't be included)
    - circular include dependency
    - Text does not know what Book is
    
- Are these includes really needed?

## Compilation Dependencies
When does a compilation dependency exist?

Consider: 

```c++
class A{...}; // a.h
class B: public A{...}; // #include "a.h"
class C // #incldue "a.h"
{
    A a;
};
class D // class A;
{
    A *ap; // all ptrs are the same size
};
class E // class A;
{
    A f(A a);
};
```
- B and A have a compilation dependency
    - Need to know how big B + C are

If there is no compilation dependency necessiated by the code, don't create one with extra includes.

When class A changes, only A, B, C need to recompile

Now, in the implementations of D, E:

```c++
#include "a.h"

void D::f()
{
    ap->someMethod(); // Need to know about about class A here
}
```

Do the `#include` in the .cc file, instead of the .h file (where possible).

Now consider the XWindow class:

```c++
class XWindow
{
    Display *d;
    Window w;
    int s;
    GC gc;
    unsigned long colours[10];
public:
    ...
};
```
- This is private data, yet, we can look at it.
- Do we know what it all means? Do we care?

What if we add/change a private member? All clients must recompile.
- Would be better to hide these details away.

Sol'nL the pimpl idiom ("pointer to implementation")
- Create a second class XWindowImpl:

```c++
// XWindowImpl.h
#include <x11/Xlib.h>

struct XWindowImpl
{
    Display *d;
    Window w;
    int s;
    GC gc;
    unsigned long colours[10];
};

// window.h
class XWindowImpl;
class XWindow
{
    XWindowImpl *pimpl;
public:
    // No change
    ...
};

// window.cc
#include "window.h"
#include "XWindowImpl.h"

XWindow::XWindow(_): pimpl{new XWindowImpl}...{}
// other methods: replace fields, d,w,s,etc. with pimpl->d, pimpl->w, etc.
```

If you confine all private fields to XWindowImpl, ten only window.cc needs to recompile if you change XWindow's implementation.

Generalization: What if there are several possible window implementations?
- e.g. XWindows + YWindows
- Then make the Impl Structure a Superclass

Pimpl idiom with class hierarchy of implementations: called the Bridge Pattern

## Measures of Design Quality
- coupling + cohesion

Coupling: 
- the degree to which distinct program modules depend on each other
- low:
    - modules communicate via f'n calls with basic params/results
    - modules pass arrays/structs back + forth
    - modules affect each other's contorl flow
    - modules share global data
- high
    - modules have access to each other's implementation (friends).
    - high coupling => changes to one module require greater changes to other modules
    - harder to reuse individual modules
- perfect coupling: everything is in 1 module

Cohesion:
- how closely elements of a module are related to each other
- low:
    - arbitrary grouping of unrelated elements (<utility>)
    - elements share a common theme, otherwise unrelated, perhaps share some basic code (<algorithm>)
    - elements manipulate state over the lifetime of an object (e.g. open/read/close files)
    - elements pass data to each other
- high:
    - elements cooperate to perform exactly one task
- low cohesion => poorly organized code
    - hard to understand, maintain, reuse

Goal: low coupling, high cohesion.

## Decoupling the Interface (MVC)
Your primary program classes should not be printing things!

E.g.:
```c++
class ChessBoard
{
    ...
    cout << "Your move";
};
```

Bad design - inhibits code reuse.

What if you want to reuse ChessBoard, but not have it communicate via stdout