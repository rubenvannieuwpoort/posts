# Division by constant unsigned integers

Most modern processors have an integer divide instruction which, for technical reasons, is very slow compared to other integer arithmetic operations. When the divisor is constant, it is possible to transform the division instruction to other instructions which execute faster. Most optimizing compilers perform this optimization, as can be seen on [Matt Godbolt’s compiler explorer](https://godbolt.org/z/xrYsbs).

There are some tricks to divide by special divisors: A division by one can be ignored, a division by a power of two can be replaced by a bit shift, and a division by a number that is larger than half of the maximum value of the datatype of the dividend can be replaced by a comparison. By using fixed-point arithmetic, we can speed up division by all constant divisors. However, this is not as simple, and for this reason I will focus only on positive or 'unsigned' divisors.

When it is necessary to repeatedly divide floating point numbers by the same constant $c$, it is often preferred to precompute $\frac{1}{c}$ and multiply by this instead. This is usually a lot more efficient. We can do something analogous for integers by using [fixed point arithmetic](https://en.wikipedia.org/wiki/Fixed-point_arithmetic). The basic idea is to pick a large constant $L$ that is easy to divide by. In decimal you can take some power of ten, in binary you would take a power of two. If we would have $c = \frac{L}{d}$ we would have $\frac{n \cdot c}{L} = \frac{n \cdot \frac{L}{d}}{L} = \frac{n}{d}$. However, $\frac{L}{d}$ is usually not an integer, and we need to round. Still, if $c \approx \frac{L}{d}$ we still expect that $\frac{n \cdot c}{L} \approx \frac{n}{d}$.

In the following section, I'll discuss the mathematical background. In the sections after that, I'll discuss how division by constant unsigned integers can be optimized at compile-time and at runtime.


## Mathematical background

### Preliminaries

In this article, I will assume that we optimize for an $N$-bit machine. By this, I mean that I will assume that the normal integer datatype has $N$ bits. I will also assume that the machine can efficiently compute the full $2N$-bit product of two $N$-bit unsigned integers. I will use the notation $\mathbb{U}_N$ for the set of unsigned integers that can be represented with $N$ bits. That is,
$$ \mathbb{U}_N = \{ 0, 1, ..., 2^N - 1 \} $$

I will use the notation $\text{mod}_d(n)$ to denote the integer in the range $\{ 0, 1, ..., d - 1 \}$ that is equivalent to $n$ modulo $d$. That is, $\text{mod}_d(n)$ is the unique integer such that
$$ 0 \leq \text{mod}_d(n) < d $$

$$ \text{mod}_d(n) = n + m \cdot d $$

for some $m \in \mathbb{Z}$.

The following lemma will be very useful:

**Lemma 1**: *Suppose that $n, d \in \mathbb{N}$ with $d > 0$. If $\frac{n}{d} \leq x < \frac{n + 1}{d}$ then $\lfloor x \rfloor = \lfloor \frac{n}{d} \rfloor$.*

**Proof**: We have $\frac{n}{d} = \lfloor \frac{n}{d} \rfloor + \frac{k}{d}$ for some nonnegative integer $k < d$. So $\frac{n}{d} \leq \lfloor \frac{n}{d} \rfloor + 1$. It follows that
$$ \frac{n}{d} \rfloor x < \lfloor \frac{n}{d} \rfloor + 1 $$

It follows that $x \in [ \lfloor \frac{n}{d} \rfloor, \lfloor \frac{n}{d} \rfloor + 1)$, so that $\lfloor x \rfloor = \lfloor \frac{n}{d} \rfloor$.
$\square$


### ???

With the preliminaries out of the way, let's continue to the meat of the article. In the introduction, we have already established that we want to compute some $m \approx \frac{2^k}{d}$ for some large enough $k$ so that $\frac{m \cdot n}{2^k} \approx \frac{n}{d}$. We want the evaluation of $\frac{m \cdot n}{2^k}$ to be efficient, therefore we demand that we round down $\frac{m \cdot n}{2^k}$, since in this case we can use a bit shift.

Since for any $d > 1$ we can pick $k \leq N$ and still have $m \in \mathbb{U}_N$, we will set $k = N + l$ for some nonnegative integer $l$. Now, the remaining question is how to pick $l$ and $m$ so that $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$.  Let's focus on $m$ first. Reasonable choices are to round either up or down: $m = \lfloor \frac{2^{N + l}}{d} \rfloor$ or $m = \lceil \frac{2^{N + l}}{d} \rceil$.

If we set $m = \lfloor \frac{2^{N + l}}{d} \rfloor$, we will have $m < \frac{2^{N + l}}{d}$ so that $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor < \lfloor \frac{n}{d} \rfloor$. So this clearly won't work. However, we will see that simply using $n + 1$ instead of $n$ is a viable alternative. So we now have the following two methods:
  1. Round-up method: Take $m = \lceil \frac{2^{N + l}}{d} \rceil$ and approximate $\lfloor \frac{n}{d} \rfloor$ by $\lfloor \frac{m \cdot n}{2^{N + l}} \lfloor$
  2. Round-down method: Take $m = \lfloor \frac{2^{N + l}}{d} \rfloor$ and approximate $\lfloor \frac{n}{d} \rfloor$ by $\lfloor \frac{m \cdot (n + 1)}{2^{N + l}} \lfloor$

We now demand that these methods are *correct*. That is, we should have $\lfloor \frac{m \cdot n}{2^{N + l}} \lfloor = \lfloor \frac{n}{d} \rfloor$ in case of the round-up method, and $\lfloor \frac{m \cdot (n + 1)}{2^{N + l}} \lfloor = \lfloor \frac{n}{d} \rfloor$ in case of the round-down method. Furthermore, we want the methods to be *efficient*. For that, we demand that $m \in \mathbb{U}_N$.

Now, I will present some theorems about the conditions under which the methods are correct.

**Theorem 2 (Round-up method)**: *Let $d, m, N, l \in \mathbb{N}$ be nonnegative integers with $d > 0$. If*
$$ 2^{N + l} \leq m \cdot d \leq 2^{N + l} + 2^l $$

*then*
$$ \forall n \in \mathbb{U}_N : \lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor $$

**Proof**: Multiplying the inequality by $\frac{n}{d \cdot 2^{N + l}}$ we get
$$ \frac{n}{d} \leq \frac{m \cdot n}{2^{N + l}} \leq \frac{n}{d} + \frac{1}{d} \cdot \frac{n}{2^N} $$

For all $n \in \mathbb{U}_N$ we have $n < 2^N$, so that $\frac{n}{2^N} < 1$. It follows that
$$ \forall n \in \mathbb{U}_N : \frac{n}{d} \leq \frac{m \cdot n}{2^{N + l}} \leq \frac{n}{d} + \frac{1}{d} $$

By lemma 1, it follows that $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

The following corollary makes clear why I call theorem 2 the "round-up method".

**Corollary 3**: *If $\text{mod}_d(-2^{N + l}) \leq 2^l$ and $m = \lceil \frac{2^{N + l}}{d} \rceil$, then $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.*

**Proof**: Suppose we have $\text{mod}_d(-2^{N + l}) < 2^l$. Then there exists an integer $m$ such that $0 \leq m \cdot d - 2^{N + l} \leq 2^l$ Adding $2^{N + l}$, we see that $2^{N + l} \leq m \cdot d \leq 2^{N + l} + 2^l$. In particular, the lowest $m$ that satisfies this equation is $m = \lceil \frac{2^{N + l}}{d} \rceil$, since $d \cdot \lceil \frac{2^{N + l}}{d} \rceil$ is the first multiple of $d$ that is greater than or equal to $2^{N + l}$. By theorem 2, it follows that $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

**Example**: TODO

**Example**: TODO

The following theorem is a natural counterpart of theorem 2. I haven’t found this exact theorem anywhere else, though analogues have been proven in [2] and [4].

**Theorem 4**: *Let $d, m, N, l \in \mathbb{N}$ be nonnegative integers with $d > 0$. If*
$$ 2^{N + l} - 2^l \leq m \cdot d < 2^{N + l}$$

*then*
$$ \forall n \in \mathbb{U}_N : \lfloor \frac{m \cdot (n + 1)}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor $$

**Proof**: Multiply the inequality by $\frac{n + 1}{d \cdot 2^{N + l}}$ to get
$$ \frac{n}{d} + \frac{1}{d} \cdot \left( 1 - \frac{n + 1}{2^N} \right) \leq \frac{m \cdot (n + 1)}{2^{N + l}} < \frac{n + 1}{d} $$

Looking at the expression on the left side, we have $1 \leq n + 1 \leq 2^N$, so that $0 \leq 1 - \frac{n + 1}{2^N} < 1$. It follows that $\frac{n}{d} \leq \frac{n}{d} + \frac{1}{d} \cdot (1 - \frac{n + 1}{2^N})$, so
$$ \frac{n}{d} \leq \frac{m \cdot (n + 1)}{2^{N + l}} < \frac{n + 1}{d} $$

for all $n \in \mathbb{U}_N$. So the lemma applies and we have $\lfloor \frac{m \cdot (n + 1)}{2^{N + l}} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

**Corollary 5**: If $0 < \text{mod}_d(2^{N + l}) < 2^l$ and $m = \lfloor \frac{2^{N + l}}{d} \rfloor$, then $\lfloor \frac{m \cdot (n + 1)}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.

**Proof**: If $0 < \text{mod}_d(2^{N + l}) \leq 2^l$ and $m - \lfloor \frac{2^{N + l}}{d} \rfloor$, then $\lfloor \frac{m \cdot (n + 1)}{2^{N + l}} = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$. Subtracting $2^{N + l}$, we see that $-2^{N + l} < -m \cdot d \leq -2^{N + l} + 2^l$. Multiplying by $-1$, we get $2^{N + l} - 2^l \leq m \cdot d < 2^{N + l}$. In particular, this condition must hold for $m = \lfloor \frac{2^{N + l}}{d}$, since $d \cdot \lfloor \frac{2^{N + l}}{d} \rfloor$ is the biggest multiple of $d$ that is less than $2^{N + l}$. By theorem 4, it follows that $\lfloor \frac{m \cdot (n + 1)}{2^{N + l}} = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

**Example**: TODO




Now that we have established under which conditions the methods are correct, let's turn to the question of efficiency. First, let's make our notion formal:

**Definition:** *We call a positive divisor $d \in \mathbb{U}_d$ **efficient** for the $N$-bit round-up method (or round down method) if there exists an $l \in \mathbb{N}$ and an $m \in \mathbb{U}_N$ such that for all $n \in \mathbb{U}_N$, we have $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ ($\lfloor \frac{m \cdot (n + 1)}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for the round-down method).*

We will use the following lemma about the number of bits that we need for the magic number $m$.

**Lemma:** *Let $d \in \mathbb{U}_N$ and define $x = \frac{2^{N + 1 - \lceil \log_2(d) \rceil}}{d}$. Now*
$$2^{N - 1} \leq \lfloor x \rfloor \leq \lceil x \rceil \leq 2^N - 1$$

That is, the binary representations of $\lfloor x \rfloor$ and $\lceil x \rceil$ have exactly $N$ bits.

**Proof:** TODO
$\square$

The following theorem states that for any $N$-bit positive divisor, there is an $(N + 1)$-bit magic number that works for the round-up method. In other words, for any given divisor, the round-up method is *almost* efficient.

**Theorem:** *For any positive divisor $d \in \mathbb{U}_N$, there exist $l \in \mathbb{N_+}$, $m \in \mathbb{U}_{N + 1}$ such that $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.*

**Proof**: Set $l = \lceil \log_2(d) \rceil$. The range $\{ 2^N, 2^N + 1, ..., 2^N + 2^l \}$ consists of $2^l + 1$ consecutive numbers. We have $2^l + 1 = 2^{} + 1 > d$, so there must be a multiple of $d$ in this range. Using theorem 2, we see that there exists an $m$ such that $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$. It remains to show that $m \in \mathbb{U}_{N + 1}$. TODO
$\square$

The following theorem says that for any given positive divisor, either the round-up method or the round-down method is efficient.

theorem: every d is either efficient for the round-up or round-down method















We can use theorem 2 and 4 and corollary 3 and 5 to see if a given divisor is efficient for the round-up method. The following condition can be used to implement a more efficient test.

**Lemma 7**: *Let $d$ be a cooperative divisor and let $l = \lceil \log_2(d) \rceil - 1$. If $\text{mod}_{2^N}(\lceil \frac{2^{N + l}}{d} \rceil \cdot d) \leq 2^l$ then $d$ is a cooperative divisor.*

**Proof**: The product $m \cdot d = \lceil \frac{2^{N + l}}{d} \rceil \cdot d$ is the first multiple of $d$ that is equal to or larger than $2^{N + l}$. So this product will be of the form $2^{N + l} + q$ for some $q < d < 2^N$. We have $\text{mod}_{2^N}(m \cdot d) = q$. It follows that $2^{N + l} \leq m \cdot d \leq 2^{N + l} + 2^l$ if and only if $\text{mod}_{2^N}(m \cdot d) \leq 2^l$.
$\square$

Finally, the following theorem will be useful for the implementation.

**Theorem 8**: *If $d$ is a divisor of $2^N - 1$, then $d$ is a cooperative divisor.*

**Proof**: TODO
$\square$


## Implementation

### Division by runtime constants

### Code generation

**Example**: TODO

**Example**: TODO

## References

[1]: [Division by Invariant Integers using Multiplication](https://gmplib.org/~tege/divcnst-pldi94.pdf), Torbjörn Granlund and Peter L. Montgomery, 1994.

[2]: [N-Bit Unsigned Divison Via N-Bit Multiply-Add](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.512.2627&rep=rep1&type=pdf), Arch D. Robinson, 2005.

[3]: [Labor of Divison (Episode I)](https://ridiculousfish.com/blog/posts/labor-of-division-episode-i.html), fish, 2010.

[4]: [Labor of Divison (Episode III): Fast Unsigned Division by Constants](https://ridiculousfish.com/blog/posts/labor-of-division-episode-iii.html), fish, 2011.

[5] [Faster Remainder by Direct Computation: Applications to Compilers and Software Libraries](https://arxiv.org/pdf/1902.01961), Daniel Lemire, Owen Kaser, Nathan Kurz, 2019.
