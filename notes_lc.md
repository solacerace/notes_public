

<!-- TOC -->

- [1. Code snippet](#1-code-snippet)
  - [1.1 Initialize array](#11-initialize-array)
  - [1.2 Numeric limits](#12-numeric-limits)
- [2. Heap](#2-heap)
  - [2.1 C++ Priority Queue](#21-c-priority-queue)
- [3. Linked List](#3-linked-list)
- [4. Trees](#4-trees)
  - [4.1 BFS](#41-bfs)
  - [4.2 DFS](#42-dfs)

<!-- /TOC -->
<!-- /TOC -->(#41-bfs)
  - [4.2 DFS](#42-dfs)

<!-- /TOC -->
<!-- /TOC -->2-heap)

<!-- /TOC -->


# 1. Code snippet

## 1.1 Initialize array

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

## 1.2 Numeric limits

```
#include <limits>

std::cout << std::numeric_limits<int>::min() << "\t│ "
        << std::numeric_limits<int>::max() << '\n';


std::numeric_limits<int>::min()           // -2,147,483,648
std::numeric_limits<int>::max()           //  2,147,483,647
std::numeric_limits<unsigned int>::max()  //  4,294,967,295

```


# 2. Heap

## 2.1 C++ Priority Queue
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






# 3. Linked List


# 4. Trees
## 4.1 BFS
## 4.2 DFS





