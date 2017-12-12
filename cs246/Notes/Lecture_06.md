# CS 246 lecture 6

## Overloading << and >>

```c++
struct Grade
{   
    int theGRade;
};

ostream &operator<<(ostream &out, const Grade &g)
{
    return out << g.theGrade << '%';
}

istream &operator>>(istream &in, Frade &g)
{
    in >> g.theGrade;
    if(g.theGrade < 0) g.theGrade = 0;
    if(g.theGrade > 100) g.theGrade = 100;
    return in;
}
```

## The Preprocessor
- Transforms the program before the compiler sees it.
- `#___` 
    - Preprocessor directive
    - e.g. `#include`
- Including old C headers - new naming convention.
    - e.g. Instead of:
        - `#include <stdio.h>`
    - Use:
        - `#include <cstdio>`
- `#define VAR VALUE`
    - Sets a preprocessor variable
    - THen all occurences of `VAR` in the source file are replaced with `VALUE`.
    - e.g. 
    ```c++ 
    #define MAX 10
    int x[MAX]; // transformed to: int x[10];
    ```
    - Use const definitions instead.
    ```c++
    #define ever ;;
    ...
    for(ever){}
    
    // sets the variable FLAG to the empty string
    #define FLAG
    // makes all instances of "int" dissapear from the program
    #define int 
    ```
- `#define` constants are useful for conditional compilation.

```c++
#if SECURITYLEVEL == 1
    short int
#elif SECURITYLEVEL == 2
    long long int
#endif
    publickey;
```
- Special case:

```c++
#if 0  // Never true, all inner text is removed befoe the compiler sees it.
...
#endif // Acts as a heavy-duty "comment out"
```
- Can also define symbols via compiler args.

```c++
int main()
{
    cout << X << endl;
}
```
```shell
g++14 -DX=15 define.cc -o -define
```

```c++
#ifdef NAME   // True if NAME has been defined 
#ifndef NAME  // True if NAME has not been defined
```

Eg.
```c++
int main()
{
    #ifdef DEBUG
        cout << "Setting x = 1" << endl;
    #endif
        int x=1;
        while(x < 10)
        {
            ++i;
            #ifdef DEBUG
                cout << "x is now " << x << endl;
            #endif
        }
        cout << x << endl;
}
```

## Seperate Compilation
Split programs into composable modules, which each provide:
- interface: type definitions, function headers - .h file
- implementation: full definitions for every provided function - .cc file

Recall:
- declaration: asserts existence
- definition: full details, and allocates space (variables/functions)

E.g.

```c++
// interface (vec.h)
struct Vec
{
    int x, y;
};

Vec operator+(const Vec &v1, const Vec &v2);

// main.cc
#include "vec.h"
int main()
{
    Vec v{1,2};
    v = v + v;
}

// implementation (vec.cc)
#include "vec.h"
Vec operator+(const Vec &v1, const Vec &v2)
{
    return {v1.x + v2.x, v1.y + v2.y};
}
```

- Compiling separately:

```shell
g++14 -c vector.cc
g++14 -c main.cc
g++14 main.0 vector.o -o main
```
- Means compile only, don't link (-c)
- This creates vector.o (object file)

What if we want a module to provide a global variable?
- e.g. abc.h: `int globalNum;`
    - Is a declaration AND a definition
    - Every file that includes "abc.h" defines a separate globalNum
    - Program will not link.
- Sol'n: put the global variable in a .cc
    - abc.h: `extern int globalNum;` <- a declaration but NOT a def'n
    - abc.cc: `int globalNum;` <- def'n

Suppose we write a linear algebra module:

```c++
// linalg.h:
#include "vector.h"
...

// linalg.cc:
#include "linalg.h"
#include "vector.h"
...
```

- Will result in an error since we defined `struct Vec` twice in both vector.h AND linalg.h
    - Not allowed!
- Need to prevent from being included more than once.

Sol'n: include guards in .h files

```c++
// vector.h:
#ifndef VECTOR_H // Any unique name (suggested: based on filename)
// First time vector.h is included, the symbol VECTOR_H is not defined, so the file is included.
#define VECTOR_H
    .
    . // File contents
    .
#endif
// After that, symbol IS defined, so file is suppresed.
```

Always:
- put include guards in .h files
- Never EVER EVER EVER EVER EVER EVER!
    - Compile .h files
    - Include .cc files
    - Never put `using namespace std;` in headers 
        - forces the client to open the namespace
- Always use the `std::` prefix in headers.