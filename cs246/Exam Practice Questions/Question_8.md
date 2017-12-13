## Conceptual Questions
#### Question 1
What is stack unwinding? When does stack unwinding happen? Is it true that all statements in the program are not executed if stack unwinding is happening? Explain.

#### Answer
- Stack unwinding is when control goes backwards in the call stack until an appropiate handler is called
- Occurs when you throw an exception in a function. The stack frame "pops" and throws the exception to the caller
- When the stack pops, it goes through and destructs each local variable

***

#### Question 2
Explain the benefits of reducing compilation dependencies.

#### Answer
Reducing compilation dependencies is very beneficial for the following reasons:
1. It reduces the size of .h files
2. Prevents circular dependencies from preventing your program to compile
3. Prevents unecessary recompilation of files (use forward declaration whenever possible)

***

#### Question 3
What is RTTI? Briefly explain how RTTI is implemented in C++

#### Answer
- RTTI is Run-time type information, a mechanism that exposes information about an object's data type at run-time (Straight from wikipedia)
- RTTI can be used to do safe typecasts, using `dynamic_cast`, and to manipulate type info at run-time.
- RTTI is available only to polymorphic classes (at least one virtual method).
- The programmer can choose at compile time whether to include the functionality.

***

#### Question 4
Does the move assignment operator suffer from the partial assignment problem? What about the mixed assignment problem? Explain

***

#### Question 5
What are the 4 steps of object creation and the 4 steps of object destruction?

#### Answer
- Object creation:
    1. Space is allocated
    2. Superclass part is constructed
    3. Fields are constructed in declaration order
    4. Constructor body runs
- Object destruction
    1. Fields are destructed in declaration order
    2. Superclass destructor runs
    3. Fields are destructed in reverse declaration order
    4. Space is deallocated

***

#### Question 6
What is the auto keyword? When is it useful? Do you recommend using auto for all declarations? Why?

#### Answer
- It used to mean that this variable is stack allocated. Now, it's used for type inference (type deduction).
- I do not recommend using auto for all declarations as it does not always deduce the type you want.
    - For example, say you had `auto x = 1;`. x's type would be deduced to int; however, you wanted x to be size_t
****

#### Question 7
Some people claim that accessor and mutator methods are bad since there is no difference between using these methods and making the variable public. Do you think they are right? Why?

#### Answer
- No, accessor and mutator methods are not as bad as making the variables public.
- Accessor methods can be used to return copies of the object's fields. This way the original object cannot be altered.
- Mutator methods are also not as bad as making the variables public as you still have control over what the variables can be changed to.

***