# Linear congruential generators

Linear congruential generators are a quick-and-dirty way to generate pseudorandom numbers. While the statistical properties of the pseudorandom numbers are not perfect, an LCG with properly chosen parameters seems to be a solid choice in practice. LCGs are simple to implement, fast, and require almost no storage.

This post is meant as a reference for implementing a decent LCG, without going into all the why's of the implementation. So I won't dive into the theory behind them.

LCGs use the recurrence
$$ x_{n + 1} = (ax_n + c) \mod{m} $$

Here, $x_0$ is called the *seed*, $a$ is called the *multiplier*, $c$ is called the *increment*, and $m$ is called the *modulus*. When $c = 0$ we call the generator a *multiplicative congruential generator* or a *Lehmer generator*. When $c \neq 0$ we call the generator a *mixed congruential generator*.

The modulus is chosen as a power of two for efficient evaluation, and to ensure that the distribution of the pseudorandom number is uniform. When a power of two is used, the $k$th bit has a period of $2^k$. So to avoid low-quality pseudorandom numbers, only the most significant portion of the state should be used (I use the most significant half in the example implementation).


## Multiplicative congruential generators

A multiplicative congruential generator with $m$ a power of two has a maximum period of $m / 4$. The lower three bits of $x_n$ alternate between two states and should not be part of the pseudorandom number.

In [1], the following multipliers with good properties for MCGs are listed:

For $m = 2^{32}$: 2480367069

For $m = 2^{64}$: 17380933483125451205

For $m = 2^{128}$: 227125521124990501218943255231830569685


## Mixed congruential generators

In general, mixed congruential generators can obtain the maximum possible period of $m$. The increment can be chosen as any odd number.

In [1], the following multipliers with good properties for LCGs are listed:

For $m = 2^{32}$: 2438952949

For $m = 2^{64}$: 15074714826142052245

For $m = 2^{128}$: 291382399519485789170309121576895642645


## Example

This is a simple C++ header-only implementation of an LCG with a 64-bit state space and a 32-bit output.

```
#include <cstdint>

class LCG {
public:
	LCG(uint64_t multiplier, uint64_t increment, uint64_t seed)
		: mMultiplier(multiplier),
		  mIncrement(increment),
		  mState(seed)
	{ }

	uint32_t next() {
		mState = mState * mMultiplier + mIncrement;
		return (uint32_t)(mState >> 32);
	}

private:
	uint64_t mMultiplier, mIncrement, mState;
};
```


## References

[1] ['Computationally Easy, Spectrally Good Multipliers for Congruential Pseudorandom Number Generators' by Guy Steele and Sebastiano Vigna, arXiv](https://arxiv.org/abs/2001.05304)

[2] ['Linear congruential generator', wikipedia](https://en.wikipedia.org/wiki/Linear_congruential_generator#History).

