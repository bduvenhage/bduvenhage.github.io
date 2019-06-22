---
layout: post
title:  "The High Performance Time-Stamp Counter"
date: 2019-06-21
published: false
comments: true
categories: [performance]
tags: [performance, timer, time, tsc]
---

Your computer has a high performance Time-Stamp Counter (TSC) that increments at a similar rate as the CPU clock. On modern processors this counter increments at a constant rate and may be used as a wall clock timer. The benefit of the TSC compared to the Linux system timeofday function, for example, is that the TSC counter takes only a few clock cycles to read.

From Section 17.15 _Time Stamp Counter_ of the Intel 64 and IA-32 Architectures Software Developer’s Manual, Volume 3B: "Constant TSC behavior ensures that the duration of each clock tick is uniform and supports the use of the TSC as a wall clock timer even if the processor core changes frequency. This is the architectural behavior moving forward." 

So the architectural behaviour now and moving forward is that the TSC increments at a constant rate. This will be even though the actual CPU clock rate may drop to save power or increase during turbo boost. Note that on certain processors, the TSC frequency may not be the same as the frequency in the brand string.

On virtual hosts modern processors also got your back with two features called _TSC offsetting_ and _TSC scaling_. Virtualisation software can appropriately set the scale and offset of the TSC when read by guest software so that the guest wouldn't notice being migrated from one platform to another. Intel has a 'Timestamp-Counter Scaling for Virtualization White Paper' that you can read for more info. I'm not sure how widely modern processors has adopted scaling yet, but offsetting seems pretty standard.

## The RDTSC and RDTSCP instructions
The RDTSC and RDTSCP instructions can be used by user-mode / guest code to read the TSC. These instructions read the 64-bit TSC value into EDX:EAX. The high-order 32 bits of each of RAX and RDX are cleared. 

The RDTSC instruction can be called as shown below:
{% highlight c++ %}
  uint32_t countlo, counthi;

  __asm__ volatile ("rdtsc" : "=a" (countlo), "=d" (counthi));
  
  const uint64_t count = (uint64_t(counthi) << 32) | countlo;
{% endhighlight %}

The RDTSC instruction is not a serializing instruction so it does not wait until all previous instructions have been executed before reading the counter and subsequent instructions may begin execution before the read operation is performed. This means that without adding fences the measurement might be contaminated by instructions that come before and after RDTSC within the out of order window of the processor. This impacts on the measurement bias and variance by a few cycles. Adding fences would improve the accuracy of timing, but at the cost of some measurement overhead.

The RDTSCP instruction can be called as shown below:
{% highlight c++ %}
  uint32_t countlo, counthi;
  uint32_t chx; // Set to processor signature register - set to chip/socket & core ID by recent Linux kernels.

  __asm__ volatile("rdtscp" : "=a" (countlo), "=d" (counthi), "=c" (chx));

  const uint64_t count = (uint64_t(counthi) << 32) | countlo;
  const int chip = (chx & 0xFFF000)>>12;
  const int core = (chx & 0xFFF);
{% endhighlight %}

The RDTSCP instruction also returns the processor ID (chip and core) that the instruction was executed on. RDTSCP does wait until all previous instructions have executed and all previous loads are globally visible, but it still does not wait for previous stores to be globally visible and subsequent instructions may begin execution before the read operation is performed.

On a Macbook Intel Core i5-5287U 06_3D @ 2.90GHz the RDTSC and RDTSCP instructions seem to take around 23 cycles or 8 ns. RDTSCx are therefore fairly long latency instructions. 23 cycles is more than the 10 cycles reported in the Intel 64 and IA-32 Architectures Optimization Reference Manual, but the point is that RDTSCx (and any timing) instructions shouldn't be used in the inner loop of your code if timing overhead is a concern.

## Measuring Wall Clock Time
If the 

## Multi-Socket Behaviour
If the 

## Summary
...

The full [source](https://github.com/bduvenhage/Bits-O-Cpp/tree/master/time) with execution timing
is available in my [Bits-O-Cpp](https://github.com/bduvenhage/Bits-O-Cpp) GitHub repo.
