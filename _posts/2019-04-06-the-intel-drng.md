---
layout: post
title:  "The Intel DRNG"
date: 2019-04-06
published: true
comments: true
categories: [rng]
tags: [Intel, DRNG]
---

I recently got around to testing Intel's Secure Key Digital Random Number Generator (DRNG). Intel Secure Key (code-named Bull Mountain Technology) is the name for the Intel® 64 and IA-32 Architectures instructions RDRAND and RDSEED and the underlying hardware implementation. 

The RDRAND and RDSEED instructions address the need for a fast source of entropy. From Intel's [Software Implementation Guide](https://software.intel.com/en-us/articles/intel-digital-random-number-generator-drng-software-implementation-guide): "The DRNG using the RDRAND instruction is useful for generating high-quality keys for cryptographic protocols, and the RDSEED instruction is provided for seeding software-based pseudorandom number generators (PRNGs)."

Section 2 of the software implementation guide gives an overview of Random Number Generators (RNGs). For another overview of RNGs watch the excellent [talk by Melissa O'Neill](http://www.pcg-random.org/posts/stanford-colloquium-talk.html), or watch [my talk](https://www.youtube.com/watch?v=jWXZ07YBsPM&feature=youtu.be). I've also found [Daniel Lemire's blog](https://lemire.me/blog/?s=random) to be an excellent resource on implementing RNGs.

Section 3 of the software implementation guide gives an overview of how Intel's DRNG works. Thermal noise is the fundamental source of entropy. A hardware CSPRNG (Cryptographically secure PRNG) digital random bit generator feeds the RDRAND instructions over all cores while an ENRNG (Enhanced Non-deterministic Random Number Generator) feeds the RDSEED instructions over all cores. The RDRAND DRNG is continuously reseeded from the hardware entropy source while the RDSEED generator makes conditioned entropy samples directly available. 

## Determining Support for Intel's DRNG
Support for RDRAND can be determined by examining bit 30 of the ECX register returned by CPUID, and support for RDSEED can be determined by examining bit 18 of the EBX register.

{% highlight c++ %}
void get_cpuid(cpuid_t *info, unsigned int leaf, unsigned int subleaf) {
    asm volatile("cpuid"
                 : "=a" (info->eax), "=b" (info->ebx), "=c" (info->ecx), "=d" (info->edx)
                 : "a" (leaf), "c" (subleaf));
}

bool is_intel_cpu() {
    cpuid_t info;
    get_cpuid(&info, 0, 0);

    if (memcmp((char *) &info.ebx, "Genu", 4) ||
        memcmp((char *) &info.edx, "ineI", 4) ||
        memcmp((char *) &info.ecx, "ntel", 4)) {
        return false;
    } else {
        return true;
    }
}

bool is_drng_supported() {
    bool rdrand_supported = false;
    bool rdseed_supported = false;

    if (is_intel_cpu()) {
        cpuid_t info;
        get_cpuid(&info, 1, 0);

        if ((info.ecx & 0x40000000) == 0x40000000) {
            rdrand_supported = true;
        }

        get_cpuid(&info, 7, 0);

        if ( (info.ebx & 0x40000) == 0x40000 ) {
            rdseed_supported = true;
        }
    }

    return rdrand_supported & rdseed_supported;
}
{% endhighlight %}

## Using RDRAND and RDSEED
The RDRAND and RDSEED instructions may be called as shown below. The size of the operand register determines whether 16-, 32- or 64-bit random numbers are returned. If the carry flag is zero after a DRNG instruction it means a random number wasn't available yet and the software should retry if a random number is still required. 

Similar to a splitmix64_stateless generator (shown for reference), the rdseed64 generator shown below may be used to seed RNGs:
{% highlight c++ %}
//! Stateless [0,2^64) splitmix64 by Daniel Lemire https://github.com/lemire/testingRNG . Useful for seeding RNGs.
ALWAYS_INLINE uint64_t splitmix64_stateless(const uint64_t index) {// 1.3 ns on local.
    uint64_t z = index + UINT64_C(0x9E3779B97F4A7C15);
    z = (z ^ (z >> 30)) * UINT64_C(0xBF58476D1CE4E5B9);
    z = (z ^ (z >> 27)) * UINT64_C(0x94D049BB133111EB);
    return z ^ (z >> 31);
}

//! 64-bit Intel RDSEED. Useful for seeding RNGs.
ALWAYS_INLINE uint64_t rdseed64() { // 450 ns on local.
    uint64_t rand;
    unsigned char ok;
    
    asm volatile ("rdseed %0; setc %1"
                  : "=r" (rand), "=qm" (ok));
    
    while (!ok) {
        asm volatile ("pause" : );
        asm volatile ("rdseed %0; setc %1"
                      : "=r" (rand), "=qm" (ok));
    }
    return rand;
}
{% endhighlight %}

The assembler for the above rdseed64 generator would look similar to the below snippet. Notice the 'pause' instruction which is recommended by Intel so that the core can perhaps still do other work while waiting for a random number.

{% highlight nasm %}
rdseed64():
      jmp .L6
  .L3:
      pause
  .L6:
      rdseed rax; setc dl
      test dl, dl
      je .L3
  ret
{% endhighlight %}

Below is a Lehmer RNG class (shown for reference) and an Intel 32-bit DRNG class using RDRAND:
{% highlight c++ %}
//! Lehmer RNG with 64bit multiplier, derived from https://github.com/lemire/testingRNG.
class TC_MCG_Lehmer_RandFunc32 {
public:
    TC_MCG_Lehmer_RandFunc32(const uint32_t seed = 0) {init(seed);}
    
    //!Calc LCG random number in [0,2^32)
    ALWAYS_INLINE uint32_t operator()() {// 1.0 ns on local.
        state_.s128_ *= UINT64_C(0xda942042e4dd58b5);
        return uint32_t(state_.s64_[1]);
    }
    
    void init(const uint32_t seed) {state_.s128_ = (__uint128_t(splitmix64_stateless(seed)) << 64) + splitmix64_stateless(seed + 1);}
    static constexpr double max_plus_one() {return 4294967296.0;} //0x1p32
    static constexpr double recip_max_plus_one() {return (1.0 / 4294967296.0);} //1.0/0x1p32
    static constexpr int num_bits() {return 32;}
    
private:
    union{__uint128_t s128_; uint64_t s64_[2];} state_; //Assumes little endian so that s64[0] is the low 64 bits of s128_.
};

// 32-bit RNG using Intel's DRNG CPU instructions. Warning: It is slow! 100x slower than PCG!
class TC_IntelDRNG_RandFunc32 {
public:
    TC_IntelDRNG_RandFunc32(const uint32_t seed = 0) {init(seed);}
    
    //!Intel DRNG random number in [0,2^32)
    ALWAYS_INLINE uint32_t operator()() {//??ns on TC's EC2! 120ns on local!!!
        uint32_t rand;
        unsigned char ok;
        do {
            asm volatile ("rdrand %0; setc %1"
                          : "=r" (rand), "=qm" (ok));
        } while (!ok);
        return rand;
    }
    
    void init(const uint32_t seed) {} //No seeding required.
    
    static constexpr double max_plus_one() {return 4294967296.0;} //0x1p32
    static constexpr double recip_max_plus_one() {return (1.0 / 4294967296.0);} //1.0/0x1p32
    static constexpr int num_bits() {return 32;}
};
{% endhighlight %}

## Performance Results:
In the Intel software implementation guide it is stated that "On real-world systems, a single thread executing RDRAND continuously may see throughputs ranging from 70 to 200 MB/sec, depending on the SPU architecture." 

I also ran some performance measurements on my laptop (which is a 2.9 GHz Intel Core i5 and has cpu_ticks_per_ns = 2.89991):

{% highlight c++ %}
int main() {
    const uint32_t rng_seed_ = 0;
    TC_MCG_Lehmer_RandFunc32 lehmer_rng(rng_seed_);
    
    //Should check is_intel_cpu()
    //Should check is_drng_supported()
    TC_IntelDRNG_RandFunc32 intel_rng_(rng_seed_);

    std::cout << "Generating some random numbers...";    
    TCTimer::init_timer(2.89992e+09); // The param is the initial guess of your CPU's clock rate in GHz.
    
    const uint64_t num_iterations = uint64_t(1) << 27;
    uint32_t ri = 0;    
    const double start_time = TCTimer::get_time();

    for (uint64_t i=0; i <= num_iterations; ++i) {
        //ri += lehmer_rng();
        ri += intel_rng_();
        //ri += rdseed64();
    }
    
    const double end_time = TCTimer::sync_tsc_time(); // Same as get_time(), but also estimates CPU's seconds_per_tick_!    
    std::cout << "done.\n";
    std::cout << ri << "\n"; // Print the sum so that the RNG doesn't get optimised out.
    
    const double cpu_ticks_per_ns = TCTimer::get_clock_freq() * 0.000000001;
    const double ns_per_iteration = (end_time-start_time) / num_iterations * 1000000000.0;
    const double cpu_ticks_per_iteration = cpu_ticks_per_ns * ns_per_iteration;    
    const double millions_numbers_per_second = num_iterations / (end_time-start_time) * 0.000001;   

    return 0;
}
{% endhighlight %}


Result for lehmar64:
```
ns_per_number = 0.90284
cpu_ticks_per_number = 2.61816
mbits_per_second = 70887.4
```

Result for rdrand32:
```
ns_per_number = 112.152
cpu_ticks_per_number = 325.231
mbits_per_second = 285.327
```

Result for rdseed64
```
ns_per_number = 445.784
cpu_ticks_per_number = 1292.73
mbits_per_second = 143.5672
```

RDRAND and RDSEED is slower than, for example, the Lehmar generator. However, it provides cryptographically secure hardware entropy based random numbers significantly faster than seems to be otherwise possible.

## The Code
The full code is available in my [Bits-O-Cpp GitHub repo](https://github.com/bduvenhage/Bits-O-Cpp/tree/master/random). That code uses some headers for timing and platform info from the repo, but the Bits-O-Cpp/random/README.md file contains info on how to compile the example.
