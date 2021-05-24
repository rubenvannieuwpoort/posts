# Bernoulli numbers

This article investigates sums of the form $\sum_{k = 0}^{n - 1} k^d$. From this investigation we naturally arrive at the Bernoulli numbers.

The derivation is based on section 9.4.3 in "Getallen - van natuurlijk tot imaginair" by Keune, and section 6.5 in "Concrete mathematics - a foundation for computer science" by Graham, Knuth, and Patashnik.

First, we define an operator which takes the difference of two subsequent elements of a sequence.

**Definition 1**: *The **forward difference operator** $\Delta$ is defined by its operation on a sequence $(a_k)_{k = 0}^\infty$*
$$ \Delta a_k = a_{k + 1} - a_k $$

The forward difference operator and summation behave similarly to differentation and integration: They behave roughly like inverses of each other. More precisely, we have
$$ \sum_{k = 0}^{n - 1} \Delta a_k = a_n - a_0 $$

and
$$ \Delta (\sum_{k = 0}^{n - 1} a_k) = a_n $$

The binomial theorem enables us to find an explicit formula for the forward difference of a series of the form $0^d, 1^d, 2^d, ...$ for any power $d \in \mathbb{N}_0$. We will use the convention that $0^0 = 1$.

**Theorem 2**: *When $d \geq 1$ we have $\Delta n^d = p(n)$ for a polynomial $p$ of degree $d - 1$.*

**Proof**: Suppose that $p(n) = \Delta n^d$. Then $p(n) = (n + 1)^d - n^d$. By the binomial theorem, we have $p(n) = \sum_{k = 0}^{d - 1} \binom{d}{k} n^k$, which is a polynomial of degree $d - 1$.
$\square$

**Corollary 3**: *Let $p$ be a polynomial of degree $d \geq 1$. Then $\Delta p(k) = q(n)$ for some polynomial of degree $d - 1$.*

It is just a little more complicated to show that the sequence obtained by summing the first $n$ elements of the sequence $0^d, 1^d, 2^d, ...$ is a polynomial of degree $d + 1$.

**Theorem 4**: When $d \geq 0$ we have $\sum_{k = 0}^{n - 1} k^d = q(n)$ for some polynomial $q$ of degree $d + 1$.

**Proof**: We use induction. The base case is $d = 0$ for which we have $\sum_{k = 0}^{n - 1} k^0 = n$, so the result holds.

Now define $p_j(n) = \sum_{k = 0}^{n - 1} k^j$. Assume that $p_j$ is a polynomial of degree $j + 1$ for every $j < d$. Then
$$ \sum_{k = 0}^n \Delta k^{d + 1} = \sum_{k = 0}^n (k + 1)^{d + 1} - k^{d + 1} = (n + 1)^{d + 1} $$

By the binomial theorem we can also write this as
$$ \sum_{k = 0}^{n - 1} \sum_{j = 0}^d \binom{d + 1}{j} k^j = \sum_{j = 0}^d \binom{d + 1}{j} \sum_{k = 0}^{n - 1} k^j = \sum_{j = 0}^d \binom{d + 1}{j} p_j(n) $$

Combining the two gives
$$ p_d(n) = \frac{n^{d + 1} - \sum_{j = 0}^{d - 1} \binom{d + 1}{j} p_j}{d + 1} $$
 
Since $p_d$ is a polynomial of degree $d + 1$, the induction step is completed.
$\square$

**Corollary 5**: *Let $p$ be a polynomial of degree $d$. Then $\sum_{k = 0}^{n - 1} p(k) = q(n)$ for some polynomial $q$ of degree $d + 1$.*

While the proof of theorem 4 gives an explicit formula for $p_d$, it is recursively defined. To compute $p_d$, we need to know all the polynomials $p_0$, $p_1,$, ..., $p_{d - 1}$. It would be more efficient if we didn't have to find all the polynomials $p_0, p_1, ..., p_{d - 1}$ before we could start calculating the coefficients of $p_d$. We will now explore a different method which allows us to calculate $p_d$ directly.

Since we know that the $p_d$ is a polynomial of degree $d + 1$, we can write $p_d(n) = \sum_{j = 0}^{d + 1} c_{d, j} n^j$. By substituting $n = 0$ in $\sum_{j = 0}^{d + 1} c_{d, j} n^j = \sum_{k = 0}^{n - 1}k^d$ we get $c_{d, 0} = 0$. So we will omit the constant coefficient $c_{d, 0}$ and write $p_d = \sum_{j = 1}^{d + 1} c_{d, j} n^j$ from now on.

We then have
$$ \sum_{j = 1}^{d + 1} c_{d, j} n^j = \sum_{k = 0}^{n - 1} k^d $$

Applying the forward difference operator with respect to $n$ on both sides results in an equation with a polynomial of degree $d$ on both sides. If two polynomials of degree $d$ or less are equal in $d + 1$ points, they must be the same polynomial. So we can group the coefficients that correspond to the same power. This way, we get system of $d + 1$ equations that we can solve for the coefficients $c_{d, 1}, c_{d, 2}, ..., c_{d, d + 1}$ of the polynomial $p_d$.

**Theorem 6**: *Assume that $d \geq 0$. Define $p_d(n) = \sum_{k = 1}^{n - 1} k^d$. Then $p_d(n) = \sum_{j = 1}^{d + 1} c_{d, j} n^j$ where the coefficients $c_{d, 0}, c_{d, 1}, ..., c_{d, d + 1}$ satisfy*
$$ \sum_{j = k + 1}^{d + 1} \binom{j}{k} c_{d, j} = \begin{cases} 1 & \text{when $k = d$} \\ 0 & \text{otherwise} \end{cases} $$

*for $k = 0, 1, ..., d$*

**Proof**: Suppose that $p_d(n) = \sum_{k = 1}^{n - 1} k^d$. Taking the forward difference of both sides of the equality yields
$$\sum_{j = 1}^{d + 1} c_{d, j} \left( (n + 1)^j - n^j \right) = n ^d$$

By the binomial theorem we have $(n + 1)^j - n^j = \sum_{i = 0}^{j - 1} \binom{j}{i} n^i$. So we have
$$\sum_{j = 1}^{d + 1} c_{d, j} \sum_{i = 0}^{j - 1} \binom{j}{i} n^k = n ^d$$

Now, we switch the order of summation of the left side
$$ \sum_{j = 1}^{d + 1} c_{d, j} \sum_{i = 0}^j \binom{j}{i} n^i = \sum_{0 \leq i < j \leq d + 1} c_{d, j} \binom{j}{i} n^i = \sum_{i = 0}^d n^i \sum_{j = i + 1}^{d + 1} c_{d, j} \binom{j}{i} $$

We now have
$$ \sum_{i = 0}^d n^i \sum_{j = i + 1}^{d + 1} c_{d, j} \binom{j}{i} = n^d$$

Since the left and right side are both polynomials and need to agree for all $n$, it follows that the polynomials must be equal. So for $i =0, 1, ..., d$ we have
$$ \sum_{j = i + 1}^{d + 1} c_{d, j} \binom{j}{i} = \begin{cases} 1 & \text{when $k = d$} \\ 0 & \text{otherwise} \end{cases} $$

Now, when $p_d(n) = \sum_{j = 1}^d c_{d, j} n^j$ we have $p_d(0) = 0$ and $p_d(n + 1) - p_d(n) = n^d$ for every $n \geq 0$, since this is the system that we solved. It follows by induction that $p_d(n) = \sum_{k = 0}^{n - 1}k^d$.
$\square$

If we use this linear system to find $p_d$ for $d = 1, 2, 3, 4, 5$ we can make a table:

$$ \begin{aligned}
p_0(n) &= n \\
p_1(n) &= \frac{1}{2} n^2 - \frac{1}{2} n \\
p_2(n) &= \frac{1}{3} n^3 - \frac{1}{2} n^2 + \frac{1}{6} n \\
p_3(n) &= \frac{1}{4} n^4 - \frac{1}{2} n^3 + \frac{1}{4} n^2 \\
p_4(n) &= \frac{1}{5} n^5 - \frac{1}{2} n^4 + \frac{1}{3} n^3  - \frac{1}{30} n\\
p_5(n) &= \frac{1}{6} n^6 - \frac{1}{2} n^5 + \frac{5}{12} n^4  - \frac{1}{12} n^2\\
p_6(n) &= \frac{1}{7} n^7 - \frac{1}{2} n^6 + \frac{1}{2} n^5  - \frac{1}{6} n^3 + \frac{1}{42}n\\
p_7(n) &= \frac{1}{8} n^8 - \frac{1}{2} n^7 + \frac{7}{12} n^6  - \frac{7}{24} n^4 + \frac{1}{12}n^2\\
\end{aligned}$$

There seems to be a pattern in the coefficients. After some investigation, it looks like we have
$$ c_{d, j} = \frac{1}{d + 1} \binom{d + 1}{j} B_{d + 1 - j} $$

for some constant $B_{d + 1 - j}$. If this is correct, it is interesting to compute these numbers since knowing the sequence $B_0, B_1, ...$ up to $B_{d + 1}$ allows us to easily compute all the polynomials $p_0, p_1, ..., p_d$.

**Definition 7**: *The **Bernoulli numbers** $B_0, B_1, ...$ are defined by the system*
$$ B_0 = 1 $$

$$ \sum_{k = 0}^m \binom{m + 1}{k} B_k = 0 $$

*for $m = 1, 2, ...$*

The system is infinite, but the first $N$ equations suffice to compute the first $N$ Bernoulli numbers.

Now, we will prove that the Bernoulli numbers can indeed be used to find coefficients $c_{d, 0}, c_{d, 1}, ...$ for the polynomial $p_d(n) = \sum_{j = 0}^{d + 1} c_{d, j} n^j$ such that $p_d(n) = \sum_{k = 0}^{n - 1} k^d$.

**Theorem 8**: *Define $c_{d, j} = \frac{1}{d + 1} \binom{d + 1}{j} B_{d + 1 - j}$ where $B_0, B_1, B_2, ...$ are the Bernoulli numbers. Then*
$$ \sum_{j = 0}^{d + 1} c_{d, j} n^j = \sum_{k = 0}^{n - 1} k^d $$

**Proof**: From the previous theorem we know that we have $\sum_{j = 0}^{d + 1} c_{d, j} n^j = \sum_{k = 0}^{n - 1} k^d$ when the coefficients $c_{d, j}$ satisfy the system
$$\sum_{j = k + 1}^{d + 1} \binom{j}{k} c_{d, j} = \begin{cases} 1 & \text{when $k = d$} \\ 0 & \text{otherwise} \end{cases}$$

for $k = 0, 1, ..., d$

Now we simply substitute $c_{d, j} = \frac{1}{d + 1} \binom{d + 1}{j} B_{d + 1 - j}$. Now we can write
$$ \begin{aligned} \binom{j}{k} c_{d, j} & = \frac{1}{d + 1} \binom{j}{k} \binom{d + 1}{j} B_{d + 1 - j} \\ &= \frac{1}{d + 1} \cdot \frac{j!}{k! (j - k)!} \cdot \frac{(d + 1)!}{j! (d + 1 - j)!} \\
& = \frac{1}{d + 1} \cdot \frac{(d + 1)!}{k! (d + 1 - k)!} \cdot \frac{(d + 1 - k)!}{(j - k)! (d + 1 - j)!} \\ &= \frac{1}{d + 1} \binom{d + 1}{k} \binom{d + 1- k}{d + 1 - j} B_{d + 1 - j} \end{aligned} $$

So our system for the coefficients is equivalent to
$$\sum_{j = k + 1}^{d + 1} \frac{1}{d + 1} \binom{d + 1}{k} \binom{d + 1- k}{d + 1 - j} B_{d + 1 - j} = \begin{cases} 1 & \text{when $k = d$} \\ 0 & \text{otherwise} \end{cases}$$

Substituting $j' = d + 1 - j$ and $m = d - k$ and adjusting the bounds of the summation gives
$$\sum_{j' = 0}^{k'} \frac{1}{d + 1} \binom{d + 1}{d - m} \binom{m + 1}{j'} B_{j'} = 0$$

Using $\binom{n}{n - k} = \binom{n}{k}$ and bringing out the factors that independent on $j'$ out of the summation, this can be simplified to
$$\frac{1}{d + 1} \binom{d + 1}{m} \sum_{j' = 0}^{m} \binom{k' + 1}{j'} B_{j'} = 0$$

which is equivalent to
$$ \sum_{j' = 0}^{m} \binom{m + 1}{j'} B_{j'} = 0 $$

Since the Bernoulli numbers $B_0, B_1, ...$ satisfy these equations by definition, the coefficients $c_{d, j}$ defined in terms of the Bernoulli numbers satisfy the condition of theorem 6. So the desired result follows.
$\square$

**Example**: From the definition of the Bernoulli numbers, we see that $B_0 = 1$. From setting $m = 1$ in definition 6 we get $\binom{2}{0} B_0 + \binom{2}{1} B_1 = 0$, which is equivalent to $B_1 = -\frac{1}{2}$. In the same fashion we have the formula $\binom{3}{0} B_0 + \binom{3}{1} B_1 + \binom{3}{2} B_2 = 0$ which implies that $B_2 = -\frac{1}{6}$, and $\binom{4}{0} B_0 + \binom{4}{1} B_1 + \binom{4}{2} B_2 + \binom{4}{3} B_3 = 0$ which implies that $B_3 = 0$.

From theorem 4 we know that we can write $\sum_{k = 0}^{n - 1} k^d$ as a polynomial $p_d$ of degree $d + 1$ in $n$. From theorem 8 we know that
$$ c_{d, j} = \frac{1}{d + 1} \binom{d + 1}{j} B_{d + 1 - j} $$

Setting $d = 1$ we can find the coefficients $c_{1, 1}, c_{1, 2}$ of $p_1$. We get $c_{1, 1} = B_1 = -\frac{1}{2}$, $c_{1, 2} = \frac{1}{2} B_0 = \frac{1}{2}$ and find $\sum_{k = 0}^{n - 1} k = -\frac{1}{2}n^2 + \frac{1}{2}n^2$.

In the same way we can find the coefficients of $p_3$: $c_{3, 1} = 0, c_{3, 2} = \frac{1}{4}, c_{3, 3} = -\frac{1}{2}, c_{3, 4} = \frac{1}{4}$. So $\sum_{k = 0}^{n - 1} k^3 = \frac{1}{4}n^2 - \frac{1}{2}n^3 + \frac{1}{4} n^4$.

Now, we can see that $p_3(n) = \frac{1}{4}n^2 - \frac{1}{2}n^3 + \frac{1}{4} n^4 = (-\frac{1}{2}n^2 + \frac{1}{2}n^2)^2 = (p_2(n))^2$. So it follows that
$$ \sum_{k = 0}^{n - 1} k^3 = \left( \sum_{k = 0}^{n - 1} k \right)^2 $$
