## Big 5 Revisited, Smart Pointers
Read the following code and answer the following questions:

```c++
class Node{
    int data;
    unique_ptr<Node> next;
    // All of the nodes in one linked list share the same id.
    shared_ptr<int> id;
    
    Node(int data, int id): data{data}, next{nullptr}, 
        id{make_shared<int>(id)}{}
        
    Node(int data, unique_ptr<Node> &next): ??? {???}
    //Write the Big 5 of this class as they appear here.
};
```
a) Complete the implementation of the constructor above by filling in the sections marked by ???.

b) Write the Big 5 of this class as they appear inside the class definition.

#### Answer
```c++
#include <memory>
#include <utility>
using namespace std;

class Node
{
    int data;
    unique_ptr<Node> next;
    // All of the nodes in one linked list share the same id.
    shared_ptr<int> id;

public:
    Node(int data, int id):data{data},next{nullptr},id{make_shared<int>(id)}{}

    Node(int data, unique_ptr<Node> &next):data{data}, next{std::move(next)},
        id{this->next->id}{}
	// Do you need to write the dtor?
    // ~Node(){}

    // copy ctor
    Node(const Node &other): data{other.data},
        next{(other.next == nullptr) ? nullptr : make_unique<Node>(*other.next)},
        id{(other.next == nullptr) ? 
            make_shared<int>(*(other.id)) : this->next->id}{}

    // copy assignment operator
    Node &operator=(const Node &other)
    {
        Node copy = other;
        std::swap(data, copy.data);
        std::swap(next, copy.next);
        std::swap(id, copy.id);
        return *this;
    }

    // move ctor
    Node(Node &&other): 
        data{other.data}, next{std::move(other.next)}, 
            id{std::move(other.id)}{}

    // move assignment operator
    Node &operator=(Node &&other)
    {
        std::swap(data, other.data);
        std::swap(next, other.next);
        std::swap(id, other.id);
        return *this;
    }
};

int main()
{
    unique_ptr<Node> n = make_unique<Node>(2,1);
    Node n1{1, n};
    Node n2 = n1;
    Node n3(1,2);
    n3 = n1;
    Node n4{std::move(n1)};
    Node n5(0,0);
    n5 = std::move(n2);
}
```
***