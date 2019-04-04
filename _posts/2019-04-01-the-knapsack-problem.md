---
layout: post
title:  "The knapsack problem"
date:   2019-04-04
published: true
comments: true
categories: [algorithms, dynamic programming]
tags: [knapsack]
---

The knapsack problem comes up quite often and it is important to know how 
to solve it. For example, given a certain material budget and the cost 
vs. perceived value of building various edges in a potential road network, 
which edges should one build? The goal is to optimise the perceived value 
of the built roads within the fixed material budget. 

I recently encountered this problem within a TopCoder marathon
match. This post discusses two solutions and the code that I'll likely reuse for 
this problem in future.

<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-MML-AM_CHTML' async></script>

In terms of a knapsack, given a collection of $$n$$ objects (each with a weight 
$$w_i$$ and value $$v_i$$) as well as a knapsack that can carry a certain weight 
$$W$$, which objects would you choose to pack? The goal is to optimise the total 
value of the objects that one can fit into the weight budget of the knapsack.  

The below solutions are for the 0-1 knapsack problem which restricts the 
number of 'copies' of each of the n objects to zero or one. 

More formally, the goal is to maximize $$\sum _{i=1}^{n}v_{i}x_{i}$$ subject to $$\sum _{i=1}^{n}w_{i}x_{i}\leqslant W$$ and $$x_{i}\in \{0,1\}$$.

## A Greedy Approximate Solution
The 'greedy' approach to solving the problem is to repeatedly choose the best 
value per weight object until no more objects can be added to the knapsack.

For example, given three objects with weights $$w_1=3,\,w_2=4,\,w_3=2$$ and 
values $$v_1=8,\,v_2=12,\,v_3=5$$ and $$W = 5$$. The value per weight scores 
for these objects are $$s_1=2\frac{2}{3}$$, $$s_2=3$$, $$s_3=2\frac{1}{2}$$.  

For a greedy approach one would try to first 
choose object two, then object one and then object three. However, one can 
only fit object two ($$w_2=4$$) in the knapsack. Adding either object one or 
three ($$w_1=3,\,w_3=2$$) would make the knapsack heavier than $$W = 5$$. 
Therefore, the greedy solution is a knapsack value $$\{v_2\} = 12$$.

A greedy solution is however often not optimal. A better solution would be 
objects one and three with weight $$\{w_1=3,\,w_3=2\} = 5$$ and value 
$$\{v_1=8,\,v_3=5\} = 13$$. 

## A Dynamic Programming Solution
A solution that uses [dynamic programming](https://en.wikipedia.org/wiki/Dynamic_programming)
exists that can find the optimal solutions. Dynamic 
programming refers to simplifying a complicated problem by breaking it down 
into simpler sub-problems. Another 
well known example of such a solution is 
[Dijkstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm) 
for the shortest path problem.

For the 0-1 knapsack problem define $$m[n,W]$$ to be the maximum value that 
can be attained with weight less than or equal to $$W$$ using the set of $$n$$ 
objects.

We can define $$m[i,w]$$ recursively as follows:
- $$m[0,\,w] = 0$$,
- $$m[i,\,w] = m[i-1,\,w]$$ if $$w_{i} > w$$,
- $$m[i,\,w] = \max(m[i-1,\,w],\,m[i-1,w-w_{i}] + v_{i})$$ if $$w_{i} \leqslant w$$.

The C++ code for this would look like:
{% highlight c++ %} 
  for (int i=1; i <= n; ++i) {
      for (int w=0; w <= W_; ++w) {
          if (w[i-1] > w) {
              m[i][w] = m[i-1][w];
          }
          else {
              m[i][w] = std::max(m[i-1][w], m[i-1][w-w[i-1]] + v[i-1]);
          }
      }
  }
{% endhighlight %}

For the above three object example, $$m$$ ends up as a table of n+1 rows by W+1 columns:

|     | w=0 | w=1 | w=2 | w=3 | w=4 | w=5 |
|-----|-----|-----|-----|-----|-----|-----|
| i=0 |0|0|0|0|0|0|
| i=1 |0|0|0|8|8|8|
| i=2 |0|0|0|8|12|12|
| i=3 |0|0|5|8|12|13|

From the table one can see that $$m[3,5]$$ is a knapsack of value 13.

The maximum value of the objects that can be packed in the knapsack may then 
be found by calculating $$m[n,W]$$. A key part of the solution is to tabulate 
the intermediate results of $$m[i,w]$$.

The full [source](https://github.com/bduvenhage/Bits-O-Cpp/tree/master/knapsack) is 
available in my Bits-O-Cpp repo. I use this repo as a reference for myself, but I'll
try and add more examples of how anyone can use those bits of sweet C++ code.
  
