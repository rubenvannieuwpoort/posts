# Division by constant unsigned integers

Most modern processors have an integer divide instruction which, for technical reasons, is very slow compared to other integer arithmetic operations. When the divisor is constant, it is possible to transform the division instruction to other instructions which execute faster. Most optimizing compilers perform this optimization, as can be seen on [Matt Godbolt’s compiler explorer](https://godbolt.org/z/xrYsbs).

This article tries to be a self-contained reference for optimizing division by constant unsigned integers in this way. It focuses on rigor and conciseness, while not sacrificing readability too much. The results presented are mostly from existing literature (except theorem 4, although in [2] an analogous theorem is proved). The proof for theorem 2 has been simplified from the result in [1].

There are some tricks to divide by special divisors: A division by one can be ignored, a division by a power of two can be replaced by a bit shift, and a division by a number that is larger than half of the maximum value of the datatype of the dividend can be replaced by a comparison. By using fixed-point arithmetic, we can speed up division by all constant divisors. However, this is not as simple, and for this reason I will focus only on positive (or 'unsigned') divisors. For a number $n$ and a divisor $d$, I assume that we are interested in the *quotient* $\lfloor \frac{n}{d} \rfloor$.

When it is necessary to repeatedly divide floating point numbers by the same constant $c$, it is often preferred to precompute $\frac{1}{c}$ and multiply by this instead. This is usually a lot more efficient. We can do something analogous for integers by using [fixed point arithmetic](https://en.wikipedia.org/wiki/Fixed-point_arithmetic). The basic idea is to pick a large constant $L$ that is easy to divide by. In decimal you can take some power of ten, in binary you would take a power of two. If we would have $c = \frac{L}{d}$ we would have $\frac{n \cdot c}{L} = \frac{n \cdot \frac{L}{d}}{L} = \frac{n}{d}$. However, $\frac{L}{d}$ is usually not an integer, and we need to round. Still, if $c \approx \frac{L}{d}$ we still expect that $\frac{n \cdot c}{L} \approx \frac{n}{d}$.

In the following section, I'll discuss the mathematical background. In the sections after that, I'll discuss optimization of division by unsinged integers.


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
  2. It should be *efficient to evaluate*. For our purposes, this means that we only want to multiply numbers of at most $N$ bits, so that they fit in a single register on our $N$-bit machine, and that we implement the division by $2^k$ by a bit shift.

Guided by these demands, we can start our analysis. The following lemma will be useful:

**Lemma 1**: *Suppose that $n, d \in \mathbb{N}$ with $d > 0$. If $\frac{n}{d} \leq x < \frac{n + 1}{d}$ then $\lfloor x \rfloor = \lfloor \frac{n}{d} \rfloor$.*

**Proof**: We have $\frac{n + 1}{d} = \lfloor \frac{n}{d} \rfloor + \frac{k}{d}$ for some nonnegative integer $k \leq d$. So $\frac{n + 1}{d} = \lfloor \frac{n}{d} \rfloor + \frac{k}{d} \leq \lfloor \frac{n}{d} \rfloor + 1$. It follows that $x \in [ \lfloor \frac{n}{d} \rfloor, \lfloor \frac{n}{d} \rfloor + 1)$, so that $\lfloor x \rfloor = \lfloor \frac{n}{d} \rfloor$.
$\square$

At this point, we have only decided that we want an expression that evaluates to $\lfloor \frac{n}{d} \rfloor$. In the introduction, we have established that when
$$ m \approx \frac{2^k}{d} $$

then
$$ \frac{n \cdot m}{2^k} \approx \frac{n}{d} $$

So, it is natural to take $\lfloor \frac{m \cdot n}{2^k} \rfloor$ with $m \approx \frac{2^k}{d}$ as our starting point, since we know that $\frac{m \cdot n}{2^k} \approx \frac{n}{d}$. We now need to answer the following questions. We now need to decide how exactly to pick $m$ and $k$.

The obvious choice for $m$ are the integers that minimize the error to $\frac{2^k}{d}$, which are $m_\text{down} = \lfloor \frac{2^k}{d} \rfloor$ and $m_\text{up} = \lceil \frac{2^k}{d} \rceil$. Note that when $d$ is not a power of two, we have $\lfloor \frac{d \cdot m_\text{down}}{2^k} \rfloor = 0$. To overcome this, we can either use $m_\text{up}$ or simply use $n + 1$ instead of $n$. This gives us the following two methods:
  - The **round-up method**, which approximates $\lfloor \frac{n}{d} \rfloor$ by $\lfloor \frac{m_\text{up} \cdot n}{2^k} \rfloor$ with $m_\text{up} = \lceil \frac{2^k}{d} \rceil$.
  - The **round-down method**, which approximates $\lfloor \frac{n}{d} \rfloor$ by $\lfloor \frac{m_\text{down} \cdot (n + 1)}{2^k} \rfloor$ with $m_\text{down} = \lfloor \frac{2^k}{d} \rfloor$.

First, we determine the conditions under which the round-up and round-down method produce the correct result. Note that from this point on we will assume that $k$ is larger than $N$. We will assume $k = N + \ell$ and use $k$ and $N + \ell$ interchangeably.

The following theorem gives us a condition under which the round-up method is correct, that is, $\lfloor \frac{m_\text{up} \cdot n}{2^k} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.

**Theorem 2 (round-up method)**: *Let $d, N, l \in \mathbb{N}$ be nonnegative integers with $d > 0$. If there exists an $m \in \mathbb{N}$ with*
$$ 2^{N + \ell} \leq m \cdot d \leq 2^{N + \ell} + 2^\ell $$

*then*
$$ \forall n \in \mathbb{U}_N : \lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor $$

*If there exists any $m$ with this property, then $m_\text{up} = \lceil \frac{2^{N + \ell}}{d} \rceil$ has this property as well.*

**Proof**: Multiplying the inequality by $\frac{n}{d \cdot 2^{N + \ell}}$ we get
$$ \frac{n}{d} \leq \frac{m \cdot n}{2^{N + \ell}} \leq \frac{n}{d} + \frac{1}{d} \cdot \frac{n}{2^N} $$

For all $n \in \mathbb{U}_N$ we have $n < 2^N$, so that $\frac{n}{2^N} < 1$. It follows that
$$ \forall n \in \mathbb{U}_N : \frac{n}{d} \leq \frac{m \cdot n}{2^{N + \ell}} \leq \frac{n}{d} + \frac{1}{d} $$

By lemma 1, it follows that $\lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$. Now, $d \cdot m_\text{up} = d \cdot \lceil \frac{2^{N + \ell}}{d} \rceil$ is simply the first multiple of $d$ larger than or equal to $2^{N + \ell}$, so if there is any multiple of $d$ between $2^{N + \ell}$ and $2^{N + \ell} + 2^\ell$, we have $2^{N + \ell} \leq d \cdot m_\text{up} \leq 2^{N + \ell} + 2^\ell$.
$\square$

From this theorem we can derive the following condition, which is more formal, but easier to check.

**Corollary 3**: *If $\text{mod}_d(-2^{N + \ell}) \leq 2^\ell$ and $m_\text{up} = \lceil \frac{2^{N + \ell}}{d} \rceil$, then $\lfloor \frac{m_\text{up} \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.*

**Proof**: Suppose we have $\text{mod}_d(-2^{N + \ell}) < 2^\ell$. Then there exists an integer $m$ such that $0 \leq m \cdot d - 2^{N + \ell} \leq 2^\ell$ Adding $2^{N + \ell}$, we see that $2^{N + \ell} \leq m \cdot d \leq 2^{N + \ell} + 2^\ell$. In particular, the lowest $m$ that satisfies this equation is $m = \lceil \frac{2^{N + \ell}}{d} \rceil$, since $d \cdot \lceil \frac{2^{N + \ell}}{d} \rceil$ is the first multiple of $d$ that is greater than or equal to $2^{N + \ell}$. By theorem 2, it follows that $\lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

**Example**: TODO
**Example**: TODO

The following theorem gives us a condition under which the round-down method is correct, that is, $\lfloor \frac{m_\text{down} \cdot (n + 1)}{2^k} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.

**Theorem 4 (round-down method)**: *Let $d, N, l \in \mathbb{N}$ be nonnegative integers with $d > 0$. If there exists an $m \in \mathbb{N}$ with*
$$ 2^{N + \ell} - 2^\ell \leq m \cdot d < 2^{N + \ell}$$

*then*
$$ \forall n \in \mathbb{U}_N : \lfloor \frac{m \cdot (n + 1)}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor $$

**Proof**: Multiply the inequality by $\frac{n + 1}{d \cdot 2^{N + \ell}}$ to get
$$ \frac{n}{d} + \frac{1}{d} \cdot \left( 1 - \frac{n + 1}{2^N} \right) \leq \frac{m \cdot (n + 1)}{2^{N + \ell}} < \frac{n + 1}{d} $$

Looking at the expression on the left side, we have $1 \leq n + 1 \leq 2^N$, so that $0 \leq 1 - \frac{n + 1}{2^N} < 1$. It follows that $\frac{n}{d} \leq \frac{n}{d} + \frac{1}{d} \cdot (1 - \frac{n + 1}{2^N})$, so
$$ \frac{n}{d} \leq \frac{m \cdot (n + 1)}{2^{N + \ell}} < \frac{n + 1}{d} $$

for all $n \in \mathbb{U}_N$. So lemma 1 applies and we have $\lfloor \frac{m \cdot (n + 1)}{2^{N + \ell}} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

Again, there is a more formal form of this result, which is easier to check.

**Corollary 5**: If $0 < \text{mod}_d(2^{N + \ell}) < 2^\ell$ and $m = \lfloor \frac{2^{N + \ell}}{d} \rfloor$, then $\lfloor \frac{m \cdot (n + 1)}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.

**Proof**: If $0 < \text{mod}_d(2^{N + \ell}) \leq 2^\ell$ and $m - \lfloor \frac{2^{N + \ell}}{d} \rfloor$, then $\lfloor \frac{m \cdot (n + 1)}{2^{N + \ell}} = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$. Subtracting $2^{N + \ell}$, we see that $-2^{N + \ell} < -m \cdot d \leq -2^{N + \ell} + 2^\ell$. Multiplying by $-1$, we get $2^{N + \ell} - 2^\ell \leq m \cdot d < 2^{N + \ell}$. In particular, this condition must hold for $m = \lfloor \frac{2^{N + \ell}}{d}$, since $d \cdot \lfloor \frac{2^{N + \ell}}{d} \rfloor$ is the biggest multiple of $d$ that is less than $2^{N + \ell}$. By theorem 4, it follows that $\lfloor \frac{m \cdot (n + 1)}{2^{N + \ell}} = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.
$\square$

Note that we have $0 < \text{mod}_d(2^{N + \ell})$ whenever $d$ is not a power of two. So in practice it often suffices to only check $\text{mod}_d(2^{N + \ell}) < 2^\ell$.

**Example**: TODO
**Example**: TODO

These two theorems give us a way to know when the methods produce the correct result. Now, the methods are only efficient when $m$ fits in $N$ bits. Let's define this more rigorously:

**Definition**: *We call a positive divisor $d \in \mathbb{U}_d$ efficient for the $N$-bit round-up method (or round down method) if there exists a $k \in \mathbb{N}$ such that $\lfloor \frac{m_\text{up} \cdot n}{2^k} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$. Likewise, we call a positive divisor $d \in \mathbb{U}_d$ efficient for the $N$-bit round-down method if there exists a $k \in \mathbb{N}$ such that $\lfloor \frac{m_\text{down} \cdot (n + 1)}{2^k} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.*

The following result tells us the maximum $k$ that we can pick so that $m_\text{up}$ and $m_\text{down}$ still fit in $N$ bits.

**Lemma 6**: *Let $d \in \mathbb{U}_N$ and define $m' = \frac{2^{N + \lceil \log_2(d) \rceil - 1}}{d}, m_\text{down} = \lfloor m' \rfloor, m_\text{up} = \lceil m' \rceil$. Now*
$$2^{N - 1} \leq \lfloor x \rfloor \leq \lceil x \rceil \leq 2^N - 1$$

That is, the binary representations of $m_\text{down}$ and $m_\text{up}$ have exactly $N$ bits.

**Proof**: We have $2^{N - 1} = \frac{2^{N - 1} \cdot d}{d} = \frac{2^{N - 1 + \log_2(d)}}{d} \leq \frac{2^{N - 1 + \lceil \log_2(d)} \rceil}{d} = m'$. Since the floor function is a nondecreasing function and $\lfloor m' \rfloor \leq \lceil m' \rceil$ we have $2^{N - 1} \leq m_\text{down} \leq m_\text{up}$. It remains to show that $m_\text{up} \leq 2^N - 1$.

When $N = 1$, it follows that $d = 1$ and the bound holds. When $N > 1$, the ratio $\frac{2^{\lceil \log_2(d) \rceil}}{d}$ is maximized over $d \in \mathbb{U}_N$ by $d = 2^{N - 1} + 1$, so $\frac{2^{\lceil \log_2(d) \rceil}}{d} \leq \frac{2^N}{2^{N - 1} + 1} < \frac{2^N - 1}{2^{N - 1}}$ (this last inequality can be seen by multiplying both sides by $2^{N - 1} \cdot (2^{N - 1} + 1)$). So we have $\frac{2^{\lceil \log_2(d) \rceil}}{d} < \frac{2^N - 1}{2^{N - 1}}$; multiplying both sides by $2^{N - 1}$ gives $m' = \frac{2^{N + \lceil \log_2(d) \rceil - 1}}{d} < 2^N - 1$. After rounding up both sides, it follows that $m_\text{up} \leq 2^N - 1$.
$\square$

The following theorem tells us that the round-down method is *almost* efficient; for any divisor $d$ we can store $m_\text{up}$ in at most $N + 1$ bits.

**Theorem 7**: *Let $N \in \mathbb{N}_+$. For any positive divisor $d \in \mathbb{U}_N$, there exist $l \in \mathbb{N_+}$, $m \in \mathbb{U}_{N + 1}$ such that $\lfloor \frac{m \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.*

**Proof**: Set $\ell = \lceil \log_2(d) \rceil$. The range $\{ 2^N, 2^N + 1, ..., 2^N + 2^\ell \}$ consists of $2^\ell + 1$ consecutive numbers. We have $2^\ell + 1 > d$, so there must be a multiple of $d$ in this range. Using theorem 2, we see when $\ell = \lceil \log_2(d) \rceil$ we have $\lfloor \frac{m_\text{up} \cdot n}{2^{N + \ell}} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$ with $m_\text{up} = \lceil \frac{2^{N + \lceil \log_2(d) \rceil}}{d} \rceil$. In the proof of lemma 6 we saw that $\frac{2^{N + \lceil \log_2(d) \rceil - 1}}{d} < 2^N - 1$. By multiplying both sides by two and rounding up, we see that for $\ell = \lceil \log_2(d) \rceil$ we have $m_\text{up} \leq 2^{N + 1} - 1$, so $m_\text{up} \in \mathbb{U}_{N + 1}$.
$\square$

The following result states that we can combine the round-up and round-down method to a method that works for any positive divisor. So this finally concludes our quest for an efficient method to calculate the quotient $\lfloor \frac{n}{d} \rfloor$.

**Theorem 8**: Any positive divisor $d \in \mathbb{U}_N$ is either efficient for the $N$-bit round-up method, or efficient for the $N$-bit round-down method.

**Proof**: Set $\ell = \lceil \log_2(d) \rceil - 1$. By lemma 6, we know that $m_\text{up}, m_\text{down} \in \mathbb{U}_N$. Now consider the range $\{ 2^N - 2^\ell, 2^N - 2^\ell + 1, ..., 2^N + \ell  \}$. This is a range of $2\ell + 1$ numbers. Since $d < 2\ell + 1$ there must be at least one multiple $m$ of $d$ in this range. When this multiple $m$ satisfies $2^N - 2^\ell \leq m < 2^{N + \ell}$ the condition for the round-down method is satisfied. Since $m_\text{down} \in \mathbb{U}_N$, $d$ is efficient for the $N$-bit round-down method. Otherwise, we have $2^{N + \ell} \leq m \leq 2^{N + \ell} + 2^\ell$ and the condition for the round-up method is satisfied. Again, since $m_\text{up} \in \mathbb{U}_N$ we have that $d$ is efficient for the $N$-bit round-up method.
$\square$

The round-up method is slightly more efficient to evaluate, so this is the preferred method. To test if a given divisor is efficient for the round-up method, the condition from theorem 2 or corollary 3 can be used. The following result gives a more efficient condition to check if a given divisor is efficient for the round-up method.

**Lemma 9**: *Let $d$ be a positive integer, $\ell = \lfloor \log_2(d) \rfloor$, and $m_\text{up} = \lceil \frac{2^{N + \ell}}{d} \rceil$. If $\text{mod}_{2^N}(m_\text{up} \cdot d) \leq 2^\ell$ then $d$ is efficient for the round-up method.*

**Proof**: The product $m_\text{up} \cdot d = \lceil \frac{2^{N + \ell}}{d} \rceil \cdot d$ is the first multiple of $d$ that is equal to or larger than $2^{N + \ell}$. This product will be of the form $2^{N + \ell} + q$ for some $q < d \leq 2^\ell$, so we have $\text{mod}_{2^N}(m \cdot d) = q$. It follows that $2^{N + \ell} \leq m_\text{up} \cdot d \leq 2^{N + \ell} + 2^\ell$ if and only if $\text{mod}_{2^N}(m \cdot d) \leq 2^\ell$.
$\square$


## Implementation

### Compile-time optimization

### Runtime optimization

### Testing

## Appendix

**Lemma 10**: *If $d$ is a divisor of $2^N - 1$, then $d$ is efficient for the $N$-bit round-up method.*

**Proof**: Take $\ell = \lfloor \log_2(d) \rfloor$ and define $e = \text{mod}_d(-2^{N + 1})$ and $e' = \text{mod}_d(2^{N + 1})$. Using $\text{mod}_d(2^N) = 1$ and $2^{\lfloor \log_2(d) \rfloor} < d$, we see:
$$ e' = \text{mod}_d(2^{N + \ell}) = \text{mod}_d(2^N \cdot 2^{\lfloor \log_2(d) \rfloor}) = \text{mod}_d(2^{\lfloor \log_2(d) \rfloor}) = 2^{\lfloor \log_2(d) \rfloor} $$

Here, we used that $2^{\lfloor \log_2(d) \rfloor} < d$. Since $d$ is a divisor of $2^N - 1$, it can't be a power of two, so we have $e, e' \in \{ 1, 2, ..., d - 1 \}$. It follows that $e + e' = d$ since $e + e' \equiv 0 \mod d$. We find that
$$ e = d - e' \leq 2^{\lceil \log_2(d) \rceil} - 2^{\lfloor \log_2(d) \rfloor} = 2^{\lfloor \log_2(d) \rfloor} = 2^\ell $$

So the condition $e = \text{mod}_d(2^{N + \ell}) \leq 2^\ell$ for corollary 5 is satisfied. Since $d$ is not a power of two, we have that $\ell = \lfloor \log_2(d) \rfloor = \lceil \log_2(d) \rceil - 1$. By using lemma 6, we now see that $m_\text{up} \in \mathbb{U}_N$ for $\ell = \lfloor \log_2(d) \rfloor$. So $d$ is efficient for the $N$-bit round-up method.
$\square$


## References and further reading

The classic reference for optimization of division by both signed and unsigned integers is [1], which states and proves theorem 2 (round-up method). In [2], the approach is extended to the round-down method. The resources [3] and [4] are by far the easiest to read, and cover essentially everything in this article in a more accessible way. They sacrifice some rigor and completeness, though. If you are interested in optimizing division, I recommend reading these articles as your starting point. Finally, in [5] it is shown that we can also use fixed point math to compute the modulo operation.

[1]: [Division by Invariant Integers using Multiplication](https://gmplib.org/~tege/divcnst-pldi94.pdf), Torbjörn Granlund and Peter L. Montgomery, 1994.

[2]: [N-Bit Unsigned Divison Via N-Bit Multiply-Add](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.512.2627&rep=rep1&type=pdf), Arch D. Robinson, 2005.

[3]: [Labor of Divison (Episode I)](https://ridiculousfish.com/blog/posts/labor-of-division-episode-i.html), fish, 2010.

[4]: [Labor of Divison (Episode III): Fast Unsigned Division by Constants](https://ridiculousfish.com/blog/posts/labor-of-division-episode-iii.html), fish, 2011.

[5] [Faster Remainder by Direct Computation: Applications to Compilers and Software Libraries](https://arxiv.org/pdf/1902.01961), Daniel Lemire, Owen Kaser, Nathan Kurz, 2019.
