---
layout: post
title:  "The Heap Allocation Behaviour of the C++ std::unordered_set"
date: 2019-04-22
published: true
comments: true
categories: [performance]
tags: [unordered_set, hash table]
---

Hash tables are very useful whenever a constant time lookup of previously calculated results is required. It is therefore good to know how these containers are implemented and used. The C++ std::unordered_map and std::unordered_set are good examples of a hash table and hash set respectively. This post will investigate the heap allocation behaviour of std::unordered_set. One can expect the behaviour of std::unordered_map to be similar.

## The std::unordered_set Implementation
std::unordered_set is a set container implemented as a hash set. The container maintains a number of 'buckets' that can each hold more than one item. When an item needs to be inserted into the container a bucket index is first calculated from a hash of the item's value. The item is then added to the bucket which is typically implemented as a linked list.

The C++ std::unordered_set keeps track of its load factor i.e. the average number of items per bucket. Once the load factor goes beyond the maximum load factor (default 1.0) the container is resized to double its number of buckets which halves its load factor. Keeping the load factor low means that the chance of having multiple items per bucket would be relatively small which in turn keeps the cost of the linked list implementation of the bucket low.

## The Heap Allocation Behaviour
The below results show the behaviour of the unordered set container when inserting 20 million random 32-bit unsigned integers. The tests were compiled on Apple LLVM (clang) compiler version 10.0.1 with -O3 (default Xcode release flags). 

The container was allowed to automatically resize to keep its load factor from exceeding the default max load factor of 1.0. There is a small probabilty of randomly generating the same number more than once which resulted in an actual set size of 19.95 million items.

I would have liked to use valgrind's massif tool (i.e. valgrind --tool=massif ...) to analyse the memory usage of the std::unordered_set, but it seems that valgrind does not yet support macOS Mojave. So I wrote a minimal allocator to track the number of bytes and breakdown of allocations on the heap. The tracking allocator calls the default allocator's allocate and deallocate functions. The [source code](https://github.com/bduvenhage/Bits-O-Cpp/blob/master/containers/main_hash_table.cpp) for the tests is available in my [Bits-O-Cpp GitHub repo](https://github.com/bduvenhage/Bits-O-Cpp).

As random numbers are inserted into the set the load factor increases. When the load factor exceeds the max load factor of 1.0 the number of buckets are doubled which halves the load factor. This behaviour of the load factor and number of buckets is shown below.

<img src="/assets/images/unordered_set_load_factor.pdf" width="600" />

<img src="/assets/images/unordered_set_buckets.pdf" width="600" />

Increasing the number of buckets requires a reallocation of buckets and a redistribution of items. The below figure shows the time lost whenever the number of buckets is doubled. 

<img src="/assets/images/unordered_set_running_time.pdf" width="600" />

Storing 20 million uint32_t values in an unordered_set required 657 MB of heap memory. After adding all the items, the allocation breakdown looked like:

| Element Size | Total Num. Elements Allocated | Num. Currently on Heap |
|--------------|-------------------------------|------------------------|
| size = 8     | 52679739                      | 26339969               |
| size = 24    | 19953544                      | 19953544               |

The reserved buckets are stored as an array of 64-bit (8 byte) pointers. Each linked list bucket entry is stored as the item's hash, its uint32_t value (with 4 bytes of padding) and a pointer to the next item in the bucket. A bucket entry therefore requires 24 bytes and looks similar to this struct:

{% highlight c++ %}
struct BucketItem
{
    size_t hash_;
    uint32_t item_; 
    //4 bytes of padding lay here.
    BucketItem *next_;
};
{% endhighlight %}

It is interesting to note that the number of the 24 byte bucket entry allocations is the same as the number of items in the set. This shows that an allocated bucket entry is reused when the hash set is resized. In contrast to this, the total number of buckets that are allocated is about twice the number of buckets eventually used which is as expected for an array that is resized by doubling.

## Summary
This post has hopefully clarified how the size of the C++ unordered_set (and similarly the unordered_map) is related to the number of items by the max load factor. A good rule of thumb for the container overhead on a 64-bit system and a max load factor of 1.0 would be 8+16 bytes per item. A future investigation could perhaps explore the performance to memory tradeoff of different container max load factors.

