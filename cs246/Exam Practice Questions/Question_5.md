## Iterators
In this question, you are writing a template binary tree class Bintree<T> where T is name of a type.
a) Write the definition of the class, with data members (the data, left child and right child) and the big 5.
    
b) Change your class definition so that the pImpl idiom is followed.

c) Write an iterator `class Bintree<T>::Iterator` with in-order traversal and the as- sociated method `Bintree<T>::begin()` and `Bintree<T>::end()`. 

How should you change your solution so that pre-order and post-order traversals are also implemented?
    
#### Answer

```c++
#include <memory>
#include <utility>

using namespace std;

template<typename T>
class BinTree
{
    struct BinNode
    {
        T data;
        shared_ptr<BinNode> left;
        shared_ptr<BinNode> right;
        shared_ptr<BinNode> parent;
        
        /* Big Five */

        // Ctor
        BinNode(T data): data{data}, left{nullptr}, right{nullptr}, 
            parent{nullptr}{}

        // Copy ctor
        BinNode(const BinNode &other): data{other.data}, 
            left{other.left ? make_shared<BinNode>(other.left) : nullptr},
            right{other.right ? make_shared<BinNode>(other.right) : nullptr},
            parent{other.parent ? make_shared<BinNode>(other.parent) : nullptr}{}
    };
    unique_ptr<BinNode> node;
    
public:
    /* Big five */

    // Ctor
    BinTree(T data): node{make_unique<BinNode>(BinNode(data))}{}

    // Copy ctor
    BinTree(const BinTree &other): node{make_unique<BinNode>(*other.node)}{}

    // Copy assn
    BinTree &operator=(const BinTree &other)
    {
        BinTree temp{other};
        swap(node, temp.node);
        return *this;
    }

    // Move ctor
    BinTree(BinTree &&other): node{std::move(other.node)}{}

    // Move assn
    BinTree &operator=(BinTree &&other)
    {
        std::swap(node, other.node);
        return *this;
    }

    // Dtor, not necessary
};

```
***