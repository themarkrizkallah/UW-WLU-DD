# CS 246 Lecture 4

### Reading strings

```c++
#include <string>
// Type is std::string

int main()
{
    string s;
    cin >> s; // read skipping leading whitespce
    cout << s << endl;
}
```
What if we want whitespace?

```c++
string s;
getline(cin, s); // Input is "95"
cout << s << endl; // prints 95
```

What if we want to print a number in hex?

```c++
cout << hex << 95 << endl; // prints 5f
```

- `hex` is an IO manipulator prints all subsequent ints in hex

```c++
cout << dec; //go back to decimal
```
- Need to `#include <iomanip>`
- Stream obstruction applies to other sources of data

#### Files
- `std::ifstream`
- `std::ofstream`

Eg. Read from a file in C
```c
int main()
{
    char s [256];
    FILE *f = fopen("suite.txt", "r");
    while(true)
    {
        fscanf(f, "%255s", s);
        if(feof(f)) break;
        printf("%s\n", s);
    }
    fclose(f);
}
```
Eg. Read from a file in C++

```c++
int main(){
    ifstream f {"suite.txt"};
    string s;
    while(f >> s) cout << s << endl;
}
```
- Anything you can do with `cin`/`cout`, you can do with an `ifstream`/`ofstream`

Eg. Strings:
- Attach a stream to a string variable & read from/write to the string.
- `std::istringstream` - Read from a string
- `std::ostringstream` - Write to a string

```c++
#include <sstream>

int hi = _, lo = _;
ostringstream oss;
oss << "Enter a # between " << lo << " and " << hi;
string s = oss.str();
cout << s << endl;
```

Eg. Convert strings to #'s
```c++
int n;
while(true)
{
    string s;
    cout << "Enter a #" << endl;
    cin >> s;
    istringstream iss {s};
    if (iss >> n) break; // Stop if the string contains a #
    cout << "I said, ";
}
cout << "You entered: " << n << endl;
```

Example revisited: Echo all #'s, skip non-#'s
```c++
int main()
{
    string s;
    while(cin >> s)
    {
        istringstream iss {s};
        int n;
        if(iss >> n) cout << n << endl;
    }
}
```

### Strings
In C: 
- array of char (char * or char []) terminated by '\0'.
- must explicitly manage memory - manage more memory as strings get larger.
- easy to overwrite the null terminator '\0' and corrupt memory.

In C++:
- strings grow as needed, no need to manage memory,
- safer to manipulate.

Ex: ```string s = "Hello";```
- "Hello" = C-style string (char array)
- s = C++ string created from the C string initialization.

### String Operations
Equality/inequality
- In C++, you can use `s1 == s2 , s1 != s2`

Comparison (lexicographic)
- `s1 <= s2`

Length 
- `s.length()` O(1)

Get individual chars: 
- `s[0], s1[1], s[2], ...`

Concat:
- `s3 = s1 + s2; s3 += s4`

### Default F'n Params
```c++
void printSuiteFile(string name = "suite.txt")
{
    ifstream f{name};
    string s;
    while(f >> s) cout << s << endl;
}
printSuiteFile("suite.txt");
printSuiteFile(); // prints from suite.txt
```

## Overloading
C: 
```c
int negInt(int n){return -n;}
bool negBool(bool b){return !b;}
```

C++: Fn's with diffeent param lists can share the same name
```c++
int neg(int n){return -n;}
bool neg(bool b){return !b;}
```

Overloads must differ in # or types of args 
- may not differ on just the return type.

We've seen this already: `>>`, `<<`, `+`, `-`, etc are all overloaded.

## Structs 
```c++
struct Node
{
    int data;
    Node * next; // No need for Struct here
};
```

## Constants 
```c++
const int maxGrade = 100; // Must be initialized.
```
- Declare as many things const as you can, helps catch errors.

```c++
struct Node n1 = {5,nullptr}; // Syntax for null. Don't say NULL or 0 in this class.
const Node n2 = n1; // Immutable copy of n1. Can't change n2's fields
const Node *pn = &n1; // Can change where the pointer points, can't change n1's fields
```