
# Division by constant unsigned integers

Most modern processors have an integer divide instruction which, for technical reasons, is very slow compared to other integer arithmetic operations. When the divisor is constant, it is possible to transform the division instruction to other instructions which execute faster. Most optimizing compilers perform this optimization, as can be seen on [Matt Godbolt’s compiler explorer](https://godbolt.org/z/xrYsbs).

There are some straightforward tricks to divide by special divisors: A division by one can be ignored, a division by a power of two can be replaced by a bit shift, and a division by a number that is larger than half of the maximum value of the datatype of the dividend can be replaced by a comparison. By using fixed-point arithmetic, we can speed up division by all constant divisors. However, this is not as simple, and for this reason I will focus only on positive or 'unsigned' divisors.

If many divisions by a floating-point constant $d$ need to be done, we can precompute $c = \frac{1}{d}$ and multiply by $c$ instead of dividing by $d$. With integer arithmetic, we can do something similar. If we just set $c = \frac{1}{d}$ we will get either zero or one, depending on $d$ and how we round. The solution is to use [fixed point arithmetic](https://en.wikipedia.org/wiki/Fixed-point_arithmetic). The basic idea is to pick a large constant $L$ that is easy to divide by. In decimal you can take some power of ten, in binary you would take a power of two. If we would have $c = \frac{L}{d}$ we would have $\frac{n \cdot c}{L} = \frac{n \cdot \frac{L}{d}}{L} = \frac{n}{d}$. However, $\frac{L}{d}$ is usually not an integer, and we need to round. Still, if $c \approx \frac{L}{d}$ we still expect that $\frac{n \cdot c}{L} \approx \frac{n}{d}$.

If this confuses you, consider the following example. In base ten, we expect that the result of multiplying a number $n$ by 0.3333 approximates $\frac{n}{3}$. Since $0.3333 = \frac{3333}{100000}$ we would expect that $\lfloor \frac{n \cdot 3333}{10000} \rfloor \approx \lfloor \frac{n}{3} \rfloor$ In binary, division by powers of two can be done efficiently. So, working with $N$-bit numbers, we take some value $k$ and compute $\lfloor \frac{n}{d} \rfloor$ as $\lfloor \frac{n \cdot m}{2^k} \rfloor$, where $m \approx \frac{2^k}{d}$.


## Mathematical background

Throughout this article, $N$ denotes the number of bits in a bit. Furthermore, I will use the notation $\mathbb{U}_N$ for the set of unsigned integers that can be represented with $N$ bits. That is,
$$ \mathbb{U}_N = \{ 0, 1, ..., 2^N - 1 \} $$

I will use the notation $\text{mod}_d(n)$ to denote the integer in the range $\{ 0, 1, ..., d - 1 \}$ that is equivalent to $n$ modulo $d$. That is, $\text{mod}_d(n)$ is the unique integer such that
$$ 0 \leq \text{mod}_d(n) < d $$

$$ \text{mod}_d(n) = n + m \cdot d $$

for some $m \in \mathbb{Z}$.

The following lemma will be useful.

**Lemma 1**: *Suppose that $n, d \in \mathbb{N}$ with $d > 0$. If $\frac{n}{d} \leq x < \frac{n + 1}{d}$ then $\lfloor x \rfloor = \lfloor \frac{n}{d} \rfloor$.*

**Proof**: We have $\frac{n}{d} = \lfloor \frac{n}{d} \rfloor + \frac{k}{d}$ for some nonnegative integer $k < d$. So $\frac{n}{d} \leq \lfloor \frac{n}{d} \rfloor + 1$. It follows that
$$ \frac{n}{d} \rfloor x < \lfloor \frac{n}{d} \rfloor + 1 $$

It follows that $x \in [ \lfloor \frac{n}{d} \rfloor, \lfloor \frac{n}{d} \rfloor + 1)$, so that $\lfloor x \rfloor = \lfloor \frac{n}{d} \rfloor$.
$\square$

The following theorem gives a condition to check if $m$ can be used to optimise division by a constant divisor $d$. The theorem is stated in [1], although the proof is adapted to be shorter and easier to understand.

**Theorem 2**: *Let $d, m, N, l \in \mathbb{N}$ be nonnegative integers with $d > 0$. If*
$$ 2^{N + l} \leq m \cdot d \leq 2^{N + l} + 2^l $$

*then*
$$ \forall n \in \mathbb{U}_N : \lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor $$

**Proof**: Multiplying the inequality by $\frac{n}{d \cdot 2^{N + l}}$ we get
$$ \frac{n}{d} \leq \frac{m \cdot n}{2^{N + l}} \leq \frac{n}{d} + \frac{1}{d} \cdot \frac{n}{2^N} $$

For all $n \in \mathbb{U}_N$ we have $n < 2^N$, so that $\frac{n}{2^N} < 1$. It follows that
$$ \forall n \in \mathbb{U}_N : \frac{n}{d} \leq \frac{m \cdot n}{2^{N + l}} \leq \frac{n}{d} + \frac{1}{d} $$

By lemma 1, it follows that $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

**Corollary 3**: *If $\text{mod}_d(-2^{N + l}) \leq 2^l$ and $m = \lceil \frac{2^{N + l}}{d} \rceil$, then $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.*

**Proof**: Suppose we have $\text{mod}_d(-2^{N + l}) < 2^l$. Then there exists an integer $m$ such that $0 \leq m \cdot d - 2^{N + l} \leq 2^l$ Adding $2^{N + l}$, we see that $2^{N + l} \leq m \cdot d \leq 2^{N + l} + 2^l$. In particular, the lowest $m$ that satisfies this equation is $m = \lceil \frac{2^{N + l}}{d} \rceil$, since $d \cdot \lceil \frac{2^{N + l}}{d} \rceil$ is the first multiple of $d$ that is greater than or equal to $2^{N + l}$. By theorem 2, it follows that $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

**Example**: TODO

**Example**: TODO

The following theorem is a natural counterpart of theorem 2. I haven’t found this exact theorem anywhere else, though analogues have been proven in [2] and [4].

**Theorem 4**: *Let $d, m, N, l \in \mathbb{N}$ be nonnegative integers with $d > 0$. If$
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

Theorem 4 is short and easy to prove, but not very insightful. The following exercises guide you through the approach taken in [3], which is more instructive.

**Exercise**: TODO

**Exercise**: TODO

We can use the following lemma to figure out how many bits are needed to store the magic number $m$.

**Lemma 6**: *Let $d, k \in \mathbb{N}_+$ with $d \leq 2^k$. Then the binary representations of $\lfloor \frac{2^k}{d} \rfloor$ and $\lceil \frac{2^k}{d} \rceil$ have $b = k + 1 - \lceil \log_2(d) \rceil$ bits. That is,*
$$2^{b - 1} \leq \lfloor \frac{2^k}{d} \leq \lceil \frac{2^k}{d} \rceil \leq 2^b - 1 $$

**Proof**: TODO

Ideally, $m$ fits in a single word. We can ensure that an $m$ that satisfies the condition in theorem 2 exists when there are at least $d$ numbers in the range $\{ 2^N, 2^N + 1, ..., 2^l \}$. To ensure this, we can set $l = \lceil \log_2(d - 1) \rceil$.

Assuming that $d$ is not of the form $2^p + 1$, we have $\lceil \log_2(d - 1) \rceil = \lceil \log_2(d) \rceil$. So in the worst case $\lceil \log_2(d) \rceil$ is the first value for $l$ for which there is an $m$ that satisfies the condition in theorem 2. In this case we need $b = N + 1$ bits, which is a single bit too many to fit in an $N$-bit word.

The following definition defines a name for divisors for which we can use theorem 2 with a magic number that fits in $N$ bits.

**Definition**: *If $d \in \mathbb{N}$ is a divisor such that there exist $l, m \in \mathbb{N}$ such that $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N​$, then $d$ is called a **cooperative divisor**.*

So, for cooperative divisors it is straightforward to apply theorem 2. For non-cooperative divisors, we can use a technique introduced in [1] to multiply an $N$-bit unsigned number by an $(N + 1)$-bit unsigned number. As we will later see, it is more efficient to use theorem 4 for uncooperative divisors. I will document the trick here anyway. To obtain $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor$, where $n$ is an $N$-bit word and $2^{N - 1} \leq m \leq 2^N - 1$, the following function can be used:
```
uint divide(uint n) {
	uint hiword = (m' * n) >> N;
	return (hiword + ((n - hiword) >> s1)) >> s2;
}
```

where $m' = m - 2^N$, $s_1 = \min(l, 1)$, and $s_2 = \max(l - 1, 0)$.

If we combine theorem 2 and theorem 4, it will become clear that we never need to deal with $(N + 1)$-bit constants. Setting  l = \lceil\log_2(d) \rceil - 1l=⌈log2​(d)⌉−1  we see from lemma 6 that the number of bits will be $b = N + l + 1 - \lceil \log_2(d) \rceil = N$. The range $\{ 2^{N + l} - 2^l, 2^{N + l} - 2^l + 1, ..., 2^{N + l} + 2^l \}$ contains $2^{l + 1} + 1 = 2^{\lceil \log_2(d) \rceil} + 1$ integers. Since  $2^{\lceil \log_2(d) \rceil} + 1 > d$, there must be at least one multiple of $d$ in this range. If this multiple is in the range $\{ 2^{N + l} - 2^l, 2^{N + l} - 2^l + 1, ..., 2^{N + l} - 1 \}$ we can apply theorem 4. Otherwise,  the multiple must be in the range $\{ 2^{N + l}, 2^{N + l} + 1, ..., 2^{N + l} + 2^l \}$ and we can apply theorem 2. In any case we find an $m$ which fits in a single $N$-bit word.

The following condition can be used to decide if a divisor is cooperative or not.

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
