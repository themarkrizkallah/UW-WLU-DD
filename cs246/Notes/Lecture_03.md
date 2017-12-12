# CS 246 Lecture 3

## C++

#### Hello World in C
```c
#include <stdio.h>

int main()
{
    printf("Hello world!\n");
    return 0;
}
```

#### Hello world in C++
```c++
#include <iostream>
using namespace std;

int main()
{
    cout<<"Hello world!"<<endl;   
    return 0;
}
```

- main must return an int in C++
- stdio.h and printf are still available (banned in CS246)
- prefered C++ io is `<iostream>`
- Output

```c++
std::cout <<____<<___ << std::endl;
```

- `using namespace std` can omit `std::`

##### Compiling C++ programs

```shell
g++ -std=c++14 -Wall hello.cc -o hello
```

- hello is the executable or the output

or

```shell
g++14 hello.cc -o hello

./hello
```

- Most C programs work in C++

#### Input / Output
##### 3 I/O streams
- `cout` : print to stdout
- `cin` : for reading from stdin
- `cerr` : print to stderr

##### I/O operators
- `<<`"put to" (output)
- `>>`"get from" (input)
- points in the direction of information flow

E.g. add 2 numbers 

```c++
#include <iostream>
using namespace std;

int main()
{
   int x,y;
   cin >> x >> y;
   cout << x + y << endl;
   return 0;
}
```

- if you put in invalid input then read fails
- if the read fails, the expression `cin.fail()` will evaluate to true
- if EOF, then `cin.eof()` and `cin.fail()` will both be true (but not until the attempted read fails)

E.g. Read all ints from stdin and echo them 1 per line to stdout and stop on bad input or on EOF

```c++
int main()
{
    int i;
    while (true)
    {
        cin >> i;
        if(cin.fail()) break;
        cout << i << endl;
    }
}
```

Note: there is an implicit conversion from `cin` to `bool`
- lets `cin` be used as a condition
    - `true` = success 
    - `false` = failure

Example:

```c++
int main()
{
    int i;
    while(true)
    {
        cin >> i;
        if (!cin) break;
        cout << i << endl;
    }
    return 0;
}
```

Note: `>>`
- C's right bitshift operator
- `a >> b` shifts a's bits to the right by b spots

eg: `21 >> 3`

21 = 10101 >> 3 -> 10 = 2

`operator>>`

- inputs:
    - `cin` (type istream)
    - `data` (various types)
- output:
    - Returns `cin` back (istream)
- This is why we can write:

```c++
cin >> x >> y >> z;
cin >> y >> z;
cin z;
```

Ex. V.3:

```c++
int main()
{
    int i;
    while (true)
    {
        if (!(cin >> i)) break;
        cout << i << endl;
    }
}
```

Ex. V. 4

```c++
int main()
{
    int i;
    while(cin >> i)
    {
        cout << i << endl;
    }
}
```

E.g. read all ints and echo to stdout until EOF and skip invalid input

```c++
int main()
{
    while(true)
    {
        if(!(cin >> i))
        {
            if(cin.eof()) break; //EOF
            cin.clear();
            cin.ignore(); // Ignore current character
        }
        // Read was ok
        else cout << i << endl;
    }
    return 0;
}
```