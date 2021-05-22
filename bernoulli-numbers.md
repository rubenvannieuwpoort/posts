# Bernoulli numbers

This article investigates sums of the form $\sum_{k = 0}^{n - 1} k^d$. From this investigation we naturally arrive at the Bernoulli numbers.

The derivation is based on section X in X and section Y in Y. TODO

**Definition**: *The **forward difference operator** $\Delta$ is defined by its operation on a sequence $(c_k)_{k = 0}^\infty$*
$$ \Delta c_k = c_{k + 1} - c_k $$

The binomial theorem enables us to find an explicit formula for the forward difference of a series of the form $(n^d)_{n = 0}^\infty$.

**Theorem**: *When $d \geq 1$ we have $\Delta n^d = p(n)$ for a polynomial $p$ of degree $d - 1$.*

**Proof**: Suppose that $p(n) = \Delta n^d$. Then $p(n) = (n + 1)^d - n^d$. By the binomial theorem, we have $p(n) = \sum_{k = 0}^{d - 1} \binom{d}{k} n^k$, which is a polynomial of degree $d - 1$.
$\square$

TODO: now we want to find a formula for $\sum k^d$
The following theorem says there exists a polynomial of degree this and that TODO

**Theorem**: When $d \geq 0$ we have $\sum_{k = 0}^{n - 1} = q(n)$ for some polynomial $q$ of degree $d + 1$.

**Proof**: We use induction. The base case is $d = 0$ for which we have $\sum_{k = 0}^{n - 1} k^0 = n$ (using the convention that $0^0 = 1$ TODO: investigate), so the result holds.

Now define $p_j(n) = \sum_{k = 0}^{n - 1} k^j$. Assume that $p_j$ is a polynomial of degree $j + 1$ for every $j < d$. Then
$$ \sum_{k = 0}^n \Delta k^{d + 1} = \sum_{k = 0}^n (k + 1)^{d + 1} - k^{d + 1} = n^{d + 1} $$

By the binomial theorem we can also write this as
$$ \sum_{k = 0}^{n - 1} \sum_{j = 0}^d \binom{d + 1}{j} k^j = \sum_{j = 0}^d \binom{d + 1}{j} \sum_{k = 0}^{n - 1} k^j = \sum_{j = 0}^d \binom{d + 1}{j} p_j(n) $$

Combining the two gives
$$ p_d(n) = \frac{n^{d + 1} - \sum_{j = 0}^{d - 1} \binom{d + 1}{j} p_j}{d + 1} $$
 
 Obviously $p_d$ is a polynomial of degree $d + 1$. This completes the induction step.
 $\square$

While the proof gives an explicit formula for $p_d$, it is recursively defined. To compute $p_d$, we need to know all the polynomials $p_0$, $p_1,$, ..., $p_{d - 1}$. There is a more efficient method.

Since we know that the $p_d$ is a polynomial of degree $d + 1$, we can write $p_d(n) = \sum_{j = 0}^{d + 1} c_{d, j} n^j$. We then have
$$ \sum_{j = 0}^{d + 1} c_{d, j} n^j = \sum_{k = 0}^{n - 1} k^d $$

Applying the forward difference operator with respect to $n$ on both sides results in a system of equations that we can solve for the coefficients $c_{d, 0}, c_{d, 1}, ..., c_{d, d + 1}$ of the polynomial $p_d$.

**Theorem**: *Assume that $d \geq 0$. Define $p_d(n) = \sum_{k = 0}^{n - 1} k^d$. Then $p_d(n) = \sum c_{d, j} n^j$ where the coefficients $c_{d, 0}, c_{d, 1}, ..., c_{d, d + 1}$ satisfy*
$$ \binom{d + 1}{d} c_{d, d+ 1} = 1 $$

*and*
$$ \sum_{j = k + 1}^{d + 1} \binom{j}{k} c_{d, j} = 0 $$

*for $k = 0, 1, ..., d - 1$*

**Proof**: TODO
$\square$

TODO: give some formulas and state general formula
TODO: introduce bernoulli numbers

If we use this linear system to find $p_d$ for $d = 1, 2, 3, 4, 5$ we can make a table:

TODO: make table

There seems to be a pattern in the coefficients. After some investigation, it looks like we have
$$ c_{d, j} = \frac{1}{d + 1} \binom{d + 1}{j} B_{d + 1 - j} $$

for some constant $B_{d + 1 - j}$. If this is correct, it is interesting to compute these numbers since knowing the sequence $B_0, B_1, ...$ up to $B_{d + 1}$ allows us to easily compute all the polynomials $p_0, p_1, ..., p_d$.

**Definition**: *The **Bernoulli numbers** $B_0, B_1, ...$ are defined by the system*
$$ B_0 = 1 $$

$$ \sum_{k = 0}^m \binom{m + 1}{k} B_k = 0 $$

*for $m = 1, 2, ...$*

Even though the system is infinite, taking the first $N$ equations suffices to compute the first $N$ Bernoulli numbers. When the first $N$ Bernoulli numbers are known, the $(N + 1)$th number can easily be found just from the $(N + 1)$th equation and the first $N$ Bernoulli numbers.

Now, 

**Theorem**: *We have*
$$ \sum_{j = 0}^{d + 1} c_{d, j} n^j = \sum_{k = 0}^{n - 1} k^d $$

*where*
$$ c_{d, j} = \frac{1}{d + 1} \binom{d + 1}{j} B_{d + 1 - j} $$

**Proof**: From the previous theorem we know that we have $\sum_{j = 0}^{d + 1} c_{d, j} n^j = \sum_{k = 0}^{n - 1} k^d$ when the coefficients $c_{d, j}$ satisfy the system
$$\binom{d + 1}{d} c_{d, d+ 1} = 1$$

and
$$\sum_{j = k + 1}^{d + 1} \binom{j}{k} c_{d, j} = 0$$

for $k = 0, 1, ..., d$

Now we simply substitute $c_{d, j} = \frac{1}{d + 1} \binom{d + 1}{j} B_{d + 1 - j}$. Now we can write
$$ \binom{j}{k} c_{d, j} = \frac{1}{d + 1} \binom{j}{k} \binom{d + 1}{j} B_{d + 1 - j} $$

$$ = \frac{1}{d + 1} \cdot \frac{j!}{k! (j - k)!} \cdot \frac{(d + 1)!}{j! (d + 1 - j)!} = \frac{1}{d + 1} \cdot \frac{(d + 1)!}{k! (d + 1 - k)!} \cdot \frac{(d + 1 - k)!}{(j - k)! (d + 1 - j)!} $$

$$ = \frac{1}{d + 1} \binom{d + 1}{k} \binom{d + 1- k}{d + 1 - j} B_{d + 1 - j} $$

So our system for the coefficients is equivalent to
$$\sum_{j = k + 1}^{d + 1} \frac{1}{d + 1} \binom{d + 1}{k} \binom{d + 1- k}{d + 1 - j} B_{d + 1 - j} = 0$$

TODO (not always 0 in reality)

Substituting $j' = d + 1 - j$ and $m = d - k$ and adjusting the bounds of the summation gives
$$\sum_{j' = 0}^{k'} \frac{1}{d + 1} \binom{d + 1}{d - m} \binom{m + 1}{j'} B_{j'} = 0$$

Using $\binom{n}{n - k} = \binom{n}{k}$ and bringing out the factors that independent on $j'$ out of the summation, this can be simplified to
$$\frac{1}{d + 1} \binom{d + 1}{m} \sum_{j' = 0}^{m} \binom{k' + 1}{j'} B_{j'} = 0$$

which is equivalent to
$$ \sum_{j' = 0}^{m} \binom{m + 1}{j'} B_{j'} = 0 $$

Since the Bernoulli numbers $B_0, B_1, ...$ satisfy these equations by definition, the coefficients $c_{d, j}$ defined in terms of the Bernoulli numbers also satisfy TODO. So the desired result follows.
$\square$
