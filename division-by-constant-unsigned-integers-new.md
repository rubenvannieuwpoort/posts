# Division by constant unsigned integers

Most modern processors have an integer divide instruction which, for technical reasons, is very slow compared to other integer arithmetic operations. When the divisor is constant, it is possible to transform the division instruction to other instructions which execute faster. Most optimizing compilers perform this optimization, as can be seen on  [Matt Godbolt’s compiler explorer](https://godbolt.org/z/xrYsbs).

There are some straightforward tricks to divide by special divisors: A division by one can be ignored, a division by a power of two can be replaced by a bit shift, and a division by a number that is larger than half of the maximum value of the datatype of the dividend can be replaced by a comparison.

A less straightforward, but more general trick is to use fixed-point arithmetic. When using real numbers, a division by $x$ can be replaced by a multiplication by $\frac{1}{x}$. This is more efficient, since multiplication is a faster operation than division.

With integer arithmetic, we can do something similar. In base ten, we expect that the result of multiplying a number $n$ by 0.3333 approximates $\frac{n}{3}$. Since $0.3333 = \frac{3333}{100000}$ we would expect that $\lfloor \frac{n \cdot 3333}{10000} \rfloor \approx \lfloor \frac{n}{3} \rfloor$ In binary, division by powers of two can be done efficiently. So, working with $N$-bit numbers, we take some value $k$ and compute $\lfloor \frac{n}{d} \rfloor$ as $\lfloor \frac{n \cdot m}{2^k} \rfloor$, where $m \approx \frac{2^k}{d}$.

Of course, if we want to apply this trick in our programs, we’d better be sure it works. So we need to do some error analysis to know exactly for which numbers this optimization works.

## Mathematical background

Throughout this article, $N$ denotes the number of bits in a bit. Furthermore, I will use the notation $\mathbb{U}_N$ for the set of unsigned integers that can be represented with $N$ bits. That is,
$$ \mathbb{U}_N = \{ 0, 1, ..., 2^N - 1 \} $$

I will use the notation $\text{mod}_d(n)$ to denote the integer in the range $\{ 0, 1, ..., d - 1 \}$ that is equivalent to $n$ modulo $d$. That is, $\text{mod}_d(n)$ is the unique integer such that
$$ 0 \leq \text{mod}_d(n) < d $$

$$ \text{mod}_d(n) = n + m \cdot d $$

for some $m \in \mathbb{Z}$.

The following lemma will be useful.

**Lemma 1:** *Suppose that $n, d \in \mathbb{N}$ with $d > 0$. If $\frac{n}{d} \leq x < \frac{n + 1}{d}$ then $\lfloor x \rfloor = \lfloor \frac{n}{d} \rfloor$.*

**Proof:** We have $\frac{n}{d} = \lfloor \frac{n}{d} \rfloor + \frac{k}{d}$ for some nonnegative integer $k < d$. So $\frac{n}{d} \leq \lfloor \frac{n}{d} \rfloor + 1$. It follows that
$$ \frac{n}{d} \rfloor x < \lfloor \frac{n}{d} \rfloor + 1 $$

It follows that $x \in [ \lfloor \frac{n}{d} \rfloor, \lfloor \frac{n}{d} \rfloor + 1)$, so that $\lfloor x \rfloor = \lfloor \frac{n}{d} \rfloor$.
$\square$

The following theorem gives a condition to check if $m$ can be used to optimise division by a constant divisor $d$. The theorem is stated in [1], although the proof is adapted to be shorter and easier to understand.

**Theorem 2:** *Let $d, m, N, l \in \mathbb{N}$ be nonnegative integers with $d > 0$. If*
$$ 2^{N + l} \leq m \cdot d \leq 2^{N + l} + 2^l $$

*then*
$$ \forall n \in \mathbb{U}_N : \lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor $$

**Proof:** Multiplying the inequality by $\frac{n}{d \cdot 2^{N + l}}$ we get
$$ \frac{n}{d} \leq \frac{m \cdot n}{2^{N + l}} \leq \frac{n}{d} + \frac{1}{d} \cdot \frac{n}{2^N} $$

For all $n \in \mathbb{U}_N$ we have $n < 2^N$, so that $\frac{n}{2^N} < 1$. It follows that
$$ \forall n \in \mathbb{U}_N : \frac{n}{d} \leq \frac{m \cdot n}{2^{N + l}} \leq \frac{n}{d} + \frac{1}{d} $$

By lemma 1, it follows that $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

**Corollary 3:** *If $\text{mod}_d(-2^{N + l}) \leq 2^l$ and $m = \lceil \frac{2^{N + l}}{d} \rceil$, then $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.*

**Proof:** Suppose we have $\text{mod}_d(-2^{N + l}) < 2^l$. Then there exists an integer $m$ such that $0 \leq m \cdot d - 2^{N + l} \leq 2^l$ Adding $2^{N + l}$, we see that $2^{N + l} \leq m \cdot d \leq 2^{N + l} + 2^l$. In particular, the lowest $m$ that satisfies this equation is $m = \lceil \frac{2^{N + l}}{d} \rceil$, since $d \cdot \lceil \frac{2^{N + l}}{d} \rceil$ is the first multiple of $d$ that is greater than or equal to $2^{N + l}$. By theorem 2, it follows that $\lfloor \frac{m \cdot n}{2^{N + l}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

**Example:** TODO

**Example:** TODO
