---
layout: post
title:  "Generating Equidistant Vectors"
date: 2019-07-31
published: false
comments: true
categories: [geometry]
tags: [geometry, computer graphics]
---

In this post I'll revive work I did during my PhD to generate 3D vectors that are equally spaced (i.e equidistant) on the unit sphere. Equidistant vectors are useful for many operations over the sphere, to precalculate unit vectors and to tesselate a sphere. It was originally implemented for a paper on [Numerical Verification of Bidirectional Reflectance Distribution Functions for Physical Plausibility](https://dl.acm.org/citation.cfm?id=2513499). The PDF of a pre-print of the paper is also available from [ResearchGate](https://www.researchgate.net/publication/259885429_Numerical_Verification_of_Bidirectional_Reflectance_Distribution_Functions_for_Physical_Plausibility) and via my [Google Scholar page](https://scholar.google.com/citations?user=jqhH0o4AAAAJ).

I'll show two methods for generating equidistant vectors. One using the golden section spiral a.k.a. the Fibonacci spiral sphere and another using a sub-division of an icosahedron. The golden section spiral has the benefit that one can exactly specify the desired number of vectors while the sub-division method has the benefit of approximately circular (pentagon and hexagon) shaped bins.

## The Fibonacci Spiral Sphere
<img src="/assets/images/fibogeodual.jpg" width="640" />

## Sub-division of an Icosahedron
<img src="/assets/images/icosageodual.jpg" width="640" />

## Symmary
...
