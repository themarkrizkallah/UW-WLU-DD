# CS 246 Lecture 13
Recall:

```c++
class Book
{
    string title, author;
    int numPages;
    
public:
    Book(_);
};

class Text:public Book
{
    string topic;
    
public:
    Text(_);
};

class Comic: public Book
{
    string hero;
    
public:
    comic(_);
};
```
- `Text` and `Comic` inherit `title, author, numPages`.
- Any method that can be called on `Book` can be called on `Text` and `Comic`
- Who can see these members?
    - `title`, `author`, `numPages` are private in `Book` (outsiders can't see them)
- Can `Text` and `Comic` see them?
    - NO! 
    - Subclasses can't see their superclass(es)' private fields

E.g: `Text t;` 
- Can't access `t.author` (private)
- Can call `t.getAuthor()` (assuming its public)

How do we initialize Text?

```c++
class Text: public Book
{
    string topic;
    
public:
    Text(string title, string author, int numPages, string topic):
    title{title}, author{author}, numPages{numPages}, topic{topic}{}
    ...
}; // WRONG!
```

Wrong for 2 reasons:
- `title`, etc. are not accessible in `Text`

### Steps in Object Construction/Destruction *IMPORTANT*
- When an object is created:
    1. Space is allocated
    2. Superclass part is constructed `// NEW`
    3. Fields constructed in declaration order
    4. Ctor body runs
- When an object is destroyed:
    1. Destructor body runs
    2. Fields are destructed in reverse declaration order
    3. Superclass part is destructed `// NEW`
    4. Space is deallocated

Since Book has no default constructor, Step 2 can't run

Fix: Invoke `Book`'s ctor in Text's MIL

```c++
class Text: public Book
{
    string topic;
    
public:
    Text(string title, string author, int numPages, string topic):
    Book{title, autor, numPages}, topic{topic}{} // CORRECT!
    ...
};
```
- If the superclass has no default ctor, the subclass MUST invoke a ctor in the MIL

Good reasons to keep superclass fields inaccessible to subclasses
- If you want to give subclasses access to certain members, use `protected` visibility:

```c++
class Book
{
protected: // Accessible to Book and subclasses ONLY!
    string title, author;
    int numPages;
    
public:
    Book(_);
    ...
};

class Text: public Book
{
    ...
public:
    void addAuthor(string auth)
    {
        author += auth; // OK if protected
    }
};
```
- Not a good idea to give subclasses unlimited access to fields.
- Better: make fields private, but provide protected accessors and mutators

```c++
class Book
{
    string title, author;
    int numPages;
    
protected: // Only a subclass can call this
    void setAuthor(string auth);
    
public:
    ...
};
```
- Relationship among `Book`, `Text`, `Comic` called "is a"
    - a `Text` is a `Book`
    - a `Comic` is `Book`
    - `protected` = `#` in UML
- Implement "is a" by public inheritance

```c++
class Book
{
    string title, author;
    int numPages;
    
protected: // Only a subclass can call this
    void setAuthor(string auth);
    
public:
    ...
    bool isHeavy() const;
};
```

Consider:
- `isHeavy`: When is a `Book` heavy?
    - For ordinary Books, > 200 pages
    - For textbooks, > 500 pages
    - For comics,  > 30 pages

```c++
class Book
{
    ...
protected:
    int numPages;
    
public:
    ...
    bool isHeavy() const {return numPages > 200;}
};

class Comic: public Book
{
    ...
public:
    bool isHeavy() const {return numPages > 30;}
};

Book b {"small book", ____, 50};
Comic c {"big comic", ____, 40, ___};

cout << b.isHeavy() << endl; // false
cout << c.isHeavy() << endl; // true
```

Since inheritance means "is a", we can do this:

```c++
Book b = Comic {"big comic", ___, 40, ___}; // legal
```

Q: Is b heavy? `b.isHeavy() // true or false?`
- Does `Book:isHeavy` or `Comic::isHeavy` run?

A: NO! `b` is not heavy, `Book::isHeavy` runs
- Tries to fit a Comic obj, where there is only space for a Book obj
    - What happens? Comic is sliced!
    - hero field is chopped off, Comic becomes a Book!

*IMPORTANT*
- When accessing objects through ptrs, slicing is unnecessary so it doesn't happen

```c++
Comic c {__, __, 40, __};
Book *pb = &c;
Comic *pc = &c;

cout << pc->isHeavy() << endl; // true
cout << pb->isHeavy() << endl; // false
```

Still, `Book::isHeavy` runs when we accessed `pb->isHeavy()`;
- Same object behaves differently depending on what type of pointer points at it
- Compiler uses the type of the pointer (or ref) to decide which `isHeavy` to run.
    - It doesn't consider the actual type of the object.
    
How do we make Comic act like a Comic, even when pointed to by a Book ptr?
- Declare the method `virtual`

```c++
class Book
{
    ...
public:
    ...
    virtual book isHeavy() const {return numPages > 200;}
};

class Comic: public Book
{
    ...
public:
    bool isHeavy() const override {return numPages > 30;}
};

Comic c{__, __, 40, __};
Book *pb = &c;
Book &rb = c;
Comic *pc = &c;

cout << pc->isHeavy() << endl; // true
cout << bc->isHeavy() << endl; // true
cout << rb.isHeavy() << endl; // true
```
- Comic::isHeavy runs in each case

Virtual methods
- Choose which class' method to run, based on the actual type of the object at runtime.

E.g. My book collection

```c++
Book *myBooks[20];
...
for(int i = 0; i < 20; ++i) cout << myBooks[i]->isHeavy() << endl;
/* Uses:
    Book::isHeavy for Books
    Text::isHeavy for Texts
    Comic::isHeavy for Comics */
```
- Accomodates multiple types under 1 abstraction
    - Polymorphism (many forms)