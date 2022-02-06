# Bounds for rational approximations to square roots

A consequence of [Dirichlet's approximation theorem](https://en.wikipedia.org/wiki/Dirichlet%27s_approximation_theorem) is that for any real number $x \in \mathbb{R}$, there exist infinitely many rational approximations $\frac{p}{q} \in \mathbb{Q}$ that satisfy
$$ |x - \frac{p}{q}| < \frac{1}{q^2} $$

In particular, the convergents of the continued fraction for $x$ satisfy this inequality. [Hurwitz's theorem](https://en.wikipedia.org/wiki/Hurwitz%27s_theorem_(number_theory)) gives a slightly tighter bound and states that there exist infinitely many rational approximations $\frac{p}{q} \in \mathbb{Q}$ that satisfy
$$ |x - \frac{p}{q}| < \frac{1}{\sqrt{5}} \cdot \frac{1}{q^2} $$

Surprisingly, this bound is tight, since for the [golden ratio](https://en.wikipedia.org/wiki/Golden_ratio) $\phi = \frac{1 + \sqrt{5}}{2}$ since for any $c < \frac{1}{\sqrt{5}}$ there exist only finitely many solutions $\frac{p}{q} \in \mathbb{Q}$ to $|\phi - \frac{p}{q}| < c \cdot \frac{1}{q^2}$.

It turns out that in general, there are bounds on how well rational numbers can approximate algebraic numbers. This is the basis of [Liouville's theorem](https://proofwiki.org/wiki/Liouville%27s_Theorem_(Number_Theory)), which was used to construct the first known transcedental numbers.

In Frits Beukers' "Getaltheorie voor Beginners", he briefly mentions an explicit bound for $\sqrt{2}$: any rational approximation $\frac{p}{q} \in \mathbb{Q}$ to $\sqrt{2}$ satisfies
$$ |\sqrt{2} - \frac{p}{q}| > \frac{1}{4} \cdot \frac{1}{q^2} $$

He proves this by supposing that $|\sqrt{2} - \frac{p}{q}| \leq \frac{1}{4} \cdot \frac{1}{q^2}$ and deriving a contradiction. Multiplying both sides with $\sqrt{2} + \frac{p}{q}$ gives
$$ |2 - \frac{p^2}{q^2}| \leq \frac{\sqrt{2} + \frac{p}{q}}{4} \cdot \frac{1}{q^2} < \frac{1}{q^2} $$

where the last inequality follows from
$$ \sqrt{2} + \frac{p}{q} = 2 \sqrt{2} - (\sqrt{2} - \frac{p}{q}) < 2 \sqrt{2} + \frac{1}{4q^2} \leq 2 \sqrt{2} + \frac{1}{4} < 4 $$

Finally, multiplying by $q^2$ gives
$$ | 2q^2 - p^2 | < 1 $$

Now $p$ and $q$ are integers, so the left side is integer. The only integer with absolute value less than one is zero. However, this implies that $\frac{q^2}{p^2} = 2$ so $\frac{p}{q} = \sqrt{2}$, which is a contradiction since $\sqrt{2}$ is irrational.

This bound can obviously be improved, since there are places where inequalities are used that are not tight. For example, we use $2 \sqrt{2} + \frac{1}{4} < 4$, while $2 \sqrt{2} + \frac{1}{4} \approx 3.078$ is much closer to $3$ than to $4$.

We can derive a tighter bound by supposing $| \sqrt{2} - \frac{p}{q} | < \epsilon \cdot \frac{1}{q^2}$ for some $\epsilon > 0$ and following the same steps until. We end up with
$$ | 2q^2 - p^2 | < \epsilon(2 \sqrt{2} + \epsilon) $$

Now, we want to pick $\epsilon$ so that the right hand side becomes one. So we solve $\epsilon (2 \sqrt{2} + \epsilon) = 1$ and find the positive solution $\epsilon = \sqrt{3} - \sqrt{2}$. By this, we have found the tighter bound $|\sqrt{2} - \frac{p}{q}| > (\sqrt{3} - \sqrt{2}) \cdot \frac{1}{q^2} = \frac{1}{\sqrt{2} + \sqrt{3}} \cdot \frac{1}{q^2}$.

In fact, we can generalize this for any irrational $\sqrt{n}$:

**Theorem**: *Let $n \in \mathbb{N}$ be a natural number that is not a square. Then any $\frac{p}{q} \in \mathbb{Q}$ satisfies*
$$ | \sqrt{n} - \frac{p}{q} | > c_n \cdot \frac{1}{q^2} $$

*with $c_n = \sqrt{n + 1} - \sqrt{n} = \frac{1}{\sqrt{n} + \sqrt{n + 1}}$.*

**Proof**: Set $\epsilon = \sqrt{n + 1} - \sqrt{n}$ and suppose there exists a $\frac{p}{q} \in \mathbb{Q}$ with
$$ | \sqrt{n} - \frac{p}{q} | < \epsilon \cdot \frac{1}{q^2} $$

Multiply both sides by $\sqrt{n} + \frac{p}{q} = 2 \sqrt{n} - (\sqrt{n} - \frac{p}{q}) < 2 \sqrt{n} + \epsilon$ to get
$$ | n - \frac{p^2}{q^2} | < \frac{\epsilon^2 + 2\sqrt{n} \epsilon}{q^2} $$

If we substitute $\epsilon = \sqrt{n + 1} - \sqrt{n}$ in $\epsilon^2 + 2\sqrt{n} \epsilon$ and simplify we see $\epsilon^2 + 2\sqrt{n} \epsilon = 1$. So the right side of this equation equals $\frac{1}{q^2}$. Multiplying both sides by $q^2$ gives us
$$ |nq^2 - p^2| < 1 $$

Like before, this is a contradiction: it implies that $|nq^2 - p^2| = 0$, so $|n - \frac{p^2}{q^2}| = 0$, so $\frac{p^2}{q^2} = n$, so $\frac{p}{q} = \sqrt{n}$. However, by assumption $n$ is not a square, so $\sqrt{n}$ is irrational. So this is a contradiction.
$\square$


I wonder if this can be generalized to $n$th roots and numbers of the form $\frac{a + b \sqrt{n}}{c}$.

From a high-level perspective we have used the expression
$$ e_x(\frac{p}{q}) = q^2 |x - \frac{p}{q}| $$

to measure how well a rational number $\frac{p}{q} \in \mathbb{Q}$ approximates a real number. We can now rephrase some results in terms of $e_x$:
  - For any $x \in \mathbb{R}$ there are infinitely many $\frac{p}{q} \in \mathbb{Q}$ with $e_x(\frac{p}{q}) < \frac{1}{\sqrt{5}}$.
  - For any natural number $n \in \mathbb{N}$ that is not a square root we have $e_{\sqrt{n}}(\frac{p}{q}) > \sqrt{n + 1} - \sqrt{n}$