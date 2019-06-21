---
layout: post
title:  "The High Performance Time-Stamp Counter"
date: 2019-06-21
published: false
comments: true
categories: [performance]
tags: [performance, timer, time, tsc]
---

Your computer has a high performance Time-Stamp Counter (TSC) that increments at the same rate as your CPU clock. On modern processors this counter increments at a constant rate and may be used as a wall clock timer. The benefit of the TSC compared to the Linux system time timeofday function, for example, is that the TSC counter takes only a few clock cycles to read.

From Section 17.15 _Time Stamp Counter_ of the Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 3B, "Constant TSC behavior ensures that the duration of each clock tick is uniform and supports the use of the TSC as a wall clock timer even if the processor core changes frequency. This is the architectural behavior moving forward."

So the architectural behaviour now and moving forward is that the TSC increments at a constant rate even though the actual CPU clock may drop to save power or increase during turbo boost.

On virtual hosts modern processors also got your back with two features called _TSC offsetting_ and _TSC scaling_. VM software can appropriately set the scale and offset of the TSC when read by guest software so that the guest doesn't notice being migrated from one platform to another. Intel has a 'Timestamp-Counter Scaling for Virtualization White Paper' that you can read for more info. I'm not sure how widely modern processors has adopted scaling yet, but offsetting seems pretty standard.

## The RDTSC and RDTSCP instructions
The RDTSC and RDTSCP instructions can be used to read the TSC. The instructions read the 64-bit TSC value into EDX:EAX. The high-order 32 bits of each of RAX and RDX are cleared. 

{% highlight c++ %}
  uint32_t countlo, counthi;

  __asm__ volatile ("rdtsc" : "=a" (countlo), "=d" (counthi));
  
  const uint64_t count = (uint64_t(counthi) << 32) | countlo;
{% endhighlight %}

The RDTSC instruction is not a serializing instruction so it does not necessarily wait until all previous instructions have been executed before reading the counter and subsequent instructions may begin execution before the read operation is performed.

{% highlight c++ %}
  uint32_t countlo, counthi;
  uint32_t chx;

  __asm__ volatile("rdtscp" : "=a" (countlo), "=d" (counthi), "=c" (chx));

  const uint64_t count = (uint64_t(counthi) << 32) | countlo;
  const int chip = (chx & 0xFFF000)>>12;
  const int core = (chx & 0xFFF);
{% endhighlight %}

The RDTSCP instruction also returns the processor ID (chip and core) that the instruction was executed on. RDTSCP is not a serializing instruction, but it does wait until all previous instructions have executed and all previous loads are globally visible. It still does not wait for previous stores to be globally visible, and subsequent instructions may begin execution before the read operation is performed.

On my Macbook Intel Core i5-5287U @ 2.90GHz the RDTSC and RDTSCP instructions to read the 64-bit TSC counter seem to take around 20 cycles or 7 ns. I'm not sure why the instruction takes 20 cycles.

## Measuring Wall Clock Time

## Summary
...

The full [source](https://github.com/bduvenhage/Bits-O-Cpp/tree/master/time) with execution timing
is available in my [Bits-O-Cpp](https://github.com/bduvenhage/Bits-O-Cpp) GitHub repo.
