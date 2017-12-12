# CS 246 Lectures 22 onwards
## How Virtual Methods Work 
```c++
class Vec
{
    int x, y;
    void f();
};

class Vec2
{
    int x, y;
    virtual void f();
};

// What's the difference?

Vec v{1, 2};
Vec2 w{1, 2}; // Do these look the same in memory?

// Answer:
sizeof(v) == 8;
sizeof(w) == 16;
```

- First note: 8 bytes is space for 2 ints
    - no space for methods
- Compiler stores the methods with all other functions

Recall:
```c++
Book *pb = new (Book|Text|Comic);
pb->isItHeavy();
```
- `isItHeavy()` is virtual 
    - Choose which version of `isItHeavy()` to run based on the type of the actual object at runtime. How?

### Vtables
For each class with at least 1 virtual method, the compiler creates a table of function ptrs 
- the "vtable"

E.g.

```c++
class C
{
    int x, y;
    virtual void f();
    virtual void g();
    void h();
    virtual ~C();
};  // see image
```

C objects have an extra ptr (the vptr) that points to C's vTable

E.g. Book, Text (see image)

*IMPORTANT*

Calling a virtual method: (this all happens at runtime)
1. follow vptr to the vtable
2. fetch ptr to actual method from vtable
3. follow the function ptr & call the function

Therefore, virtual method calls incur a small overhead cost in time

Also, having virtual methods adds a vptr to the object.

Thus, classes with virtual methods produce larger objects than if all methods were non-virtual

Concretely, how is an object laid out? -> Compiler-dependent...
(see image)

### Multiple Inheritance 
A class can inherit from more than one class!

```c++
class A
{
public:
    int a;
};

class B
{
public:
    int b;
};

class C: public A, public B
{
    void f(){ cout << a << ' ' << b; }
};
```

Challenges:
- Suppose we have class A, with a public 'a' field, and class B, with a public 'b' field, which inherits A
- Then we have a class C with a public 'c' field that inherits A.. then we do this:
```c++
class D: public B, public C
{
public:
    int d;
};

D dobj;
dobj.a; //now we have a problem: which a? the one from B, or the one from C?
dobj.B::a OR dobj.C::a; // This works!
```
- BUT if B & C both inherit from A, should there be ONE A part of D, or should there be 2 (which is default)?
    - i.e. should `B::a` and `C::a` be the same or different? (currently they're different)
- What if we want B and C to inherit from the same A? Then `B::a == C::a`
- Such a pattern is known as the "deadly diamond" OR "the deadly diamond of death"

Make A a virtual base class: that is, use "virtual inheritance"
```c++
class B: virtual public A{...};
class C: virtual public A{...};
```
E.g.: IO stream hierarchy (see image)

How will this be laid out?
- B must be laid out so that we can find its A part, but the distance to the A part varies
- Solution: location of superclass obj is stored in vtables.
- Diagram doesn't look like A,B,C,D simultaneously, but slices of it do look like A,B,C,D
    - Therefore, ptr assignment among A,B,C,D changes the address stored in ptr
    ```c++
    D *d = ...;
    A *a = d; // Changes address
    ```
- `static_cast` & `dynamic_cast` under multiple inheritance also changes the value of the ptr
- `reinterpret_cast` will not

### Template Functions
```c++
template<Typename T> T min(T x, T y)
{
    return x < y ? x : y;
}

int f()
{
    int x = 1, y = 2;
    int z = min(x, y); // Don't have to say min<int>
}
```
- C++ infers that `T = int` from the types of x & y
    - APPLIES ONLY to template functions!
- If C++ can't determine T, you can tell it:

```c++
    z = min<int>(x,y);
    char w = min('a', 'b'); // T = char
    auto f = min(1.0, 3.0); // T = double
```
- For what type T can min be used?
    - Any type for which `operator<` is defined

Recall:
```c++
void for_each(AbstractIterator &start, AbstractIterator &finish, int (*f)(int))
{
    while (start != finish)
    {
        f(*start);
        ++start;
    }
}
```

Requirements:
- `AbstractIterator` must support !=, *, ++
- `f` must be callable as a function
- Make these template args
```c++
template<typename Iter, typename Fn> void for_each(Iter start, Iter finish, fn f)
{
    //as before
}
```
- Now `Iter` can be ANY type that supports ++, !=, and *
    - (including raw pointers)
    
```c++
void f(int n) {cout << n << endl;}
int a[] = {1,2,3,4,5};
for_each(a, a+5, f)     //prints the array
```

### C++ STD algorithm Library
Suite of template functions, many of which work on iterators

E.g. for_each (as above)
```
template<typename Iter, typename T>
Iter find(Iter first, Iter last, const T &val)
{
    /* Returns an iterator to the first element in [first, last) matching val
        or last if val not found */
}
```
- `count` - like count, but returns # of occurences of val

```c++
template<typename InIter, typename OutIter> OutIter copy(InIter first, InIter last, OutIter result)
{
    // Copies one container range [first, last) to another, starting at result
}   // Note: does NOT allocate memory
```

E.g.
```c++
vector<int> v = {1, 2, 3, 4, 5, 6, 7};
vector<int> w(4);   //space for 4 ints
copy(v.begin() + 1, v.begin() + 5, w.begin());  // w == {2, 3, 4, 5}

template<typename InIter, typename OutIter, typename Fn> 
OutIter transform(InIter first, InIter last, OutIter result, Fn f)
{
    while(first != last)
    {
        *result = f(*first);
        ++first;
        ++result;
    }
    return result;
}
```

E.g.

```c++
    int add1(int n){ return n+1; }
    vector<int> v{2, 3, 5, 7, 11};
    vector<int> w (v.size());
    transform(v.begin(), v.end(), w.begin(), add1); // w == {3, 4, 6, 8, 12}
```

How general is this code?
- What can we use for Fn?
- What can we use for InIer/OutIter?

- How is Fn even used? (f*(first))
- Fn can be anything that can be called as a function
- We could write `operator()` for objects!

```c++
class Plus1
{
public:
    int operator()(int n) {return n + 1;}
};

Plus1 p;
p(4); // Produces 5
transform(v.begin(), v.end(), w.begin(), Plus1{}); 
// Note: Plus1{} is a ctor call to return an object
```

Generalize:

```c++
class Plus
{
    int m;
    
public:
    Plus(int m): m {m} {}
    int operator()(int n) {return n+m;}
};

transform(v.begin(), v.end(), w.begin(), Plus{1}); 
// Plus{1} is called a "function object"
```
The advantage of function objects is that we can maintain state
```c++
class IncreasingPlus
{
    int m = 0;
public:
    int operator()(int n) { return n + (m++); }
    void reset(){m = 0;}
};

vector<int> v(5, 0); // {0, 0, 0, 0, 0}
vector<int> w(v.size());

transform(v.begin(), v.end(), w.begin(), IncreasingPlus{}); 
// w == {0, 1, 2, 3, 4}
```

Consider: how many ints in a vector v are even?
```c++
vector<int> v {...};
bool even(int n){ return n%2 == 0; }
int x = count_if(v.begin(), v.end(), even);
```
- It seems a waste to explicitly create the function even
- Also, we can't keep even close to count_if because functions must be declared at file or namespace scope (i.e. not inside other functions)
- If this were Racket, we would use lambda
- Do the same here:

```c++
// "lambda" function inside the [] brackets
int x = count_if(v.begin(), v.end(), []int(n)) {return n%2 == 0;}
auto even = [](int n) {return n%2 == 0;}
```

### Iterators
Apply the notion of iteration to other data sources

E.g. streams
```c++    
#include <iterator>

vector<int> v {1, 2, 3, 4, 5};
ostream_iterator<int> out{cout, ", "};
copy(v.begin(), v.end(), out);

vector<int> v {1, 2, 3, 4, 5};
vector<int> w; // empty
copy(v.begin(), v.end(), w.begin()); // WRONG: copy doesn't allocate space, and w has size 0, so this will not work!
```

But what if we had an iterator whose assignment operator inserts a new item?

```c++
cop(v.begin(), v.end(), back_inserter(w));
```
- Copies to the end of w, allocating space as it goes
- This is available for any container that supports push_back