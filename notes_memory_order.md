
- [1. Cache Coherency](#1-cache-coherency)
- [2. Cache Line](#2-cache-line)
  - [2.1. How to get cacheline Size (C++17 and beyond)](#21-how-to-get-cacheline-size-c17-and-beyond)
  - [2.2. **Types of Caches**](#22-types-of-caches)
  - [2.3. **Cache Sizes**](#23-cache-sizes)
  - [2.4. cache hit/miss](#24-cache-hitmiss)
- [3. Latency of L1, L2, L3 caches.](#3-latency-of-l1-l2-l3-caches)
- [4. Core Pinning](#4-core-pinning)
- [5. Measurement of Time RDTSC](#5-measurement-of-time-rdtsc)
- [6. Memory Order](#6-memory-order)


# 1. Cache Coherency
The same cache line copies exists in different L1 caches and in the main memory, if the core-0 overwrites one value then all the other cache line will get invalidated


# 2. Cache Line

Cache lines are of usually 64 bytes

You may choose to align large objects and arrays by the cache line size, which is typically 64 bytes. This makes sure that the beginning of the object or array coincides with the beginning of a cache line. Some compilers will align large static arrays automatically but you may as well specify the alignment explicitly by writing: 
 
alignas(64) int BigArray[1024]; 
alignas(64) int shared_data; 


## 2.1. How to get cacheline Size (C++17 and beyond)

std::hardware_destructive_interference_size provides the cacheline size

https://en.cppreference.com/w/cpp/thread/hardware_destructive_interference_size

```
struct keep_apart {
  alignas(std::hardware_destructive_interference_size) std::atomic<int> cat;
  alignas(std::hardware_destructive_interference_size) std::atomic<int> dog;
};
```



## 2.2. **Types of Caches**

**L1 Cache:** L1d is the level 1 data cache, L1i the level 1 instruction cache, etc

**L2 cache** is much larger than L1 but at the same time slower as well. They range from 4-8MB on flagship CPUs (512KB per core). 

**L3 cache** Each core has its own L1 and L2 cache while the last level, the L3 cache is shared across all the cores on a die.


## 2.3. **Cache Sizes**
Modern mainstream Intel CPUs (since the first-gen i7 CPUs, Nehalem) use 3 levels of cache.
L1: •	32kiB split L1i/L1d: private per-core (same as earlier Intel)

L2: 1 MB private per-core.

L3:  6 to 8 MB of L3.

https://stackoverflow.com/questions/944966/how-are-cache-memories-shared-in-multicore-intel-cpus


## 2.4. cache hit/miss

**cache hit**

When the CPU needs data, it first searches the associated core’s L1 cache. If it’s not found, the L2 and L3 caches are searched next. If the necessary data is found, it’s called a cache hit. 
 
**cache miss**

On the other hand, if the data isn’t present in the cache, the CPU has to request it to be loaded on to the cache from the main memory or storage. This takes time and adversely affects performance. This is called a cache miss.
 
Generally, the cache hit rate improved when the cache size is increased. This is especially true in the case of gaming and other latency-sensitive workloads.

# 3. Latency of L1, L2, L3 caches.

L1 - 4 to 8 ns (single digit ns)

L2 - 20 to 40 ns (single digit ns)

L3 - 60 to 80 ns (single digit ns)


Good article on measuring L1,L2,L4 cache latency
https://stackoverflow.com/questions/21369381/measuring-cache-latencies



# 4. Core Pinning

On the machine - certains cpu's can be isolated via  configuration so that the OS will not use them for any OS tasks, nor schedule OS related process on them.

Taskset

This is a unix command - which forces the process to run only on the provided cpu list in the option. 

```
taskset -c 5,6 ./launch-threads-report-cpu - Taskset restrict the affinity of the process to only two CPUs - 5 and 6
```

**pthread_setaffinity_np**







# 5. Measurement of Time RDTSC

Very good read: http://btorpey.github.io/blog/2014/02/18/clock-sources-in-linux/

https://github.com/btorpey/clocks

**How Linux Keeps the Time**
 
When the system boots is that the TSC (Time Stamp Counter) starts running. The TSC is a register counter that is also driven from a crystal oscillator – the same oscillator that is used to generate the clock pulses that drive the CPU(s). As such it runs at the frequency of the CPU, so for instance a 2GHz clock will tick twice per nanosecond. A 4GHz clock will tick 4 times per nanosecond. Thus one counter in TSC on 4GHZ processor is 0.25ns
 
2.	How to measure time with rdtsc and rdtscp?
 
Start = rdtscp()
End = rdtscp()
 
Time diff in NanoSecond = (start - end) / TICKS_PER_NANOSEC
 
TICKS_PER_NANOSEC ON 4GHZ IS 4.


Below are the various ways time can be measured, ranked in terms of efficiency. The stats are here.
 http://btorpey.github.io/blog/2014/02/18/clock-sources-in-linux/

Although the rdtsc is fastest but it has jitters and rdtscp() should be preferred. 


-	rdtsc
-	rdtscp
-	rdtsc + cpuid
-	clock_gettime() + time()
-	gettimeofday()
-	clock()


```
ClockBench.cpp - All timings are in nanoseconds

                   Method       samples     min     max     avg  median   stdev
           CLOCK_REALTIME       255       54.00   58.00   55.65   56.00    1.55
    CLOCK_REALTIME_COARSE       255        0.00    0.00    0.00    0.00    0.00
          CLOCK_MONOTONIC       255       54.00   58.00   56.20   56.00    1.46
      CLOCK_MONOTONIC_RAW       255      650.00 1029.00  690.35  839.50   47.34
   CLOCK_MONOTONIC_COARSE       255        0.00    0.00    0.00    0.00    0.00
              cpuid+rdtsc       255       93.00   94.00   93.23   93.50    0.42
                    rdtsc       255       24.00   28.00   25.19   26.00    1.50
```

As you might expect, the combination of CPUID and RDTSC is slower than RDTSCP, which is slower than RDTSC alone. In general, this would suggest that RDTSCP should be preferred if available, with a fallback to CPUID+RDTSC if not. (While RDTSC alone is the fastest, the fact that it can be inaccurate as a result of out-of-order execution means it is only useful for timing relatively long operations where that inaccuracy is not significant – but those are precisely the scenarios where its speed is less important).




# 6. Memory Order





typedef enum memory_order {
    memory_order_relaxed,
    memory_order_consume,		//C++ 20 and beyond – its discouraged to use now
    memory_order_acquire,
    memory_order_release,
    memory_order_acq_rel,
    memory_order_seq_cst
} memory_order;

std::memory_order specifies how regular, non-atomic memory accesses are to be ordered around an atomic operation. Absent any constraints on a multi-core system, when multiple threads simultaneously read and write to several variables, one thread can observe the values change in an order different from the order another thread wrote them. Indeed, the apparent order of changes can even differ among multiple reader threads. Some similar effects can occur even on uniprocessor systems due to compiler transformations allowed by the memory model.
The default behavior of all atomic operations in the library provides for sequentially consistent ordering (see discussion below). That default can hurt performance, but the library's atomic operations can be given an additional std::memory_order argument to specify the exact constraints, beyond atomicity, that the compiler and processor must enforce for that operation.


 **memory_order_relaxed:**
-	This is the most relaxed operation: 
-	there are no synchronization or ordering constraints imposed on other reads or writes, only this operation's atomicity is guaranteed

**memory_order_consume:	**
1.	A load operation with this memory order performs a consume operation on the affected memory location: 
2.	No reads or writes in the current thread dependent on the value currently loaded can be reordered before this load. Writes to data-dependent variables in other threads that release the same atomic variable are visible in the current thread. On most platforms, this affects compiler optimizations only

**memory_order_acquire:**
1.	A load operation with this memory order performs the acquire operation on the affected memory location: 
2.	No reads or writes in the current thread can be reordered before this load. 
3.	All writes in other threads that release the same atomic variable are visible in the current thread

**memory_order_release	**
1.	A store operation with this memory order performs the release operation: 
2.	No reads or writes in the current thread can be reordered after this store. 
3.	All writes in the current thread are visible in other threads that acquire the same atomic variable (see Release-Acquire ordering below) and writes that carry a dependency into the atomic variable become visible in other threads that consume the same atomic (see Release-Consume ordering below).

**memory_order_acq_rel	**
1.	A read-modify-write operation with this memory order is both an acquire operation and a release operation. 
2.	No memory reads or writes in the current thread can be reordered before or after this store. 
3.	All writes in other threads that release the same atomic variable are visible before the modification and the modification is visible in other threads that acquire the same atomic variable.

This can be used as a memory order parameter for load, store and read-modify-write operation.
Atomic<int> a;
a.load(memory_order_acq_rel)
a.store(4, memory_order_acq_rel)
a.compare_exchange_weak(1,2,memory_order_acq_rel)


**memory_order_seq_cst	**
1.	A load operation with this memory order performs an acquire operation, a store performs a release operation, 
2.	And read-modify-write performs both an acquire operation and a release operation, 
3.	plus a single total order exists in which all threads observe all modifications in the same order (see Sequentially-consistent ordering below)
4.	This is the default memory ordering used by atomic operations. 
atomic<int> a;
a.load() // uses memory_order_seq_cst by default
a.store(4) // uses memory_order_seq_cst by default

5.	This is also the most expensive memory order among all – due to its promise of total ordering and sequential consistency. Which means all threads have consistent view of the atomic value.  If an atomic value is X in one thread and cache would mean the same value has to be visible by all the threads and cache all this has cost associated. Whereas the same thing in release-acquire it is pair wise ordering i.e the thread which releases(stores) and the thread which acquires the same atomic variable need to have consistent view of the atomic and non-atomic variables before the store with the thread which does the acquire. Whereas the other threads can have old/stale values of the atomic(relaxed) and non-atomic values.


