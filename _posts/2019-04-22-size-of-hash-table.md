---
layout: post
title:  "The size of the C++ std::unordered_set / Hash-Table"
date: 2019-04-22
published: false
comments: true
categories: [performance]
tags: [unordered_set, hash table]
---

T

## The STL Deque Implementation
T

## The Performance of std::deque vs. std::vector
T 

|                    |  Vector    |  Deque   |
|--------------------|------------|----------|
| push_back (50M)    | 0.36s      | 0.33s    |
| insert front (50M) | $$\approx \infty$$ | 0.30s    |
| sort (100M)        | 8.80s      | 10.0s    |
| iterate (100M)     | 0.041s     | 0.063s   |
| pop_back (50M)     | 0.013s     | 0.13s    |


The [source code](https://github.com/bduvenhage/Bits-O-Cpp/blob/master/containers/main_vector_vs_deque.cpp) for the tests is available in my [Bits-O-Cpp GitHub repo](https://github.com/bduvenhage/Bits-O-Cpp).

## Summary
T
