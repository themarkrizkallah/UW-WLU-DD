# CS 246 Lecture 14
## Destructor Revisited
```c++
class X
{
    int *x;
    
public:
    X(int n): x{new int[n]}{}
    ~X(){ delete {} x; }
};

class Y: public X
{
    int *y;
    
public:
    Y(int m, int n): X{n}, y{new int[m]}{}
    ~Y(){ delete [] y; } 
    // Don't need to delete x, Y's dtor will call X's dtor (Step 3)
};

X *myX = new Y{5, 10};
delete myX; // leaks, why?
```
- Calls `~X`, but not `~Y`
- Only x is freed as a result, y leaks

- How can we ensure that deletion through a superclass ptr will call the subclass dtor?
    - Make the dtor virtual!

```c++
class X
{
    ...
public:
    ...
    virtual ~X(){ delete [] x; }
};
```

*IMPORTANT*
- ALWAYS make the dtor `virtual` in classes that are meant to have subclasses
    - Even if the dtor doesn't do anything
- If a class is not meant to have subclasses, declare it `final`

```c++
class Y final: public X{...}
```

## Pure Virtual Methods + Abstract Classes
```c++
class Student
{
public:
    virtual float fees();
};
```
- 2 kinds of student: Regular & Co-op

```c++
class Regular: public Student
{
public:
    float fees() override;
};

class Coop: public Student
{
public:
    float fees() override;
};
```

- What should we put for `Student::fees`?
    - Not sure, every student should be `Regular` or `Coop`
    - Can explicitly give `Student::fees` no implementation
    
```c++
class Student
{
    ...
public:
    virtual float fees() = 0; // Pure virtual method
    /* - Method has no implementation 
        - Pure Virtual Destructors HAVE to be implemented
       - Pure virtual methods cannot be instantiated */
};

Student s; // WRONG! Called an abtract class
Student *p = new Regular;
```

- Subclasses of abstract-classes are also abstract, unless they implement all pure virtual methods.
- Non-abstract classes are called concrete.

UML:
- virtual & pure virtual methods: italics
- abstract classes: class name in italics
- static (underlined)

## Inheritance and Copy/Move
```c++
class Book
{
    ...
public:
    ... // Defines copy/move ctor/assign
};

class Text: public Book
{
    string topic;
    
public:
    ... // Does not define copy/move operations
};

Text t {"Algorithms", "C L R S", 50000, "CS"};
text t2 = t; // No copy ctor in Text: what happens?
```

- Calls Book's copy ctor
- Then goes field-by-field (default behaviour) for the Text part
- Some for other operations

To write your own operations:

```c++
// Copy ctor
Text::Text(const Text &other): Book{other}, topic{other.topic}{}

// Copy assn
Text &Text::operator=(const Text &other)
{
    Book::operator=(other);
    topic = other.topic;
    return *this;
}

// Move ctor
Text::Text(Text &&other): Book{std::move(other)}, topic{std::move(other.topic)}{}

// Move assn
Text &Text::operator=(Text &&other)
{
    Book::operator=(std::move(other));
    topic = std::move(other.topic);
    return *this;
}
```

Note:
- Even though through other/other.topic refer to rvalues, they themselves are lvalues
- `std::move(x)` forces an lvalue `x` to be treated as an rvalue, so that the "move" versions of the operations run

Now, consider:

```c++
Text t1{____}, t2{____};
Book *pb1 = &t1, *pb2 = &t2;
```

- What if we do `*pb1 = *pb2;` ?
    - It is `Book::operator=` that runs

Partial assignment: copies only the Book part
- How can we fix this?
    - try making `operator=` virtual

```c++
class Book
{
    ...
public:
    virtual Book &operator=(const Book &other){...}
};

class Text: public Book
{
    ...
public:
    Text &operator=(const Text &other) override {...} // Won't compile
    Text &operator=(const Book &other) override {...} // Will compile
    // Assignment from a Book to a Text is allowed
};
```

Note: different return type, but param types must be the same, or it's not an override (and won't compile)
- Violates "is a"

If `operator=`is non-virtual:
- Partial assignment through base class ptrs, BAD!

If virtual:
- Compiler allows mixed assignment

Recommendation: *IMPORTANT*
- All superclasses should be abstract