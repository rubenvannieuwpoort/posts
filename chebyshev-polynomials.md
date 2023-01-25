# Chebyshev polynomials

We have the following trigonometric identities for the sine and cosine of the sum of angles
$$ \begin{aligned} \sin(\alpha + \beta) &= \sin(\alpha) \cos(\beta) + \cos(\alpha) \sin(\beta) \\
\cos(\alpha + \beta) &= \cos(\alpha) \cos(\beta) - \sin(\alpha) \sin(\beta) \end{aligned} $$

Substituting $\alpha = \theta, \beta = \theta$ gives
$$ \begin{aligned} \sin(2 \theta) &= 2 \sin(\theta) \cos(\theta) \\
\cos(2 \theta) &= \cos^2(\theta) - \sin^2(\theta) = 2 \cos^2(\theta) - 1 \end{aligned} $$

The formula for $\cos(2 \theta)$ is simplified using $\sin^2(\theta) = 1 - \cos^2(\theta)$, which follows from $\sin^2(\theta) + \cos^2(\theta) = 1$.

In the same way, we can derive formulas for $\sin(3 \theta)$ and $\cos(3 \theta)$: We substitute $\alpha = 2 \theta$, $\beta = \theta$ and get
$$ \begin{aligned} \sin(3 \theta) &= \sin(2 \theta) \cos(\theta) + \cos(2 \theta) \sin(\theta) = \sin(\theta) (4 \cos^2(\theta) - 1) \ \\
\cos(3 \theta) &= \cos(2 \theta) \cos(\theta) - \sin(2 \theta) \sin(\theta) = 4 \cos^3(\theta) - 3 \cos(\theta) \end{aligned} $$

This strategy of substituting $\alpha = n \theta$, $\beta = \theta$ in the sum of angle formula, and using $\sin^2(\theta) = 1 - \cos^2(\theta)$ seem to work more generally. We can use this method to compile a list of multiple angle formulas:
$$ \begin{aligned} \sin(2 \theta) &= 2 \sin(\theta) \cos(\theta) \\
\cos(2 \theta) &= 2 \cos^2(\theta) - 1 \end{aligned} $$

$$ \begin{aligned} \sin(3 \theta) &= \sin(\theta) (4 \cos^2(\theta) - 1) \\
\cos(3 \theta) &= 4 \cos^3(\theta) - 3 \cos(\theta) \end{aligned} $$

$$ \begin{aligned} \sin(4 \theta) &= \sin(\theta) \cdot (8 \cos^3(\theta) - 4 \cos(\theta)) \\
\cos(4 \theta) &= 8 \cos^4(\theta) - 8 \cos^2(\theta) + \cos(\theta) \end{aligned} $$

$$ \begin{aligned} \sin(5 \theta) &= \sin(\theta) \cdot (16 \cos^4(\theta) - 12 \cos^2(\theta) + 1) \\
\cos(5 \theta) &= 16 \cos^5(\theta) - 20 \cos^3(\theta) + 5 \cos(\theta) \end{aligned} $$

$$ \begin{aligned} \sin(6 \theta) &= \sin(\theta) \cdot (32 \cos^5(\theta) - 32 \cos^3(\theta) + 6 \cos(\theta) \\
\cos(6 \theta) &= 32 \cos^6(\theta) - 48 \cos^4(\theta) + 18 \cos^2(\theta) - 1 \end{aligned} $$

Looking at these multiple angle formulas from the last section suggests the existence of a pattern that allows us to write
$$ \begin{aligned} \cos(n \theta) &= T_n(\cos(\theta)) \\
\sin(n \theta) &= \sin(\theta) \cdot U_{n - 1}(\cos(\theta)) \end{aligned} $$

for polynomials $T_n, U_{n - 1}$ of degree $n$, $n - 1$ respectively. It is not very complicated to check that this is indeed the case. We start from [De Moivre's formula](https://en.wikipedia.org/wiki/De_Moivre%27s_formula):
$$ \cos(n \theta) + i \sin(n \theta) = (\cos(\theta) + i \sin(\theta))^n $$

We take the real part of this equation and get an equality with $\cos(n \theta)$ on the left hand side and a polynomial in $\cos(\theta)$ and $\sin(\theta)$ on the right side. Now, only terms where $i \sin(\theta)$ is raised to an even exponent contribute to the real part of the expression. Hence the expression on the right side will only involve $\cos(\theta)$ and $\sin^2(\theta)$. So we can use $\sin^2(\theta) = \cos^2(\theta) - 1$ and write the right hand side as a polynomial in $\cos(\theta)$.

The same idea works for $\sin(n \theta)$, except we now need to take the imaginary part of the equation. In this case only the terms where $i \sin(\theta)$ has an odd exponent contribute to the imaginary part of the expression. Now we write every term of the form $\sin^{2a + 1}(\theta) \cos^b(\theta)$ as $\sin(\theta)(1 - \cos^2(\theta))^a \cos^b(\theta)$.

So we have proved the following result:

**Theorem**: *There are sequences of polynomials $T_0, T_1, ...$ and $U_1, U_2, ...$ where $T_n$ and $U_n$ are polynomials of degree $n$, such that*
$$ \cos(n \theta) + i \sin(n \theta) = T_n(\cos(\theta)) + i \sin(\theta) \cdot U_{n - 1}(\cos(\theta)) $$

The polynomials $T_n$ and $U_n$ are the famous *Chebyshev polynomials*.

**Definition**: *The **Chebyshev polynomials of the first kind** $T_n$ are defined by*
$$ T_n(\cos(\theta)) = \cos(n \theta) $$

*Likewise, the **Chebyshev polynomials of the second kind** $U_n$ are defined by*
$$ \sin(\theta) \cdot U_n(\cos(\theta)) = \sin((n + 1) \theta) $$

So, rewriting the results we found earlier in terms of polynomials, we can find the Chebyshev polynomials of the first kind:
$$ \begin{aligned}
T_0(x) & = 1 \\
T_1(x) & = x \\
T_2(x) & = 2 x^2 - 1\\
T_3(x) & = 4 x^3 - 3 x \\
T_4(x) & = 8 x^4 - 8 x^2 + 1 \\
T_5(x) & = 16 x^5 - 20 x^3 + 5x\\
T_6(x) & = 32 x^3 - 48x^4 + 18x^2 - 1 \\
...
\end{aligned} $$

And the Chebyshev polynomials of the second kind:
$$ \begin{aligned}
U_0(x) & = 1 \\
U_1(x) & = 2x \\
U_2(x) & = 4x^2 - 1 \\
U_3(x) & = 8x^3 - 4x \\
U_4(x) & = 16x^4 - 12x^2 + 1 \\
U_5(x) & = 32x^5 - 32x^3 + 6x \\
U_6(x) & = 64x^6 - 80x^4 + 24x^2 - 1 \\
...
\end{aligned} $$

I will now focus on the Chebyshev polynomials of the first kind, and in the remainder of the article the term "Chebyshev polynomials" will refer to the Chebyshev polynomials of the first kind specifically.


## Properties of the Chebyshev polynomials

Since $T_n(x) = \cos(n \arccos(x))$, the Chebyshev polynomials have a graph that essentially equals the graph of $\cos(n x)$ under a non-uniform horizontal rescaling:

**TODO: graph of first 4 Chebyshev polynomials**

**TODO: graph of cos nx for n = 0, 1, 2, 3**

This insight has some immediate applications. For example, we know the zeroes and extrema of the graph of the cosine function, so we are able to easily derive the zeroes and extrema of the Chebyshev polynomials as well.

We are only interested in extrema and zeroes $\theta \in [0, \pi)$ so that there is a one-to-one correspondence between $\theta$ and $x = \cos(\theta)$. The function $\cos(n \theta)$ has minima at $\frac{\pi + 2k \pi}{n}$ for $k \in \{ 0, 1, ..., \lfloor \frac{n - 2}{2} \rfloor \}$, maxima at $\frac{2k \pi}{n}$ for $k \in \{ 0, 1, ..., \lfloor \frac{n - 1}{2} \rfloor \}$, and zeroes at $\frac{\pi + 2k \pi}{2n}$ for $k \in \{ 0, 1, ..., n - 1 \}$. Applying the transformation $x = \cos(\theta)$ we obtain the following.

**Theorem**: *The $n$th Chebyshev polynomial $T_n(x)$ has minima at $\cos(\frac{\pi + 2k \pi}{n})$ for $k \in \{ 0, 1, ..., \lfloor \frac{n - 2}{2} \rfloor \}$, maxima at $\cos(\frac{2k \pi}{n})$ for $k \in \{ 0, 1, ..., \lfloor \frac{n - 1}{2} \rfloor \}$, and zeroes at $\cos(\frac{\pi + 2k \pi}{2n})$ for $k \in \{ 0, 1, ..., n - 1 \}$.*

From the list of Chebyshev polynomials, we see that the leading term of $T_{n + 1}$ equals $2x$ times the leading term of $T_n$. So, we can write
$$ T_{n + 1}(x) = 2x T_n(x) + r_n(x) $$

where $r(x)$ is some residual polynomial of degree $\leq n$. Writing out some of the residual polynomials:
$$ \begin{aligned} r_1(x) &= -1 \\
r_2(x) &= -x \\
r_3(x) &= -2x^2 + 1 \\
r_3(x) &= -4x^3 + 3x \end{aligned} $$

This suggests that $r_n(x) = -T_{n - 1}(x)$, so that $T_{n + 1}(x) = 2x T_n(x) - T_{n - 1}(x)$.

The proof for this turns out to be quite simple, although coming up with it is far from obvious. Take the angle addition formula for $\cos(\theta)$ at the start of this article. Substituting $\alpha = n \theta, \beta = \theta$ and $\alpha = n \theta, \beta = -\theta$, we get
$$ \begin{aligned} \cos((n + 1) \theta) &= \cos(n \theta) \cos(\theta) - \sin(n \theta) \sin(\theta) \\
\cos((n - 1) \theta) &= \cos(n \theta) \cos(\theta) + \sin(n \theta) \sin(\theta) \end{aligned} $$

Adding these expressions and re-arranging terms gives
$$ \cos((n + 1) \theta) = 2 \cos(\theta) \cdot \cos(n \theta) - \cos((n - 1) \theta) $$

After substituting $x = \cos(\theta)$ and using that, by definition of the Chebyshev polynomials, $T_n(x) = \cos(n \theta)$, we obtain the desired result:

**Theorem**: *The Chebyshev polynomials $T_0, T_1, ...$ satisfy $T_0(x) = 1$, $T_1(x) = x$, and*
$$ T_{n + 1}(x) = 2x T_n(x) - T_{n - 1}(x) $$


4. minimal/maximal property and best approximation property
5. orthonormality


## Approximation with Chebyshev polynomials


## References

[1] "The Chebyshev Polynomials: Patterns and Derivation", Benjamin Sinwell, 2004.

[2] "Chebyshev Polynomials", John D. Cook, 2008. From https://www.johndcook.com/ChebyshevPolynomials.pdf

[3] "Chebyshev polynomials", wikipedia. From https://en.wikipedia.org/wiki/Chebyshev_polynomials
