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

I've been considering implementing a ring buffer STL containor or container adaptor for doing breadth first search faster than would perhaps be possible with an STL deque. However, an STL deque is a much cooler (and faster) container than I initially thought. The deque stores its elements in cache friendly chunks and still allows constant time random access (although with one more dereference than required with a vector). 

Given its design, the std::deque has the potential to be very efficient. This post discusses std::deque's internal workings and compares its performance to that of std::vector.

## The STL Deque Implementation
Post is in progress ...

## The Performance of std::deque vs. std::vector
I compared the performance of std::deque to std::vector on Apple LLVM (clang) compiler version 10.0.1. The code was compiled with -O3 (default Xcode release flags). 

I'm specifically interested in the relative performance of adding to the end and iterating over elements. One can expect adding to and removing from the front of the container to be much faster for a deque. The test does a push_back of 50 million random integers, then inserts 50 million random integers at the front, then sorts the container and finally iterates over and sets all the elements to a constant value. The total time these operations take are shown below:

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
