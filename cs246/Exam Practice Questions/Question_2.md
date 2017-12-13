## Forward Declaration
Consider the code below. Which classes do we need to include the definition for and which ones can we forward declare?

```c++
class K: public A{
    B b;
    C* c;
public:
    D& d;
    void foo(E);
    void foo(F*);
    void foo(G&);
    H func();
    I* func2();
    J& func3();
};
std::istream& operator<<(std::istream&, const K&);
```

#### Answer
- `A`: include the definition
- `B`: include the definition
- `C`: forward declare
- `D`: forward declare
- `E`: forward declare
- `F`: forward declare
- `G`: forward declare
- `H`: forward declare
- `I`: forward declare
- `J`: forward declare
- `K`: forward declare
***