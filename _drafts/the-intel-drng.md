---
layout: post
title:  "The Intel DRNG"
<!-- date: 2019-04-01 -->
published: true
comments: true
categories: [rng]
tags: [Intel DRNG]
---

Intel® Secure Key, code-named Bull Mountain Technology, is the Intel name for the Intel® 64 and IA-32 Architectures instructions RDRAND and RDSEED and the underlying Digital Random Number Generator (DRNG) hardware implementation.

Solves the problem of having a fast source of entropy ...

From their [Software Implementation Guide](https://software.intel.com/en-us/articles/intel-digital-random-number-generator-drng-software-implementation-guide) ... among other things, the DRNG using the RDRAND instruction is useful for generating high-quality keys for cryptographic protocols, and the RSEED instruction is provided for seeding software-based pseudorandom number generators (PRNGs).

Section 2 gives an overview of Random Number Generators (RNGs)... For an overview of RNGs watch the [talk](http://www.pcg-random.org/posts/stanford-colloquium-talk.html) by Melissa O'Neill, or watch my [talk](https://www.youtube.com/watch?v=jWXZ07YBsPM&feature=youtu.be). I've found [Daniel Lemire's blog](https://lemire.me/blog/?s=random) to be an excellent resource on RNGs.

Section 3 gives an overview of how Intel DRNG works ...
Thermal noise is the fundamental source of entropy ...
https://en.wikipedia.org/wiki/Hardware_random_number_generator
https://en.wikipedia.org/wiki/Johnson–Nyquist_noise
A hardware CSPRNG (Cryptographically secure PRNG) that is based on AES in CTR mode and is compliant with SP800-90A. In SP800-90A terminology, this is referred to as a DRBG (Deterministic Random Bit Generator), a term used throughout the remainder of this document.
An ENRNG (Enhanced Non-deterministic Random Number Generator) that is compliant with SP800-90B and C.

Access to the DRNG is provided throught he new RdRand and RdSeed instructions ...
If the carry flag was not set it means a random number wasn't available ...
"On real-world systems, a single thread executing RDRAND continuously may see throughputs ranging from 70 to 200 MB/sec, depending on the SPU architecture."

support for RDRAND can be determined by examining bit 30 of the ECX register returned by CPUID, and support for RDSEED can be determined by examining bit 18 of the EBX register.

Essentially, developers invoke this instruction with a single operand: the destination register where the random value will be stored. Note that this register must be a general purpose register, and the size of the register (16, 32, or 64 bits) will determine the size of the random value returned.

After invoking the RDRAND instruction, the caller must examine the carry flag (CF) to determine whether a random value was available at the time the RDRAND instruction was executed. As Table 3 shows, a value of 1 indicates that a random value was available and placed in the destination register provided in the invocation. A value of 0 indicates that a random value was not available. In current architectures the destination register will also be zeroed as a side effect of this condition.

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
    do {
        asm volatile ("rdseed %0; setc %1"
                      : "=r" (rand), "=qm" (ok));
    } while (!ok);
    return rand;
}
{% endhighlight %}

{% highlight c++ %}
//! Lehmer RNG with 64bit multiplier, derived from https://github.com/lemire/testingRNG.
class TC_MCG_Lehmer_RandFunc32 {
public:
    TC_MCG_Lehmer_RandFunc32(const uint32_t seed = 0) {init(seed);}
    
    //!Calc LCG random number in [0,2^32)
    ALWAYS_INLINE uint32_t operator()() noexcept {// 1.0 ns on local.
        state_.s128_ *= UINT64_C(0xda942042e4dd58b5);
        return uint32_t(state_.s64_[1]);
    }
    
    void init(const uint32_t seed) {state_.s128_ = (__uint128_t(splitmix64_stateless(seed)) << 64) + splitmix64_stateless(seed + 1);}
    static constexpr double max_plus_one() noexcept {return 4294967296.0;} //0x1p32
    static constexpr double recip_max_plus_one() noexcept {return (1.0 / 4294967296.0);} //1.0/0x1p32
    static constexpr int num_bits() noexcept {return 32;}
    
private:
    union{__uint128_t s128_; uint64_t s64_[2];} state_; //Assumes little endian so that s64[0] is the low 64 bits of s128_.
};
{% endhighlight %}

{% highlight c++ %}
// 32-bit RNG using Intel's DRNG CPU instructions. Warning: It is slow! 100x slower than PCG!
class TC_IntelDRNG_RandFunc32 {
public:
    TC_IntelDRNG_RandFunc32(const uint32_t seed = 0) {init(seed);}
    
    //!Intel DRNG random number in [0,2^32)
    ALWAYS_INLINE uint32_t operator()() noexcept {//??ns on TC's EC2! 120ns on local!!!
        uint32_t rand;
        unsigned char ok;
        do {
            asm volatile ("rdrand %0; setc %1"
                          : "=r" (rand), "=qm" (ok));
        } while (!ok);
        return rand;
    }
    
    void init(const uint32_t seed) {} //No seeding required.
    
    static constexpr double max_plus_one() noexcept {return 4294967296.0;} //0x1p32
    static constexpr double recip_max_plus_one() noexcept {return (1.0 / 4294967296.0);} //1.0/0x1p32
    static constexpr int num_bits() noexcept {return 32;}
};
{% endhighlight %}


cpu_ticks_per_ns = 2.89991

lehmar64
ns_per_iteration = 0.90284
cpu_ticks_per_iteration = 2.61816
mbits_per_second = 70887.4

rdrand32
ns_per_iteration = 112.152
cpu_ticks_per_iteration = 325.231
mbits_per_second = 285.327

rdseed64
ns_per_iteration = 445.784
cpu_ticks_per_iteration = 1292.73
mbits_per_second = 143.5672
