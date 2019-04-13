---
layout: post
title:  "The Performance of C++ std::deque vs. std::vector"
date: 2019-04-12
published: true
comments: true
categories: [performance]
tags: [deque, vector]
---

Many algorithms make use of a double ended queue (a.k.a. a deque). For example, in breadth first search a deque is used to hold the search frontier. Having a fast implementation of a deque is very useful. 

If the size of the deque is bounded then one can implement it as a ring buffer over a pre-allocated array. If the size of the deque is not bounded then one would have to trade some performance for the ability to dynamically grow the capacity of the deque.

<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-MML-AM_CHTML' async></script>

I've been considering implementing a ring buffer STL container or container adaptor for doing breadth first search faster than would perhaps be possible with an STL deque. However, an STL deque is a much cooler (and faster) container than I initially thought. The deque stores its elements in cache friendly chunks and still allows constant time random access (although with one more dereference than required with a vector). 

Given its design, the std::deque has the potential to be very efficient. This post discusses std::deque's internal workings and compares its performance to that of std::vector.

## The STL Deque Implementation
The image below shows the memory layout of a std::deque. The data is stored in blocks or chunks which are referenced from a list of chunks called a map. The beginning and end of the map as well as the beginning of the first chunk and the end of the last chunk can have empty slots.

<img src="/assets/images/deque.pdf" width="600" />

The push_back operations adds an element to the next available slot of the last chunk. If no space is available in the last chunk then a new chunk is added to the next unused slot in the map. If no space is available at the end of the map then the map is first resized by a reallocation and move similar to how a vector is resized. A similar mechanism is used to add elements to the front via push_front. Popping elements might make a chunk unused at which point the chunk may be freed or returned to a chunk pool. 

It is worth noting that the deque can keep growing in size without having to reallocate and move any of the data elements. However, a drawback of the container is that to reference an element one first needs to reference its chunk and then the element within the chunk.

## The Performance of std::deque vs. std::vector
I compared the performance of std::deque to std::vector on Apple LLVM (clang) compiler version 10.0.1. The code was compiled with -O3 (default Xcode release flags). 

I'm specifically interested in the relative performance of adding to the end of the container and also the performance of iterating over elements. One can expect adding to and removing from the front of the container to be much faster for a deque than for a vector. The test does a push_back of 50 million random integers, then inserts 50 million random integers at the front, then sorts the container and finally iterates over and sets all the elements to a constant value. 

The total time these operations take are shown below:

|                    |  Vector    |  Deque   |
|--------------------|------------|----------|
| push_back (50M)    | 0.36s      | 0.33s    |
| insert front (50M) | $$\approx \infty$$ | 0.30s    |
| sort (100M)        | 8.80s      | 10.0s    |
| iterate (100M)     | 0.041s     | 0.063s   |
| pop_back (50M)     | 0.0        | 0.13s    |

Note that some of the time spent during push back and insert at front is probably spent in making the allocated memory pages available to the process. The vector space was reserved before doing the tests. Without reserving the vector space the push_back vector operations take about double the above time. The time taken to generate 50M random numbers is around 0.06 seconds.

The [source code](https://github.com/bduvenhage/Bits-O-Cpp/blob/master/containers/main.cpp) for the tests is available in my [Bits-O-Cpp GitHub repo](https://github.com/bduvenhage/Bits-O-Cpp).

## Summary
The push_back operations on a vector and deque require similar time if the vector space is already reserved. If the vector space is not already reserved then growing the vector would require it to be moved every time the vector runs out of space. As expected, inserting elements at the front of a deque is much quicker than inserting at the front of a vector.

Sorting a deque is almost 15% slower than sorting a vector. This is likely due to the additional dereference required when accessing a random element of a deque. Iterating over the deque is about 50% slower than interating over the vector. Popping from a vector just decrements the size variable and doesn't free any memory so it takes almost no time.

The std::deque is slower than a ring buffer based deque would be, but probably not by more than 50%. std::deque is likely a good implementation choice when the maximum size of the deque is unknown. In a future post I'll show what a ring buffer based deque looks like and how it performs relative to std::vector and std::deque.
