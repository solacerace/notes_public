

<!-- TOC -->

- [Code snippet](#code-snippet)
  - [Initialize array](#initialize-array)
  - [Numeric limits](#numeric-limits)
- [Heap](#heap)
  - [C++ Priority Queue](#c-priority-queue)
- [Linked List](#linked-list)
- [Trees](#trees)
  - [Topological Sort](#topological-sort)
  - [BFS](#bfs)
  - [DFS](#dfs)
  - [Traversal](#traversal)
- [STL](#stl)
- [lower\_bound and upper\_bound](#lower_bound-and-upper_bound)
- [Some standard programs](#some-standard-programs)
  - [Topological Sort.](#topological-sort-1)
  - [Find shortest Path in Graph.](#find-shortest-path-in-graph)
  - [Find if path exists between two nodes.](#find-if-path-exists-between-two-nodes)
  - [Stack from Queues](#stack-from-queues)
  - [Shared pointer and Unique Pointer code](#shared-pointer-and-unique-pointer-code)

<!-- /TOC -->




# Code snippet

## Initialize array

* 1D array to zero
```
// Initializes 2D array of size M*N
vector<int> arr(N,0);
```


* 2D array to false
```
// Initializes 2D array of size M*N
vector<vector<bool>> dp(M, vector<bool>(N, false));
```

## Numeric limits

```
#include <limits>

std::cout << std::numeric_limits<int>::min() << "\t│ "
        << std::numeric_limits<int>::max() << '\n';


std::numeric_limits<int>::min()           // -2,147,483,648
std::numeric_limits<int>::max()           //  2,147,483,647
std::numeric_limits<unsigned int>::max()  //  4,294,967,295

```


# Heap

## C++ Priority Queue
By default the c++ priority queue is **MaxPriorityQueue **of **Max Heap**
https://en.cppreference.com/w/cpp/container/priority_queue
```
template<
    class T,
    class Container = std::vector<T>,
    class Compare = std::less<typename Container::value_type>
> class priority_queue;
```

- A priority queue is a container adaptor that provides **constant time lookup of the largest (by default) O(1)** element, at the expense of logarithmic insertion and extraction **O(log(N))**.

- A user-provided Compare can be supplied to change the ordering, e.g. using std::greater<T> would cause the smallest element to appear as the top().

- Note that the compare operator std::less<T> causes the lesser element to appear at the end of the heap. Thus the Max element taking the  first position.

```
int main()
{
    const auto data = {1, 8, 5, 6, 3, 4, 0, 9, 7, 2};
    print("data", data);
 
    // Max priority queue default - Max at top()
    std::priority_queue<int> q1; 
    for (int n : data)
        q1.push(n);
 
    print("q1", q1);
 
    // Min priority queue
    // Min priority queue - Min at top()
    // std::greater<int> makes the max priority queue act as a min priority queue
    std::priority_queue<int, std::vector<int>, std::greater<int>>
        minq1(data.begin(), data.end());
 
    print("minq1", minq1);
}
```
**pop** removes the top element<br>
**top** accesses the top element<br>
**push** inserts element and sorts the underlying container<br>

**Usages for problems statements**<br>
**For K Minimum elements - use Max Heap**
- Use the MaxHeap, which is priority_queue (default).
- for (elem: in 1 to N)
  - priority_queue.push(elem)
  - if (priority_queue.size() > K)
  - {
    - priority_queue.pop() // Removes the top element (max element)
  - }
  - The priority_queue contains K Minimum Elements.

**For K Maximum elements - use Min Heap**<br>
 std::priority_queue<int, std::vector<int>, std::greater<int>>
        minq1;

- The top will contain the minimum element as soon as the count crosses K remove the top element( which is minimum element) and you will always be left with K Maximum element.

[LeetCode Example: K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/)






# Linked List


# Trees

## Topological Sort

## BFS

Template - 1 : Assumption is no cycles in the graph - like an acyclic tree.

```
/**
 * Return the length of the shortest path between root and target node.
 */
int BFS(Node root, Node target) {
    Queue<Node> queue;  // store all nodes which are waiting to be processed
    int step = 0;       // number of steps neeeded from root to current node
    // initialize
    add root to queue;
    // BFS
    while (queue is not empty) {
        // iterate the nodes which are already in the queue
        int size = queue.size();
        for (int i = 0; i < size; ++i) {
            Node cur = the first node in queue;
            return step if cur is target;
            for (Node next : the neighbors of cur) {
                add next to queue;
            }
            remove the first node from queue;
        }
        step = step + 1;
    }
    return -1;          // there is no path from root to target
}
```

As shown in the code, in each round, the nodes in the queue are the nodes which are ```waiting to be processed```.
After each outer while loop, we are ```one step farther from the root node```. The variable ```step``` indicates the distance from the root node and the current node we are visiting.

**Template - 2: Graph with cycles**
We introduce visited hashset here so that we don't visit the same node twice and get into infinite loop.

```
/**
 * Return the length of the shortest path between root and target node.
 */
int BFS(Node root, Node target) {
    Queue<Node> queue;  // store all nodes which are waiting to be processed
    Set<Node> visited;  // store all the nodes that we've visited
    int step = 0;       // number of steps neeeded from root to current node
    // initialize
    add root to queue;
    add root to visited;
    // BFS
    while (queue is not empty) {
        // iterate the nodes which are already in the queue
        int size = queue.size();
        for (int i = 0; i < size; ++i) {
            Node cur = the first node in queue;
            return step if cur is target;
            for (Node next : the neighbors of cur) {
                if (next is not in visited) {
                    add next to queue;
                    add next to visited;
                }
            }
            remove the first node from queue;
        }
        step = step + 1;
    }
    return -1;          // there is no path from root to target
}
```



## DFS

## Traversal
**In Order Iterative Traversal**
This is a standard iterative inorder traversal using stack. This can be applied to various tree problems
```
vector<int> isValidBST(TreeNode* root) {

    /*
    Time complexity : O(N) in the worst case
    when the tree is BST or the "bad" element is a rightmost leaf.

    Space complexity : O(N) to keep stack.
    */

    // This is a DFS solution - inorder iterative traversal
    if (root == nullptr) return true;
    stack<TreeNode*> st;
    TreeNode* prev = nullptr;
    vector<int> result;
    while(root != nullptr || !st.empty())
    {
        while(root != nullptr)
        {
            st.push(root);
            root = root->left;
        }

        root = st.top(); st.pop();
        result.push_back(root->val);

        prev = root;
        root = root->right;            
    }
    return result;
}
```
Question [Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)

```
    bool isValidBST(TreeNode* root) {

        // https://leetcode.com/problems/validate-binary-search-tree/solutions/32112/learn-one-iterative-inorder-traversal-apply-it-to-multiple-tree-questions-java-solution/


        /*
        Time complexity : O(N) in the worst case
        when the tree is BST or the "bad" element is a rightmost leaf.

        Space complexity : O(N) to keep stack.
        */

        // This is a DFS solution - inorder iterative traversal
        if (root == nullptr) return true;
        stack<TreeNode*> st;
        TreeNode* prev = nullptr;
        while(root != nullptr || !st.empty())
        {
            while(root != nullptr)
            {
                st.push(root);
                root = root->left;
            }

            root = st.top(); st.pop();
            if (prev != nullptr && root->val <= prev->val ) return false;
            
            prev = root;
            root = root->right;            
        }
        return true;
    }
```

Question [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)

```
    int kthSmallest(TreeNode* root, int k) {

        stack<TreeNode*> st;
        while( root != nullptr || !st.empty())
        {
            while(root != nullptr)
            {
                st.push(root);
                root = root->left;
            }

            root = st.top();
            st.pop();
            if (--k == 0)  break;

            root = root->right;
        }

        // Assumption is k is smaller than the total number of elements
        return root->val;
    }
```


# STL

# lower_bound and upper_bound


```
value a a a b b b c c c
index 0 1 2 3 4 5 6 7 8
bound       l     u
```
Where l represents the lower bound of b, and u represents the upper bound of b.

So if there are range of values that are "equal" with respect to the comparison being used, lower_bound gives you the first of this, upper_bound gives you one-past-the-end of these. This is the normal pattern of STL ranges **[first, last)**.

```
if (lower_bound() == upper_bound())
    // then the element does not exists
```

```std::lower_bound``` - returns iterator to first element in the given range which is EQUAL_TO or Greater than val.

```std::upper_bound``` - returns iterator to first element in the given range which is Greater than val.


# Some standard programs

## Topological Sort.

## Find shortest Path in Graph.

## Find if path exists between two nodes.

## Stack from Queues

## Shared pointer and Unique Pointer code

**Unique Ptr**
```
namespace ThorsAnvil {
    template<typename T>
// UNIQUE POINTER
    class UP {              
            T*   data;
        public: 
  // Explicit constructor
            explicit UP(T* data) : data(data) {}
            ~UP() {
                delete data;
            }
   // Remove compiler generated methods.
            UP(UP const&)            = delete;    
    // Remove compiler generated methods.
            UP& operator=(UP const&) = delete;      
    // Const correct access owned object
            T* operator->() const {return data;}     
    // Const correct access owned object
            T& operator*()  const {return *data;}    
    // Access to smart pointer state
            T* get()                 const {return data;}     
            explicit operator bool() const {return data;}
           // Modify object state
            T* release() {                       
                T* result = nullptr;
                std::swap(result, data);
                return result;
            }
    };
}
```

**shared_ptr** code
```
namespace ThorsAnvil {
    template<typename T>
    class SP {        //SHARED POINTER 
        T*      data;
        int*    count;
        public:
            explicit SP(T* data) // Explicit constructor
            try                 : data(data), count(new int(1)) {}
            catch(...) {
                // If we failed because of an exception delete the pointer and rethrow the exception.
                delete data;
                throw;
            }
            ~SP()  {
                --(*count);
                if (*count == 0)  {
                    delete data;
                }
            }
            SP(SP const& copy)  : data(copy.data), count(copy.count)  {
                ++(*count);
            }
            // Use the copy and swap idiom, It works perfectly for this situation.
            SP& operator=(SP rhs)  {
                rhs.swap(*this);
                return *this;
            }
            SP& operator=(T* newData)  {
                SP tmp(newData);
                tmp.swap(*this);
                return *this;
            }
// Always good to have a swap function, make sure it is noexcept
            void swap(SP& other) noexcept {
                std::swap(data, other.data);
                std::swap(count, other.count);
            }
            // Const correct access owned object
            T* operator->() const {return data;}
            T& operator*() const {return *data;}
 
            // Access to smart pointer state
            T* get()                 const {  return data;  }
            explicit operator bool() const {return data;}
        };
}

```


