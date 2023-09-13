- [1. How do you implement consolidated Queue.](#1-how-do-you-implement-consolidated-queue)
- [2. Implement order throttle program.](#2-implement-order-throttle-program)
- [3. Implement serializer and deserializer of array of int.](#3-implement-serializer-and-deserializer-of-array-of-int)
- [4. Market is moving fast as option market maker how do you adjust passive open position.](#4-market-is-moving-fast-as-option-market-maker-how-do-you-adjust-passive-open-position)
- [5. How do you implement tick size correction on the prices?](#5-how-do-you-implement-tick-size-correction-on-the-prices)
- [6. Data structure to handle addition/deletion in middle](#6-data-structure-to-handle-additiondeletion-in-middle)
- [7. How-do-I-sort-a-very-large-file-with-a-limited-amount-of-memory](#7-how-do-i-sort-a-very-large-file-with-a-limited-amount-of-memory)



# 1. How do you implement consolidated Queue.


# 2. Implement order throttle program. 
The question will be to send 20 orders per second.
There can be two possible solution based on the problem 
1. if the number of orders are small. 

Implement a circular Queue with size of number of orders, where each element contains the timestamp when the order was sent.

circular_queue<int> q(20)

```
bool can_send_order()
{
    cur_time = get_curTime();
    while(!q.empty() &&
        (cur_time - q.head() >= 1second) )
    {
        q.pop();
    }
    
    if (q.size() < MAX_COUNT)
        return true;
}


void send_order(order& o)
{
    if (can_send_order())
    {
        cur_time = get_curTime();
        q.push(cur_time);
    }
}
```

2. if the max number of orders are huge 1000.

We can bucket the total time into various buckets, for example 1 second could be divided into 100 buckets of 10ms.


# 3. Implement serializer and deserializer of array of int.

The BigEndian: The bigger part of the byte is stored at the lowest address in memory. That is at the end
The LittleEndian: The Little part or the lowest part of the byte is stored at the lowest address in the memory.





```

vector<char> serialize( vector<char>& arr)
{
    vector<char> res;
    for (auto c: arr)
    {
        res.push_back(htobe8(c));
    }
    return res;
}

//
vector<char> deserialize( vector<char>& arr)
{
    // Same as serialize 
    // but use be8toh
}

// change the call methods to
be16toh, be32toh, b64toh
and viceversa based on the datatype. 

```

# 4. Market is moving fast as option market maker how do you adjust passive open position.


# 5. How do you implement tick size correction on the prices?


```
correct_tick_size = (price/(int)tick) * tick;
```

# 6. Data structure to handle addition/deletion in middle
You have a huge array/vector with lot of deletion happening at the middle of the vector. What datastrucure could you use?

Intrusive list is best suited here.

# 7. How-do-I-sort-a-very-large-file-with-a-limited-amount-of-memory

https://www.quora.com/How-do-I-sort-a-very-large-file-with-a-limited-amount-of-memory


This process is similar to sorting/merging - mutliple sorted list.
As other answers have mentioned, you can use an approach similar to the merge-sort algorithm.

Imagine that your data is on a hard drive (secondary storage).

1. Read 1 GB (or less) chunks from the file into an array.
2. Sort the elements in this array using any in-place sorting algorithm.
3. Write the result into a new file (we will call them intermediate output files).
4. Repeat steps 1 to 3 until all data is read from the input file.

During the merging process, follow these steps:

1. Open all intermediate-output-files simultaneously.
2. Create a sorted list. The elements of this list are a pair of values. One value is the ID of the file from which the data-element was read, and the second value is the data-element itself. The list is sorted on the data-element values. Insert one data-element from each intermediate-output-file.
3. Now read one element from the sorted list (the smallest element) and write it into your final merged-output-file.
Every time you pull the smallest element from the sorted list, use the file ID of that element to pull a new data-element from the corresponding intermediate-output-file and insert it into the sorted list.
4. Continue this process until all data-elements are written into the final merged-output-file.



