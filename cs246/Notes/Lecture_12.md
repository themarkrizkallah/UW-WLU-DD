# CS 246 Lecture 12
Recall: Accessors + Mutators

```c++
class Vec
{
    int x, y;
public:
    int getX() const {return x;} // accessor
    void setY(int z){y = z;} // Mutator
};
```
- What about operator<<?
    - Needs x + y, but can't be a member - if getX, getY defined, ok
    - If not, make operator << a friend function.
    
```c++
// .h file
class Vec
{
    ...
    friend ostream &oeprator<<(ostream &out, const vec &v);
};

// .cc file
ostream &operator<<(ostream &out, const Vec &v)
{
    return out << v.x << ' ' << v.y;
}
```
### Static Fields + Methods
What if we want to track the # of times a method is ever called, or # of Students created?
- Static members: associated with the class itself, not with any specific instance (object).

```c++
// .h file
struct Student
{
    ...
    static int numInstances;
    Student(_): _____ 
    {
        ++numInstances;
    }
};
// .cc file
int Student::numInstances = 0;
```
- Static methods: 
    - Don't depend on a specific instance of the class (no `this` parameter)
    - Can only access static fields and all other static methods
    
```c++
struct Student
{
    ...
    static int numIstances;
    ...
    static void printNumInstances()
    {
        cout << numInstances << endl;
    }
};

int Student::numInstances = 0;

Student billy{60, 70, 80};
Student jane{70, 80, 90};

Student::printNumInstances(); // prints 2;
```

## System Modelling
Building an OO system involves planning:
- Identify abstractions and relationships among them.

Diagramming Standard: UML (Unified Modelling Language)
- Modelling a class
    - Name
    - Fields (optional)
    - Methods (optional)
    - Visibility: '-' = private, '+' = public
    
### Relationship: Composition of Classes
```c++
class Vec
{
    int x, y, z;
    
public:
    Vec(int x, int y, int z): x{x}, y{y}, z{z} {}
};

// Two vecs define a plane
class Plane
{
    Vec v1, v2;
    ...
}; // Can't initialize v1, v2. No default ctor for Vec
// This is a better set up
class Plane
{
    Vec v1, v2;
public:
    Plane(): v1{1, 0, 0}, v2{0,1,0}{} // initialize v1 to Vec{1,0,0}
}
```
- Embedding one obj (Vec) inside another (Plane) is called composition.
- The relationship between Plane + Vec is called "owns a"
    - Plane owns two Vec objects
    - If A owns a B, then typically B has no ientity outside A (no interdependent existence)
    - If A is destroyed, then B is destroyed
    - If A is copied, then B is copied (deep copy)

E.g: A car owns 4 wheels. A wheel is part of a car.
- Destroy a car => Destroy the wheels
- Copy the car => Copy the wheels

Implementation: usually as composition of classes.

### Aggregation
- Compare car parts in a car ("owns a") vs. car parts in a catalogue.
- The catalogue contains the parts, but the parts have an independent existence.
- This is a "has a" relationship (aggregation)
- If A "has a" B, then typically:
    - B has an existence apart from its association with A
    - If A is destroyed, B lives on.
    - If A is copied, B is not (shallow copy), copies of A share the same B

E.g. Ducks in a Pond

```c++
class Pond
{
    Duck *ducks[maxDucks];
    ...
};
```

## Specialization (Inheritance)
- Suppose you want to track your collection of Books:

```c++
class Books
{
    string title, author;
    int numPages;
    
public:
    Book(_);
};

class Text
{
    string title author;
    int numPages;
    string topic;
    
public:
    Text(_);
};

class Comic
{
    string title, author;
    int numPages;
    string hero;
    
public:
    Comic(_);
};
```
- This is ok, does not capture the relationship among those classes.
- How do we create ab array (or list) that contains a mix of these?
- Could:
    - Union:
    
```c++
union BookTypes
{
    Book *b;
    Text *c;
    Comic *c;
}

BookTypes myBooks[20];
```
    - Array of void pointers (`void *p`), could be any type
    - Neither one is type safe
- BUT Texts + Comics are kinds of Books, Books with special features.

### C++ model by inheritance:
```c++
class Books
/* Base class (or Superclass) */
{
    string title, author;
    int numPages;
    
public:
    Book(_);
};

/* Derived classes (or subclasses) */
class Text: public Book
{
    string topic;
    
public:
    Text(_);
};

class Comic: public Book
{
    string hero;
    
public:
    Comic(_);
};
```
- Derived classes inherit fields + methods from the base class.