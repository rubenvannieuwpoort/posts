# The Euler-Lagrange equation

In this article, I'll prove the *Euler-Lagrange equation* and give some examples of applications. I'll use some other theorems without proof: Fermat's theorem, the fundamental lemma of the calculus of variations, the multivariate chain rule, and integrations by parts.

**Theorem**: *Consider the functional*
$$ I(f) = \int_a^b L(f, f', x)\ \text{d}x $$

*If $f$ is a differentiable function for which this functional has a local extremum, then it must satisfy the Euler-Lagrange equation:*
$$ \frac{\partial L}{\partial f} = \frac{\text{d}}{\text{d}x} \frac{\partial L}{\partial f'} $$

**Proof**: Set $h(\alpha) = f + \alpha \eta$, where $\eta$ is any smooth function with $\eta(a) = \eta(b) = 0$. By Fermat's theorem, we have $\frac{\text{d} }{\text{d} \alpha} I(h(\alpha)) |_{\alpha = 0} = 0$ if $f$ is a local extremum of $I$. Working out $\frac{\text{d} }{\text{d} \alpha} I(h(\alpha))$ gives
$$ \frac{\text{d}}{\text{d} \alpha} I(h(\alpha)) = \frac{\text{d}}{\text{d} \alpha} \int_a^b L(f, f', x)\ \text{d}x = \int_a^b \frac{\text{d}}{\text{d} \alpha} L(h, h', x)\ \text{d}x$$

Using the multivariate chain rule, we see this last expression equals
$$ \int_a^b \frac{\partial L}{\partial h} \frac{\text{d} h}{\text{d} \alpha} + \frac{\partial L}{\partial h'} \frac{\text{d} h'}{\text{d} \alpha}\ \text{d}x $$

Note that the term $\frac{\partial L}{\partial x} \frac{\text{d} x}{\text{d} \alpha}$ does not occur since $\frac{\text{d} x}{\text{d} \alpha} = 0$.

Now we use that $\frac{\text{d} h}{\text{d} \alpha} = \eta$ and $\frac{\text{d} h'}{\text{d} \alpha} = \eta'$, and obtain

$$\int_a^b \frac{\partial L}{\partial h} \eta(x) + \frac{\partial L}{\partial h'} \eta'(x)\ \text{d}x $$

Now, consider the integral over only the second term. By using integration by parts, we see
$$ \int_a^b \frac{\partial L}{\partial h'} \eta'(x)\ \text{d}x = \frac{\partial L}{\partial h'} \eta(b) - \frac{\partial L}{\partial h'} \eta(a) - \int_a^b \frac{\text{d}}{\text{d}x}\frac{\partial L}{\partial h'} \eta(x)\ \text{d}x$$

Using that $\eta(a) = \eta(b) = 0$, this simplifies to just
$$ -\int_a^b \frac{\text{d}}{\text{d}x}\frac{\partial L}{\partial h'} \eta(x)\ \text{d}x $$

Substituting this back in our original equation yields
$$\frac{\text{d}}{\text{d} \alpha} I(h(\alpha)) =  \int_a^b \left( \frac{\partial L}{\partial h} - \frac{\text{d}}{\text{d}x}\frac{\partial L}{\partial h'} \right) \eta(x)\ \text{d}x$$

Evaluating at $\alpha = 0$ gives $h = f$ and $h' = f'$. So we obtain
$$\int_a^b \left( \frac{\partial L}{\partial f} - \frac{\text{d}}{\text{d}x}\frac{\partial L}{\partial f'} \right) \eta(x)\ \text{d}x = 0$$

For any smooth $\eta$ with $\eta(a) = \eta(b) = 0$. By the fundamental lemma of the calculus of variations, it follows that
$$ \frac{\partial L}{\partial f} - \frac{\text{d}}{\text{d}x}\frac{\partial L}{\partial f'} = 0 $$

The Euler-Lagrange equation follows after adding $\frac{\text{d}}{\text{d}x}\frac{\partial L}{\partial f'}$ to both sides.
$\square$

If the integrand $L(f, f', x)$ does not depend on $f$ or on $x$, the Euler-Lagrange equation can be simplified. Let's first consider the case where the integrand does not depend on $f$:

**Corollary**: *Consider the functional*
$$ I(f) = \int_a^b L(f', x)\ \text{d}x $$

*If $f$ is a differentiable function for which this functional has a local extremum, then*
$$ \frac{\partial L}{\partial f'} $$

*is constant.*

**Proof**: Using, the Euler-Lagrange equation and substituting $\frac{\partial L}{\partial f} = 0$, we get $\frac{\text{d}}{\text{d}x} \frac{\partial L}{\partial f'} = 0$. Taking the integral of this expression from $x_0$ to $x$, we get
$$ \int_{x_0}^x \frac{\text{d}}{\text{d}x} \frac{\partial L}{\partial f'} = \frac{\partial L}{\partial f'}(x) - \frac{\partial L}{\partial f'}(x_0) = 0 $$

Setting $\frac{\partial L}{\partial f'}(x) = C$ gives
$$ \frac{\partial L}{\partial f'} = C $$

$\square$

The case where $L$ does not depend on $x$ is similar, but less straightforward to prove.

**Corollary**: *Consider the functional*
$$ I(f) = \int_a^b L(f', f)\ \text{d}x $$

*If $f$ is a differentiable function for which this functional has a local extremum, then*
$$ \frac{\partial L}{\partial f'} f' - L $$

*is constant.*

**Proof**: By the chain rule, we can write
$$ \frac{\text{d}L}{\text{d}x} = \frac{\partial L}{\partial f} \frac{\text{d}f}{\text{d}x} + \frac{\partial L}{\partial f'} \frac{\text{d}f'}{\text{d}x} $$

Substituting the Euler-Lagrange equation $\frac{\partial L}{\partial f} = \frac{\text{d}}{\text{d}x} \frac{\partial L}{\partial f'}$ in this expression yields
$$ \frac{\text{d}L}{\text{d}x} = \left(\frac{\text{d}}{\text{d}x} \frac{\partial L}{\partial f'} \right)  \frac{\text{d}f}{\text{d}x} + \frac{\partial L}{\partial f'} \frac{\text{d}f'}{\text{d}x} $$

Rewriting the right side yields
$$ \left(\frac{\text{d}}{\text{d}x} \frac{\partial L}{\partial f'} \right)  f' + \frac{\partial L}{\partial f'} \left( \frac{\text{d}}{\text{d}x} f' \right) = \frac{\text{d}}{\text{d}x}( \frac{\partial L}{\partial f'} f') $$

The equality follows from the product rule. We now have:
$$ \frac{\text{d}L}{\text{d}x} = \frac{\text{d}}{\text{d}x}( \frac{\partial L}{\partial f'} f') $$

After subtracting $\frac{\text{d}L}{\text{d}x}$ from both sides and integrating from $x_0$ to $x$ we obtain the desired expression:
$$ \frac{\partial L}{\partial f'} f' - L= C $$

for some constant $C$.
$\square$


## Examples

The Euler-Lagrange equation is a helpful tool, but it usually requires some work to arrive at a solution. Here, I show some well-known applications of the Euler-Lagrange equation.


### The shortest path between two points is a straight line

This intuitively obvious statement is not trivial to prove. The Euler-Lagrange equation provides one way to do this.

The length of a linear line segment is $\sqrt{\Delta x^2 + \Delta y^2}$, where $\Delta x$ is the difference in the x-coordinate between the start- and endpoint, and likewise for $\Delta y$. Letting $\Delta x \rightarrow 0$, we get $\sqrt{\text{d} x^2 + \text{d} y^2}$, and integrating yields the expression
$$ \int_a^b \sqrt{\text{d} x^2 + \text{d} y^2} = \int_a^b \sqrt{1 + \left( \frac{\text{d} y}{\text{d} x} \right)^2}\ \text{d} x $$

for the length of the curve $y$ from $(a, y(a)$ to $(b, y(b))$. This "derivation" is not rigorous, but it shows how one can work with differentials intuitively.

The Euler-Lagrange equation now gives
$$ \frac{\text{d}}{\text{d}x} \frac{\partial}{\partial y} \sqrt{1 + (y')^2} = 0 $$

Calculating the derivatives gives
$$ \frac{\text{d}}{\text{d}x} \frac{y'}{\sqrt{1 + (y')^2}} = \frac{y''}{\sqrt{1 + (y')^2}^3} = 0 $$

Since the denominator is always positive, it follows that $y'' = 0$, and $y$ must be of the form
$$ y(x) = ax + b $$


### Brachistochrone

A [*brachistochrone curve*](https://en.wikipedia.org/wiki/Brachistochrone_curve) through two points $A$ and $B$ on a plane is, by definition, the curve that minimizes the time that it takes from a ball to roll from $A$ to $B$ from a standstill. Of course, this assumes that the height of $B$ is less than the height of $A$.

Tackling this problem requires some physics. The potential energy, which is the energy that the ball gets from its height is $mgh$, where $m$ is the mass of the ball, $g$ is some gravitational constant, and $h$ is the height of the ball. The kinetic energy, that the ball gets from its speed, is $\frac{1}{2} mv^2$. Since we pretend there's no friction and ignore other types of energy, the sum of these two energies is constant by the law of conservation of energy. If we assume that $m$ is constant as well, we find that
$$ \frac{1}{2} v^2 + gh$$

is constant. Re-arranging, we find $v = \sqrt{2g} \sqrt{c - h}$. Since $g$ is a constant, the speed depends only on the height $h$ of the ball, which is an interesting result in itself. It is particularly convenient to assume that the height of point $A$ is zero, so that $y = d - h$. Note that the $y$ axis points downwards in this case. In this case, we obtain:
$$v = \sqrt{2gy}$$

Now, as we used before, the length of an infinitesimal curve segment is $\sqrt{1 + (y')^2}\ \text{d}x$. The speed at $y$ is $\sqrt{2gy}$. The time that is takes a ball to roll over the line segment is simply the length of the segment divided by its speed. So we can express the time it takes to roll from $A$ to $B$ as
$$ \int_{x_A}^{x_B} \sqrt{ \frac{1 + (y')^2}{2gy}}\ \text{d}x $$

With this we finally have the integral to minimize, and we can use the Euler-Lagrange equation with $L = \sqrt{ \frac{1 + (y')^2}{2gy}}$. In fact, the integrand does not depend on $x$, so we can use a simplified version. From this, we gather that
$$ \frac{\partial L}{\partial y'} y' - L = C $$

for some constant $C$. Now, first, compute $\frac{\partial L}{\partial y'}$:
$$\frac{\partial L}{\partial y'} = \frac{1}{\sqrt{2gy}} \frac{y'}{\sqrt{1 + (y')^2}}$$

Substituting this in the previous equation yields

$$ \frac{1}{\sqrt{2gy}} \left( \frac{(y')^2}{\sqrt{1 + (y')^2}} - \sqrt{ 1 + (y')^2} \right) = C $$

Simplifying gives
$$ \frac{1}{\sqrt{2gy}} \frac{-1}{\sqrt{1 + (y')^2}} = C $$

$$ \sqrt{y (1 + (y')^2)} = -\frac{1}{\sqrt{2g}C} $$

$$ y (1 + (y')^2) = r^2 $$

with $r = \frac{1}{\sqrt{2g}C}$. Now, write $y' = \frac{\text{d}y}{\text{d}x}$. We can then rewrite to
$$ y(\text{d}x^2 + \text{d}y^2) = r^2 \text{d}x^2 $$

Now, we want to write $x$ and $y$ as a function of a parameter $t$. Divide both sides by $\text{d}t$ to get
$$ y\left(\left(\frac{\text{d}x}{\text{d}t}\right)^2 + \left(\frac{\text{d}y}{\text{d}t}\right)^2 \right) = r^2 \left( \frac{\text{d}x}{\text{d}t} \right)^2 $$

Then it can be checked that
$$ x(t) = \frac{r^2}{2}(t - \sin(t)) $$

$$ y(t) = \frac{r^2}{2}(1 - \cos(t)) $$

is a solution to this differential equation.
