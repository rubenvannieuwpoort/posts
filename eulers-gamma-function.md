# Euler's gamma function

**Definition**: *The **gamma function $\Gamma$** is defined as*
$$ \Gamma(x) = \int_0^\infty t^{x - 1} e^{-t}\ \text{d}t $$

First, do we even expect the integral in the definition of the gamma function to converge? The integrand $t^{x - 1} e^{-t}$ is the product of two factors. The first factor, $t^{x - 1}$, goes to infinity as $t$ goes to infinity. The second factor, $e^{-t}$, goes to zero as $t$ goes to infinity.

By the [integral test](https://en.wikipedia.org/wiki/Integral_test_for_convergence) the integral converges if and only if the sum
$$ \sum_{k = 0}^\infty k^{x - 1} e^{-k} $$

converges.

If $x$ is a nonpositive integer, $x - 1$ is a negative integer and the factor $k^{x - 1}$ is undefined at $k = 0$.

Otherwise, the ratio between two terms is
$$ \frac{(n + 1)^{x - 1} e^{-(n + 1)}}{n^{x - 1} e^{-n}} = \left(\frac{n + 1}{n} \right)^{x - 1} \cdot e^{-1} $$

Now $(\frac{n + 1}{n})^{x - 1}$ goes to $1$ as $n$ goes to infinity, so the ratio is certainly below $1$. By the [ratio test](https://en.wikipedia.org/wiki/Ratio_test) the sum converges. So the integral converges as well.

**Theorem**: *When $x \in \mathbb{R}$ is not a nonpositive integer, the integral*
$$ \int_0^\infty t^{x - 1} e^{-t}\ \text{d}t $$

*converges.*

Now suppose that $x = 1$. We have
$$ \Gamma(1) = \int_0^\infty e^{-t}\ \text{d}t = [-e^{-t}]_{t = 0}^\infty = 1 $$

Now suppose that $x = n$ for some $n \in \mathbb{N}_+$. Then we have
$$ \Gamma(n) = \int_0^\infty t^{n - 1} e^{-t}\ \text{d}t $$

Using [integration by parts](https://en.wikipedia.org/wiki/Integration_by_parts) we have
$$ \int_0^\infty t^{n - 1} e^{-t}\ \text{d}t = [-t^{x - 1} e^{-t}]_{t = 0}^\infty - \int_0^\infty -(x - 1) t^{x - 2}e^{-t} \ \text{d}t = (x - 1) \cdot \Gamma(x - 1) $$

So we have $\Gamma(1) = 1$, $\Gamma(n) = (n - 1) \Gamma(n - 1)$. This is analogous to the definition of $n!$
$$ \begin{aligned} 0! &= 1\\n! &= n \cdot (n - 1)! \end{aligned} $$

This implies the following result:

**Theorem**:
$$ \Gamma(n) = (n - 1)! $$

In fact, the integral in the definition of the gamma function is well defined for any complex number that is not a nonpositive integer (which is why the wikipedia page writes the gamma function as a function of a parameter $z$, which is usually used for complex variables).

Why do we care about the gamma function? For one, it is just a beautiful mathematical object that is worthy of study on it's own. It is also related to the [Riemann zeta function](https://en.wikipedia.org/wiki/Riemann_zeta_function) and the [Beta function](https://en.wikipedia.org/wiki/Beta_function).

Other than that, I have to be honest and tell you *I don't really know* what the significance of the gamma function is. It is commonly accepted to be one of the "special" functions which are of great importance, but I am struggling to give an application of the gamma function itself. It's application seems to be mostly its relatedness to other "special" functions. If I were more familiar with the field of special functions I could probably give better examples here.
