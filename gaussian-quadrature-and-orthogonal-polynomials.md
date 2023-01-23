# Gaussian quadrature and orthogonal polynomials

The proofs here are based on the lecture notes from a course in approximation theory [1]. I think most of the results and proofs are standard, so I expect most of the texts on this subject to contain similar results and proofs.

Suppose $\mu(x)$ is a nonnegative weight function and we want to compute the integral
$$ \int f(x)\ \mu(x)\ \text{d}x $$

for many different functions $f$.

Depending on what $f$ and $\mu$ are, we might be able to evaluate the integral analytically (i.e. by integrating and obtaining an exact answer). However, in computational contexts it is often both more efficient and simpler to use a *quadrature rule*.

**Definition 1**: *A **quadrature rule** is a collection of weights $w_1, w_2, ..., w_n$ and positions $x_1, x_2, ..., x_n$ such that*
$$ \sum_{k = q}^n w_k f(x_k) \approx \int f(x)\ \mu(x)\ \text{d}x $$

Since the approximation given by the quadrature rule only depends on the value of $f$ on $x_1, x_2, ..., x_n$, the approximation will be better for smoother functions, for which the value of $f$ doesn't fluctuate too wildly, so the behavior is captured well by its value on some positions.

In practice, we often require the quadrature rule to be exact for some set $S$ of functions. Often, we take $S$ as the set of polynomials up to some maximum degree. We call this the *exactness conditions*. We can write the exactness conditions as a finite number of equations by taking a basis $f_1, f_2, ...$ of $S$. By the linearity of the integral the quadrature rule is also exact for any linear combination of $f_1, f_2, ...$ which means that the quadrature rule is exact for any $f \in S$.

In the following, we will denote the space of polynomials with degree up to $n$ as $\mathbb{P}_n$.


## Quadrature rules, the easy way

It is not too complicated to derive quadrature rules that are exact for any polynomial $p \in \mathbb{P}_n$ for a given $n$. Give a function $f$ and $d + 1$ distinct points $x_0, x_1, ..., x_d$ we can use [Lagrange interpolation](https://en.wikipedia.org/wiki/Lagrange_polynomial) to construct a polynomial $f_d$ of degree $d$ that interpolates $f$ at $x_0, x_1, ..., x_d$. That is,
$$ f_d(x_k) = f(x_k)\ \text{for $k = 0, 1, ..., d$} $$

The polynomial $f_d$ is defined as $f_d(x) = \sum_{k = 0}^n f(x_k) l_k(x)$, where $l_k$ are the Lagrange polynomials defined by
$$ l_k(x) = \frac{(x - x_0)(x - x_1) \cdots (x - x_{k - 1}) (x - x_{k + 1}) \cdots (x - x_d)}{(x_k - x_0)(x_k - x_1) \cdots (x_k - x_{k - 1}) (x_k - x_{k + 1}) \cdots (x_k - x_d)} $$

We can now integrate the interpolation polynomial $f_d$ of $f$ as
$$ \int f_d(x)\ \mu(x)\ \text{d}x = \int \sum_{k = 0}^n f(x_k)l_k(x)\ \mu(x)\ \text{d}x $$

Bringing the summation and the terms $f(x_k)$ outside of the integral, we see
$$ \int f_d(x)\ \mu(x)\ \text{d}x = \sum_{k = 0}^n f(x_k) \int l_k(x)\ \mu(x)\ \text{d}x $$

So if we set $w_k = \int l_k(x)\ \mu(x)\ \text{d}x$, we have
$$ \int f_d(x)\ \mu(x)\ \text{d}x = \sum_{k = 0}^n w_k f(x_k) $$

Now, since $f_d$ interpolates $f$, we expect that $f_d \approx f$, so that
$$ \int f(x)\ \mu(x)\ \text{d}x \approx \int f_d(x)\ \mu(x)\ \text{d}x = \sum_{k = 0}^n w_k f(x_k) $$

So we have found a quadrature rule that is exact for any $f \in \mathbb{P}_d$.

It is also possible to find the weights in a more computational way. If we want the quadrature rule to be exact for a function space of dimension $d$, we can pick a basis of $d$ functions, pick $d$ distinct positions $x_1, x_2, ..., x_d$, write out the exactness conditions, and solve the resulting linear system of equations for the weights.

For example, suppose that we want the quadrature rule to be exact for any $p \in \mathbb{P}_d$. We pick $d + 1$ basis functions $b_0, b_1, ..., b_d$ (for example $b_k(x) = x^k$ would work) and distinct positions $x_0, x_1, ..., x_d$. Then the exactness requirements become
$$ \sum_{j = 0}^d w_j b_k(x_j) = \int b_k(x)\ \mu(x)\ \text{d}x\ \text{for $k = 0, 1, ..., d$} $$

which is a linear system of $d + 1$ equations in the $d + 1$ unknowns $w_0, w_1, ... w_d$. In particular, if we use $b_k(x) = x^k$ the resulting system is a [Vandermonde system](https://en.wikipedia.org/wiki/Vandermonde_matrix).

For now, we have regarded the positions $x_1, x_2, ..., x_d$ as given. It is usually a good choice to pick them as [Chebyshev nodes](https://en.wikipedia.org/wiki/Chebyshev_nodes), since this ensures that the polynomial $f_d$ that interpolates $f$ is actually close to $f$ -- it minimizes oscillatory behavior of $f_d$.

If we change our point of view and interpret the positions $x_1, x_2, ..., x_d$ as unknowns we can interpret the exactness requirements as a nonlinear system in $2d$ unknowns. If we expect this system to behave like a non-singular linear one (which is not a given, but let's be optimistic), we would expect to be able to solve a system of $2d$ equations with it. This is by analogy with a linear system, where a non-singular system has a unique solution if there are as many unknowns as equations.

So, this would mean that for each $n$ there exists a quadrature rule with $n$ points that is exact for $\mathbb{P}_{2n - 1}$. These quadrature uses roughly half the number of points of the quadrature rules derived via the method presented earlier, at the cost of not having the freedom of choosing the positions $x_0, x_2, ..., x_{n - 1}$ freely.

It is possible to derive quadrature rules like this using *orthogonal polynomials*.


## Orthogonal polynomials

Consider the following inner product on a space of functions
$$ \left< f, g \right> = \int f(x) g(x)\ \mu(x)\ \text{d}x $$

Like before, $\mu(x)$ is a nonnegative weight function. This inner product makes the function space into a [vector space](https://en.wikipedia.org/wiki/Vector_space).

The usual notions of orthogonality and orthonormality apply.

**Definition 2**: *A sequence of polynomials $p_0, p_1, ...$ is **orthogonal** if*
  1. *$\text{deg}(p_k) = k$*
  2. *$\left< p_i, p_j \right> = 0$ whenever $i \neq j$*

**Definition 3**: *A sequence of polynomials $p_0, p_1, ...$ is **orthonormal** if it is orthogonal and $\left< p_k, p_k \right> = 1$ for every $k = 0, 1, ...$*

We will use the following lemmas, which are easy to understand and prove.

**Lemma 4**: *If $p_0, p_1, ...$ is a sequence of orthogonal polynomials and $p$ is a polynomial with $\text{deg}(p) < n$, then*
$$ \left< p, p_n \right> = 0 $$

**Proof**: The polynomials $p_0, p_1, ..., p_{n - 1}$ form a basis of $\mathbb{P}_n$. Since $p \in \mathbb{P}_n$, we can write $p(x) = \sum_{k = 0}^{n - 1} c_k p_k(x)$. It follows that $\left< p, p_n \right> = \left< \sum_{k = 0}^{n - 1} c_k p_k, p \right> = \sum_{k = 0}^{n - 1} c_k \left< p_k, p_n \right>$. Since $p_0, p_1, ...$ is an orthogonal sequence of polynomials and $k < n$, all terms in the summation are zero, so it follows that $\left< p, p_n \right> = 0$.

$\square$

The following lemma states that polynomials can be easily expressed as a linear combination of orthonormal polynomials.

**Lemma 5**: *If $p_0, p_1, ...$ is a sequence of orthonormal polynomials and $p$ is a polynomial with $\text{deg}(p) = n$, then*
$$ p(x) = \sum_{k = 0}^n \left< p, p_k \right> p_k(x) $$

**Proof**: Since $p_0, p_1, ..., p_n$ is a basis of $\mathbb{P}_n$ and $p \in \mathbb{P}_n$, we can write $p$ as $p(x) = \sum_{k = 0}^n c_k p_k(x)$ for some $c_0, c_1, ..., c_n \in \mathbb{R}$. Taking the inner product of $p$ and $p_k$, we see
$$ \left< p, p_k \right> = \left< \sum_{j = 0}^n c_j p_j, p_k \right> = \sum_{j = 0}^n c_j \left< p_j, p_k \right> = c_k $$

So $c_k = \left< p, p_k \right>$. Substituting this in $p(x) = \sum_{k = 0}^n c_k p_k(x) gives the required result.

$\square$

A sequence of orthonormal polynomials can be computed by using the Gram-Schmidt method, but it is more efficient to use the following result.

**Theorem 6**: *If $p_0, p_1, ...$ is a sequence of polynomials then*
$$ \left< p_n, x p_{n + 1} \right> p_{n + 1}(x) = xp_n(x) - \left< p_n, x p_{n - 1} \right> p_{n - 1}(x) - \left< p_n x p_n \right> p_n(x) $$

**Proof**: From lemma 5 we have $x p_n(x) = \sum_{k = 0}^{n + 1} \left< x p_n, p_k \right> p_k$. Now
$$ \left< x p_n, p_k \right> = \int x p_n(x) p_k(x)\ \mu(x)\ \text{d}x = \left< p_n, x p_k \right> $$

So we have
$$ x p_n = \sum_{k = 0}^{n + 1} \left< p_n, x p_k \right> p_k $$

By lemma 4, $\left< p_n, x p_k \right>$ is zero when $\text{deg}(x p_k) = k + 1 < n$, so $k = n - 1, n, n + 1$ are the only $k \leq n + 1$ for which $\left< p_n, x p_k \right>$ is nonzero. So we have
$$ x p_n = \left< p_n, x p_{n - 1} \right> p_{n - 1}(x) + \left< p_n, x p_n \right> p_n(x) + \left< p_n, x p_{n + 1} \right> p_{n + 1}(x) $$

$\square$

Using theorem 6, we can compute the orthonormal polynomial $p_{n + 1}$ from the previous polynomials as follows. First, we compute a polynomial $q_{n + 1}(x)$ which is orthogonal to $p_0, p_1, ..., p_n$ as
$$ q(x) = x p_n(x) - \left< p_n, x p_{n - 1} \right> p_{n - 1}(x) - \left< p_n, x p_n \right> p_n(x) $$

Then, we compute $p_{n + 1}$ by normalizing $q_{n + 1}$:
$$ p_{n + 1}(x) = \frac{q_{n + 1}(x)}{\sqrt{\left< q_{n + 1}, q_{n + 1} \right>}} $$

There is the following nice result about the roots of orthogonal polynomials.

**Lemma 7**: *Let $p_0, p_1, ...$ be a sequence of orthonal polynomials with respect to the inner product $\left< f, g \right> = \int f(x) f(x)\ \mu(x)\ \text{d}x$, and let the support of $\mu$ be contained in $[a, b]$. Then $p_n$ has $n$ distinct real roots $x_1, x_2, ..., x_n \in (a, b)$.*

**Proof**: Let $x_1, x_2, ..., x_m$ be the zeroes of $p_n$ that have odd multiplicity and lie in $(a, b)$. Define $q(x) = (x - x_1)(x - x_2) \cdots (x - x_m)$. Now $q(x) p_n(x)$ does not change sign on $[a, b]$, so $\left< q, p_n \right> \neq 0$. It follows from $m \leq n$ and lemma 4 that $\text{deg}(q) = n$. So $m = n$, which implies that $p_n$ has $n$ distinct roots in $(a, b)$.

Now we are ready to prove the following theorem.

**Theorem 8**: *Let $p_0, p_1, ...$ be a sequence of orthonal polynomials with respect to the inner product $\left< f, g \right> = \int f(x) f(x)\ \mu(x)\ \text{d}x$ and let $x_1, x_2, ..., x_n$ be the zeroes of $p_n$. Define $w_k = \int l_k(x)\ \mu(x)\ \text{d}x$ for $k = 1, 2, ..., n$, where $l_1, l_2, ..., l_n$ are the Lagrange polynomials for $x_1, x_2, ..., x_n$. Then*
$$ \sum_{k = 1}^n w_k f(x_k) = \int f(x)\ \mu(x)\ \text{d}x $$

*for $f \in \mathbb{P}_{2n - 1}$.*

**Proof**: Let $f_n \in \mathbb{P}_n$ be the polynomial of degree $n$ that interpolates $f$ at $x_1, x_2, ..., x_n$. If we can show that $\int (f(x) - f_n(x))\ \mu(x)\ \text{d}x = 0$, it follows that $\int f(x)\ \mu(x)\ \text{d}x = \int f_n(x)\ \mu(x)\ \text{d}x$.

First, we note that $f_n(x)$ assumes the same values as $f(x)$ for $x = x_1, x_2, ..., x_n$. So $f(x) - f_n(x)$ has zeroes at $x_1, x_2, ..., x_n$. Since $x_1, x_2, ..., x_n$ are the zeroes from $p_n$, the function $f(x) - f_n(x)$ has the same zeroes as $p_n$ and must be some multiple of $p_n$, say $f(x) - f_n(x) = c p_n(x)$. Then we can write $f(x) - f_n(x) = c p_n(x) q(x)$ for some $q \in \mathbb{P}_{n - 1}$.

Now, we can write the integral as $\int (f(x) - f_n(x))\ \mu(x)\ \text{d}x = c \int p_n(x) q(x)\ \mu(x)\ \text{d}x = c \left< p_n, q \right>$. Since $\text{deg}(q) < n$ we have $\left< p_n, q \right> = 0$ by lemma 4. So we see that $\int f(x)\ \mu(x)\ \text{d}x = \int f_n(x)\ \mu(x)\ \text{d}x$. It was shown ealier that $\int f_n(x)\ \mu(x)\ \text{d}x = \sum_{k = 1}^n w_k f(x_k)$, so it follows that
$$ \int f(x)\ \mu(x)\ \text{d}x = \int f_n(x)\ \mu(x)\ \text{d}x = \sum_{k = 1}^n w_k f(x_k) $$

$\square$

Sometimes, we want to compute the integral
$$ \int p(x) q(x)\ \mu(x)\ \text{d}x $$

for many different $p$ and $q$. Using theorem 6 and 8 and a root-finding method, we can compute a different quadrature rule for every $q$. The inconvenient thing about this method is that the quadrature points $x_1, x_2, ... will be different for every $q$.

In [2], it is proposed to instead pick one set of quadrature points and re-use these for every $q$. This means we have to use more quadrature points per $q$, but since we can use the same set of quadrature points we only have to evaluate $p$ on this one set, which is more efficient in the end. It also means that we can use the 'simple' way of computing quadrature rules outlined in the section 'quadrature rules, the easy way'.


## References

[1] Approximation Theory. Walter Groenevelt. Lecture notes WI4415, TU Delft, 2016.

[2] Matrix-free weighted quadrature for a computationally efficient isogeometric k-method. Giancarlo Sangalli and Mattia Tani, 2017. https://arxiv.org/abs/1712.08565