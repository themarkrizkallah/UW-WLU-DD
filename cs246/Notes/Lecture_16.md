# CS 246 Lecture 16
```c++
#include <stdexcept>
...
try
{
    cout << v.int(10000) << endl;
}
catch(std::out_of_range ex)
{
    cerr << "Range eror" << ex.what() << endl;
}
```

Now consider:

```c++
void f()
{
    throw out_of_range{"f"}; // out_of_range, ctor call
}

void g(){ f(); }
void h(){ g(); }

int main()
{
    try
    {
        h();
    }
    catch(out_of_range){...}
}
```

What happens?
- main calls h
- h calls g
- g calls f
- f throws out_of_range
- g has no handler for out_of_range
- Control goes back through the call chain (unwinds the stack) until a handler is found
    - all the way back to main, main handles it
- If no handler found in the entire call chain, program terminates

`out_of_range` is a class

- A handler might do part of the recovery job, execute some corrective code, and throw another exception:

```c++
try{...}
catch(SomeErrorType s)
{
    ...
    throw SomeOtherError{...};
};
```
- or throw the same exn:

```c++
try{...}
catch(SomeErrorType s)
{
    ...
    throw;
}
```
- `throw;` vs `throw s;`
    - `throw;`: throw the exact exception that was thrown at you
        - type information is retained
    - `throw s;`: rethrows a new exception of type SomeErrorType

A handler can act as a catch-all

```c++
try{___}
catch(...){} // catches all exceptions
// ... means suppress all future type checking of params to this f'n
```

You can throw anything you want, don't even have to throw objects

Define your own exn classes for errors.

E.g.: 

```c++
class BadInput{};
...
try
{
    int n;
    if(!(cin >> n)) throw BadInput{};
    ...
}
catch (BadInput &) // catch by reference (always), reduces copying, avoids slicing
{
    cerr << "Input not well-formed";
}
```

Some standard exeptions
- `length_error`: attempting to resize a vector/string too big
- `bad_alloc`: new fails

NEVER let a destructor throw an exception
- If a destructor throws, the program terminates immediately, unless the destructor was tagged noexcept(false)
- If a dtor can + does throw:
    - If that dtor was executed during stack unwinding while dealing with another exception while still looking for a handler, you now have two active, unhandled exeptions + the program WILL abort immediately
    - The standard says, it must crash
    - Defined behaviour
- Much more on exceptions later

## Design Patterns Continued
Guiding principle:
- Program to the interface, not the implementation
- Abstract base classes define the interface
    - work with ptrs to abstract base classes + call their methods
    - concrete subclasses can be swapped in and out
        - abstraction over a variety of behaviours

E.g.: Iterator pattern

```c++
class List
{
    ...
public:
    class Iterator: AbstractIterator
    {
        ...
    };
    ...
};

class Set
{
    ...
public:
    class Iterator: AbstractIterator
    {
        ...
    };
    ...
};

class AbstractIterator
{
public:
    virtual int &operator*() = 0;
    virtual AbstractIterator &operator++() = 0;
    virtual bool operator!=(const AbstractIterator &other) const = 0;
    virtual ~AbstractIterator();
};

// Then you can write code that operates over iterators:

void for_each(AbstractIterator &start, AbstractIterator &end, void (*f)(int))
{
    while(start != end)
    {
        f(*start);
        ++start;
    }
} // works for both Lists and Sets
```

### Observer Pattern
- Publish-Subscribe model
- One class: publisher/subject generates data
- One or more: subscribers/observer recieve data and react to it

E.g.:
- publisher: spreadsehet cells (observers = graphs)
    - when cells change, graphs update
- Can be many diff kinds of observer objects, subjects should not need to know all the details

Sequence of method calls:
- Subjects state changes
- `Subject::notifyObservers()` called (either by the Subject itself, or by on external controller)