#### Exceptions and Exception Safety
For each of the following functions or methods, state the level of exception safety. You should assume nothing if any extra information is not given. Assume that all optimiza- tions have been turned off.

#### Function 1
```c++
struct Node{ 
    int data;
    Node *next;
// Assume the following methods are properly defined.
    Node();
    Node(int x); // No-throw guarantee
    Node(const Node &); // basic guarantee
    Node &operator=(const Node &); // basic guarantee
};

Node giveMeANode(int a){
    Node n;
    try{
        n = Node(a);
    }catch (...){
    }
    return n;
}
```

#### Answer
- Basic guarantee 
- After `n = Node(a)` throws, it makes `n` "invalid" in the sense of basic guarantee
- `n` is regarded as the data outside in this case since it could be successfully returned to outside the function.
***

#### Function 2
```c++
class C{
    struct CImpl{
        A a;
        B b;
    };
    unique_ptr<CImpl> pImpl;
    void f(); 
};

void C::f(){
    auto temp = make_unique<CImpl>(*pImpl);
    temp->a.method1(); // No-throw
    temp->b.method2(); // No-throw
    std::swap(pImpl, temp); // No-throw
}
```

#### Answer
- No guarantee 
- `A`, `B` do not have any assumptions, and thus, we cannot guarantee anything
- If we ignore this fact (e.g. make CImpl just contain a bunch of ints) this will make it strong guarantee
***

#### Function 3
```c++
void g(){
    int *x = nullptr;
    *x = 3;
}
```

#### Answer
- No guarantee
- The function always fails.
***

#### Function 4
```c++
class C{
    int a,b,c;
    D d;
public:
    // helper function
    void swap(C &other){
        std::swap(a, other.a);
        std::swap(b, other.b);
        std::swap(c, other.c);
        std::swap(d, other.d);
}

    C &operator=(const C &other){
        unique_ptr<C> temp;
        try{
            temp = make_unique<C>(other);
        } catch (...) {return *this;}
        this->swap(*temp);
        return *this;
    } 
};
```

#### Answer
- No guarantee
- No assumptions about D either. 
- If D has no throw/move semantics, then that function has a strong guarantee
- If D has a strong guarantee for its copy semantics, then that function has a basic guarantee.