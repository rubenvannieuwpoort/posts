
# Division by constant unsigned integers

Most modern processors have an integer divide instruction which, for technical reasons, is very slow compared to other integer arithmetic operations. When the divisor is constant, it is possible to evaluate the result of the division without using the slow division instruction. Most optimizing compilers perform this optimization, as can be seen on [Matt Godbolt’s compiler explorer](https://godbolt.org/z/9G1fE7). This article tries to be a self-contained reference for optimizing division by constant unsigned divisors.

There are tricks to make division by some special divisors fast: A division by one can be ignored, a division by a power of two can be replaced by a bit shift. A more general trick exists: By using fixed-point arithmetic, we can speed up division by all constant divisors. However, this is not simple to explain, and for this reason I will focus only on positive (or 'unsigned') divisors in this article. For a number $n$ (also called the *dividend*) and a divisor $d$, I assume that we are interested in the *quotient* $\lfloor \frac{n}{d} \rfloor$.

When it is necessary to repeatedly divide floating-point numbers by the same constant $c$, it is often preferred to precompute the floating-point number $\frac{1}{c}$ and multiply by this instead. This is usually a lot more efficient. We can do something similar for integers by using [fixed point arithmetic](https://en.wikipedia.org/wiki/Fixed-point_arithmetic). The basic idea is to pick a large constant $L$ that is easy to divide by. In decimal you can take some power of ten, in binary you would take a power of two so that we can divide by $L$ using bit shifts. If we would have $c = \frac{L}{d}$ we would have $\frac{n \cdot c}{L} = \frac{n \cdot \frac{L}{d}}{L} = \frac{n}{d}$. However, $\frac{L}{d}$ is usually not an integer, and we need to round. Still, if $c \approx \frac{L}{d}$ we still expect that $\frac{n \cdot c}{L} \approx \frac{n}{d}$.

In the following section, I'll discuss the mathematical background. In the sections after that, I'll discuss optimization of division by unsigned integers.


## Mathematical background

### Preliminaries

I will assume that we are working on an $N$-bit machine which can efficiently compute the full $2N$-bit product of two $N$-bit unsigned integers. I will use the notation $\mathbb{U}_N$ for the set of unsigned integers that can be represented with $N$ bits:
$$ \mathbb{U}_N = \{ 0, 1, ..., 2^N - 1 \} $$

I will use the notation $\text{mod}_d(n)$ to denote the integer in the range $\{ 0, 1, ..., d - 1 \}$ that is equivalent to $n$ modulo $d$. That is, $\text{mod}_d(n)$ is the unique integer such that
$$ 0 \leq \text{mod}_d(n) < d $$

$$ \text{mod}_d(n) = n + m \cdot d $$

for some $m \in \mathbb{Z}$.


### Unsigned division

For a given divisor $d$, we now want to have an expression that evaluates to $\lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$. We have two demands for this expression:
  1. It should be *correct*, that is, the expression is equal to $\lfloor \frac{n}{d} \rfloor$
  2. It should be *efficient to evaluate*. For our purposes, this means that we only want to multiply numbers of at most $N$ bits, so that they fit in a single register on our $N$-bit machine, and that we implement the division by $2^k$ with a bit shift.

Guided by these demands, we can start our analysis. The following lemma will be useful:

**Lemma 1**: *Suppose that $n, d \in \mathbb{N}$ with $d > 0$. If $\frac{n}{d} \leq x < \frac{n + 1}{d}$ then $\lfloor x \rfloor = \lfloor \frac{n}{d} \rfloor$.*

**Proof**: We have $\frac{n + 1}{d} = \lfloor \frac{n}{d} \rfloor + \frac{k}{d}$ for some nonnegative integer $k \leq d$. So $\frac{n + 1}{d} = \lfloor \frac{n}{d} \rfloor + \frac{k}{d} \leq \lfloor \frac{n}{d} \rfloor + 1$. It follows that $x \in [ \lfloor \frac{n}{d} \rfloor, \lfloor \frac{n}{d} \rfloor + 1)$, so that $\lfloor x \rfloor = \lfloor \frac{n}{d} \rfloor$.
$\square$

At this point, we have decided that we want an expression that evaluates to $\lfloor \frac{n}{d} \rfloor$. In the introduction, we have established that when
$$ m \approx \frac{2^k}{d} $$

then
$$ \frac{n \cdot m}{2^k} \approx \frac{n}{d} $$

So, it is natural to take $\lfloor \frac{m \cdot n}{2^k} \rfloor$ with $m \approx \frac{2^k}{d}$ as our starting point, since we know that $\frac{m \cdot n}{2^k} \approx \frac{n}{d}$. We now need to decide how exactly to pick $m$ and $k$. The obvious choice for $m$ are the integers that minimize the error to $\frac{2^k}{d}$, which are $m_\text{down} = \lfloor \frac{2^k}{d} \rfloor$ and $m_\text{up} = \lceil \frac{2^k}{d} \rceil$. Note that when $d$ is not a power of two and put $n = d$, we have $\lfloor \frac{m_\text{down} \cdot d}{2^k} \rfloor = 0$, so this is not a viable method. So, let's round up instead.

This gives us the **round-up method**, which approximates $\lfloor \frac{n}{d} \rfloor$ by $\lfloor \frac{m_\text{up} \cdot n}{2^k} \rfloor$ with $m_\text{up} = \lceil \frac{2^k}{d} \rceil$.

First, we determine the conditions under which the round-up and round-down method produce the correct result. Note that from this point on we will assume that $k$ is larger than $N$. We will assume $k = N + \ell$ and use $k$ and $N + \ell$ interchangeably.

Now, we establish under which conditions the round-up method is correct. The following theorem can be used to derive a condition under which the round-up method is correct, that is, $\lfloor \frac{m_\text{up} \cdot n}{2^k} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.

**Theorem 2**: *Let $d, N, l \in \mathbb{N}$ be nonnegative integers with $d > 0$. If there exists an $m \in \mathbb{N}$ with*
$$ 2^{N + \ell} \leq m \cdot d \leq 2^{N + \ell} + 2^\ell $$

*then*
$$ \forall n \in \mathbb{U}_N : \lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor $$

**Proof**: Multiplying the inequality by $\frac{n}{d \cdot 2^{N + \ell}}$ we get
$$ \frac{n}{d} \leq \frac{m \cdot n}{2^{N + \ell}} \leq \frac{n}{d} + \frac{1}{d} \cdot \frac{n}{2^N} $$

For all $n \in \mathbb{U}_N$ we have $n < 2^N$, so that $\frac{n}{2^N} < 1$. It follows that
$$ \forall n \in \mathbb{U}_N : \frac{n}{d} \leq \frac{m \cdot n}{2^{N + \ell}} \leq \frac{n}{d} + \frac{1}{d} $$

By lemma 1, it follows that $\lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

**Corollary (correctness of the round-up method)**: *Let $d, \ell, N \in \mathbb{N}$ with $d > 0$ and $m_\text{up} = \lceil \frac{2^{N + \ell}}{d} \rceil$. If $m_\text{up} \cdot d \leq 2^{N + \ell} + 2^{\ell}$ we have $\lfloor \frac{m_\text{up} \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$. If $m_\text{up} \cdot d > 2^{N + \ell} + 2^\ell$ then there exists no $m$ such that $2^{N + \ell} \leq m \cdot d \leq 2^{N + \ell} + 2^\ell$.*

**Proof**: TODO
$\square$

Let's do some examples to see how we can use the round-up method in practice.

**Example**: TODO

**Example**: TODO

This last example shows that $m_\text{up}$ does not always fit in $N$ bits. The following theorem shows that $m_\text{up}$ always fits in $N + 1$ bits.

**Theorem 3**: *Let $N, d \in \mathbb{N}_+$ and define $\ell = \lceil \log_2(d) \rceil, m_\text{up} = \lceil \frac{2^{N + \ell}}{d} \rceil$. Then $m_\text{up} \in \mathbb{U}_{N + 1}$ and $\lfloor \frac{m_\text{up} \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.*

**Proof**: The range $\{ 2^N, 2^N + 1, ..., 2^N + 2^\ell \}$ consists of $2^\ell + 1$ consecutive numbers. We have $\ell = \lceil \log_2(d) \rceil$, so $2^\ell + 1 > d$ and there must be a multiple of $d$ in this range. Since $d \cdot m_\text{up} = d \cdot \lceil \frac{2^{N + \ell}}{d} \rceil$ is simply the first multiple greater than or equal to $d$, this is a multiple of $d$ in this range. By theorem 2, it follows that $\lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$. In the proof of lemma 6 we saw that $\frac{2^{N + \lceil \log_2(d) \rceil - 1}}{d} < 2^N - 1$. By multiplying both sides by two and rounding up, we see that we have $\lceil \frac{2^{N + \lceil \log_2(d) \rceil}}{d} \rceil = m_\text{up} \leq 2^{N + 1} - 1$, so $m_\text{up} \in \mathbb{U}_{N + 1}$.
$\square$

One way to handle the case when $m$ does not fit in $N$ bits, is to use the following trick from [1] to multiply by an $(N + 1)$-bit constant: We can evaluate $\lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor$ by defining $m' = m - 2^N$ and using the following code:
```
uint high_word = (((big_uint)m') * n) >> N;
uint result = (high_word + ((n - high_word) >> 1)) >> (l - 1);
```

The reason for mentioning this method is mainly that it is used by some compilers. We will now focus on a variation of the round-up method that will usually be more efficient. Instead of rounding up $\frac{2^{N + \ell}}{d}$, we can round down but increase $n$.

This gives us the **round-down method**, which approximates $\lfloor \frac{n}{d} \rfloor$ by $\lfloor \frac{m_{down} \cdot (n + 1)}{2^{N + \ell}} \rfloor$ by $m_\text{down} = \lfloor \frac{2^{N + \ell}}{d} \rfloor$.

We proceed as we did for the round-up method, by deriving a condition under which the method is correct.

**Theorem 4**: *Let $d, N, l \in \mathbb{N}$ be nonnegative integers with $d > 0$. If there exists an $m \in \mathbb{N}$ with*
$$ 2^{N + \ell} - 2^\ell \leq m \cdot d < 2^{N + \ell}$$

*then*
$$ \forall n \in \mathbb{U}_N : \lfloor \frac{m \cdot (n + 1)}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor $$

**Proof**: Multiply the inequality by $\frac{n + 1}{d \cdot 2^{N + \ell}}$ to get
$$ \frac{n}{d} + \frac{1}{d} \cdot \left( 1 - \frac{n + 1}{2^N} \right) \leq \frac{m \cdot (n + 1)}{2^{N + \ell}} < \frac{n + 1}{d} $$

Looking at the expression on the left side, we have $1 \leq n + 1 \leq 2^N$, so that $0 \leq 1 - \frac{n + 1}{2^N} < 1$. It follows that $\frac{n}{d} \leq \frac{n}{d} + \frac{1}{d} \cdot (1 - \frac{n + 1}{2^N})$, so
$$ \frac{n}{d} \leq \frac{m \cdot (n + 1)}{2^{N + \ell}} < \frac{n + 1}{d} $$

for all $n \in \mathbb{U}_N$. So lemma 1 applies and we have $\lfloor \frac{m \cdot (n + 1)}{2^{N + \ell}} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

**Corollary (correctness of the round-down method)**: TODO

**Proof**: TODO
$\square$

Let's do an example to see the round-down method in practice:

**Example**: TODO

When using the round-up or round-down method, we want $m$ to be an $N$-bit number in order for the multiplication to be efficient. As we have seen, such an $m$ does not always exist. Let us call divisors for which such an $m$ exists *efficient*. The following definition makes this rigorous.

**Definition**: *We call a positive divisor $d \in \mathbb{U}_d$ efficient for the $N$-bit round-up method (or round down method) if there exists an $\ell \in \mathbb{N}$ such that $\lfloor \frac{m_\text{up} \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$. Likewise, we call a positive divisor $d \in \mathbb{U}_d$ efficient for the $N$-bit round-down method if there exists an $\ell \in \mathbb{N}$ such that $\lfloor \frac{m_\text{down} \cdot (n + 1)}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.*

The following result tells us that $\ell = \lceil \log_2(d) \rceil - 1$ is the biggest value for $\ell$ that we can pick so that $m_\text{up}$ and $m_\text{down}$ still fit in $N$ bits.

**Lemma 5**: *Let $d \in \mathbb{U}_N$ and define $m' = \frac{2^{N + \lceil \log_2(d) \rceil - 1}}{d}, m_\text{down} = \lfloor m' \rfloor, m_\text{up} = \lceil m' \rceil$. Now*
$$2^{N - 1} \leq \lfloor x \rfloor \leq \lceil x \rceil \leq 2^N - 1$$

That is, the binary representations of $m_\text{down}$ and $m_\text{up}$ have exactly $N$ bits.

**Proof**: We have $2^{N - 1} = \frac{2^{N - 1} \cdot d}{d} = \frac{2^{N - 1 + \log_2(d)}}{d} \leq \frac{2^{N - 1 + \lceil \log_2(d)} \rceil}{d} = m'$. Since the floor function is a nondecreasing function and $\lfloor m' \rfloor \leq \lceil m' \rceil$ we have $2^{N - 1} \leq m_\text{down} \leq m_\text{up}$. It remains to show that $m_\text{up} \leq 2^N - 1$.

When $N = 1$, it follows that $d = 1$ and the bound holds. When $N > 1$, the ratio $\frac{2^{\lceil \log_2(d) \rceil}}{d}$ is maximized over $d \in \mathbb{U}_N$ by $d = 2^{N - 1} + 1$, so $\frac{2^{\lceil \log_2(d) \rceil}}{d} \leq \frac{2^N}{2^{N - 1} + 1} < \frac{2^N - 1}{2^{N - 1}}$ (this last inequality can be seen by multiplying both sides by $2^{N - 1} \cdot (2^{N - 1} + 1)$). So we have $\frac{2^{\lceil \log_2(d) \rceil}}{d} < \frac{2^N - 1}{2^{N - 1}}$; multiplying both sides by $2^{N - 1}$ gives $m' = \frac{2^{N + \lceil \log_2(d) \rceil - 1}}{d} < 2^N - 1$. After rounding up both sides, it follows that $m_\text{up} \leq 2^N - 1$.
$\square$

The following result states that we can combine the round-up and round-down method to a method that works for any positive divisor. So this finally concludes our quest for an efficient method to calculate the quotient $\lfloor \frac{n}{d} \rfloor$.

**Theorem 6**: *Any positive divisor $d \in \mathbb{U}_N$ is either efficient for the $N$-bit round-up method, or efficient for the $N$-bit round-down method.*

**Proof**: Set $\ell = \lceil \log_2(d) \rceil - 1$. By lemma 6, we know that $m_\text{up}, m_\text{down} \in \mathbb{U}_N$. Now consider the range $\{ 2^N - 2^\ell, 2^N - 2^\ell + 1, ..., 2^N + \ell  \}$. This is a range of $2\ell + 1$ numbers. Since $d < 2\ell + 1$ there must be at least one multiple $m$ of $d$ in this range. When this multiple $m$ satisfies $2^N - 2^\ell \leq m < 2^{N + \ell}$ the condition for the round-down method is satisfied. Since $m_\text{down} \in \mathbb{U}_N$, $d$ is efficient for the $N$-bit round-down method. Otherwise, we have $2^{N + \ell} \leq m \leq 2^{N + \ell} + 2^\ell$ and the condition for the round-up method is satisfied. Again, since $m_\text{up} \in \mathbb{U}_N$ we have that $d$ is efficient for the $N$-bit round-up method.
$\square$

The round-up method is slightly more efficient to evaluate, so this is the preferred method. To test if a given divisor is efficient for the round-up method, the condition from theorem 2 or corollary 3 can be used. The following result gives a more efficient condition to check if a given divisor is efficient for the round-up method.

**Lemma 7**: *Let $d$ be a positive integer, $\ell = \lceil \log_2(d) \rceil - 1$, and $m_\text{up} = \lceil \frac{2^{N + \ell}}{d} \rceil$. If $\text{mod}_{2^N}(m_\text{up} \cdot d) \leq 2^\ell$ then $d$ is efficient for the round-up method.*

**Proof**: The product $m_\text{up} \cdot d = \lceil \frac{2^{N + \ell}}{d} \rceil \cdot d$ is the first multiple of $d$ that is equal to or larger than $2^{N + \ell}$. This product will be of the form $2^{N + \ell} + q$ for some $q < d \leq 2^\ell$, so we have $\text{mod}_{2^N}(m \cdot d) = q$. It follows that $2^{N + \ell} \leq m_\text{up} \cdot d \leq 2^{N + \ell} + 2^\ell$ if and only if $\text{mod}_{2^N}(m \cdot d) \leq 2^\ell$.
$\square$

On some architectures it is faster to shift by fewer bits. So, for the purpose of optimization, we might be interested in finding the smallest $\ell$ such that $m_\text{up}$ satisfies the condition of theorem 2 (or the smallest). Surprisingly, there is an easy way to find the smallest $m_\text{up}$ or $m_\text{down}$ that satisfies the condition of theorem 2 or theorem 4.

**Lemma 8**: *Let $d \in \mathbb{U}_N$ be a positive integer that is not a power of two and let $0 < \ell \leq \lfloor \log_2(d) \rfloor$, such that $l, m$ satisfy the condition of theorem 2 (or theorem 4, respectively). If $m$ is odd, this is the smallest $m$ that satisfies this condition. If $m$ is even, $\ell' = \ell - 1, m' = \frac{m}{2}$ also satisfy the condition.*

**Proof**: Suppose that $m$ satisfies the condition of theorem 2. In this case, we have $2^{N + \ell} \leq m \cdot d \leq 2^{N + \ell} + 2^\ell$. It is easy to see that when $m$ is even all expressions in the inequality are even, so we can divide by two and see that $2^{N + \ell - 1} \leq \frac{m}{2} \cdot d \leq 2^{N + \ell - 1} + 2^{\ell - 1}$. The case for an $m$ that satisfies the condition of theorem 4 is analogous.

Suppose that there is a smaller pair $\ell', m'$ that satisfies the condition: $2^{N + \ell'} \leq m' \cdot d \leq 2^{N + \ell'} + 2^{\ell'}$. By multiplying the whole thing by $2^{\ell - \ell'}$, we see that $2^{N + \ell} \leq 2^{\ell - \ell'}m' \cdot d \leq 2^{N + \ell} + 2^\ell$. The set $\{ 2^{N + \ell}, 2^{N + \ell} + 1, ..., 2^{N + \ell} + 2^\ell \}$ has $2^\ell + 1$ elements. We have $2^{N + \ell'} \leq 2^{\lfloor \log_2(d) \rfloor} + 1 < d$ (the last inequality is strict because $d$ is not a power of two), so there can only be one multiple of $d$ in this set, which is $m \cdot d$. So we have $m = 2^{\ell - \ell'} \cdot m'$, so $m$ must be even.
$\square$

TODO: simplify proof of above theorem

So, to find the smallest $m_\text{up}$ that works, we can start by computing $\ell_0 = \lfloor \log_2(d) \rfloor, m_0 = \lceil \frac{2^{N + \ell}}{d} \rceil$ and keep iterating while $m_k$ is even:

$$ m_{k + 1} = \frac{m}{2} $$

$$ \ell_{k + 1} = \ell_k - 1 $$

Once $m_k$ is odd, we set $m_\text{up} = m_k$.

**Example**: ...

**Example**: N=32, d = 641

TODO: division by even divisors which are not efficient for round-up method.


## Implementation

We distinguish between compile-time optimization and runtime optimization of constant unsigned integers. To illustrate, the divisor `d` in the following code is a compile-time constant:
```
	const unsigned int d = 61;
	
	for (int i = 0; i < size; i++) {
		quotient[i] = dividend[i] / d;
	}
```

The divisor `d` in the following code is a runtime constant:
```
	unsigned int d = read_divisor();
	
	for (int i = 0; i < size; i++) {
		quotient[i] = dividend[i] / d;
	}
```

In case of a runtime constant divisor, most compilers will not optimize the division. However, we can do it ourselves, by doing something like:
```
	unsigned int d = read_divisor();
	divdata_t divisor_data = precompute(divisor);
	
	for (int i = 0; i < size; i++) {
		quotient[i] = fast_divide(dividend[i], divisor_data);
	}
```

In this section, we will discuss both how a compiler can generated optimized code and how to efficiently implement the `precompute` and `fast_divide` functions for runtime constant divisors. The [libdivide](https://libdivide.com/) library, which was started by the author of [3] and [4], is a mature implementation that can optimize division by runtime constant integers. The library is conveniently included in a single header file, contains a nice C++ interface, and also implements vector implementations of integer division, which can make division even faster.

During compile-time, there is a lot of room for optimizations. Typically, programmers are OK with waiting a fraction of a second longer if this means that their code executes faster. So for division by compile-time constants, there is time for extensive optimizations. During runtime, time is more precious, so we want to keep the precomputation reasonably efficient. Further, the `fast_divide` function is used for all divisors; We want to keep it efficient and, if possible, branchless.

For compile-time optimization, we can spend some more effort to produce better optimized code. For runtime optimization, we need to do the precomputation in runtime, which means that we want the precomputation to be efficient as well. Simply put, for compile-time optimization it will pay off to distinguish many special cases for which we can produce more efficient code. For runtime optimization the challenge is the other way around: We want to have a single, fast codepath which handles all cases with good efficiency.


### Runtime optimization

Let's take the example from before as a starting point. I will assume that we have a constant `N` which denotes the number of bits in the unsigned integer datatype `uint`. I'll also assume the existence of `big_uint`, an unsigned integer datatype with $2N$ bits. Using these datatypes, the example from before becomes:
```
	uint divisor = get_number_from_user();
	divdata_t divisor_data = precompute(divisor);
	
	for (int i = 0; i < size; i++) {
		quotient[i] = fast_divide(dividend[i], divisor_data);
	}
```

Both the round-up and the round-down method can be implemented using an expression of the form `(n * m + add) >> (N + shift)` to compute $\lfloor \frac{n}{d} \rfloor$, so it is natural to define the struct `divdata_t` with these fields:
```
typedef struct {
	uint mul, add, shift;
} divdata_t;
```

The `fast_divide` function is now straightforward to write:
```
uint fast_divide(uint n, divdata_t dd) {
	big_uint full_product = ((big_uint)n) * dd.mul + dd.add;
	return (full_product >> N) >> dd.shift;
}
```

Let's start with the implementation of the `precompute` function. First, we define a variable `divdata` to hold the result. I'll save you the trouble and note that the case $d = 1$ needs to be handled separately. If we work out this case by hand we see that when we set $\ell = \lceil \log_2(1) \rceil - 1$ we get $m = 2^{N - 1}$ and $\frac{m \cdot n}{2^{N - 1}} = n$. While this is mathematically correct, we can't use the `fast_divide` function to evaluate this expression, because it divides by $2^{N - 1}$ while the `fast_divide` function implements this as a right shift by $N$, followed by another right shift. This last right shift would have to be a right shift by negative one. Technically, you could say that this is a right shift by one, but most architectures don't work this way and even if it would, we would lose a bit that has been shifted out by the first shift.

Luckily, we use a trick. If we set both `divdata.mul` and `divdata.add` to $2^N - 1$, things will work out since
$$ n \cdot 2^N \leq n \cdot (2^N - 1) + 2^N - 1 < n \cdot 2^N $$

So let's write a skeleton for the `precompute` function that only handles the case $d = 1$:
```
divdata_t precompute(uint d) {
	divdata_t divdata;
	
	// d = 1 is a special case
	if (d == 1) {
		divdata.mul = max;
		divdata.add = max;
		divdata.shift = 0;
		return divdata;
	}
	
	// normal path
	...
	
	return divdata;
}
```

For the normal path (that is, all the cases except $d = 1$) we can use the test from TODO:
```
	// normal path
	uint l = ceil_log2(d) - 1;
	uint m_down = (((big_uint)1) << (N + l)) / d;
	uint m_up = (is_power_of_two(d)) ? m_down : m_down + 1;
	uint temp = m_up * d;
	bool use_round_up_method= temp <= (1 << l);
	
	// set fields of divdata
	...
```

Note that the `temp` variable is an $N$-bit unsigned integer. It is just used to compute $m_\text{up} \cdot d$ modulo $2^N$. Also note that we need a special case for when $d$ is a power of two for rounding.

Setting the `mul`, `add`, and `shift` fields of `divdata` is straightforward:
```
	// set fields of divdata
	if (use_round_up_method) {
		divdata.mul = m_up;
		divdata.add = 0;
	}
	else {
		divdata.mul = m_down;
		divdata.add = m_down;
	}
	
	divdata.shift = l;
```

When you stitch the snippets together and include `bits.h` from the appendix you should get a working implementation of `precompute`. However, we can make it a bit more efficient. Note that we have a special case for $d = 1$ and also special cases for when $d$ is a power of two. The trick we used for $d = 1$ actually can be generalized to handle all powers of two. 

We can set `mul = max` and `add = max` so that the high word contains the dividend. Then we only need need to shift the high word right by $\log_2(d)$ bits. Now, we can also compute $\ell = \lfloor \log_2(d) \rfloor$ instead of $\lceil \log_2(d) \rceil - 1$ (which is slightly more efficient), since there is only a difference for powers of two.
```
divdata_t precompute(uint d) {
	divdata_t divdata;
	uint l = floor_log2(d);
	
	if (is_power_of_two(d)) {
		divdata.mul = max;
		divdata.add = max;
    }
	else {
		uint m_down = (((big_uint)1) << (N + l)) / d;
		uint m_up = m_down + 1;
		uint temp = m_up * d;
		bool use_round_up_method = temp <= (1 << l);
		
		if (use_round_up_method) {
			divdata.mul = m_up;
			divdata.add = 0;
		}
		else {
			divdata.mul = m_down;
			divdata.add = m_down;
		}
    }

	divdata.shift = l;
	
	return divdata;
}
```

There are probably lots of optimizations possible, but this is a reasonably simple and efficient version. It is basically what is presented in TODO.


### Compile-time optimization

Here, I will show a sample implementation of the idea. I will pretend we're working on a compiler and implementing the code generation for a division by an unsigned compile-time constant. In particular, I will implement a function `div_by_const_uint` that takes a constant divisor `const uint d` and an expression `expression_t n` that represents the dividend. It will return an expression that represents the instructions that will be executed. As an example, the expression $a \cdot (b + 5)$ can be written as `mul(a, add(b, const(5))` in this way.

TODO: Present all instructions?

As mentioned before, for compile-time optimization we can distinguish a lot of cases to squeeze out every last bit of performance. Some special cases that can be implemented particularly efficient are:
  - Division by one, which can be handled by setting the quotient equal to the dividend.
  - Division by a power of two, which can be implement by a bit shift.
  - Division by an integer larger than half of the maximum value of the dividend, which can be implemented by setting the quotient to zero if it is smaller than the divisor, and to one otherwise.

For divisors that do not fall in one of these special cases, we use fixed-point arithmetic to efficiently implement the division. That is, we either use the round-up method or the round-down method.
```
expression_t div_by_const_uint(expression_t n, const uint d) {
	if (d == 1) return n;
	if (is_power_of_two(d)) return shr(n, constant(floor_log2(d)));
	if (d >= MAX / 2) return gte(n, constant(d));
	return div_fixpoint(d, n);
}
```

Now, we use TODO as a test and select $m$ accordingly:
```
expression_t div_fixpoint(expression_t n, const uint d) {
		uint l = floor_log2(d);
		uint m_down = (((big_uint)1) << (N + l)) / d;
		uint m_up = m_down + 1;
		uint temp = m_up * d;
		bool use_round_up_method = temp <= (1 << l);
		
		uint m = use_round_up_method ? m_up : m_down;
		if (use_round_up_method) {
			while ((m_up & 1) == 0 && l > 0) {
				m_up >>= 1;
				l--;
			}
			return shr(umulhi(m_up, n), l);
		}
		
		if ((d & 1) == 0) {
			// TODO
		}
		
		return 
}
```

The optimized implementation of a division by a divisor that is efficient for the round-up method is a straightforward implementation of the evaluation of the expression TODO. Many processors have an instruction that computes the full $2N$-bit product of two $N$-bit unsigned registers. After that, the register holding the high $N$ bits can be right shifted by $\ell$ bits. Taking the high $N$ bits is equivalent to right shifting $N$ bits, so in total we have right shifted by $k = N + \ell$ bits.

One optimization that we can do, is to shift by the least amount possible. That is, we want to find the smallest $\ell$ such that $m_\text{up}$ satisfies the condition in theorem 2 (or $m_\text{down}$ satisfies the condition in theorem 4, for divisors which are not efficient for the round-up method). We can do this by TODO
```
expression_t optimize_div_fixpoint(const unsigned int d, expression_t n) {
	l = floor_log2(d);
	m_down = ...
	m_up = m_down + 1;
	multiple = d * m_up
	if (multiple < (1 << l)) {
		shift = l;
		// efficient for round-up method
		while (multiple / 2 <=
	}
}
```

For a divisor that is not efficient for the round-up method the situation is not as simple. The round-down method is slightly more expensive to use than the round-up method. For even divisors that are not efficient, we can apply a trick to use the round-up method: Instead of calculating $\lfloor \frac{n}{d} \rfloor$, we calculate $\lfloor \frac{\lfloor \frac{n}{2} \rfloor}{\lfloor \frac{d}{2} \rfloor} \rfloor$. In this case, the dividend and divisor are $(N - 1)$-bit numbers. Using theorem 2 and theorem 7 with $N - 1$ instead of $N$ bits, we see that $m_\text{up} = \lceil \frac{2^{TODO}}{d} \rceil$ now fits in $N$ bits.

Optimizing division by a divisor that is not efficient for the round-up method is not as straightforward. By theorem TODO, we can use the round-down method, but this has a subtle quirk. When we implement the expression TODO, $(n+1)$ will overflow when $n$ equals the maximum value $2^N - 1$. Instead, we can implement this as $m \cdot n + m$. This is an addition of two $2N$-bit integers, and it depends on the target processor how efficiently this can be implemented. On most processors this can be done by an $N$-bit addition followed by an $N$-bit addition with carry. Other processors have dedicated instructions for adding $2N$-bit numbers (for example, this is the case when optimizing division of 32-bit numbers on a 64-bit architecture). Some instruction sets even support integer fused multiply-add instructions (or even high word of ...), which can be used ... TODO

In [4] another approach is presented, where a saturating increment is used. This saturating increment only increments the dividend when it is less than the maximum value $2^N - 1$. So it effectively computes TODO when $n = 2^N - 1$ and TODO otherwise. The difference will only matter when $d$ is a divisor of $2^N - 1$, and according to theorem 10 (see appendix), every divisor of $2^N - 1$ is efficient for the round-up method. So if we prefer the round-up method for divisors that are efficient for the round-up method, this approach will always give the correct result. On most architectures, the saturating increment can be implemented by an N-bit increment followed by an N-bit decrement with borrow instruction.

A division by one can be handled by setting the quotient equal to the dividend. A division by a power of two can be handled 

Listing the cases roughly in order of efficiency, we have:

  1. Division by one, which can be implemented as a move or no-op.
  2. Division by a power of two, which can be implemented as a bit shift.
  3. Division by an integer that is larger than half of the maximum value of the dividend, which can be implemented as a comparison.
  4. Division by a divisor that is efficient for the round-up method.
  5. Division by an even divisor that is not efficient for the round-up method
  6. Division by a divisor that is efficient for the round-down method.


### Testing

When writing code for $N = 8$ or $N = 16$, I recommend that you check the result is correct for every pair $n, d \in \mathbb{U}_N$ with $d > 0$. This can be as simple as:
```
void main(void) {
	for (uint d = 1; true; d++) {
		divdata_t dd = precompute(d);
		for (uint n = 0; true; n++) {
			assert(fast_divide(n, dd) == n / d);
			if (n == max) break;
		}
		if (d == max) break;
	}
}
```
For $N = 8$, this code will run more or less instantly, for $N = 16$ it will take a couple of minutes. For $N = 32$ this program won't terminate anytime soon. However, what you can you is test, for every divisor $d \in \mathbb{U}_N$ with $d > 0$, all numbers of the form $n \cdot d, n \cdot d - 1 \in \mathbb{U}_N$:
```
TODO
```

This will still take a long time, but it is feasible; The above code took slightly over 40 minutes to run on my machine.

For $N = 64$, exhaustive testing is completely infeasible. I recommend making a set $S$ of special values consisting of all numbers from 0 up to 256, all numbers of the form $2^k - 1$, $2^k$, $2^k + 1$, the divisors of these numbers, and the divisors of $2^{64} + 1$. Then we can test if the result is correct for each $n \in S$, $d \in S \setminus \{ 0 \}$. On top of that, you can run some randomized testing. I recommend that you pick $n$ and $d$ uniformly random in $\mathbb{U}_{64}$ and then mask out each byte with some probability. Then, you can run random tests for some time. If you find a bug in the implementations, add $n$ and $d$ for which the result is not correct to $S$ and verify that your testset triggers the bug before you fix the bug.


## References and further reading

The classic reference for optimization of division by both signed and unsigned integers is [1], which states and proves theorem 2 (round-up method). In [2], the approach is extended to the round-down method. The resources [3] and [4] are by far the easiest to read, and cover essentially everything in this article in a more accessible way. They sacrifice some rigor and completeness, though. If you are interested in optimizing division, I recommend reading these articles as your starting point. Finally, in [5] it is shown that we can also use fixed point math to compute the modulo operation.

[1]: [Division by Invariant Integers using Multiplication](https://gmplib.org/~tege/divcnst-pldi94.pdf), Torbjörn Granlund and Peter L. Montgomery, 1994.

[2]: [N-Bit Unsigned Divison Via N-Bit Multiply-Add](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.512.2627&rep=rep1&type=pdf), Arch D. Robinson, 2005.

[3]: [Labor of Divison (Episode I)](https://ridiculousfish.com/blog/posts/labor-of-division-episode-i.html), fish, 2010.

[4]: [Labor of Divison (Episode III): Fast Unsigned Division by Constants](https://ridiculousfish.com/blog/posts/labor-of-division-episode-iii.html), fish, 2011.

[5]: [Faster Remainder by Direct Computation: Applications to Compilers and Software Libraries](https://arxiv.org/pdf/1902.01961), Daniel Lemire, Owen Kaser, Nathan Kurz, 2019.

TODO: Put this lemma somewhere it fits.

**Lemma 10**: *If $d$ is a divisor of $2^N - 1$, then $d$ is efficient for the $N$-bit round-up method.*

**Proof**: Take $\ell = \lfloor \log_2(d) \rfloor$ and define $e = \text{mod}_d(-2^{N + 1})$ and $e' = \text{mod}_d(2^{N + 1})$. Using $\text{mod}_d(2^N) = 1$ and $2^{\lfloor \log_2(d) \rfloor} < d$, we see:
$$ e' = \text{mod}_d(2^{N + \ell}) = \text{mod}_d(2^N \cdot 2^{\lfloor \log_2(d) \rfloor}) = \text{mod}_d(2^{\lfloor \log_2(d) \rfloor}) = 2^{\lfloor \log_2(d) \rfloor} $$

Here, we used that $2^{\lfloor \log_2(d) \rfloor} < d$. Since $d$ is a divisor of $2^N - 1$, it can't be a power of two, so we have $e, e' \in \{ 1, 2, ..., d - 1 \}$. It follows that $e + e' = d$ since $e + e' \equiv 0 \mod d$. We find that
$$ e = d - e' \leq 2^{\lceil \log_2(d) \rceil} - 2^{\lfloor \log_2(d) \rfloor} = 2^{\lfloor \log_2(d) \rfloor} = 2^\ell $$

So the condition $e = \text{mod}_d(2^{N + \ell}) \leq 2^\ell$ for corollary 5 is satisfied. Since $d$ is not a power of two, we have that $\ell = \lfloor \log_2(d) \rfloor = \lceil \log_2(d) \rceil - 1$. By using lemma 6, we now see that $m_\text{up} \in \mathbb{U}_N$ for $\ell = \lfloor \log_2(d) \rfloor$. So $d$ is efficient for the $N$-bit round-up method.
$\square$

<div class="pagebreak"></div>

## Appendices

### Appendix A: Related results

This appendix shows some related results which are not necessary to understand the state-of-the-art algorithms presented in the main article. They are meant for the interested reader.

The following theorem tells us that the round-up method is *almost* efficient for any divisor: Independently of the divisor $d$, we can store $m_\text{up}$ in at most $N + 1$ bits.

**Corollary 3**: *If $\text{mod}_d(-2^{N + \ell}) \leq 2^\ell$ and $m_\text{up} = \lceil \frac{2^{N + \ell}}{d} \rceil$, then $\lfloor \frac{m_\text{up} \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.*

**Proof**: Suppose we have $\text{mod}_d(-2^{N + \ell}) < 2^\ell$. Then there exists an integer $m$ such that $0 \leq m \cdot d - 2^{N + \ell} \leq 2^\ell$ Adding $2^{N + \ell}$, we see that $2^{N + \ell} \leq m \cdot d \leq 2^{N + \ell} + 2^\ell$. In particular, the lowest $m$ that satisfies this equation is $m = \lceil \frac{2^{N + \ell}}{d} \rceil$, since $d \cdot \lceil \frac{2^{N + \ell}}{d} \rceil$ is the first multiple of $d$ that is greater than or equal to $2^{N + \ell}$. By theorem 2, it follows that $\lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

**Corollary 5**: If $0 < \text{mod}_d(2^{N + \ell}) < 2^\ell$ and $m = \lfloor \frac{2^{N + \ell}}{d} \rfloor$, then $\lfloor \frac{m \cdot (n + 1)}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.

**Proof**: If $0 < \text{mod}_d(2^{N + \ell}) \leq 2^\ell$ and $m - \lfloor \frac{2^{N + \ell}}{d} \rfloor$, then $\lfloor \frac{m \cdot (n + 1)}{2^{N + \ell}} = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$. Subtracting $2^{N + \ell}$, we see that $-2^{N + \ell} < -m \cdot d \leq -2^{N + \ell} + 2^\ell$. Multiplying by $-1$, we get $2^{N + \ell} - 2^\ell \leq m \cdot d < 2^{N + \ell}$. In particular, this condition must hold for $m = \lfloor \frac{2^{N + \ell}}{d}$, since $d \cdot \lfloor \frac{2^{N + \ell}}{d} \rfloor$ is the biggest multiple of $d$ that is less than $2^{N + \ell}$. By theorem 4, it follows that $\lfloor \frac{m \cdot (n + 1)}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$


### Appendix B: Header files

`bits.h`:
```
#ifndef BITS_H
#define BITS_H

#include <stdint.h>
#include <stdbool.h>
#include <assert.h>

#ifndef N
#undef N
#define N 32
#endif

#if N == 8
typedef uint8_t uint;
typedef uint16_t big_uint;
#endif

#if N == 16
typedef uint16_t uint;
typedef uint32_t big_uint;
#endif

#if N == 32
typedef uint32_t uint;
typedef uint64_t big_uint;
#endif

const uint max = (uint)((int)-1);

uint floor_log2(uint x) {
	assert(x > 0);
	uint count = 0;
#if N >= 64
	if (x & 0xffffffff00000000) { x >>= 32; count += 32; }
#endif
#if N >= 32
	if (N >= 32 && x & 0x00000000ffff0000) { x >>= 16; count += 16; }
#endif
#if N >= 16
	if (N >= 16 && x & 0x000000000000ff00) { x >>= 8; count += 8; }
#endif
	if (N >= 8 && x & 0x00000000000000f0) { x >>= 4; count += 4; }
	if (x & 0x000000000000000c) { x >>= 2; count += 2; }
	if (x & 0x0000000000000002) { x >>= 1; count += 1; }
	return count;
}

uint ceil_log2(uint x) {
	uint floor = floor_log2(x);
	assert(x >= ((uint)1 << floor));
	if (x > ((uint)1 << floor)) floor++;
	return floor;
}

bool is_power_of_two(uint k) {
	return k && (k & (k - 1)) == 0;
}

#endif
```
