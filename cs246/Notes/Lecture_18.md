# CS 246 Lecture 18
### Template Method Pattern
Want subclasses to override superclass behaviour, but some aspects must stay the same.

E.g. There are red turtles and green turtles

```c++
class Turtle
{
public:
    void draw()
    {
        drawHead();
        drawShell();
        DrawFeet();
    }

private:
    void drawHead(){...}
    void drawFeet(){...}
    virtual void drawShell() = 0;
};

class RedTurtle: public Turtle
{
    void drawShell() override {/* draw red shell */}
};

class GreenTurtle: public Turtle
{
    void drawShell() override {/* draw green shell */}
};
```
- Subclasses can't change the way the turtle is drawn (head, then shell, then feet), but CAN change the way the shell is drawn.

Generalization:
- The non-virtual interface (NVI) idiom
    - A public virtual method is really 2 things: 
        - public
            - interface to the client
            - indicated provided behaviour with pre/post conditions
        - virtual
            - interface to the subclasses
            - a hook to insert specialized behaviour
    - Hard to seperate these ideas if they are tied to the same function
    - What if you wanted to "split" a virtual method into two, without affecting the client interface?
    - How can you make overriding f'ns respect the pre/post conditions you guaranteed the client?

NVI idiom:
- All public methods should be non-virtual
- All virtual methods should be non-public (private or protected)
- Except the destructor

E.g.:

```c++
class DigitalMedia
{
public:
    virtual void play() = 0;
};
```
Instead, we would write:

```c++
class DigitalMedia
{
    virtual void doPlay() = 0;
    
public:
    void play() // Easily add before or after code
    {
        ...
        doPlay();
        ...
    }
    // E.g. check copyright, update play counts, etc.
};
```

Generalizes Template Method:
- puts every virtual method inside a template method

## STL: Revisited
### Map: for creating dictionaries
E.g: "arrays" that map strings to int

```c++
#include <map>
using namespace std;

map<string, int> m;
m["abc"] = 1; // New entry
m["def"] = 4; // New entry

cout << m["ghi"] << endl; // What happens?, prints 0
cout << m["abc"] << endl; // 1

m.erase("abc");
if(m.count("def")) // 0 = not found, 1 = found
```
- If key is not present, it is inserted + value is default-constructed (for ints, that value is 0)

Iterating over a map: results are present in sorted key order

```c++
for(auto &p: m) // std::pair<key, value> (<utility>)
{
    cout << p.first << ' ' << p.second;
}
```

## Visitor Pattern
For implementing double dispatch.
- Virtual methods: chosen based on the actual type (at runtime) of the object which they are called

What if you want to choose based on 2 objects?

E.g. Strike an enemy with a weapon. We would want something like:

```c++
virtual void(Enemy, Weapon)::strike(){...}
```
- If strike is virtual in Enemy, choose based on Enemy but not on Weapon.
- If strike is virtual in Weapon, choose based on Weapon but not on Enemy
- Trick to get dispatch based on both: combination of overriding and overloading

```c++
class Enemy
{
public:
    virtual void beStruckBy(Weapon &w) = 0;
};

class Turtle: public Enemy
{
public:
    void beStruckBy(Weapon &w) override{ w.strike(*this); }
};

class Bullet: public Enemy
{
public:
    void beStruckBy(Weapon &w) override{ w.strike(*this); }
};

class Weapon
{
public:
    virtual void strike(Turtle &t) = 0;
    virtual void stirke(Bullet &b) = 0;
};

class Stick: public Weapon
{
public:
    void strike(Turtle &t) override
    {
        // strike turtle with stick
    }
<<<<<<< HEAD
    void strike(Bullet &b) override
    {
        // strike bullet with stick
=======
    //Now this function works for both lists and sets

## Observer Pattern

"Publish-Subscribe model"
One class: publisher/subject - generates data
One or more subscriber/observer classes - receive and react to data

E.g.
    publisher = spreadsheet cells
    observers = graphs
when cells change, graphs update
Can be many different kinds of observer objects
    thus, the subject should not need to know all the details

Now the actual pattern:
(UML)
            _Subject_
            +notifyObservers()
            +attach(Observer)
            +detach(Observer)

This code is common to ALL subjects
(UML)
            _Observer_
            +notify()

Interface is common to all Observers
Subject-Observer relationship is a "has a"
(UML)
            ConcreteSubject
            +getState()
            (+setState(?))

            //Note ConcreteSubject "is a" Subject

(UML)
            ConcreteObserver
            +notify()

            //ConcreteObserver "is a" Observer and "has a" ConcreteSubject

Sequence of method calls
1. Subject's state changes
2. Subject::notifyObservers() called (either by the Subject or by an external controller)
    - will call each Observer's notify()
3. each Observer calls ConcreteSubject::getState to get the new state & react appropriately

Example: Horse Race
    Subject - publishes winners (some source of info/data on who won the race)
    Observer - individual bettors; declare victory when their horse wins

    class Subject {
        vector<Observer *> observers;
    public:
        void attach(Observer *ob) { observers.emplace.back(ob); }
        void detach(Observer *ob) { /* remove from observers */ }
        void notifyObservers() {
            for (auto &ob:observers) ob->notify();
        }
        virtual ~Subject();
>>>>>>> 18043ebceb6a7f953c60e6f3fe9c9d960f7936ce
    }
};

Enemy *e = new Bullet{_};
Weapon *w = new Stick{_};

e->beStruckBy(*w);
```

What happens?
- `Bullet::beStruckBy` runs (virt.method lookup on Enemy)
- Calls `Weapon::strike`, `*this` is `Bullet` 
- `Bullet` version chosen at compile time
- virtual strike method call resolves to `Stick::strike(Bullet &)`


Visitor can be used to add functionality to existing classes without changing or recompiling the classes themselves

E.g. add a visitor to the Book hierarchy:

```c++
class Book
{
public:
    ...
    virtual void accept(BookVisitor &v){v.visit(*this);}
};

<<<<<<< HEAD
class Text:public Book
{
public:
    ...
    void accept(BookVisitor &v) override{v.visit(*this);}
};

// Comic - similar

class BookVisitor
{
public:
    virtual void visit(Book &b) = 0;
    virtual void visit(Text &t) = 0;
    virtual void visit(Comic &c) = 0;
};
```

Application: track how many of each kind of Book I have
- Books: by Author
- Texts: by Topic
- Comics: by Hero

Use a map<string, int>, add `virtual void addMeToMap(_);`, or write a visitor

```c++
class Catalogue: public BookVisitor
{
public:
    map<string, int> theCatalogue;
    void visit(Book &b) override { ++theCatalogue[b.getAuthor()]; }
    ...
};
```
=======
## Decorator Pattern

Want to enhance an object at runtime: for instance, adding functionality/features
E.g. Windowing System - start with a basic window
                      - add scrollbar
                      - add menu
    We want to choose these enhancements at runtime!

* See UML Diagram *

Class Component - defines the interface: operations your objects will provide
Concrete Component - implements the interface
Decorators - all inherit from Decorator, which inherits from Component
Therefore, every Decorator "is a" Component, and every Decorator "has a" Component!

E.g. a Window w/ scrollbar "is a" Window AND "has a" ptr to the underlying plain Window
Window w/ a scrollbar, which has a ptr to a plain Window

All inherit from abstract Window class, so Window methods can be used polymorphically on all of them

E.g. Pizza!

    class Pizza {
    public:
        virtual float price() const = 0;
        virtual string desc() const = 0;
        virtual ~Pizza();
    };

    class CrustAndSauce: public Pizza {
    public:
        float price() { return 5.99; }
        string desc() { return "Pizza"; }
    };

    class Decorator: public Pizza {
    protected:
        Pizza *component;
    public:
        Decorator(Pizza *p): component{p} { }
        ~Decorator() { delete component; }
    };

    class StuffedCrust: public Decorator {
    public:
        StuffedCrust(Pizza *p): Decorator{p} { }
        float price() const override { return component->price() + 2.69; }
        string desc() const override { return component->desc() + " with stuffed crust"; }
    };

    class Topping: public Decorator {
        string topping;
    public:
        Topping(string topping, Pizza *p): Decorator{p}, topping{topping} { }
        float price() const override { return component->price() + 0.75; }
        string desc() const override { return component->desc() + " with " + topping; }
    };

// Usage:

    Pizza *p1 = new CrustAndSauce;
    p1 = new Topping{"Cheese", p1};
    p1 = new Topping{"Mushrooms", p1};
    p1 = new StuffedCrust(p1);

    cout << p1.desc() << ' ' << p1.price(); 
    delete p1;

## Factory Method Pattern (also called Virtual Constructor Pattern)

Write a video game w/ 2 kinds of enemies: turtles, and bullets
system randomly send turtles and bullets... but bullets become more frequent in harder levels

* See UML Diagram *

We never know which enemy comes next, so we can't call turtle/bullet ctors immediately
Instead, we can put a factory method inside Level that creates Enemies

    class Level {
    public:
        virtual Enemy *createEnemy() = 0;
    };

    class Easy: public Level {
    public:
        Enemy *createEnemy() override {
            //create mostly turtles
        }
    };

    class Hard: public Level {
    public:
        Enemy *createEnemy() override {
            //mostly bullets
        }
    };

    Level *l = new Easy;
    Enemy *e = l->createEnemy();
>>>>>>> 18043ebceb6a7f953c60e6f3fe9c9d960f7936ce
