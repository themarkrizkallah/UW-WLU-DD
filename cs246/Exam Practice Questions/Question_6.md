## Template Patterns
Implement `foldl` and `foldr` in C++.

Here is the description of those functions in How To Design Programs, serving as a re-fresher:

```
;;foldr:(X Y -> Y) Y (listof X) -> Y
;; (foldr f base (list x-1 ... x-n)) = (f x-1 ... (f x-n base)) 
(define (foldr f base alox) ...)
;;foldl:(X Y -> Y) Y (listof X) -> Y
;; (foldl f base (list x-1 ... x-n)) = (f x-n ... (f x-1 base)) 
(define (foldl f base alox) ...)
```
The parameters of those functions in C++ should be the same, except the last parameter: instead of passing a list to the fucntions, you are to pass two iterators to the function, which are beginning and end of the iteration.

Assume `f`, the function passed to `foldl` and `foldr`, has no side effects except output to an `ostream`.

Here are some usage examples:

```c++
// assume foldl, foldr defined here.
int add(int x, int y){ return x + y; }
int sub(int x, int y){ return x - y; }
int print(int e, int i){ cout << e << endl; return i;}
int main(){
    vector<int> v{1,2,3,4,5};
    cout << foldl(add,0,v.begin(),v.end()) << endl << endl;
    cout << foldr(sub,0,v.begin(),v.end()) << endl << endl;
    foldl(print,0,v.begin(),v.end());
    cout << endl;
    foldr(print,0,v.begin(),v.end());
}
```

The code outputs:

```
15

3

1
2
3
4
5

5
4
3
2
1
```

#### Answer
```c++
#include <iostream>
#include <vector>
using namespace std;

template<typename Y, typename Fn, typename XIter>
Y foldl(Fn f, Y init, XIter begin, XIter end)
{
    for (; begin != end; ++begin) init = f(*begin, init);
    return init;
}

template<typename Y, typename Fn, typename XIter>
Y foldr(Fn f, Y init, XIter begin, XIter end)
{
    XIter next = begin;
    ++next;
    if (next == end) return f(*begin, init);
    return f(*begin, foldr(f, init, next, end));
}

int add(int x, int y){ return x + y;}
int sub(int x, int y){ return x - y;}

int print(int e, int i)
{
    cout << e << endl; return i;
}

int main(){
    vector<int> v{1,2,3,4,5};

    cout << foldl(add,0,v.begin(),v.end()) << endl << endl;
    cout << foldr(sub,0,v.begin(),v.end()) << endl << endl;

    foldl(print,0,v.begin(),v.end());
    cout << endl;
    foldr(print,0,v.begin(),v.end());
    
    return 0;
}
```
***