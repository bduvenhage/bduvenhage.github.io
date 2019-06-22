---
layout: post
title:  "The High Performance Time-Stamp Counter"
date: 2019-06-22
published: true
comments: true
categories: [performance]
tags: [performance, timer, time, tsc]
---

Your computer has a high performance Time-Stamp Counter (TSC) that increments at a rate similar to the CPU clock. On modern processors this counter increments at a constant rate and may be used as a wall clock timer. The benefit of the TSC compared to the Linux system timeofday function, for example, is that the TSC counter takes only a few clock cycles to read.

From Section 17.15 _Time Stamp Counter_ of the 'Intel 64 and IA-32 Architectures Software Developer’s Manual, Volume 3B': "Constant TSC behaviour ensures that the duration of each clock tick is uniform and supports the use of the TSC as a wall clock timer even if the processor core changes frequency. This is the architectural behaviour moving forward." 

So the architectural behaviour now and moving forward is that the TSC increments at a constant rate. This will true even though the actual CPU clock rate may drop to save power or increase during turbo boost. Note that on certain processors, the TSC frequency may not be the same as the frequency in the brand string.

On virtual hosts modern processors also got your back with two features called _TSC offsetting_ and _TSC scaling_. Virtualisation software can appropriately set the scale and offset of the TSC when read by guest software so that the guest wouldn't notice being migrated from one platform to another. Intel has a 'Timestamp-Counter Scaling for Virtualization White Paper' that you can read for more info. I'm not sure how widely modern processors has adopted scaling yet, but offsetting seems pretty standard.

## The RDTSC and RDTSCP instructions
The RDTSC and RDTSCP instructions can be used by user-mode / guest code to read the TSC. These instructions read the 64-bit TSC value into EDX:EAX. The high-order 32 bits of each of RAX and RDX are cleared. 

The RDTSC instruction can be called as shown below:
{% highlight c++ %}
  ALWAYS_INLINE uint64_t _get_tsc_ticks_since_reset() {
    uint32_t countlo, counthi;

    __asm__ volatile ("RDTSC" : "=a" (countlo), "=d" (counthi));
    return (uint64_t(counthi) << 32) | countlo;
  }
{% endhighlight %}

The RDTSC instruction is not a serializing instruction so it does not wait until all previous instructions have been executed before reading the counter and subsequent instructions may begin execution before the read operation is performed. This means that without adding fences the measurement might be contaminated by instructions that come before and after RDTSC within the out of order window of the processor. This might impact on the measurement bias and variance by a few cycles. Adding fences would improve the accuracy of timing, but at the cost of some measurement overhead.

The RDTSCP instruction can be called as shown below:
{% highlight c++ %}
  ALWAYS_INLINE uint64_t _get_tsc_ticks_since_reset_p(int &chip, int &core) {
    uint32_t countlo, counthi;
    uint32_t chx; // Set to processor signature register - set to chip/socket & core ID by recent Linux kernels.

    __asm__ volatile("RDTSCP" : "=a" (countlo), "=d" (counthi), "=c" (chx));
    chip = (chx & 0xFFF000)>>12;
    core = (chx & 0xFFF);
    return (uint64_t(counthi) << 32) | countlo;
  }
{% endhighlight %}

The RDTSCP instruction also returns the processor ID (chip and core) that the instruction was executed on. RDTSCP does wait until all previous instructions have executed and all previous loads are globally visible, but it still does not wait for previous stores to be globally visible and subsequent instructions may begin execution before the read operation is performed.

On a Macbook Intel Core i5-5287U 06_3D @ 2.90GHz the RDTSC and RDTSCP instructions seem to take around 23 cycles or 8 ns. RDTSCx are therefore fairly long latency instructions. The 'Intel 64 and IA-32 Architectures Optimization Reference Manual' reports an instruction throughput of 10 for DisplayFamily_Display_Model 06_3D which is what one can probably expect if the timing instructions are spaced further apart than in the testing code I used. The point is that RDTSCx (and any timing) instructions shouldn't be used in the inner loop of your code if timing overhead is a concern.

## Measuring Wall Clock Time
Given that the TSC frequency is invariant, if the frequency of the counter is known then it can be used to measure wall clock time in seconds.

If `init_tick` is the reference 'zero' tick and `seconds_per_tick_` is the timer period (reciprocal of the frequency) then the wall clock time in seconds since a reference init time is returned by:
{% highlight c++ %}
  ALWAYS_INLINE double get_tsc_time() {
    return (_get_tsc_ticks_since_reset() - init_tick_) * seconds_per_tick_;
  }
{% endhighlight %}

As mentioned, the TSC frequency can be different from the clock frequency in the CPUs brand string returned by the CPUID instruction. One solution to finding the TSC frequency is to count the change in the TSC between two known times and then calculate the TSC's `seconds_per_tick`.  

If `init_time_` is the known reference time and `_get_tod_seconds()` returns the wall clock time using timeofday, for example, then `seconds_per_tick` may be updated with: 
{% highlight c++ %}
  void sync_tsc_time() {
      const double dTime = _get_tod_seconds() - init_time_;
      const uint64_t dTicks = _get_tsc_ticks_since_reset_p(chip_, core_) - init_tick_;
      seconds_per_tick_ = dTime / dTicks;
  }
{% endhighlight %}

I usually init my timer then do the setup of my app and by the time I need to start using the timer enough time has passed to get an accurate estimate of `seconds_per_tick_`.

## Multi-Socket Behaviour
On modern platforms the TSC is synchronised between cores of the same socket and is reset with the processor reset signal. The processor reset signal is, similar to the reference clock, synchronised between multiple processor sockets on the same motherboard. It is therefore reasonable to assume that the TSC is at least approximately synchronised between sockets.

The code I've implemented assumes that the TSC is synchronised over sockets, but I don't have enough experience with multi-socket systems to confirm this behaviour. I'll update this post if I find that the TSC is not adequately synchronised across sockets.

To accommodate TSC skew between sockets, the code could be adapted to maintain init_tick_ values and perhaps also seconds_per_tick_ values for each socket using a list or an unordered map indexed by chip/socket.

## Summary
The TSC can be used as a high performance timer. Moving forward, the architectural behaviour of the TSC is to be invariant and available to do wall clock time measurements. This is also true for guest code running on virtualisation software. 

Al alternative to directly using the TSC is to use C++'s chrono timer with the high resolution option ... busy with an example for this...

The full [source](https://github.com/bduvenhage/Bits-O-Cpp/tree/master/time) with execution timing
is available in my [Bits-O-Cpp](https://github.com/bduvenhage/Bits-O-Cpp) GitHub repo.
