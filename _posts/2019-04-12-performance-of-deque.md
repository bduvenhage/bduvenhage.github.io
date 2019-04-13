---
layout: post
title:  "The Performance of C++ std::deque vs. std::vector"
date: 2019-04-12
published: true
comments: true
categories: [performance]
tags: [deque, vector]
---

Post is in progress ...

<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-MML-AM_CHTML' async></script>

Many algorithms make use of a double ended queue (a.k.a. a deque). For example, in breadth first search a deque is used to hold the search frontier. ...

Having a fast implementation of a deque is very useful. If the size of the deque is bounded then one can implement it as a ring buffer over a pre-allocated array.

I've been considering implementing a ring buffer STL containor or container adaptor for doing breadth first search faster than would perhaps be possible with an STL deque.

However, an STL deque is a much cooler (and faster) container than I initially thought. The deque stores its elements in cache friendly chunks and still allows constant time random access (although with one more dereference than required by a vector). Given its design, a deque has to potential to be very efficient.

## The STL Deque Implementation

## The Performance of std::deque vs. std::vector
I compared the performance of std::deque to std::vector on Apple LLVM (clang) compiler version 10.0.1. I measured the performance of adding elements to the end of the container, adding to the front and iterating over the entire container. I expect adding to and removing from the front of the container to be MUCH faster for a deque than for a vector. I'm more interested in the relative performance of adding to the end and iterating over elements.

The test does a push_back of 50M elements, insert at front of 50M elements and then iterates over all elements for deque and for vector. The code is compiled with -O3 (default Xcode release flags). The total time these operations take are shown below:

|                    |  Vector  |  Deque   |
|--------------------|----------|----------|
| push_back (50M)    | 0.36s    | 0.33s    |
| insert front (50M) | &&\inf&& | 0.30s    |
| sort (100M)        | 8.80s    | 10.0s    |
| iterate (100M)     | 0.041s   | 0.063s   |
| pop_back (50M)     | 0.0      | 0.13s    |

Some of the push back/front time is actually spent in making the allocated mem pages available to the process. Note also that the vector space was reserved before doing the tests. Without reserve the push_back vector operations take about double the above time.

The testing code is available at ...

## Summary
The push_back operations on the vector and deque take very similar time given that the vector space is already reserved. If the vector space was not already reserved then growing the vector would require elements to be moved every time the vector runs out of space. Inserting elements at the front is much quicker with a deque as expected.
