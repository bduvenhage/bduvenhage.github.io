---
layout: post
title:  "Making Fire and Water"
date: 2019-04-30
published: true
comments: true
categories: [memory_bank]
tags: [demoscene]
---

Future Crew's 1993 PC demo called [Second Reality](https://en.wikipedia.org/wiki/Second_Reality) had a Brobdingnagian impact on my interest in programming. Much of what I researched and played with at a young age was due to this demo. The demo won a couple of prizes and eventually made it onto [Slashdot's Top 10 Hacks of All Time](https://slashdot.org/story/99/12/13/0943241/slashdots-top-10-hacks-of-all-time). The [source code](https://github.com/mtuomi/SecondReality) which contains lots of PAS, POV and ASM files was released to celebrate the 20th anniversary of the demo. A [high quality video](https://www.youtube.com/watch?v=iw17c70uJes) of the demo is also available.

This post is my first "memory bank" post. In these posts I'll dig up some of my old code, make it run and preserve the code by committing it to github. In this post I'll revive some of my old code for making fire and water.

## Coding like its 1998.
[Mode 13h](https://en.wikipedia.org/wiki/Mode_13h) was a standard 320x200 VGA graphics mode with 256 colours. Mode 13h provided a linear 320x200 block of video memory at 0xA000:0000, where each byte represents one pixel. 

To set the RGB value of each of the 256 colours in the palette one would write the colour number first to the DAC Write Index register at 0x3C8 and then writing three 6-bit RGB components to the DAC Data register at 0x3C9 using the outp() function from conio.h in MSDOS. [Michael Abrash’s Graphics Programming Black Book](http://www.jagregory.com/abrash-black-book/) is a good reference for more info on how this all worked.

For interest sake, I looked up the compiler from the binaries I had compiled back then. The compiler was Borland Turbo C++ 3.2 Copyright 1991 Borland Intl. The IDE looked like:

<img src="/assets/images/turbo_cpp_3.2.jpg" width="400" />

To get the code running I used an SDL texture as a framebuffer and added some code to emulate a 256 colour palette. The [source code](https://github.com/bduvenhage/Bits-O-Cpp/tree/master/demoscene) is available in my [Bits-O-Cpp GitHub repo](https://github.com/bduvenhage/Bits-O-Cpp).


## Simple Fire 
Simple fire can be created by having a heat source with random noise that is continually convolved with an asymmetrical filter kernel. The asymmetrical filter creates a flow effect. On a modern processor this effect runs at almost a 1000 FPS.

<img src="/assets/images/demoscene_fire.gif" width="600" />

## Doom Fire 
Doom style fire can be created by having a heat source that is propagated up while being randomly extinguised and scattered. I generate one more random number than the [method recently documented by Fabien Sanglard](http://fabiensanglard.net/doom_fire_psx/), but it is almost as fast as the simple fire above.

<img src="/assets/images/demoscene_doom_fire.gif" width="600" />

## Water 
This water effect is quite cool. A water heightmap is maintained between two buffers. An [archived explanation](https://web.archive.org/web/20160418004149/http://freespace.virgin.net/hugo.elias/graphics/x_water.htm) contains more details on how this works. Here I render the water height directly, but one should really create a refractive offset into a texture map. A demo of this effect is available at http://www.onlinetutorialsweb.com/demo/javascript-water-ripple/ .

<img src="/assets/images/demoscene_water.gif" width="600" />


## Summary
The [source code](https://github.com/bduvenhage/Bits-O-Cpp/tree/master/demoscene) is available in my [Bits-O-Cpp GitHub repo](https://github.com/bduvenhage/Bits-O-Cpp). Thanks for the inspiration [Future Crew](https://en.wikipedia.org/wiki/Future_Crew).
