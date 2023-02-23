# Dirichlet's approximation theorem

Dirichlet's approximation theorem is a result about rational approximation. First, let's derive a trivial bound on the error of the best rational approximation so that we can compare it with Dirichlet's theorem.

If we approximate a real number $\alpha \in \mathbb{R}$ by a rational $\frac{p}{q}$ with $p \in \mathbb{Z}$, $q \in \mathbb{N}_+$, it is straightforward to see that there exists a $p \in \mathbb{Z}$ such that
$$ \frac{p}{q} \leq \alpha \leq \frac{p + 1}{q} $$

If both $\alpha - \frac{p}{q} > \frac{1}{2q}$ and $\frac{p + 1}{q} - \alpha > \frac{1}{2q}$, we could add the inequalities to get
$$ (\alpha - \frac{p}{q})+ (\frac{p + 1}{q} - \alpha) > \frac{1}{q} $$

The left side is obviously equal to $\frac{1}{q}$, so this is a contradiction. So either $0 \leq \frac{p}{q} - \alpha \leq \frac{1}{2q}$ or $0 \leq \frac{p + 1}{q} - \alpha \leq \frac{1}{2q}$. So, we have proved the following lemma:

**Lemma 1**: *Let $\alpha \in \mathbb{R}$. For any $q \in \mathbb{N}_+$ there exists a $p \in \mathbb{Z}$ such that*
$$ \left| \alpha - \frac{p}{q} \right| \leq \frac{1}{2q} $$

It follows from Dirichlet's theorem that there exist infinitely many rationals $\frac{p}{q}$ such that $\left| \alpha - \frac{p}{q} \right| < \frac{1}{q^2}$. The quadratic order of the error turns out to be optimal; This is what Hurwitz' theorem proves.

It seems that Dirichlet was fond of arithmetic progressions; Dirichlet's most [famous theorem](https://en.wikipedia.org/wiki/Dirichlet%27s_theorem_on_arithmetic_progressions) is about arithmetic progressions, and they turn out to play an important role in the proof of this theorem as well.

Let $\alpha \in \mathbb{R}$. Now consider the sequence $0, \{ \alpha \}, \{ 2 \alpha \}, \{ 3 \alpha \}, ..., \{ N \alpha \}$, where $\{ x \} = x - \lfloor x \rfloor$ (so $0 \leq \{ x \} < 1$ for any $x \in \mathbb{R}$). It turns out that if two fractional parts are close, we can find a good rational approximation to $\alpha$.

Say we have $|\{ r \alpha \} - \{ s \alpha \}| < \epsilon$ for $r, s \in \mathbb{N}_0$. Expanding this with the definition of the fractional operator, we see
$$ |(r \alpha - \lfloor r \alpha \rfloor) - (s \alpha - \lfloor s \alpha \rfloor)| = |q \alpha - p| < \epsilon $$

Where $p = \lfloor r \alpha \rfloor - \lfloor s \alpha \rfloor, q = r - s$. Finally, dividing by $|q|$, we get
$$ \left| \alpha - \frac{p}{q} \right| < \frac{\epsilon}{q} $$

Noting that $q \leq N$, we have proved the following lemma:

**Lemma 2**: *Let $\alpha \in \mathbb{R}$. If $| \{ r \alpha \} - \{ s \alpha \} | < \epsilon$ for natural numbers $r, s \in \mathbb{N}_0$, then there exists a rational $\frac{p}{q} \in \mathbb{Q}$ such that*
$$ \left| \alpha - \frac{p}{q} \right| < \frac{\epsilon}{q} $$

*for some $q \leq N$*

Now to prove Dirichlet's theorem we just need to obtain a good bound $\epsilon$ which we can then plug in this lemma. We can obtain such an $\epsilon$ by splitting the interval $[0, 1)$ up in $N$ equal parts. We then consider the $N + 1$ terms of the sequence $0, \{ \alpha \}, \{ 2 \alpha \}, ..., \{ N \alpha \}$ and apply the [pigeonhole principle](https://en.wikipedia.org/wiki/Pigeonhole_principle): Since there are $N + 1$ terms and only $N$ intervals, there is at least one interval with two terms in it, say $\{ r \alpha \}$ and $\{ s \alpha \}$, so that
$$ \left| \{ r \alpha \} - \{ s \alpha \} \right| < \frac{1}{N} $$

Now, by lemma 2 there exists a rational approximation $\frac{p}{q} \in \mathbb{Q}$ of $\alpha$ that satisfies $| \alpha - \frac{p}{q} | < \frac{1}{Nq}$. So we have proved the following theorem:

**Theorem (Dirichlet)**: Let $\alpha \in \mathbb{R}$. For any positive integer $N \in \mathbb{N}_+$, there exists a fraction $\frac{p}{q} \in \mathbb{Q}$ such that
$$ \left| \alpha - \frac{p}{q} \right| < \frac{1}{Nq} $$

Now suppose that for a given $\alpha \in \mathbb{R}$ the set $S_\alpha = \{ \frac{p}{q} : | \alpha - \frac{p}{q} | < \frac{1}{q^2} \}$, (with each fraction $\frac{p}{q}$ [irreducible](https://en.wikipedia.org/wiki/Irreducible_fraction)) is finite. Then $N = \max_{\frac{p}{q} \in S_\alpha}(q)$ exists. This implies that there is no $\frac{p}{q} \in \mathbb{Q}$ such that $|\alpha - \frac{p}{q}| < \frac{1}{Nq}$. This contradicts Dirichlet's theorem, so we conclude that our assumption that $S_\alpha$ is finite was wrong.

**Corollary**: *For any $\alpha \in \mathbb{R}$, there are infinitely many rational numbers $\frac{p}{q} \in \mathbb{Q}$ such that*
$$ \left| \alpha - \frac{p}{q} \right| < \frac{1}{q^2} $$
