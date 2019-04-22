---
layout: post
title:  "The size of the C++ std::unordered_set / Hash-Table"
date: 2019-04-22
published: true
comments: true
categories: [performance]
tags: [unordered_set, hash table]
---

Hash tables are very useful whenever constant lookup time of previously calculated results is required. Therefore, it is good to know how these containers are implemented and used. The C++ std:unordered_map and std::unordered_set are good examples of hash tables. This post will investigate the heap allocation behaviour of std::unordered_set. One can expect the behaviour of std::unordered_map to be similar.

## The std::unordered_set Implementation
std::unordered_set is a set container implemented as a hash set. The container maintains a number of 'buckets' that can each hold more than one item. When an item needs to be inserted into the container a bucket index is first calculated from a hash of the item's value. The item is then added to the bucket which is typically implemented as a linked list.

The C++ std::unordered_set keeps track of its load factor i.e. how many items it contains relative to its number of buckets. Once the load factor goes beyond the maximum load factor (default 1.0) the container is resized to double its number of buckets which halves its load factor. Keeping the load factor low means that having more than one item per bucket would be unlikely which in turn keep the cost of the linked list implementation of the bucket relatively low.

## The Heap Allocation Behaviour of std::unordered_set
The below results show the behaviour of the unordered set container when inserting random 32-bit unsigned integers. The set was allowed to automatically resize to keep its load factor from exceeding the default max load factor of 1.0.

I would have liked to use valgrind's massif tool (i.e. valgrind --tool=massif ...) to analyse the memory usage of the std::unordered_set, but it seems that valgrind does not yet support macOS Mojave. So I wrote a minimal allocator to track the number of bytes and breakdown of allocations on the heap. The tracking allocator calls the default allocator's allocate and deallocate functions. The [source code](https://github.com/bduvenhage/Bits-O-Cpp/blob/master/containers/main_hash_table.cpp) for the tests is available in my [Bits-O-Cpp GitHub repo](https://github.com/bduvenhage/Bits-O-Cpp).

As random numbers are inserted into the set the loading factor increases. When the loading factor exceeds the max loading factor of 1.0 the number of buckets are doubled and the loading factor is halved. The behaviour of the load factor and number of buckets is shown below.

<img src="/assets/images/unordered_set_load_factor.pdf" width="600" />

<img src="/assets/images/unordered_set_buckets.pdf" width="600" />

Increasing the number of buckets requires a reallocation of buckets and a redistribution of items. The below figure shows the time lost whenever the number of buckets are increased.

<img src="/assets/images/unordered_set_running_time.pdf" width="600" />

Storing 20 million uint32_t values in a unordered_set used 657 MB of heap memory. After adding all the items, the allocation breakdown looked like:

|           |  Num. Total Allocations | Num. Currently on Heap |
|-----------|-------------------------|------------------------|
| size = 8  | 52679739                | 26339969               |
| size = 24 | 19953544                | 19953544               |

The reserved buckets are stored as an array of 64-bit (8 byte) pointers. Each linked list bucket entry is stored as the item's hash, its uint32_t value (with 4 bytes of padding) and a pointer to the next item in the bucket. A bucket entry requires 24 bytes looks similar to the below struct:

{% highlight c++ %}
struct BucketItem
{
    size_t hash_;
    uint32_t item_; 
    //4 bytes of padding lay here.
    BucketItem *next_;
};
{% endhighlight %}


## Summary
An unordered_set uses more memeory than just storing the items in a list, but it allows constant time checking/retrieval of items. A good rule of thumb for the overhead of the container on a 64-bit system would be 8+16 bytes per item.

