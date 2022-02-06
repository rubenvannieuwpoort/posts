# Low-discrepancy sequences with Kronecker sequences

In this article, I investigate why Kroncker sequences based on the golden ratio are so effective.

[Monte Carlo methods](https://en.wikipedia.org/wiki/Monte_Carlo_method) typically approximate an integral $\int_0^1 f(x)\ \text{d}x$ by using
$$ \lim_{N \rightarrow \infty} \frac{1}{N} \sum_{k = 0}^{N - 1} f(x_k) = \int_0^1 f(x)\ \text{d}x $$

where $x_0, x_1, ..., x_{N - 1}$ is a sequence of random variables that are uniformly distributed on $[0, 1)$. This has convergence of order $O(\frac{1}{\sqrt{N}})$.

Can we do better? If we know the number of samples $N$ beforehand, we can take the samples to be uniformly spaced on $[0, 1)$:
$$ x_k = \frac{1 + 2k}{2N}\ $$

for $k = 0, 1, ..., N - 1$. This gives $O(\frac{1}{N})$ convergence; much better.

What if we don't know the number of samples in advance? This is the case if we want to continously improve our approximation to the integral but don't want to start from scratch every iteration (many ray tracers and path tracers use this technique).

So we are looking for infinite sequences $x_0, x_1, ... \in [0, 1)$ such that every finite subsequence $x_0, x_1, ..., x_{N - 1}$ is reasonably evenly distributed on $[0, 1)$. The technical term for such a sequences is [low-discrepancy sequence](https://en.wikipedia.org/wiki/Low-discrepancy_sequence).

Using the Monte Carlo method with low-discrepancy sequences is also called the quasi-Monte Carlo method. For an $s$-dimensional integral the expected error of the Monte Carlo method is on the order of $O(\frac{1}{\sqrt{N}})$ while the quasi-Monte Carlo method has an error of $O(\frac{(\log(N))^s}{N})$.

Formally, for a finite sequence with $N$ elements $x_0, x_1, ..., x_{N - 1} \in [0, 1)$, the [discrepancy](https://en.wikipedia.org/wiki/Discrepancy_of_a_sequence) is a real number $D \in [\frac{1}{N}, 1]$ that is a measure of how evenly distributed the elemtns are on the interval $[0, 1)$: a low discrepancy means the elements are evenly distributed while a high discrepancy means they are not.

There are various methods to generate low-discrepancy sequences. Most notably, we have
  - Halton sequences
  - Sobol sequences
  - Kronecker sequences

We will focus on Kronecker sequences in this article.

Kronecker's method is a simple but effective method that works as follows:
  1. Pick an irrational $x \in \mathbb{R}$ and any $s \in \mathbb{R}$ and define $y_k = s + kx$
  2. Define $x_k$ as the fractional part of $y_k$: $x_k = \{ y_k \}$. Now $x_k$ is a low-discrepancy sequence on $[0, 1)$.

It is well known that the golden ratio $\phi$ defined by
$$ \phi = \frac{1 + \sqrt{5}}{2} $$

is the best choice for $x$ (i.e. gives the lowest discrepancy).

Most explanations simply state that the golden ratio $\phi$ is "the most irrational number$. In this article I try to give some quantitative results and give a little more context.

The discrepancy is not an easy measure to work with. Instead, I will investigate the distance between points (I'll often informally refer to this as "gap").


# OLD

This article just considers one-dimensional low-discrepancy sequences. For a non-formal but beautiful introduction to multidimensional low-discrepancy sequences, I recommend that you read Robert Martin's article [The unreasonable effectiveness of quasirandom sequences](http://extremelearning.com.au/unreasonable-effectiveness-of-quasirandom-sequences).

The *discrepancy* of a finite sequence $x_1, x_2, ..., x_n$ is a measure of how evenly the points are distributed on an interval $[a, b]$. A small discrepancy means that the points are evenly distributed, a large discrepancy means they are not.

**Definition**: *Let $X = x_1, x_2, ..., x_n \in \mathbb{R}$ and $a, b \in \mathbb{R}$ with $a < b$. The **discrepancy** $D$ with respect to the interval $[a, b]$ is defined by*
$$ D = \sup_{a \leq c \leq d \leq b} \left| \frac{ | X \cap [c, d] | }{N} - \frac{d - c}{b - a} \right| $$

So, simply put, the discrepancy measures the maximum over all intervals $[c, d]$ over the difference between the fraction of points on the interval $[c, d]$ and the relative length of the interval $[c, d]$ compared to the length of the whole interval $[a, b]$.

Considering all sequences with at most $n$ points, the minimum discrepancy is $D = \frac{1}{N + 1}$, obtained by setting $x_{k} = \frac{k \cdot a + (n + 1 - k) \cdot b}{n + 1}$ for $k = 1, 2, ..., n$, which is just an evenly distributed set of points on the interval $[a, b]$. This can be seen by taking $[c, d] = [\epsilon, \frac{1}{n + 1} - \epsilon]$ and taking the limit as $\epsilon > 0$ goes to zero. The interval contains no points, and the length approaches $\frac{1}{n + 1}$.

Now consider an infinite sequence $x_1, x_2, ...$ We define some properties 

**Definition**: *Let $x_1, x_2, ... \in \mathbb{R}$ be an infinite sequence, $a, b \in \mathbb{R}$ with $a < b$, and let $D_n$ be the discrepancy of the sequence $x_1, x_2, ..., x_n$ of the first $n$ elements with respect to the interval $[a, b]$. Then $x_1, x_2, ...$ is called **equidistributed** on the interval $[a, b]$ if*
$$ \lim_{n \rightarrow \infty} D_n = 0 $$

In practice, the property of equidistribution is not very strong. We can add any finite number of points to the sequence without "breaking" the equidistributedness of the sequence. So in practice we want a stronger condition to indicate that *every* subsequence is fairly evenly distributed.

**Definition**: *A **low-discrepancy sequence** is an infinite sequence $x_1, x_2, ... \in \mathbb{R}$ such that the subsequence $x_1, x_2, ..., x_n$ of the first $n$ elements has low discrepancy.*

Kronecker's method is a method to generate low-discrepancy sequences on the interval $[0, 1]$. It defines the $k$th element of the sequence as
$$ x_k = \{ s + kx \} $$

Here, $x$ is an irrational number, and $\{ \cdot \}$ denotes the fractional value (that is, $\{ x \} = x - \lfloor x \rfloor$).

First, we show that Kronecker's method produces an equidistributed sequence. This theorem is known as the *equidistribution theorem*.

**Lemma**: *Let 0 $\leq p \leq q \leq 1$ and $s, x \in \mathbb{R}$ with $x$ irrational. Then there exists an $n \in \mathbb{N}$ such that*
$$ \{ s + nx \} \in [p, q] $$

**Proof**: TODO
$\square$


**Theorem**: *Let $x \in \mathbb{R}$ be an irrational number. Then the sequence $x_1, x_2, x_3, ...$ defined by*
$$ x_k = \{ s + kx \} $$

*is equidistributed.*

**Proof**: TODO
$\square$


Without giving a formal definition, a low discrepancy sequence is an infinite sequence of points $x_1, x_2, ...$ such that for any $N$, the subsequence $x_1, x_2, ..., x_N$ is a good covering of the domain $[0, 1]$. This is useful for methods that rely on sampling and 

**Definition**: *The **golden ratio**, denoted $\phi$, is defined as*
$$ \phi = \frac{1 + \sqrt{5}}{2} $$

Alternatively, the golden ratio can be represented as a continued fraction:
$$ \phi = [1; 1, 1, 1, ...] $$

The golden ratio has many interesting properties, in this article I will focus

When the continued fraction contains a large coefficient, this means it is well approximated by the convergent obtained by cutting off the continued fraction just before the large coefficient. Since the continued fraction for $\phi$ only has the smallest coefficients possible, the convergents are not incredibly good approximations of $\phi$.

For this reason, the golden ratio is also known as 'the most irrational number'. In particular, there is the following theorem due to Hurwitz:

**Theorem** *...*

