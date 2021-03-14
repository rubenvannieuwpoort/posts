# Division by constant signed integers

The code accompanying this article can be found in a [github repository](https://github.com/rubenvannieuwpoort/division-by-constant-integers).

Division is a relatively slow operation. When the divisor is constant, the division can be optimized significantly. In [1] I explored how this can be done for unsigned integers. In this follow-up article, I cover how we can optimize division by constant signed integers. This article should be read as a continuation of [1]. This article essentially provides the same information as [2].

I assume that like in most programming languages, the result of the signed division is rounded toward zero. This presents some challenges which are different than those for optimizing unsigned division. We are only dealing with numbers with a magnitude of at most $2^{N - 1}$, and we will see that this means we can always use the round-up method as described in [1]. The challenge consists of efficiently rounding up the quotient when $n$ is negative.


## Mathematical background

### Preliminaries

I will assume that we are working on an $N$-bit machine which can efficiently compute the full $2N$-bit product of two $N$-bit signed integers. I will use the notation $\mathbb{U}_N$​ for the set of unsigned integers that can be represented with $N$ bits:
$$ \mathbb{U}_N = \{ 0, 1, ..., 2^N - 1 \} $$

Likewise, I will use the notation $\mathbb{S}_N$ for the set of signed integers that can be represented with $N$ bits:
$$ \mathbb{S}_N = \{ -2^{N - 1}, -2^{N - 1} + 1, ..., 2^{N - 1} - 1 \} $$

When $A$ and $B$ are sets, the set $A \setminus B$ denotes the set of elements of $A$ that are not in $B$.

For some real number $x \in \mathbb{R}$, I will denote the absolute value of $x$ by $|x|$. That is:
$$ |x| = \begin{cases} x & \text{when $x \geq 0$} \\ -x & \text{otherwise} \end{cases}$$

I will use the notation $\lfloor x \rfloor$ for the biggest integer smaller than or equal to $x$, and $\lceil x \rceil$ for the smallest integer bigger than or equal to $x$. I will use $[ x ]$ to denote the value of $x$ when rounded toward zero. That is
$$ [x] = \begin{cases} \lceil x \rceil & \text{when $x < 0$} \\ \lfloor x \rfloor & \text{when $x \geq 0$} \end{cases}$$

I will use the notation $\text{sgn}(x)$ for the *sign function*:
$$ \text{sgn}(x) = \begin{cases} -1 & \text{when $x < 0$} \\ 0 & \text{when $x = 0$} \\ 1 & \text{when $x > 0$} \end{cases}$$

Finally, I will use the notation $1_P$ where $P$ is a predicate to denote the *characteristic function*:
$$ 1_P = \begin{cases} 0 & \text{when $P$ is false} \\ 1 & \text{when $P$ is true} \end{cases}$$


### Signed division

Supposing that $d$ is positive, we want to evaluate
$$ [\frac{n}{d}] = \begin{cases} \lceil \frac{n}{d} \rceil & \text{when $n < 0$} \\ \lfloor \frac{n}{d} \rfloor & \text{when $n \geq 0$} \end{cases}$$

So, it is natural to try to find two expressions, one which equals $\lfloor \frac{n}{d} \rfloor$ and one which equals $\lceil \frac{n}{d} \rceil$.

In [1], we saw that we for every $d \in \mathbb{N}$ we can always find an $(N + 1)$-bit magic number $m$ such that $\lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for every $n \in \mathbb{U}_N$. For unsigned division this was a problem since we want $m$ to fit in $N$ bits. For nonnegative $n \in \mathbb{S}_N$ we have $n \in \mathbb{U}_{N - 1}$ so in this case $m$ will always fit in $N$ bits.

**Corollary 11**: *Let $d, N, \ell \in \mathbb{N}_0$ with $N, d > 0$. If there exists an $m \in \mathbb{N}_+$ with*
$$ 2^{N - 1 + \ell} \leq m \cdot d \leq 2^{N - 1 + \ell} + 2^\ell $$

*then $\lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all nonnegative $n \in \mathbb{S}_N$*.

**Proof**: Note that the set of nonnegative $n$ in $\mathbb{S}_N$ is exactly $\mathbb{U}_{N - 1}$. So this is just theorem 2 from [1] with $N$ replaced by $N - 1$.
$\square$

We will now prove a similar result for negative $n \in \mathbb{S}_N$. The following lemma will come in handy.

**Lemma 12**: *Let $n \in \mathbb{Z}, d \in \mathbb{N}_+$, and $x \in \mathbb{R}$. When*
$$ \frac{n - 1}{d} \leq x < \frac{n}{d} $$

*then*
$$ \lfloor x \rfloor = \left \lceil \frac{n}{d} \right \rceil - 1 $$

**Proof**: Set $\frac{n - 1}{d} = a + \frac{b}{d}$ with $a \in \mathbb{Z}$, $b \in \{ 0, 1, ..., d - 1 \}$. Then $\frac{n}{d} = a + \frac{b + 1}{d}$. Now $\lfloor \frac{n - 1}{d} \rfloor = a$ and $\lceil \frac{n}{d} \rceil = a + 1$ so that $\lceil \frac{n}{d} \rceil = \lfloor \frac{n - 1}{d} \rfloor + 1$. So $\lfloor \frac{n - 1}{d} \rfloor \leq \lfloor x \rfloor < \lfloor \frac{n - 1}{d} \rfloor + 1$. Since $\lfloor x \rfloor$ is an integer, it follows that $x = \lfloor \frac{n - 1}{d} \rfloor = \lceil \frac{n}{d} \rceil - 1$.
$\square$

The following theorem gives us a way to calculate $\lceil \frac{n}{d} \rceil$ from the same expression $\lfloor \frac{m \cdot d}{2^{N - 1 + \ell}} \rfloor$ when $n$ is negative.

**Lemma 13**: *Let $d, m, N \in \mathbb{N}_+$. If*
$$ 2^{N - 1 + \ell} < m \cdot d \leq 2^{N - 1 + \ell} + 2^\ell $$

*then*
$$\left \lfloor \frac{m \cdot n}{2^{N - 1 + \ell}} \right \rfloor = \left \lceil \frac{n}{d} \right \rceil - 1 $$

*for all negative $n \in \mathbb{S}_N$.*

**Proof**: Multiply $2^{N - 1 + \ell} < m \cdot d \leq 2^{N - 1 + \ell} + 2^\ell$ by $\frac{n}{d \cdot 2^{N + \ell}}$. Remember that $n$ is negative, so the inequality 'flips' and we get $\frac{n}{d} + \frac{n}{2^{N - 1}} \cdot \frac{1}{d} \leq \frac{m \cdot n}{2^{N - 1 + \ell}} < \frac{n}{d}$. Now, using that $-2^{N - 1} < n$ we see that $-1 \leq \frac{n}{2^{N - 1}}$, so we have $\frac{n}{d} - \frac{1}{d} \leq \frac{m \cdot n}{2^{N - 1 + \ell}} < \frac{n}{d}$. The result now follows from lemma 2.
$\square$

So, if we find an $m$ such that $2^{N - 1 + \ell} < m \cdot d \leq 2^{N - 1 + \ell} + 2^\ell$, we have $\left[ \frac{n}{d} \right] = \lfloor \frac{m \cdot n}{2^{N - 1 + \ell}} \rfloor + 1_{n < 0}$. The following theorem extends this result to the case where $d$ is negative and shows us how to pick $m$.

**Theorem 14**: *Let $d$ and $N$ be integers with $N > 0$ and define $\ell = \lceil \log_2(|d|) \rceil$ and $m = \lfloor \frac{2^{N - 1 + \ell}}{|d|} \rfloor + 1$. Then*
$$\left[ \frac{n}{d} \right] = \text{sgn}(d) \cdot \left(\left \lfloor \frac{m \cdot n}{2^{N - 1 + \ell}} \right \rfloor + 1_{n < 0} \right)$$

*for all $n \in \mathbb{S}_N$.*

**Proof**: First, observe that $m \cdot |d|$ is simply the first multiple of $|d|$ larger than $2^{N - 1 + \ell}$. Since there are $2^\ell = 2^{\lceil \log_2(|d|) \rceil} \geq |d|$, there must be at least one multiple of $d$ in the range $(2^{N - 1 + \ell}, 2^{N - 1 + \ell} + 2^\ell]$. So we have $2^{N - 1 + \ell} < m \cdot d \leq 2^{N - 1 + \ell} + 2^\ell$. Using corollary 11 we see that $\lfloor \frac{m \cdot n}{2^{N - 1 + \ell}} \rfloor + 1_{n < 0} = \lfloor \frac{m \cdot n}{2^{N - 1 + \ell}} \rfloor = \lceil \frac{n}{d} \rceil$ for nonnegative $n \in \mathbb{S}_N$. Using lemma 13, we see that $\lfloor \frac{m \cdot n}{2^{N - 1 + \ell}} \rfloor + 1_{n < 0} = \lfloor \frac{m \cdot n}{2^{N - 1 + \ell}} \rfloor + 1 = \lfloor \frac{n}{d} \rfloor$ for negative $n \in \mathbb{S}_N$. So $[ \frac{n}{|d|} ] = \left \lfloor \frac{m \cdot n}{2^{N - 1 + \ell}} \right \rfloor + 1_{n < 0}$ for all $n \in \mathbb{S}_N$. Using $[ \frac{n}{d} ] = \text{sgn}(d) \cdot [ \frac{n}{|d|} ]$ the result follows.
$\square$

Lemma 5 tells us that the magic number $m = \lceil \frac{2^{N - 1 + \lceil \log_2(d) \rceil}}{d} \rceil$ is a positive number with at most $N$ bits. In the following section on implementation, we will see that multiplying an $N$-bit signed number $n \in \mathbb{S}_N$ by an $N$-bit unsigned number $m \in \mathbb{U}_N \setminus \mathbb{U}_{N - 1}$ is slightly less efficient than multiplying an $N$-bit signed number by an $(N - 1)$-bit unsigned number. For some divisors, we can use the following result to reduce the number of bits that we need to represent $m$.

**Corollary 16**: *Let $d \in \mathbb{S}_N$ be a positive integer that is not a power of two, $m \in \mathbb{U}_N$ and let $0 < \ell \leq \lfloor \log_2(d) \rfloor$, such that $\ell, m$ satisfy the condition of lemma corollary 11 and lemma 13:*
$$ 2^{N - 1 + \ell} < m \cdot d \leq 2^{N - 1 + \ell} + 2^\ell $$

*If $m$ is even, then this condition also holds for $m' = \frac{m}{2}, \ell' = \ell - 1$, so we have $2^{N - 1 + \ell'} < m' \cdot d \leq 2^{N - 1 + \ell'} + 2^{\ell'}$. If $m$ is odd, then it is the smallest integer for which this condition holds.*

**Proof**: This follows directly from theorem 9.
$\square$

## Implementation

In this section, I use the `uint` and `sint` datatypes, which are an $N$-bit unsigned integer and an $N$-bit signed integer, respectively. I try to provide a general strategy that should work well on most instruction set architectures. Variations in the implementation might give a more efficient result. In general, you should always benchmark your implementation if performance is critical.

While theorem 3 in the previous section seems to provide a straightforward method to compute the quotient $[ \frac{n}{d} ]$ for any $n, d \in \mathbb{S}_N$, there is one subtlety we glanced over. In theorem 4, we use the $2N$-bit expression $m \cdot n$, where $m \in \mathbb{U}_N$ is an unsigned value and $n \in \mathbb{S}_N$ is a signed value. While most processors have instructions to compute the full $2N$-bit product of two $N$-bit unsigned integers or two $N$-bit signed integers, most processors do not provide an instruction to compute the $2N$-bit product of an $N$-bit unsigned integer and an $N$-bit signed integer.

While it is also possible to compute the product $m \cdot n$ by first extending $m$ and $n$ to $2N$-bit signed values and computing the product of those extended values, this is less efficient.


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

### Runtime optimization

Consider the following code:
```
sint d = read_divisor();

for (int i = 0; i < size; i++) {
	quotient[i] = dividend[i] / d;
}
```

The value of the divisor `d` is not known at compile time, but once it is read at runtime, it does not change. As such, we consider `d` to be a runtime constant, and we can optimize this code in the following way:
```
sint d = read_divisor();
divdata_t divisor_data = precompute(divisor);

for (int i = 0; i < size; i++) {
	quotient[i] = fast_divide(dividend[i], divisor_data);
}
```

Now, the `divdata_t` datatype needs to hold $m$, the number of bits to shift, and some field to indicate that we should negate the result $[\frac{n}{|d|}]$ when $d$ is negative:
```
typedef struct {
	uint mul;
	uint shift;
	bool negative;
} divdata_t;
```

Now, we compute $\ell$ such that $m$ always has the most significant bit set. This way we can always compute the upper $N$ bits of the product $m \cdot n$ in `fast_divide` by taking the signed product, taking the upper $N$ bit, and adding $n$ to this.

The precomputation is now a relatively straightforward implementation of theorem 4:
```
sdivdata_t precompute(sint d) {
	sdivdata_t divdata;
	uint d_abs = abs(d);
	
	// Compute ceil(log2(d_abs))
	uint l = floor_log2(d_abs);
	if ((1 << l) < d_abs) l++;

	// Handle case |d| = 1
	if (dabs == 1) l = 1;

	// Compute m = floor(2^(N - 1 + l) / d) + 1
	uint m = (((big_uint)1) << (N - 1 + l)) / d_abs + 1;
	
	divdata.mul = m;
	divdata.negative = d < 0;
	divdata.shift = l - 1;
	return divdata;
}
```

It should be noted that in the `fast_divide` function, the right shift by $N - 1 + \ell$ is implemented by taking the upper $N$ bits of the product $m \cdot n$, and shifting this right by $\ell - 1$ bits. If $\ell = 0$ this is not possible, since in this case we need to shift right by $N - 1$ bits, but taking the $N$ upper bits is already equivalent to a right shift of $N$ bits. We can fix this by simply setting $\ell = 1$ when $|d| = 1$. In this case the expression for $m$ becomes $2^N + 1$, which will overflow to simply $1$. Now, while theorem 4 doesn't hold anymore for $m = 1$, the calculation of the product $m \cdot n$ in `fast_divide` assumes that the most significant bit of $m$ is set, so we will end up with the correct value. In fact, setting $m = 0$ would work as well.

The `fast_divide` function has a lot of steps, but every step should be understandable.

```
sint fast_divide(sint n, sdivdata_t dd) {
	big_sint full_signed_product = ((big_sint)n) * (sint)dd.mul;
	sint high_word_of_signed_product = full_signed_product >> N;
	sint high_word_of_unsigned_product = high_word_of_signed_product + n;
	sint rounded_down_quotient = high_word_of_unsigned_product >> dd.shift;
	sint quotient_rounded_toward_zero = rounded_down_quotient - (n >> (N - 1));
	if (dd.negative) {
		quotient_rounded_toward_zero = -quotient_rounded_toward_zero;
	}
	return quotient_rounded_toward_zero;
}
```


### Compile-time optimization

In this section, I will consider how to generate optimized code for division by compile-time constant signed integers.

Most of the tricks that are applicable to calculate a quotient of unsigned integers efficiently also apply to signed integers, although we might have to do some work to handle negative integers. Of course, a division by one can be ignored and a division by minus one is equivalent to a negation. For some instruction-set architectures, it might be beneficial to implement a special case for big divisors with an absolute value of more than $2^{N - 1}$. In this case, the value of the quotient $[ \frac{n}{d} ]$ is $\text{sgn}(n)$ when $|n| \geq d$ and zero otherwise.

```
expression_t div_by_const_sint(const sint d, expression_t n) {
	if (d == 1) return n;
	if (d == -1) return neg(n);
	uint d_abs = abs(d);
	if (is_power_of_two(d_abs)) return div_by_const_signed_power_of_two(n, d);
	return div_fixpoint(d, n);
}
```

Let us first consider the case where $|d| = 2^\ell$ is a power of two. If we do an arithmetic right shift by $\ell$ bits, the result will be correct when $n$ is positive. However, this will round down the quotient when $n$ is negative. In this case we can add $2^\ell - 1$ to $n$ in order to round up.

So, we would like to have an expression which equals $2^\ell - 1$ when $n$ is negative and $0$ otherwise, so we can simply add this to $n$. The value $2^\ell - 1$ consists of $\ell$ consecutive ones in the binary representation. It can be created by doing an arithmetic right shift by $\ell$ bits on $n$, and shifting the result right by $N - \ell$ bits (with a normal right shift). The first shift produces the $\ell$ ones in the $\ell$ most significant bits when $n$ is negative (these are zero bits otherwise), the second shift puts them in the least significant bit positions.
```
expression_t div_by_const_signed_power_of_two(expression_t n, sint d) {
	uint d_abs = abs(d);
	int l = floor_log2(d_abs);
	
	// addme equals 2^l - 1 when n is negative and 0 otherwise
	// We need to add this to n to round towards zero.
	expression_t addme = shr(sar(n, constant(l - 1)), constant(N - l));
	
	expression_t result = sar(add(n, addme), constant(l));
	if (d < 0) result = neg(result);
	return result;
}
```


## References

[1] Division by constant unsigned integers, https://rubenvannieuwpoort.nl/posts/division-by-constant-unsigned-integers
[2] [Division by Invariant Integers using Multiplication](https://gmplib.org/~tege/divcnst-pldi94.pdf), Torbjörn Granlund and Peter L. Montgomery, 1994.
