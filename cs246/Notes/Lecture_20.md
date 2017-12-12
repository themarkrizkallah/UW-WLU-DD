# CS 246 Lecture 20
Recall:

```c++
class ClassBoard
{
    ...
    cout << "Your move" << endl
    ...
};
```
- Bad design, inhibits code reuse
- What if tomorrow you want to reuse ChessBoard, but not have it print to stdout?

One solution: give the class stream objects where it can send input/output:

```c++
class ChessBoard
{
    std::istream &in;
    std::ostream &out;
public:
    ChessBoard(std::istream &in, std::ostream &out): in{in}, out{out}{}
    .
     .
      .
        out << "your move" << endl;
      .
     .
    .
};
```

Better: but what if we don't want to use streams at all?
- Your ChessBoard class should not be printing at all

Single Responsibility Principle
- "A class should have only one reason to change"
- Game state, rules, communication, strategy - All reasons to change

Better: communicate with the ChessBoard via params/results/exns
- Confine user communication to outside the game class

Q: So should main do all the communication + then call ChessBoard methods?

A: NO! Hard to reuse code if it's in main
- Should have a class to manage interaction, that is separate from the game state class

Architecture: Model-View-Controller (MVC)
- Separate the distinct notions of the data (or stack), the presentation of the data, and the control of the data

- Model: The data you are manipulating
- View: How the model is displayed to the user
- Controller: How the model is manipulated

Model:
- Can have multiple views (e.g. text + graphics)
- Doesn't need to know their details
- Classic Observer Pattern

Controller:
- Mediates control flow between model + view
- Many encapsulate turn-taking, or full game rules
- Many communicate with the user for input (or this could be the view)

By decoupling presentation + control, MVC supports reuse

## Exception Safety
Consider:

```c++
void f()
{
    MyClass *p = new MyClass;
    MyClass mc;
    g();
    delete p;
}
```
- No leaks, but what if g throws?
- What is guaranteed?
- During stack unwinding, all stack allocated data is cleaned up
- Heap-allocaed memory is not freed
- If g throws, *p is leaked but mc is not

Fix:

```c++
void f()
{
    MyClass *p = new MyClass;
    MyClass mc;
    try
    {
        g();
    }
    catch(...)
    {
        delete p;
        throw;
    }
    delete p;
}
```
- Tedious + error-prone + duplication of code
- How can we guarantee that somehting will happen, no matter how we exit f? (normally or exn?)

In some languages, "finally" clauses guarantee certain final actions - not c++

Only thing you can count on in C++ is that destructors for stack allocated data will run
- Use stack-allocated data with dtor as much as possible
- Use the guarantee to your advantage

C++ Idiom: RAII = Resource Acquisition Is Initialization
- Every resource should be wrapped in a stack-allocated object, whose job is to free it

Eg.: files

```c++
istreamf("file"); // Acquiring the resource ("file") = initializing the object (f)
// the file is guaranteed to be released when f is popped from the stack (f's dtor runs)
```
This can be done with dynamic memory:
- `class std::unique_ptr<T>`
    - Takes a pointer to a T in the ctor
    - The dtor will delete the ptr
- In between, can dereference just like a ptr

```c++
f() // OR unique_ptr<MyClass> p = make_unique<MyClass>(...) <- ctor args
{
    unique_ptr<MyClass> p{new MyClass};
    MyClass mc;
    g(); // No leaks, Guaranteed
}
```

Difficulty:

```c++
class C{...};
...
unique_ptr<C> p {new C{...}};
unique_ptr<C> q = p; // WRONG!
```
- What happens when a unique_ptr is copied: don't want to delete the same ptr twice
- Instead: copying is disabled for unique_ptrs, they can only be moved.

Sample implemenation:

```c++
template <typename T> class unique_ptr
{
    T *ptr;
    
public:
    // Ctor
    explicit unique_ptr(T *p): ptr{p}{}
    // Dtor
    ~unique_ptr(){delete ptr;}
    // Deletes copy ctor
    unique_ptr(const unique_ptr &) = delete; 
    // Deletes copy assn
    unique_ptr &operator=(const unique_ptr &) = delete;
    // Move ctor
    unique_ptr(unique_ptr &&other): ptr{other.ptr}{other.otr = nullptr;}
    // Move assn
    unique_ptr &operator=(unique_ptr &&other)
    {
        std::swap(ptr, other.ptr);
        return *this;
    }
    // Unary star operator (dereference)
    T &operator*(){return *ptr;}
};
```
- If you want true shared ownership: `std::shared_ptr`

```c++
{
    auto p1 = make_shared<c>(...);
    if(_)
    {
        auto p2 = p1; 
        // p2 popped; ptr NOT deleted
    }
    // p1 popped: ptr IS deleted
}
```
- Shared-ptrs maintain a reference count
    - count if all shared_ptrs pointing at that obj
- Memory is freed when the $ of shared_ptrs pointing to it hits 0.

E.g.:
```
(define l1 (cons 1 (cons 2 (cons 3 empty))))
(define l2 (cons 4 (rest l1)))
```
- Use shared_ptrs + unique_ptrs for ownership
- Use raw ptrs for non-owning pointers
- Dramatically fewer opportunities for leaks