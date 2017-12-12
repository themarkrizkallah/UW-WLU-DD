# CS 246 lecture 7

Recall:

```c++
Node *np = new Node;
...
delete np;
```

Arrays:

```c++
int *p = new int [40];
...
delete [] p;
```

The form of delete must match the form of new.

## Classes
- Can put functions inside of structs.

Eg.

```c++
struct Student
{
    int assns, mt, final;
    
    float grade()
    {
        return assns*0.4 + mt*0.2 + final*0.4
    }
};

Student s {60,70,80}; // Class = Student, Object = s
cout << s.grade() << endl;
```

- Class
    - A structure type that can contain functions.
    - C++ has a class keyword, we will use it later.
- Object
    - An instance of a class.
- Function grade
    - Called a member function (or method)

What do `assns, mt, final` mean inside of `grade(){ ... }`?
- They are fields of the current obj.
    - The object upon which the method was called.
    
Eg.

```c++
Student billy {...};
billy.grade(); // uses billy's assns, mt, final
```

Formally:
- Methods take a hidden extra parameter called `this`
    - A ptr to the object upon which the method was called.
    
```c++
billy.grade(); // this == &billy
```

Can write (but not necesary):

```c++
struct Student
{
    ...
    float grade()
    {
        return this->assns * 0.4 + this->mt * 0.2 + this->final * 0.4;
    }
}
```

## Initializing Objects
```c++
Student billy {60,70,80}; // Ok, but quite limited
```

Better:
- A method that does initialization: a constructor

```c++
Struct Student
{
    int assns, mt, final;
    ...
    Student(int assns, int mt, int final)
    {
        this->assns = assns;
        this->mt = mt;
        this->final = final;
    }
};

Student billy(60, 70, 80); // better
```

- If a ctor has been defined, these are passed as args to the ctor.
- If not ctor has been defined, these initialize the individual fields of Student.

```c++
int x = 5; // Allowed
int x(5); // Allowed
string s = "hello"; // Allowed
string s("hello"); // Allowed
ifstream f = "hello.txt"; // Not allowed
ifstrem f("hello.txt"); // Allowed
ifstream f{"hello.txt"}; // Allowed
```

In 2010,

```c++
Student billy(60,70,80); // ctor call
Student billy{60,70,80}; // c-style only
```

Can write:

```c++
struct Student
{
    ...
    float grade()
    {
        return this->assns * 0.4 + this>mt * 0.2 + this->final * 0.4;
    }
};
```

Advantages of ctors:
- Default params, overloading, sanity checks.

Eg.

```c++
struct Student
{
    ...
    Student(int assns=0, int mt=0, int final=0)
    {
        this->assns=assns;
        ...
    }
};

Student jane{70, 80}; // 70, 80, 0
Student newKid; // 0, 0, 0
```

Note:
- Every class comes with a built-in default ctor.
    - Takes no args (Which just default constructs all fields that are objects)
    - Does nothing.

Eg.

```c++
Vec v; // Default ctor (does nothing in this case)
```
- The built-in default ctor goes away if you provide any ctor.

Eg.

```c++
struct Vec
{
    int x, y;
    Vec(int x, int y)
    {
        this->x = x;
        this->y = y;
    }
};
...
Vec v; // Will not compile! No default ctor.
Vec v{1,2}; // Ok
```

What if a struct contains constants or refs?

```c++
struct MyStruct
{
    const int myConst; // Must be initialized.
    int &myRef;
};
```

We initialize them:

```c++
int z;

struct MyStruct
{
    const int myConst = 5;
    int &myRef = z;
};
```

But does every instance of myStruct need to have the same value of myConst, etc.?

```c++
struct Student
{
    const int id; // constant, but not the same for all students.
    ...
};
```

Where do we initialize? ctor body?
- Too late, fields must be fully constructed by then.

When an object is created:
- 1) Space is allocated
- 2) Fields are constructed in declaration order
- 3) Ctor body runs

How? 

### Member initialization list (MIL)

```c++
struct Student
{
    const int id;
    int assns, mt, final;
    ...
    Student(int id, int assns, int mt, int final): 
        id{id}, assns{assns}, mt{mt}, final{final}{}
};
```

```c++
struct Student
{
    string name;
    Student(string name)
    {
        this->name = name;
    }
};
```