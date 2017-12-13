## Vtables
Draw the object layout for each of the following classes, including their vtables. Use arrow to indicate where does every single pointer points to; if the pointer is nullptr, make the arrow points to the text nullptr.

```c++
struct A{
    virtual void f() = 0;
    virtual void g() {...}
    virtual ~A(){}
};

class B : public A{
    int x, y;
public:
    void f() override { ... }
    void g() override { ... }
    void h() { ... }
};

class C : public A{
    int x;
public:
    void f() override { ... } 
    ~C(){...}
};
   
   
class D int: public B{
    int z;
public:
    virtual void h() { ... }
    ~D() { ... }
};
```

#### Answer is in q7-sol.png