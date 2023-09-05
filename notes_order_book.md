
<!-- TOC -->- [1. Github itch-order-book](#1-github-itch-order-book)
- [1. Github itch-order-book](#1-github-itch-order-book)
- [2. Custom Memory Pool](#2-custom-memory-pool)
- [3. Data Structures/Class](#3-data-structuresclass)
- [4. Design Alternatives](#4-design-alternatives)
- [5. Design Nitty Gritty](#5-design-nitty-gritty)
- [6. Pseudo Code](#6-pseudo-code)
  - [6.1. add\_order](#61-add_order)
  - [6.2. execute\_order (order\_id\_t oid, qty\_t qty)](#62-execute_order-order_id_t-oid-qty_t-qty)
  - [6.3. reduce\_order(order\_id\_t const oid, qty\_t const qty)](#63-reduce_orderorder_id_t-const-oid-qty_t-const-qty)
  - [6.4. delete\_order(order\_id\_t const oid)](#64-delete_orderorder_id_t-const-oid)
  - [6.5. replace\_order(order\_id\_t const old\_oid, order\_id\_t const new\_oid,qty\_t const new\_qty, sprice\_t new\_price)](#65-replace_orderorder_id_t-const-old_oid-order_id_t-const-new_oidqty_t-const-new_qty-sprice_t-new_price)
- [7. Internal Methods](#7-internal-methods)
  - [7.1. DELETE\_ORDER](#71-delete_order)
  - [7.2. REDUCE\_ORDER](#72-reduce_order)



<!-- /TOC -->

# 1. Github itch-order-book

1.	Orders are maintained in Vector indexed by order id. Orders contain reference to Level. So an order update can directly index Level object and update the qty
2.	Level maintains the aggregated price and qty.
3.	PRICE_LEVEL is a sorted vector containing price and reference to Level.



# 2. Custom Memory Pool
 
It uses two custom memory pools
 
1.	**Order Store**: This a simple vector<Order>, its allocated with the capacity of twice the size of highest order Id observed historically. The vector can be indexed by the incoming order_id.

static oidmap<order_t> oid_map;

2.	**Level Pool**: This is a memory pool of Level objects. It uses a non-shrinking vector as its pool, and a vector (LIFO stack) as its free list. It provides alloc and dealloc methods where the users can request a Level object by calling alloc and can return back the object to the pool by calling dealloc.
 
static constexpr size_t NUM_LEVELS = 1 << 20;
using level_vector = pool<level, level_id_t, NUM_LEVELS>; 


# 3. Data Structures/Class
 
 
**1.	Order**
Given an orderId we can find its qty, and the levelIdx - by looking up in orderstore
 
typedef struct order {
  qty_t m_qty;
  level_id_t level_idx;   (An Index to LevelPool )
} order_t;
 
This is allocated from Order Store
 
**2.	Level **
Each order and price_level holds a reference (levelIdx) to this level object. Given an LevelIdx - we can find the aggregated Quantity and Price at that level.
 
class level
{
 public:
  sprice_t m_price;
  qty_t m_qty;
};
This is allocated from Level pool


 
**3.	price_level**
 
The list of Bids and Offers is a sorted list of price_level objects. The price_level object is moved/rearranged during the sort and insert/delete. This object holds a reference (levelIdx) to this level object. This object is not permanent its physical location and reference changes based on the market movement.
 
class price_level
{
  sprice_t m_price;
  level_id_t m_ptr; (An index to Level Pool)
};

using sorted_levels_t = std::vector<price_level>;
sorted_levels_t m_bids;
         sorted_levels_t m_offers;
 
The Object life cycle is managed by vector.


# 4. Design Alternatives
 
I've discussed the Order Book design using three below techniques. Each has its own strength and weakness.
 
1.	**[Vector]**: Bids and Offers  maintained as sorted vector 
•	The bids and offers are maintained as sorted vector of price_level, the best_bid and best_ask is located at the end of the vector, thus any updates to the first few levels of best bid and offer would not result in major relocation of elements of the vector.
2.	**[Intrusive List]:** Bids and Offers maintained as sorted Intrusive list:
•	We remove the price_level and the Level structure is modified to hold left and right pointer and the sorted list of bids and offers is now an intrusive list of Level .
•	We achieve a better worst case performance for EXECUTE_ORDER and DELETE_ORDER
3.	**[Intrusive List Price Index]:** Bids and Offers maintained as sorted Intrusive list - with price index:
•	With everything remaining same as (2), we change the LevelPool such that users can index Level objects based on the price (which means we would allocate Level for every possible price point). 
•	We have achieved constant time look up O(1) in all operations using earlier design's except for ADD_ORDER event. With this approach we improve the performance of ADD_ORDER if there exists an active level for the order price. Below is a pseudo code.
•	ADD_ORDER
o	If (LevelPool[order.price].active)
•	Add the updated Qty
o	Else
•	Insert the LevelPool[order.price] in the sorted intrusive list.  // LINEAR time complexity
 

# 5. Design Nitty Gritty
1.	Prices are sorted in descending order from the end, with the highest price taking the first position at the end and then others.
2.	For the offer/sell – we multiply by the price by -1, thus the best offer becomes the highest value and occupies first position at the end. 



# 6. Pseudo Code 

## 6.1. add_order
``` 
1.	Get an Order object from oidmap
2.	Set the Qty
3.	Get the sorted vector based on bid or offer.
sorted_levels_t *sorted_levels = is_bid(price) ? &m_bids : &m_offers;
4.	Search in the sorted list from the end()
a.	If found
Set order.levelIdx with the matching price_level levelIdx
b.	Else (price> curprice.m_price)
break;
5.	If level not found - -then create a new level, set on the order, create a price_level and insert the price_level into the sorted list.
6.	Update the Qty of the sorted_list corresponding to the order levelIdx
```

```


  static void add_order(order_id_t const oid, book_id_t const book_idx,
                        sprice_t const price, qty_t const qty)
  {
#if TRACE
    printf("ADD %lu, %u, %d, %u", oid, book_idx, price, qty);
#endif  // TRACE
    oid_map.reserve(oid);
    order *order = oid_map.get(oid);
    order->m_qty = qty;
   ADD_ORDER(order, price, qty);
  }

  void ADD_ORDER(order_t *order, sprice_t const price, qty_t const qty)
  {
    sorted_levels_t *sorted_levels = is_bid(price) ? &m_bids : &m_offers;
    // search descending for the price
    auto insertion_point = sorted_levels->end();
    bool found = false;
    while (insertion_point-- != sorted_levels->begin()) {
      price_level &curprice = *insertion_point;
      if (curprice.m_price == price) {
        order->level_idx = curprice.m_ptr;
        found = true;
        break;
      } else if (price > curprice.m_price) {
        // insertion pt will be -1 if price < all prices
        break;
      }
    }
    if (!found) {
      order->level_idx = s_levels.alloc();
      s_levels[order->level_idx].m_qty = qty_t(0);
      s_levels[order->level_idx].m_price = price;
      price_level const px(price, order->level_idx);
      ++insertion_point;
      sorted_levels->insert(insertion_point, px);
    }
    s_levels[order->level_idx].m_qty = s_levels[order->level_idx].m_qty + qty;
  }

```



##  6.2. execute_order (order_id_t oid, qty_t qty)
1.	Get the order corresponding to the order id
2.	If (qty == order->m_qty) // If the whole order qty is executed.
a.	DELETE_ORDER(order)
3.	Else 
a.	REDUCE_ORDER(order, qty)

```
  static void execute_order(order_id_t const oid, qty_t const qty)
  {
#if TRACE
    printf("EXECUTE %lu %u\n", oid, qty);
#endif  // TRACE
    order_t *order = oid_map.get(oid);
    order_book *book = &s_books[MKPRIMITIVE(order->book_idx)];

    if (qty == order->m_qty) {
      book->DELETE_ORDER(order);
    } else {
      book->REDUCE_ORDER(order, qty);
    }
  }

```
## 6.3. reduce_order(order_id_t const oid, qty_t const qty)
REDUCE_ORDER(order, qty);


## 6.4. delete_order(order_id_t const oid)

DELETE_ORDER(order);

## 6.5. replace_order(order_id_t const old_oid, order_id_t const new_oid,qty_t const new_qty, sprice_t new_price)
```
-	Get the old order from OrderMap
-	Delete the order
-	Add order
-	   order_t *order = oid_map.get(old_oid);
-	    order_book *book = &s_books[MKPRIMITIVE(order->book_idx)];
-	    bool const bid = is_bid(book->s_levels[order->level_idx].m_price);
-	    book->DELETE_ORDER(order);
-	    if (!bid) {
-	      new_price = sprice_t(-1 * MKPRIMITIVE(new_price));
-	    }
-	    book->add_order(new_oid, order->book_idx, new_price, new_qty);
```                            

# 7. Internal Methods

## 7.1. DELETE_ORDER
```
// shared between delete and execute
DELETE_ORDER(order_t *order)
1.	Reduce the qty from the Level
2.	If (level.qty == 0) – 
a.	find the price_level in the sorted array 
b.	delete the price_level from the sorted array
c.	delete the level from the level pool.

  // shared between delete and execute
  void DELETE_ORDER(order_t *order)
  {
    assert(MKPRIMITIVE(s_levels[order->level_idx].m_qty) >=
           MKPRIMITIVE(order->m_qty));
    auto tmp = MKPRIMITIVE(s_levels[order->level_idx].m_qty);
    tmp -= MKPRIMITIVE(order->m_qty);
    s_levels[order->level_idx].m_qty = qty_t(tmp);
    if (qty_t(0) == s_levels[order->level_idx].m_qty) {
      // DELETE_SORTED([order->level_idx].price);
      sprice_t price = s_levels[order->level_idx].m_price;
      sorted_levels_t *sorted_levels = is_bid(price) ? &m_bids : &m_offers;
      auto it = sorted_levels->end();
      while (it-- != sorted_levels->begin()) {
        if (it->m_price == price) {
          sorted_levels->erase(it);
          break;
        }
      }
      s_levels.free(order->level_idx);
    }
  }


```



## 7.2. REDUCE_ORDER

  // shared between cancel(aka partial cancel aka reduce) and execute
  void REDUCE_ORDER(order_t *order, qty_t const qty)
```
-	Reduce the qty from the level
-	Reduce the qty from the order
-	  // shared between cancel(aka partial cancel aka reduce) and execute
-	  void REDUCE_ORDER(order_t *order, qty_t const qty)
-	  {
-	    auto tmp = MKPRIMITIVE(s_levels[order->level_idx].m_qty);
-	    tmp -= MKPRIMITIVE(qty);
-	    s_levels[order->level_idx].m_qty = qty_t(tmp);
-	
-	    tmp = MKPRIMITIVE(order->m_qty);
-	    tmp -= MKPRIMITIVE(qty);
-	    order->m_qty = qty_t(tmp);
-	  }

```



