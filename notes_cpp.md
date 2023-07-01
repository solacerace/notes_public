
- [1. Notes](#1-notes)
  - [1.1. B](#11-b)
  - [1.2. C](#12-c)
  - [1.3. J](#13-j)
  - [1.4. C](#14-c)
- [2. CPP](#2-cpp)
  - [3.1. Structure Binding](#31-structure-binding)
  - [3.2. Spin Lock](#32-spin-lock)
  - [3.2. Epoll](#32-epoll)
  - [3.2. Memory Order](#32-memory-order)
- [4. Variant and visit](#4-variant-and-visit)
- [5. Books and Materials ](#5-books-and-materials)
- [6. TBD](#4-tbd)
  
# 1. Notes
## 1.1. B
1. Multiple threads running in parallel - how do you enforce ordering?
   - Answer: Develop a Action Queue and let all threads post their action items on the queue and the items will be processed in sequential order. 
2. Generic Lambda's
3. Lambda with mutable identifier
## 1.2. C
1. Generic code for Sliding Window or throttler - the sliding window size we should be able to configure? 
2. What are your thoughts about unit testing?
3. How do you go about writing unit test code?
4. Should we have unit tests for unit test code?
5. What kind of unit test coverage you should be aiming at.

## 1.3. J
1. How do you implement tick size correction on the prices?
2. What is half-tick? How do we maximize the profit if half tick?
3. Explain how will you design a low-latency trading system? how will you minimize locks?
4.  

## 1.4. C
1. Print all permutations NpR
2. Some question on Tree 

# 2. CPP
## 3.1. Structure Binding
1. What is the command to measure the tcp route a tcp connection takes from source to destination.
2. Did you use any thread operation introduced in C++14 - like future/promise and what was the use case.
3. How do you bind a thread to a cpu core? - explain the shell command and code as well.
4. code: **pthread_setaffinity_np**  https://eli.thegreenplace.net/2016/c11-threads-affinity-and-hyperthreading/
5. commandline: **taskset**: https://eli.thegreenplace.net/2016/c11-threads-affinity-and-hyperthreading/

## 3.2. Spin Lock
1. Why do we need Spin Lock?
2. Spin Lock Implementation
   
Cache coherence which stands for the uniformity of shared resources data that ends up stored in multiple caches – which is what happens when we run on multiple CPUs that each processor has its own cache. so we end up with one copy in the main memory and one in the local cache of each processor that requested that data.

Since there might be more than one thread spinning and trying to acquire the lock a lock of cache coherency traffic is going around to make sure all the caches are valid. We can leverage the fact the readers are less expensive since they don’t affect the state of the cache. we can wait for the lock holder to first release the lock and only then try and write to it. Switching to a test and test and set tactics. where we first test which is reading only and only then attempting to test and write and only then adding coherency traffic if we have a hint that the lock is free. We can achieve this using a nested loop.

``` cpp
class Spinlock
{
private:
    std::atomic_flag atomic_flag = ATOMIC_FLAG_INIT;

public:
    void lock()
    {
        while (true)
        {
            // test and set returns oldValue & sets true
            if (!atomic_flag.test_and_set(std::memory_order_acquire))
            {
                break;
            }
            while (atomic_flag.test(std::memory_order_relaxed));
        }
    }
    void unlock()
    {
        atomic_flag.clear(std::memory_order_release);
    }
};
```
## 3.2. Epoll

Here's an example of an epoll-based server in C++ that can handle multiple clients concurrently:

```
cpp#include <iostream>
#include <sys/epoll.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <cstring>
#include <vector>
#include <stdexcept>

constexpr int MAX_EVENTS = 10;
constexpr int BUFFER_SIZE = 1024;

int main() {
    int serverSocket, epollFd;
    struct epoll_event event, events[MAX_EVENTS];
    std::vector<int> clientSockets;
    char buffer[BUFFER_SIZE];

    // Create server socket
    serverSocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serverSocket == -1) {
        std::cerr << "Failed to create socket" << std::endl;
        return 1;
    }

    // Set socket options
    int option = 1;
    if (setsockopt(serverSocket, SOL_SOCKET, SO_REUSEADDR, &option, sizeof(option)) == -1) {
        std::cerr << "Failed to set socket options" << std::endl;
        return 1;
    }

    // Bind server socket to a port
    sockaddr_in serverAddress{};
    serverAddress.sin_family = AF_INET;
    serverAddress.sin_addr.s_addr = INADDR_ANY;
    serverAddress.sin_port = htons(8080); // Change port as needed

    if (bind(serverSocket, reinterpret_cast<sockaddr*>(&serverAddress), sizeof(serverAddress)) == -1) {
        std::cerr << "Failed to bind to port" << std::endl;
        return 1;
    }

    // Start listening for incoming connections
    if (listen(serverSocket, SOMAXCONN) == -1) {
        std::cerr << "Failed to listen for connections" << std::endl;
        return 1;
    }

    // Create epoll instance
    epollFd = epoll_create1(0);
    if (epollFd == -1) {
        std::cerr << "Failed to create epoll instance" << std::endl;
        return 1;
    }

    // Add server socket to epoll
    event.events = EPOLLIN;
    event.data.fd = serverSocket;
    if (epoll_ctl(epollFd, EPOLL_CTL_ADD, serverSocket, &event) == -1) {
        std::cerr << "Failed to add server socket to epoll" << std::endl;
        return 1;
    }

    std::cout << "Server started. Listening for connections..." << std::endl;

    while (true) {
        // Wait for events
        int numEvents = epoll_wait(epollFd, events, MAX_EVENTS, -1);
        if (numEvents == -1) {
            std::cerr << "Failed to wait for events" << std::endl;
            return 1;
        }

        // Handle events
        for (int i = 0; i < numEvents; ++i) {
            if (events[i].data.fd == serverSocket) {
                // New client connection
                sockaddr_in clientAddress{};
                socklen_t clientAddressLength = sizeof(clientAddress);
                int clientSocket = accept(serverSocket, reinterpret_cast<sockaddr*>(&clientAddress),
                                          &clientAddressLength);
                if (clientSocket == -1) {
                    std::cerr << "Failed to accept client connection" << std::endl;
                    continue;
                }

                // Add client socket to epoll
                event.events = EPOLLIN | EPOLLET; // Edge-triggered mode
                event.data.fd = clientSocket;
                if (epoll_ctl(epollFd, EPOLL_CTL_ADD, clientSocket, &event) == -1) {
                    std::cerr << "Failed to add client socket to epoll" << std::endl;
                    continue;
                }

                std::cout << "New client connected: " << inet_ntoa(clientAddress.sin_addr) << std::endl;

                // Store client socket
                clientSockets.push_back(clientSocket);
            } else {
                // Existing client activity
                int clientSocket = events[i].data.fd;
                int bytesRead = read(clientSocket, buffer, BUFFER_SIZE);
                if (bytesRead == -1) {
                    std::cerr << "Failed to read from client socket" << std::endl;
                    continue;
                } else if (bytesRead == 0) {
                    // Client closed the connection
                    std::cout << "Client disconnected" << std::endl;

                    // Remove client socket from epoll and close it
                    if (epoll_ctl(epollFd, EPOLL_CTL_DEL, clientSocket, nullptr) == -1) {
                        std::cerr << "Failed to remove client socket from epoll" << std::endl;
                        continue;
                    }
                    close(clientSocket);

                    // Remove client socket from vector
                    auto it = std::find(clientSockets.begin(), clientSockets.end(), clientSocket);
                    if (it != clientSockets.end()) {
                        clientSockets.erase(it);
                    }
                } else {
                    // Process client data
                    // ...
                    std::cout << "Received data from client: " << buffer << std::endl;

                    // Echo the data back to the client
                    if (write(clientSocket, buffer, bytesRead) == -1) {
                        std::cerr << "Failed to write to client socket" << std::endl;
                        continue;
                    }
                }
            }
        }
    }

    // Cleanup
    for (int clientSocket : clientSockets) {
        close(clientSocket);
    }
    close(serverSocket);

    return 0;
}

```

Explanation:

1.  The server creates a socket, binds it to a specific port, and starts listening for incoming connections.
    
2.  An epoll instance is created using `epoll_create1`. The server socket is added to the epoll using `epoll_ctl` with the `EPOLL_CTL_ADD` operation.
    
3.  The server enters a loop where it waits for events using `epoll_wait`. This call blocks until an event occurs.
    
4.  When a new client connection is established (`events[i].data.fd == serverSocket`), the server accepts the connection using `accept` and adds the client socket to the epoll instance.
    
5.  For existing client activity (`events[i].data.fd != serverSocket`), the server reads data from the client socket using `read`. If the read fails or returns 0 (indicating the client closed the connection), the server removes the client socket from the epoll instance and closes it. Otherwise, it processes the received data (e.g., printing or echoing it).
    
6.  The loop continues to handle events as they occur.
    
7.  When the server terminates, it cleans up by closing all client sockets and the server socket.
    

Note: Error handling and additional logic, such as concurrency control and signal handling, are omitted for brevity.

## 3.2. Memory Order

# 4. Variant and visit

In C++, the terms "variant" and "visit" are related to the concept of sum types or tagged unions. They are typically used to represent a type that can hold values of different types, but only one value at a time.

1.  Variant: A variant, also known as a discriminated union or tagged union, is a data type that can hold values of different types, but only one value at a time. It can be defined using the `std::variant` template from the `<variant>` header in C++17 and later.

Here's an example of using `std::variant` to define a variant that can hold either an integer or a string:

```
cpp#include <variant>
#include <iostream>

int main() {
    std::variant<int, std::string> myVariant;
    myVariant = 42;                  // Assign an integer
    std::cout << std::get<int>(myVariant) << std::endl;

    myVariant = "Hello, World!";     // Assign a string
    std::cout << std::get<std::string>(myVariant) << std::endl;

    return 0;
}

```

In this example, `std::variant<int, std::string>` declares a variant that can hold either an `int` or a `std::string` value. We can assign different types of values to the variant using the assignment operator, and retrieve the value using `std::get` by specifying the type.

2.  Visit: The visit operation is used to perform different actions based on the currently held value in a variant. C++ provides `std::visit` as a way to achieve this functionality. It allows you to provide a set of overloaded function objects (lambdas or function pointers) that are invoked based on the type of the value held by the variant.

Here's an example using `std::visit` to perform different actions based on the type of the variant's value:

```
cpp#include <variant>
#include <iostream>

struct PrintVisitor {
    void operator()(int value) const {
        std::cout << "Integer: " << value << std::endl;
    }

    void operator()(const std::string& value) const {
        std::cout << "String: " << value << std::endl;
    }
};

int main() {
    std::variant<int, std::string> myVariant;

    myVariant = 42;
    std::visit(PrintVisitor(), myVariant);

    myVariant = "Hello, World!";
    std::visit(PrintVisitor(), myVariant);

    return 0;
}

```

In this example, we define a visitor called `PrintVisitor` that overloads the function call operator for different types. The `std::visit` function is then used to invoke the appropriate overload based on the variant's value. The `PrintVisitor` simply prints the value with a corresponding label based on its type.

Output:

```
makefileInteger: 42
String: Hello, World!

```

By using `std::visit`, you can perform different operations or behaviors based on the currently held value in a variant.

They are particularly useful in scenarios where you have a fixed set of possible types and want to handle each type differently `without resorting to runtime polymorphism`.


# Consolidated Queue

Assuming we know the list of tradable symbols in advance.

The trading symbol on the market data update assuming it comes as an integer.

We maintain two datastrcture.

1. A vector of the size of the universe symbol, initialized as nullptr.

mapUpdate.

2. A circular buffer made up of intrusive list of size of universe symbol.

cBuffer



There are 2 threads in play.

Thread 1: incoming Market Data thread.

if ( mapUpdate[symbol] == nullptr)
{
     ptr = cBuffer.push(elem)
     mapUpdate[symbol] = ptr
}
else
{
     ptr = mapUpdate[symbol]
     *ptr = elem
}


Thread 2: Circular Buffer Processing thread.


while (!cbuffer.empty())
{
   ptr = cbuffer.pop()
   mapUpdate[symbol] = nullptr
   unlock()
   process(ptr)
   lock()
}


# 5. Books and Materials

## 5.1 Read Agner Fog document

https://www.agner.org/optimize/optimizing_cpp.pdf

Read Word sizes on 32 bit and 64 bit machines.

Word size is the maximum number of bytes CPU can process at a time in one CPU cycle. on a 64bit machine it is 64 bit (or 8 byte).


## 5.2 What Every Programmer Should Know About Memory
https://www.akkadia.org/drepper/cpumemory.pdf




# 6. TBD

1. Make a list of questions asked in your interviews. What did you fail to answer.
2. Make notes on C++20, C++17
3. Make notes on Network Programming.
4. Make notes on Exchanges in USA.
5. Make notes on ideal Market Data Server.
6. Make notes on ideal Application Server.
7. Leetcode answer all questions - write to pdf.
8. Make notes on reading and writing to stream.
9. Make notes on template meta programming. 
10. Make notes on times taken by various cpu operation, L1,L2,L3 access. 
11. Make notes on L1,L2,L3 caches.
12. Read effective series again
13. Read reddit cpp questions
14. Implement your spin lock
15. Implement lock free queue - spsc, mpsc - Cameroon, moodys - 200ns
16. Shared pointer and weak pointer
17. Make notes on concepts.
18. Integral template argument array size.
19. Read about rule of 3 and rule of 5.
20. Variant and Visit
21. Alignas, false sharing
22. Epoll/Select
23. TCP/IP disconnect keep-alive/ttl/buffersize/ 
24. Memory Order
25. Atomic Wait v/s condition variable.
26. Decltype,declvalue. DeclType is perfect return,it finds type from expression.
27. taskset, cpu sched get, cpu sched set
28. Thread priority, what, how, why?
29. Static polymorphism CRTP
30. ipc between process, shared memory.
31. Talk about an application you wrote and you are proud of that. - Talk about Market Making application.
32. Atomic test v/s test and set 
33. Cache line and why?
34. structure padding 
35. Fastest way to read/write to files. (Q)
36. Simple TCP server and client program.
37. Difference b/w epoll and accept based tcp server.
38. Possible ways to branch, tag dispatch, if constexpr, 
39. Matrix multiplication with memory friendly. ( the inner loop is swapped).
40. What is the best way to measure time - RDTSP any other way (Q).
41. Datastructure for representing list of orders in OMS.
42. Threading model with hot path thread and auxiliary threads (Q).
43. Thread priority and its relevance with core pinning (Q).
44. 

