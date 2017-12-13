## Designing Object-Oriented Programs
For each of the following scenarios:
- Identify design pattern(s) and/or idiom(s) you have learned in this course that could be suitable for this scenario, if applicable.
- Explain how would the design pattern be used in this case, with an UML diagram. The classes mentioned in the UML need to have meaningful names.

#### Part (a)
You are maintaining a compiler for a programming language. A part of the language consists of statements having format (A, B), where A ∈ {A1, A2, A3, A4, · · ·} and B ∈ {B1, B2, B3, B4, · · ·}. 

For example, (A1, B1), (A2, B1), (A3, B1), ... , (A1, B2), (A2, B2), (A3, B2), ... are statements having this format. 

Different combinations (a, b) where a ∈ A and b ∈ B need to be implemented separately. 

Currently the implementation looks like this:
```c++
// This is pseudo-code
std::pair<A,B> stmt = ...;
if (stmt.first == A_1)
{
    if (stmt.second == B_1){ ... }
    else if (stmt.second == B_2){ ... }
    else if (stmt.second == B_3){ ... }
    // and so on...
}
if (stmt.first == A_2)
{
    if (stmt.second == B_1){ ... }
    else if (stmt.second == B_2){ ... }
    else if (stmt.second == B_3){ ... }
    // and so on...
}
// and so on...
```

You are asked to re-organize the logic so that object-oriented design is followed.

#### Answer

***

#### Part (b)
You are a developer at a game studio. You are asked to implement the graphics of a new game so that user can choose to render the graphics using DirectX or OpenGL without recompilation.

#### Answer

***
#### Part (c)
You are designing a new Graphical User Interface (GUI) framework. Every user interaction is considered as a “event” in the program. When an event is triggered the appropriate “event listeners” is notified and performs some action.

#### Answer

***
#### Part (d)
You are writing a PDF annotation app. How would you implement the annotations (e.g. text, highlighting and drawings) on the PDF?

#### Answer

***
#### Part (e)
Recall the AbstractIterator Example where all iterators inherit an abstrct class AbstractIterator. We want to have some unified interface to produce an iterator without losing the runtime type of iterators for different containers (this means we need an unified interface for all containers as well).

#### Answer

***