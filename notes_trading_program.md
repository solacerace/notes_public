- [1. How do you implement consolidated Queue.](#1-how-do-you-implement-consolidated-queue)
- [2. Implement order throttle program.](#2-implement-order-throttle-program)
- [3. Implement serializer and deserializer of array of int.](#3-implement-serializer-and-deserializer-of-array-of-int)
- [4. Market is moving fast as option market maker how do you adjust passive open position.](#4-market-is-moving-fast-as-option-market-maker-how-do-you-adjust-passive-open-position)



# 1. How do you implement consolidated Queue.


# 2. Implement order throttle program. 
The question will be to send 20 orders per second.
There can be two possible solution based on the problem 
1. if the number of orders are small. 

Implement a circular Queue with size of number of orders, where each element contains the timestamp when the order was sent.

circular_queue<int> q(20)

```
bool can_send_order();
void send_order(order& o);
```

1. if the max number of orders are huge 1000. 

# 3. Implement serializer and deserializer of array of int.


# 4. Market is moving fast as option market maker how do you adjust passive open position.



