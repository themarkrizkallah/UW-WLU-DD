# CS 246 Lecture 21
## 3 Levels of Exception Safety for a function
1. Basic guarantee: if an exception occurs, the program will be in a valid state
    - No leaks, class invariants maintained
2. Strong guarantee: if an exception is raised while executing `f()`
    - The state of the program will be as it was before `f()` was called
3. No-throw guarantee: `f()` will never throw an exception, and will ALWAYS accomplish a task (i.e. will succeed)

E.g.
```c++
class A { ... }; 
class B { ... };
class C 
{
    A a;
    B b;
    
public:
    void f() // Strong guarantee
    {
        a.g(); // may throw
        b.h(); // may throw
    }
};
```

Is `C::f()` safe?
- If `a.g()` throws -- nothing has happened yet, so we're OK
- If `b.h()` throws -- effects of `g()` would have to be undone to offer the strong guarantee 
    - May be hard or impossible, especially if `g()` has non-local side-effects
    
Therefore, `C::f()` is probably not exception-safe.

If `A::g()` and `B::h()` do not have non-local side-effects, we can use copy & swap
```c++
class C
{
    ...
    void f()
    {
        A atemp = a;
        B btemp = b;
        atemp.g();
        btemp.h();
        /* If any of the first 4 lines throw, original a and b are still intact
           But what if assigning throws? */
        a = atemp;
        b = btemp;
    }
}
```

Better if the swap was a no-throw.. Copying ptrs cannot throw!

Solution: use the PIMPL idiom:

```c++
struct CImpl {
    A a;
    B b;
};

class C
{
    std::unique_ptr<CImpl> pImpl;
    
public:
    void f()
    {
        auto temp = std::make_unique<CImpl>(*pImpl);
        temp->a.g();
        temp->b.h();
        std::swap(pImpl, temp); // swapping is now no-throw
    }
}; // This is a strong guarantee!
```

If either `A::g()` or `B::h()` offer no exception safety guarantees, then neither can `C::f()`

## Exception safety & the STL - Vectors
Vectors 
    - encapsulate a heap-allocated array
    - RAII: when a stack-allocated vector goes out of the internal heap-allocated array is free'd
    
```c++
void f()
{
    vector<C> v;
    ...
}
```
- v goes out of scope, array is free'd & C's dtor runs on all objects in the vector

BUT:

```c++
void g()
{
    vector<C*> v;
}
```
- Array is free'd, ptrs don't have dtors so any objs pointed at by the ptrs are NOT deleted
- v doesn't know whether the ptrs in the vector own the objects they point at
- To delete the items: `for(auto &x:v) delete x;`

BUT:
```c++
void h()
{
    vector<unique_ptr<C>> v;
}
```
- Array is free'd, unique_ptrs' dtors run and objects are deleted
- NO explicit deallocation!
- `vector<shared_ptr<C>> v;` to do the opposite

`vector<T>::emplace_back()`
- offers the strong guarantee
- if the array is full
    1. allocate a new array
    2. copy objects over (copy ctor) (*)
    3. delete old array

(*) What if this throws?
    - destroy the new array
    - old array still intact
    - strong guarantee!

BUT: 
- Copying is expensive and the old data will be thrown away!
- Wouldn't moving the objects be more efficient?
    - allocate the array
    - move the objects over (move ctor) (**)
    - delete old array

(**) If a move ctor throws, we're screwed because the old data gets thrown!
    - can't offer strong guarantee
    - original data no longer intact

Answer: don't let your move ctors throw...

If the move ctor is no-throw, emplace_back will use it!

Else, it'll use the copy ctor (slow)

Thus, your move operations should provide the no-throw guarantee, and you should indicate that they do:

```c++
class C
{
    ...
public:
    C (C &&other) noexcept{ ... }
    C &operator=(C &&other) noexcept{ ... }
};
```
- If we know a function will never throw/propagate an exception, declare it `noexcept`
    - Facilitates optimization

*IMPORTANT*

At minimum: moves & swaps should be `noexcept`

### Casting
In C: 
```c
Node n;
int *p = (int *)(&n);
```
- Forces compiler to treat `Node *` as if it were an `int *`, so we can point an `int *` at a `Node`

There are 4 ways to cast    
1. `static_cast`: "sensible" casts
    - e.g. converting `double` to `int`
    ```c++
    double d;
    int f(int x);
    f(static_cast<int>(d));
    ```
    - e.g. superclass ptr -> subclass ptr
    ```c++
    Book *b = new Text{...};
    Text *t = static_cast<Text *>(b);
    ```
        - The programmer takes responsibility that b actually points at a Text
        - If it doesn't, behaviour is undefined    
2. `reinterpret_cast`: unsafe, implementation-specific, "weird" conversions
    - Example:
    ```c++
    Student s;
    Turtle *t = reinterpret_cast<Turtle *>(&s);
    ```
3. `const_cast`: for adding/removing const
    - The only C++ cast that can "cast away const"
    - E.g. 1
    ```c++
    void g(int *p){...} // What if we know that g() doesnt modify p?
    void f(const int *p) // Can't g(p);!
    {
        ...
        g(const_cast<int *>(p)); // Legal
        ...
    }
    ```
    - E.g. 2
     (see pic)
    
4. `dynamic_cast`: is it safe to convert a `Book *` to a `Text *`?
    - Example:
    ```c++
    Book *pb = ...;
    Text *t = static_cast<Text *>(pb); // Safe only if we absolutely know that pb is a Text
    ```
    - Tentative cast:
    ```c++
    Text *t = dynamic_cast<Text *>(pb);
    ```
    - 2 outcomes:
        - success - t points at the object, proceed
        - fails - t will be nullptr