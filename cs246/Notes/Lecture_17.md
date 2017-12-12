# CS 246 Lecture 17
## Observer Pattern
"Publish-Subscribe model"
- One class publisher/subject: generates data
- One or more subscriber/observer classes: receive and react to data

E.g.
-publisher = spreadsheet cells
- observers = graphs   
- When cells change, graphs update

Can be many different kinds of observer objects
- Thus, the subject should not need to know all the details

Now the actual pattern: (UML)

            _Subject_
            
            +notifyObservers()
            
            +attach(Observer)
            
            +detach(Observer)

This code is common to ALL subjects (UML)

            _Observer_
            
            +notify()

Interface is common to all Observers (UML)

Subject-Observer relationship is a "has a"


            ConcreteSubject
            +getState()
            (+setState(?))
            //Note ConcreteSubject "is a" Subject

            ConcreteObserver
            +notify()
            //ConcreteObserver "is a" Observer and "has a" ConcreteSubject

Sequence of method calls
1. Subject's state changes
2. `Subject::notifyObservers()` called (either by the Subject or by an external controller)
    - will call each Observer's `notify()`
3. Each Observer calls `ConcreteSubject::getState` to get the new state and react appropriately

Example: Horse Race
- Subject 
    - publishes winners (some source of info/data on who won the race)
- Observer 
    - individual bettors; declare victory when their horse wins

```c++
class Subject
{
    vector<Observer *> observers;
    
public:
    void attach(Observer *ob) { observers.emplace.back(ob); }
    void detach(Observer *ob) { /* remove from observers */ }
    void notifyObservers(){for(auto &ob:observers) ob->notify();}
    virtual ~Subject();
};

// Note the class is not yet abstract... so, make the dtor pure virtual:
class Subject
{
    ...
public:
   ...
   virtual ~Subject() = 0; 
};
        
// Then somewhere in the .cc:
Subject::~Subject(){ ... }

class Observer
{
public:
    virtual void notify() = 0;
    virtual ~Observer(); // No need to make dtor pure virtual since we already have a pure virtual method
};

class HorseRace: public Subject
{
    ifstream in; //source of winners
    string lastWinner;
    
public:
    HorseRace(string source): in{source}{...}
    bool runRace()
    {
        return in >> lastWinner ? true : false;
        // true means there was a race, false means we're out of races
    }
    string getState(){return lastWinner;}
}

class Bettor: public Observer
{
    HorseRace *subject;
    string name, myHorse;
    
public:
    //ctor
    Bettor(...): ...
    {
        subject->attach(this);
    }
    ~Bettor()
    {
        subject->detach(this);
        //Note the dtor DOES NOT delete subject, since it "has a" HorseRace, does not "own a" HorseRace
    }
    void notify()
    {
        string winner = subject->getState();
        if (winner == myHorse) cout << "Win!" << endl;
        else cout << "Lose!" << endl;
    }
};

//Now in main.cc,
HorseRace hr {"file.txt"};
Bettor Larry{&hr, "Larry", "RunsLikeACow"};
//... other bettors

while(hr.runRace()) hr.notifyObservers();
```

## Decorator Pattern
Want to enhance an object at runtime: for instance, adding functionality/features

E.g. Windowing System:
- start with a basic window
- add scrollbar
- add menu

We want to choose these enhancements at runtime!

* See UML Diagram *

Class Component
- defines the interface: operations your objects will provide

Concrete Component 
- implements the interface

Decorators 
- all inherit from Decorator, which inherits from Component

Therefore, every Decorator "is a" Component, and every Decorator "has a" Component!

E.g. 
- Window w/ scrollbar "is a" Window AND "has a" ptr to the underlying plain Window

All inherit from abstract Window class, so Window methods can be used polymorphically on all of them

E.g. Pizza!

```c++
class Pizza
{
public:
    virtual float price() const = 0;
    virtual string desc() const = 0;
    virtual ~Pizza();
};

class CrustAndSauce: public Pizza
{
public:
    float price(){ return 5.99; }
    string desc(){ return "Pizza"; }
};

class Decorator: public Pizza 
{
protected:
    Pizza *component;
    
public:
    Decorator(Pizza *p): component{p}{}
    ~Decorator(){ delete component; }
};

class StuffedCrust: public Decorator 
{
public:
    StuffedCrust(Pizza *p): Decorator{p}{}
    float price() const override{ return component->price() + 2.69; }
    string desc() const override
    { 
        return component->desc() + " with stuffed crust"; 
    }
};

class Topping: public Decorator 
{
    string topping;
    
public:
    Topping(string topping, Pizza *p): Decorator{p}, topping{topping}{}
    float price() const override { return component->price() + 0.75; }
    string desc() const override 
    {
        return component->desc() + " with " + topping;
    }
};

// Usage:

Pizza *p1 = new CrustAndSauce;
p1 = new Topping{"Cheese", p1};
p1 = new Topping{"Mushrooms", p1};
p1 = new StuffedCrust(p1);

cout << p1.desc() << ' ' << p1.price(); 
delete p1;
```

## Factory Method Pattern (also called Virtual Constructor Pattern)
Write a video game w/ 2 kinds of enemies: turtles, and bullets.

System randomly send turtles and bullets... but bullets become more frequent in harder levels

* See UML Diagram *

We never know which enemy comes next, so we can't call turtle/bullet ctors immediately. 
Instead, we can put a factory method inside Level that creates Enemies

```c++
class Level
{
public:
    virtual Enemy *createEnemy() = 0;
};

class Easy: public Level
{
public:
    Enemy *createEnemy() override
    {
        ... //create mostly turtles
    }
};

class Hard: public Level
{
public:
    Enemy *createEnemy() override
    {
        //mostly bullets
    }
};

Level *l = new Easy;
Enemy *e = l->createEnemy();
```