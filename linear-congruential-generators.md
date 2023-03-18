# Linear congruential generators

Linear congruential generators (*LCG*s) are the fastest and simplest type of pseudorandom number generator (PRNG). They have somewhat of a bad name, because there have been some implementations that used bad parameters. However, with properly chosen parameters, LCGs should not have any statistical problems. However, it should be noted that LCGs are not a cryptographically secure pseudorandom number generator, and are predictable. This means that there are ways to predict the rest of a pseudorandom sequence $x_{n + 1}, x_{n}$ given the first $n$ terms $x_1, x_2, ..., x_n$.


## Implementation

An LCG uses the recursion
$$ x_{n + 1} = (ax_n + c) \mod{m} $$

to generate a sequence $x_1, x_2, ...$ of pseudorandom numbers. The parameters consist of
  - The *seed* $x_0$
  - The *multiplier* $a$
  - The *increment* $c$
  - The *modulus* $m$

Typically, $m$ is a power of two (ofen $2^{32}$, $2^{64}$, $2^{96}$, or $2^{128}$). In this case, the $k$ least significant bits have a period of $2^k$. For this reason, the lower (least significant) bits of the terms $x_1, x_2, ...$ should be discarded, depending on how large a period is acceptable. This is sometimes called a *truncated* LCG.

When $c = 0$ the generator is called a *multiplicative congruential generator* or a *Lehmer generator*. When $c \neq 0$ we call the generator a *mixed congruential generator*. Sometimes, the term LCG refers specifically to a mixed congruential generator and the abbreviation MCG is used to refer to a multiplicative congruential generator. I will follow this convention from now on as well.

Both for LCGs and MCGs, it is extremely important to pick good parameters, since the properties of the pseudorandom sequence depends on it.


## MCGs

MCGs with a modulus that is a power of two should have an odd seed. According to [1], the following multipliers are "good":

For $m = 2^{32}$ (note that this is not enough to pass most statistical tests):
| Bits | Multiplier |
|------|------------|
| 16   | 0x72ed     |
| 32   | 0x93d765dd |

For $m = 2^{64}$ (note that this is not enough to pass most statistical tests):
| Bits | Multiplier         |
|------|--------------------|
| 32   | 0xe817fb2d         |
| 64   | 0xf1357aea2e62a9c5 |

For $m = 2^{128}$:
| Bits | Multiplier                         |
|------|------------------------------------|
| 64   | 0xdefba91144f2b375                 |
| 128  | 0xaadec8c3186345282b4e141f3a1232d5 |

Some semi-interesting facts about MCGs with a modulus that is a power of two:
  - MCGs have a maximum possible period of $\frac{m}{4}$
  - The bit in position $k$ of an MCG has period $2^k$ (so the least significant bit is constant, bit 1 is alternating, etc.)


### Example implementation

This is a simple header-only C++ implementation of an MCG using the 64-bit constant for $m = 2^{128}$.

```
#include <cstdint>

class MCG {
public:
	MCG(unsigned __int128 seed) : mState(seed | 1) {
		// when the seed is zero, the first term in the sequence will be zero
		// (this is only the case when the multiplier is 64-bit)
		next();
	}

	uint64_t next() {
		mState = mState * multiplier;
		return mState >> 64;
	}

private:
	static const uint64_t multiplier = 0xdefba91144f2b375;
	unsigned __int128 mState;
};
```


## LCGs

For LCGs, any seed can be used. The increment can be any odd number. Note that when the multiplier is smaller than the modulus (e.g. half the bit size of the modulus), the first term in the sequence is zero whenever the seed is one. To avoid this, we can skip the first term of the sequence.

According to [1], the following multipliers are "good":

For $m = 2^{32}$ (note that this is not enough to pass most statistical tests):
| Bits | Multiplier |
|------|------------|
| 16   | 0xd9f5     |
| 32   | 0x915f77f5 |

For $m = 2^{64}$ (note that this is not enough to pass most statistical tests):
| Bits | Multiplier         |
|------|--------------------|
| 32   | 0xf9b25d65         |
| 64   | 0xd1342543de82ef95 |

For $m = 2^{128}$:
| Bits | Multiplier                         |
|------|------------------------------------|
| 64   | 0xfc0072fa0b15f4fd                 |
| 128  | 0xdb36357734e34abb0050d0761fcdfc15 |

An LCG generates a sequence where the bits up to position $k$ have a period of $2^k$.


### Example implementation

This is a simple header-only C++ implementation of an LCG using the 64-bit constant for $m = 2^{128}$. The multiplier is also used as the increment. Note that an 128-bit unsigned integer is used, which requires a GCC-specific compiler extension. So the code is not portable (though it should be easy-to-port to any other compiler with support for 128-bit integers).

```
#include <cstdint>

class LCG {
public:
	LCG(unsigned __int128 state) : mState(state) {
		// when the seed is zero, the first term in the sequence will be zero
		// (this is only the case when the multiplier is 64-bit)
		next();
	}

	uint64_t next() {
		mState = mState * multiplier + multiplier;
		return mState >> 64;
	}

private:
	static const uint64_t multiplier = 0xfc0072fa0b15f4fd;
	unsigned __int128 mState;
};
```


## Statistical properties

As I mentioned before, the sequences generated by LCGs and MCGs are of high quality, given that the output is truncated and the parameters are chosen properly.

The quality of PRNGs is commonly used by running statistical testsuites on them. Popular ones are
  - [TestU01](http://simul.iro.umontreal.ca/testu01/tu01.html), with the "BigCrush" testsuite being the most stringent one
  - [PractRand](https://pracrand.sourceforge.net)
  - [Diehard](https://web.archive.org/web/20110226115043/https://stat.fsu.edu/pub/diehard)
  - [Dieharder](https://webhome.phy.duke.edu/~rgb/General/dieharder.php)
  - [NIST Statistical Test Suite](https://github.com/terrillmoore/NIST-Statistical-Test-Suite)
  - [ENT](https://www.fourmilab.ch/random)
  - [gjrand](http://gjrand.sourceforge.net)
  - [RaBiGeTe](http://cristianopi.altervista.org/RaBiGeTe_MT)

People have tried running these on LCGs, here are some testimonies I found.

From [the wikipedia page on linear congruential generators](https://en.wikipedia.org/wiki/Linear_congruential_generator#Advantages_and_disadvantages): "A[n] LCG with large enough state can pass even stringent statistical tests; a modulo-2 LCG which returns the high 32 bits passes TestU01's SmallCrush suite and a 96-bit LCG passes the most stringent BigCrush suite."

Mellissa O'Neill, author of the PCG PRNG, writes that [an 128-bit MCG passes PractRand](https://www.pcg-random.org/posts/128-bit-mcg-passes-practrand.html). In [On Vigna's PCG critique](https://www.pcg-random.org/posts/on-vignas-pcg-critique.html), she mentions "A simple truncated 128-bit LCG passes all standard statistical tests once we get up to 128 bits [...]".


## References and further reading

[1] ['Computationally Easy, Spectrally Good Multipliers for Congruential Pseudorandom Number Generators' by Guy Steele and Sebastiano Vigna, arXiv](https://arxiv.org/abs/2001.05304)

[2] ['Linear congruential generator', wikipedia](https://en.wikipedia.org/wiki/Linear_congruential_generator).

For further reading on PRNGs I recommend the blog of the author of PCG, Melissa O'Neill. The following posts are especially interesting:
  - [Predictability](https://www.pcg-random.org/predictability.html)
  - [Does It Beat the Minimal Standard?](https://www.pcg-random.org/posts/does-it-beat-the-minimal-standard.html)
  - [On Trivial Predictability](https://www.pcg-random.org/posts/on-trivial-predictability.html)
  - [Too Big to Fail](https://www.pcg-random.org/posts/too-big-to-fail.html)
  - [How to Test with PractRand](https://www.pcg-random.org/posts/how-to-test-with-practrand.html)
  - [How to Test with TestU01](https://www.pcg-random.org/posts/how-to-test-with-testu01.html)
