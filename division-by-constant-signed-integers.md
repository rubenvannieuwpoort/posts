# Division by constant signed integers

Division is a relatively slow operation. When the divisor is constant, the division can be optimized significantly. In [division by constant unsigned integers](https://rubenvannieuwpoort.nl/posts/division-by-constant-unsigned-integers) I explored how this can be done for unsigned integers. In this follow-up article, I cover how we can optimize division by constant signed integers.


## Background

TODO: introduce notation $\mathbb{S}_N$
TODO: introduce notation $[\frac{n}{d}]$ for rounded toward zero
TODO: introduce notation $|x|$ for absolute value of $x$
TODO: introduce notation $\text{sgn}(x)$ for sign of $x$


## Division by signed integers

We have that $[\frac{n}{d}] = -[\frac{n}{d}]$, so for negative divisors we can just take the absolute value and negate the result at the end. So we can restrict ourselves to positive integers.

We take a slight reformulation of theorem 3 from the article on division by constant unsigned integers as our starting point. We replace $N$ by $N - 1$ and generalize the result for negative integers:

**Corollary**: *Let $N$ be a positive integer, and let $d \neq 0$ be an integer with $|d|$ not a power of two. Define $\ell = \lfloor \log_2(d) \rfloor$ and $m = \lceil \frac{2^{N + \ell}}{d} \rceil$. Then $\lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{S}_N$.*

**Proof**: TODO
$\square$

Now, using the expression $\lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor$ works for nonnegative dividends $n$, since $\lfloor \frac{n}{d} \rfloor = [\frac{n}{d}]$ in this case. However, when $n$ is negative, we have $\lfloor \frac{n}{d} \rfloor = [\frac{n}{d}] - 1$, so we need to add one when $n$ is negative.

**Example**: Take $N = 8$, $d = 3$. Taking $\ell = \lfloor \log_2(d) \rfloor = 1$, we find $m_\text{up} = \lceil \frac{2^9}{3} \rceil = 171$. Testing with $n = 100$ gives $\lfloor \frac{171 \cdot 100}{2^9} \rfloor = \lfloor \frac{17100}{512} \rfloor = 33$, which is the right answer. Setting $n = -100$ gives $\lfloor \frac{171 \cdot -100}{2^9} \rfloor = \lfloor \frac{-17100}{512} \rfloor = -34$. So indeed we need to add one before we get the correct result $[\frac{-100}{3}] = -33$.

So, we want to add an expression that is $1$ when $n$ is negative and zero otherwise to $\lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor$.

One option is to simply right shift `n` until the sign bit ends up in the least significant bit. All the other bits will be zero. So `(sint)(((uint)n) >> (N - 1))` will equal one when `n` is negative, and zero otherwise.

Alternatively, we can use an [arithmetic shift](https://en.wikipedia.org/wiki/Arithmetic_shift) for this. If we shift right a negative value by `N - 1` bits, the sign bit will end up in the least significant bit and all the more significant bits will be equal to the sign bit. So the expression `n >> (N - 1)` will equal minus one when `n` is negative and zero otherwise. So when we subtract this expression this has the same effect as adding one when `n` is negative. The advantage of this method is that when the divisor is negative, we can simply swap the order of the operands to negate the result without needing additional instructions.

So, it seems like we have a working method. If we generate assembly code, there's a subtlety to be aware of in the computation of the product $m \cdot n \in \mathbb{S}_{2N}$. The value $m$ is in $\mathbb{U}_N$, while $n$ is in $\mathbb{S}_N$. So, we need to compute the $2N$-bit product of an $N$-bit unsigned and an $N$-bit signed value. Most processors don't have an instruction for this. If you do this in in C it will be 

### Computing the product of an unsigned and a signed value

In this section, consider $m$ and $n$ to be $N$-bit bit strings. So these variables no longer represent a number, but purely a series of bits, which can hold a zero or a one. So we write $m = m_{N - 1} m_{N - 2} ... m_1 m_0$, where $m_{N - 1}, ..., m_0 \in \{ 0, 1 \}$ are the individual bits.

Now, we can provide a string $m$ with a value when we interpret it as either an unsigned value $(m)_\text{u}$, or as a signed value $(m)_\text{s}$. These interpretations are defined as
$$ (m)_\text{u} = \sum_{k = 0}^{N - 1} 2^k m_k $$

and
$$ (m)_\text{s} = -2^{N - 1} m_{N - 1} + \sum_{k = 0}^{N - 2} 2^k m_k $$

We see that $(m)_\text{u} = (m)_\text{s}$ when $m_{N - 1} = 0$ and $(m)_\text{u} = (m)_\text{s} + 2^N$ when $m_{N - 1} = 1$. So when $m_{N - 1} = 0$ we have
$$ (m)_\text{u} \cdot (n)_\text{s} = (m)_\text{s} \cdot (n)_\text{s} $$

So in this case we can just use signed multiplication. When $m_{N - 1} = 1$ we have

$$ (m)_\text{u} \cdot (n)_\text{s} = ((m)_\text{s} + 2^N) \cdot (n)_\text{s} = (m)_\text{s} \cdot (n)_\text{s} + 2^N \cdot (n)_\text{s}$$

So, the upper $N$ bits of the product $(m)_\text{u} \cdot (n)_\text{s}$ equals $\lfloor \frac{(m)_\text{s} \cdot (n)_\text{s}}{2^N} \rfloor + (n)_\text{s}$. This expression can be evaluated by multiplying $m$ and $n$ as if they where signed numbers, taking the upper $N$ bits, and adding $n$ to this.

## Implementation

Like for the division by constant unsigned integers, we consider both the case of compile-time constant divisors and the case of runtime constant divisors.

A case that we haven't covered in the first section, is when the divisor is a power of two. Suppose that $d = 2^p$. We can try to implement the division by $d$ by a right shift by $p$ bits. This will give us $\lfloor \frac{n}{2^p} \rfloor$. When $n$ is positive this works since $\lfloor \frac{n}{d} \rfloor = [\frac{n}{d}]$ for positive $n$. When $n$ is negative, we have $\lfloor \frac{n}{d} \rfloor = [\frac{n}{d}] - 1$ when $n$ is not a multiple of $d$, but $\lfloor \frac{n}{d} \rfloor = [\frac{n}{d}]$ when $n$ is a multiple of $d$. Note that this problem doesn't occur for divisors that are not powers of two, since $m \cdot n$ can never be a multiple of $2^{N + \ell}$.

The underlying problem is that we are rounding down the quotient, instead of rounding up. To fix this, we can add $2^p - 1$ to $n$ when $n$ is negative.
