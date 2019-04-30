---
layout: post
title:  "Making Fire and Water"
date: 2019-04-30
published: false
comments: true
categories: [memory_bank]
tags: [demoscene]
---

Future Crew's 1993 PC demo called [Second Reality](https://en.wikipedia.org/wiki/Second_Reality) had a Brobdingnagian impact on my interest in programming. Much of what I researched and played with at a young age was due to this demo. The demo won a couple of prises and eventually made it onto [Slashdot's Top 10 Hacks of All Time](https://slashdot.org/story/99/12/13/0943241/slashdots-top-10-hacks-of-all-time). The [source code](https://github.com/mtuomi/SecondReality) which contains lots of PAS, POV and ASM files was released to celebrate the 20th anniversary of the demo. A [high quality video](https://www.youtube.com/watch?v=iw17c70uJes) of the demo is also available.

This post is my first "memory bank" post. In these posts I'll dig up some of my old code, make it run and commit it to github.


## Coding like its 1998.
Mode 13h - 320x200 VGA graphics with 256 colours.
DOS mode and farcalloc to get access to more RAM outside of the local 64k page on the 'far' heap. I was still using Borland C. Borland C++ - Copyright 1991 Borland Intl. according to the generated binaries.

## Simple Fire 
Heat source with random noise that is continually convolved with an asymmetrical kernel ...

<img src="/assets/images/demoscene_fire.gif" width="600" />

## Doom Fire 
Heat source that is propagated up while randomly extinguised and scattered ...

<img src="/assets/images/demoscene_doom_fire.gif" width="600" />

## Water 
A water heightmap is maintained between two buffers. I'm not sure where I got the original algorithm from ...

<img src="/assets/images/demoscene_water.gif" width="600" />


## Summary
T.
