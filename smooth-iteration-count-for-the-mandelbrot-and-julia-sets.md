# Smooth iteration count for the Mandelbrot and Julia sets

The exterior of the Mandelbrot set is usually colored by using the number of iterations. As the number of iterations is a discrete quantity, this leads to the occurence of banding in the coloring. In this article, I will derive a formula for a fractional iteration count. This is by no means novel; plenty of derivations can be found on the internet. However, I found most derivations to be fairly hard to follow, and hope that my explanation may be helpful to at least some people.


## Mandelbrot formula

For the Mandelbrot and Julia fractal, we iterate
$$z_{k + 1} = z_k^2 + c$$
until we have $|z_n| > R$ for some $n$. $R$ is called the *escape radius*. I will denote the magnitude of the iterands $z_k$ as $r_k = |z_k|$. I will now suppose that the starting point $z_0$ is given. I will call the first $n$ such that $r_n > R$ for a given $z_0$ the *iteration count* for the point $z_0$.

While the iterands (and thus, their magnitudes) depend on the starting point in a continuous way, the iteration count ‘jumps’ to continuous values. A starting point for which $r_n = R + \epsilon$ will have an iteration count of $n$, while a starting point with $r_n = R - \epsilon$ will have iteration count $n + 1$. If we would have some kind of measurement for the amount by which the iterand has escaped the escape radius $R$, we could use this to assign a fractional part of the iteration count. That is, the iterand with $r_n = R + \epsilon$ would have an iteration count slightly lower than $n$, while the iterand with $r_n = R - \epsilon$ would have an iteration count slightly higher than $n$.

Assume that we have a smooth function $f : \mathbb{R} \rightarrow \mathbb{C}$ such that $f(k) = z_k$. Now we want to find the fractional iteration count $x_R$ such that $|f(x_R)| = R$. For this, we assume that $R$ is much bigger than $c$, so that we have $f(x + 1) \approx f(x)^2$ (i.e. we neglect the term $c$). By applying the recurrent relation for $f(x + 1)$, we find $|f(x_R + 1)| = R^2$, $|f(x_R + 2)| = R^4$, $|f(x_R + 3)| = R^8$, and so on. If we generalize this to non-integers $x = x_R + \Delta x$ we find
$$|f(x)| = R^{2^{\Delta x}}$$

So we find the following formula for $\Delta x$:
$$ \Delta x = \log_2(\log_R(|f(x)|)) $$
So, for the first iterand $z_n$ that has $|z_n| > R$ we have $f(n) = z_n$. If we put $x = n$ we find
$$ \Delta x = \log_2(\log_R(|z_n|)) $$
Since $x = x_R + \Delta x$ and $x = n$, we see that
$$ n = x_R + \log_2(\log_R(|z_n|))$$
The fractional iteration count $x_R$ can now be found as
$$x_R = n - \log_2(\log_R(|z_n|))$$

where $R$ is the escape radius, and $n$ is the classical iteration count (i.e. the smallest $n$ such that $|z_n| > R$).
