---
layout: post
title:  "The knapsack problem"
date:   2019-04-02
comments: true
categories: algorithms, dynamic programming
---

The knapsack problem comes up quite often and it is important to know how 
to solve it. For example, "Given a certain material budget and the cost 
vs. perceived value of building various edges in a potential road network, 
which edges should one build?". The goal is to optimise the perceived value 
of the built roads within the fixed material budget. 

<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-MML-AM_CHTML' async></script>

Given a collection of $$n$$ objects (each with a weight 
$$w_i$$ and value $$v_i$$) as well as a knapsack that can carry a certain weight 
$$W$$, which objects would you choose to pack? The goal is to optimise the total 
value of the objects that one can fit into the weight budget of the knapsack. 

I recently encountered the above problem within a TopCoder marathon
match. Below is my solutions and the code that I'll reuse for this problem
in future. 

The below solutions are for the 0-1 knapsack problem which restricts the 
number of 'copies' of each of the n objects to zero or one. 

More formally, the goal is to maximize $$\sum _{i=1}^{n}v_{i}x_{i}$$ subject to $$\sum _{i=1}^{n}w_{i}x_{i}\leqslant W$$ and $$x_{i}\in \{0,1\}$$.

## A Greedy Approximate Solution
The 'greedy' approach to solving the problem is to repeatedly choose the best 
value per weight object until no more objects can be added to the knapsack.

For example, given three objects with weights $$w_1=3,\,w_2=4,\,w_3=2$$ and 
values $$v_1=8,\,v_2=12,\,v_3=5$$ and $$W = 5$$. The value per weight scores 
for these objects are $$s_1=2\frac{2}{3}$$, $$s_2=3$$, $$s_2=2\frac{1}{2}$$.  

For the greedy approach one would therefore based on the scores try to first 
choose object two, then object one and then object three. However, one can 
only fit object two ($$w_2=4$$) in the knapsack. Adding either object one or 
three ($$w_1=3,\,w_3=2$$) would make the knapsack heavier than $$W = 5$$. 
Therefore, the greedy solution a knapsack value $$\{v_2\} = 12$$ 

The greedy solution is however often not optimal. In this example a better
solution would be objects one and three with weight $$\{w_1=3,\,w_3=2\} = 5$$ and 
value $$\{v_1=8,\,v_3=5\} = 13$$. 

## A Dynamic Programming Solution
A solution that uses [dynamic programming](https://en.wikipedia.org/wiki/Dynamic_programming)
exists that can find potentially higher valued optimal solutions. Dynamic 
programming refers to simplifying a complicated problem by breaking it down 
into simpler sub-problems. Another 
well known example of such a solution is 
[Dijkstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm) 
for the shortest path problem.

For the 0-1 knapsack problem define $$m[i,w]$$ to be the maximum value that 
can be attained with weight less than or equal to $$w$$ using objects up 
to $$i$$.

We can define $$m[i,w]$$ recursively as follows:
- $$m[0,\,w] = 0$$,
- $$m[i,\,w] = m[i-1,\,w]$$ if $$w_{i} > w$$,
- $$m[i,\,w] = \max(m[i-1,\,w],\,m[i-1,w-w_{i}] + v_{i})$$ if $$w_{i} \leqslant w$$.

The maximum value of the objects that can be packed in the knapsack may then 
be found by calculating $$m[n,W]$$.

This can be implemented ...


  
 

