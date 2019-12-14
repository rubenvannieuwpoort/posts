# The Euler-Lagrange equation

The Euler-Lagrange equation is a famous equation that states a necessary condition for a function $f$ to be a local extremum of the functional

$$ I(f) = \int_a^b L(f, f', x)\ \text{d}x $$

This theorem can be used much like Fermat's theorem, which states that if $x_0$ is a local extremum of a differentiable function $f$, then we must have $f'(x_0) = 0$. Like Fermat's theorem, the Euler-Lagrange equation is nessecary but not sufficient. For Fermat's theorem, this means that while any extreme value $x_0$ satisfies $f'(x_0) = 0$, the converse does not hold: We might find a point $x_0$ such that $f'(x_0) = 0$, while $x_0$ is not an local extremum of $f$ (for an example, look at $f(x) = x^2$ at $x_0 = 0$). For the Euler-Lagrange equation, this means that while any minimizer of the of $I(f)$ must satisfy the Euler-Lagrange equation, not any $f$ that satisfies the Euler-Lagrange equation is necessarily a local extremum.

The Euler-Lagrange equations are the basis of two fields in mathematics: the *calculus of variations* and *functional analysis*. The calculus of variations deals with minimizing functionals, while functional analysis deals is about developing more general theory about functionals.

To derive the Euler-Lagrange equation, we will need some knowledge of partial derivatives (specifically, the multivariate chain rule and integration by parts), and the following two results:

**Definition**: *Suppose that $f : X \rightarrow Y$ is differentiable on some subset $X' \subseteq X$, and $Y$ is a subset of $\mathbb{R}$. A point $x_0 \in X'$ is called a **local extremum** when there is some interval $I$ containing $x_0$ such that either $f(x_0) \geq f(x)$ for all $x \in I$, or $f(x_0) \leq f(x)$ for all $x \in I$.*

**Fermat's theorem**: *Suppose that $f : X \rightarrow Y$ is differentiable with derivative $f'$ on some subset $X' \subseteq X$, and $Y$ is a subset of $\mathbb{R}$. Any local extremum $x_0$ of the function $f$ satisfies $f'(x_0) = 0$.*

**Fundamental theorem of the calculus of variations**: *Suppose that $f$ is a continuous defined on (a superset of) the interval $(a, b)$. If, for all smooth functions $\eta$ with $\eta(a) = 0$ and $\eta(b) = 0$ we have*
$$ \int_a^b f(x) \eta(x)\ \text{d}x = 0 $$
*then $f(x) = 0$ on $(a, b)$.*

Now, we are ready to state the Euler-Lagrange equation and prove that any $f$ that yields a local extremum of the functional $I$ at the top of this article, must satisfy the Euler-Lagrange equation.

**Theorem**: *Consider the functional*
$$ I(f) = \int_a^b L(f, f', x)\ \text{d}x $$

*If $f$ is a differentiable function for which this functional has a local extremum, then it must satisfy the Euler-Lagrange equation:*
$$ \frac{\partial L}{\partial f} - \frac{\text{d}}{\text{d}x} \frac{\partial L}{\partial f'} = 0 $$
**Proof**: Set $h(\alpha) = f + \alpha \eta$, where $\eta$ is any smooth function with $\eta(a) = \eta(b) = 0$. By Fermat's theorem, we have $\frac{\text{d} h}{\text{d} \alpha}|_{\alpha = 0} = 0$ if $f$ is a local extremum of $I$. Working out $\frac{\text{d} h}{\text{d} \alpha}$ gives
$$ \frac{\text{d} h}{\text{d} \alpha} = \frac{\text{d} h}{\text{d} \alpha} \int_a^b L(f, f', x)\ \text{d}x = \int_a^b \frac{\text{d}}{\text{d} \alpha} L(h, h', x)\ \text{d}x$$

$$ = \int_a^b \frac{\partial L}{\partial h} \frac{\text{d} h}{\text{d} \alpha} + \frac{\partial L}{\partial h'} \frac{\text{d} h'}{\text{d} \alpha}\ \text{d}x $$

Now we use that $\frac{\text{d} h}{\text{d} \alpha} = \eta$ and $\frac{\text{d} h'}{\text{d} \alpha} = \eta'$, and obtain

$$\int_a^b \frac{\partial L}{\partial h} \eta(x) + \frac{\partial L}{\partial h'} \eta'(x)\ \text{d}x $$

Now, consider the integral over only the second term. By using integration by parts, we see
$$ \int_a^b \frac{\partial L}{\partial h'} \eta'(x)\ \text{d}x = \frac{\partial L}{\partial h'} \eta(b) - \frac{\partial L}{\partial h'} \eta(a) - \int_a^b \frac{\text{d}}{\text{d}x}\frac{\partial L}{\partial h'} \eta(x)\ \text{d}x$$

Using that $\eta(a) = \eta(b) = 0$, this simplifies to just
$$ -\int_a^b \frac{\text{d}}{\text{d}x}\frac{\partial L}{\partial h'} \eta(x)\ \text{d}x $$

and substituting this back in our original equation yields
$$\frac{\text{d} h}{\text{d} \alpha} =  \int_a^b \left( \frac{\partial L}{\partial h} - \frac{\text{d}}{\text{d}x}\frac{\partial L}{\partial h'} \right) \eta(x)\ \text{d}x$$

Evaluating at $\alpha = 0$ gives $h = f$ and $h' = f'$. So we obtain
$$\int_a^b \left( \frac{\partial L}{\partial f} - \frac{\text{d}}{\text{d}x}\frac{\partial L}{\partial f'} \right) \eta(x)\ \text{d}x = 0$$

For any smooth $\eta$ with $\eta(a) = \eta(b) = 0$. By the fundamental lemma of the calculus of variations, it follows that
$$ \frac{\partial L}{\partial f} - \frac{\text{d}}{\text{d}x}\frac{\partial L}{\partial f'} = 0 $$

which is exactly the Euler-Lagrange equation. $\square$
