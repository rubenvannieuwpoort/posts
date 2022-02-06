# Introduction to continued fractions

### (Euclidean algorithm)

To understand how one might discover continued fractions, we take a detour through the [Euclidean algorithm](https://en.wikipedia.org/wiki/Euclidean_algorithm).

The Euclidean algorithm is a simple but efficient algorithm to find the greatest common divisor $\gcd(a, b)$ of two nonnegative integers $a, b \in \mathbb{N}_0$ with $a \geq b$. It works by using the two properties
$$\gcd(a, b) = \gcd(b, r)$$

where $r$ is the remainder of $a$ after division by $b$, and
$$\gcd(a, 0) = a$$

The first property can be understood by observing that since both $a$ and $b$ are divisible by the greatest common divisor, surely $r$, which equals $a - nb$ for some positive integer $n \in \mathbb{N}_+$, is also divisible by it.

The second property follows from the fact that any number divides $0$, and the greatest divisor of $a$ is $a$ itself since we have $a \geq d$ for any divisor $d$ of $a$.

Now, to find the gcd of two numbers we use the first property until we get an expression of the form $\gcd(g, 0)$, at which point we can use the second property to see that the greatest common divisor equals $g$.

Applying this to find $\gcd(11, 3)$, the greatest common divisor of $11$ and $3$, we see that $\gcd(11, 3) = \gcd(3, 2) = \gcd(2, 1) = \gcd(1, 0) = 1$. Observe that $2$ is the remainder of $11$ after division by $3$, and likewise $1$ is the remainder of $3$ after division by $2$.


### (Extended Euclidean algorithm)

Now, the [extended Euclidean algorithm](https://en.wikipedia.org/wiki/Extended_Euclidean_algorithm) expresses the remainder in terms of $a$ and $b$ every time we use the first property. We can then use the relations we found to express $\gcd(a, b)$ as a linear combination of $a$ and $b$. This is best illustrated by just applying it to $11$ and $3$ again.

To make things clearer, we write $a = n \cdot b + r$ at each step.

$$ 11 = 3 \cdot 3 + 2 $$

$$ 3 = 1 \cdot 2 + 1 $$

$$ 2 = 2 \cdot 1 + 0 $$

So now we see that $\gcd(11, 3) = \gcd(1, 0) = 1$. We can write the second equation as $1 = 3 - 1 \cdot 2$ to get $\gcd(1, 0) = 1 = 3 - 1 \cdot 2$. From the first equation we derive $2 = 11 - 3 \cdot 3$ and substituting this yields $\gcd(1, 0) = 1 = 3 - 1 \cdot 2 = 3 - 1 \cdot (11 - 3 \cdot 3) = 4 \cdot 3 - 11$. So using the coefficients we were able to write the greatest common divisor of 11 and 3 as a linear combination of 11 and 3.


### (Reinterpretation)

Now, in each step of this process the coefficient $n$ we computed (which are directly right of the equal sign), is computed as $n = \lfloor \frac{a}{b} \rfloor$, and the remainder $r$ is computed as $r = a - n \cdot b$. This is very similar to reducing a fraction to an integer part and a fractional remainder. This is more obvious if we divide all equations from the process by $b$:

$$ \frac{11}{3} = 3 + \frac{2}{3} $$

$$ \frac{3}{2} = 1 + \frac{1}{2} $$

$$ \frac{2}{1} = 2 + \frac{0}{1} $$

Now, we see that the process is essentially repeatedly inverting the residual fraction (on the right side of the equation) and splitting the result in an integer part and a new residual fraction.

Using the relations gives us a way to write $\frac{11}{3}$ as a fraction of a peculiar form. Starting from 
$\frac{11}{3} = 3 + \frac{2}{3}$ and using $\frac{2}{3} = (\frac{3}{2})^{-1} = (1 + \frac{1}{2})^{-1}$ we can write
$$ \frac{11}{3} = 3 + \frac{1}{1 + \frac{1}{2}} $$

So, by using the extended Euclidean algorithm to calculate $\gcd(a, b)$, we can express the ratio $\frac{a}{b}$ as an integer plus some nested fraction with a special structure: the numerator is always one, and the denominator always consists of a natural number plus possibly another fraction (which has the same structure). We'll call fractions with this special structure *continued fractions*.

**Definition**: *An expression of the form*
$$ a_0 + \frac{1}{a_1 + \frac{1}{a_2 + \frac{1}{...}}} $$

*is called a **continued fraction**. Here, $a_0 \in \mathbb{Z}$ can be negative, while $a_1, a_2, ... \in \mathbb{N}$ are positive. The numbers $a_0, a_1, ...$ are called the **coefficients** or **terms** of the continued fraction.*

We also write $[a_0; a_1, ...]$ as a shorthand for $a_0 + \frac{1}{a_1 + \frac{1}{...}}$. Here, it can be useful to allow $a_0, a_1, ...$ to be real numbers instead of only integers.

Previously, we saw that we can see the extended Euclidean algorithm as an algorithm that works on a fraction $\frac{a}{b}$ and gives us a number of equations that allow us to express the fraction as a continued fraction. A bit more formally, we can write the equations for the coefficients $a_0, a_1, ...$ of the continued fraction:
$$ x_0 = \frac{a}{b}, x_{k + 1} = (x_k - a_k)^{-1} $$

$$ a_k = \lfloor x_k \rfloor $$

It is interesting to note that this re-interpretation of the extended Euclidean algorithm does not depend on $x_0$ being fractional. In fact, we can set $x_0 = x$ for any real number $x \in \mathbb{R}$, and this algorithm will give us a continued fraction representation for $x$. That is, we can use the same relations to calculate the coefficients of the continued fraction representation of an arbitrary real number.

**Example**: Suppose we want to find the first few coefficients of the continued fraction representation of $\pi$. Using a calculator with high precision, we can simply substitute numbers and use the recursion. I recommend [Wolfram Alpha](https://wolframalpha.com/) or, if you're using a Linux system, the `bc` program. We see that:

$$ x_0 = \pi = 3.14159265358979323844...$$

$$ a_0 = \lfloor x_0 \rfloor = 3 $$

$$ x_1 = (0.14159265358979323844)^{-1} = 7.06251330593104577092... $$

$$ a_1 = \lfloor x_1 \rfloor = 7 $$

$$ x_2 = (0.06251330593104577092...)^{-1} = 15.99659440668571960053... $$

$$ a_2 = \lfloor x_2 \rfloor = 15 $$

$$ x_3 = (0.99659440668571960053)^{-1} = 1.00341723101337289383... $$

$$ a_3 = \lfloor x_3 \rfloor = 1 $$

$$ x_4 = (0.00341723101337289383)^{-1} = 292.63459101437060689761... $$

$$ a_4 = \lfloor x_4 \rfloor = 292 $$

So we have found that
$$ \pi = [3; 7, 15, 1, 292, ...] $$

It might not be immediately obvious why continued fractions are useful. For now I'll just state without any evidence that they have many pleasing mathematical properties that make them both an incredible useful , as well as an excellent practical tool to find rational approximations.

The benefit of using this representation might not immediately be obvious. If we do similar computations for $\sqrt{2}$ and $e$, we find $\sqrt{2} = [1; 2, 2, 2, ...]$ and $e = [2; 1, 2, 1, 1, 4, 1, 1, 6, ...]$. These have obvious patterns, which is very convenient for computation.

We will also see that the coefficients of continued fractions behave somewhat like decimal digits in the sense that the number of coefficients increases the accuracy of the approximation.

So, without having any concrete quantative results, this already suggests that continued fractions might be useful to approximate these numbers. Let's introduce some terminology. The continued fraction that consists of the first $n$ coefficients is called the $n$th *convergent*.

**Definition**: *Let $[a_0; a_1, a_2, ...]$ be a (possibly infinite) continued fraction representation. If $n$ is less than or equal to the number of coefficients, the $n$th **convergent** of this continued fraction is*
$$ [a_0; a_1, ..., a_{n - 1}] $$

**Example**: Using the coefficients we found before, we can now find the first few continued fraction convergents of $\pi$:
$$ [3] = 3 $$

$$ [3; 7] = \frac{22}{7} = 3.142 $$

$$ [3; 7, 15] = \frac{333}{106} = 3.14150 $$

$$ [3; 7, 15, 1] = \frac{355}{113} = 3.1415929$$

$$ [3; 7, 15, 1, 292] = \frac{103993}{33102} = 3.1415926530 $$

Some observations from the computation:
  - The convergents give very good fractional approximations (for example, the best fraction approximation with denominator 100 is $\frac{314}{100} = 3.14$, which is only accurate up to three digits).
  - The convergents are alternately lower and higher than $\pi$.
  - The accuracy of the approximation increases with each convergent.

It is also noteworthy that $\frac{335}{113}$ is an especially good approximation, and the next coefficient is very big. We might conjecture that there is a relation between these facts. In a follow-up article I will prove the relation between the accuracy of the approximation and the coefficients.

The following lemma can be used to compute the value of a finite continued fraction.

**Lemma**: *We have*
$$ [a_0; a_1, ..., a_{n - 1}, a_n] = [a_0; a_1, ..., a_{n - 1} + \frac{1}{a_n} ] $$

**Proof**: This is just a matter of working out the definition.
$$ [a_0; a_1, ..., a_{n - 1}, a_n] = a_0 + \frac{1}{\ddots + \frac{1}{a_{n - 1} + \frac{1}{a_n}}} = [a_0; a_1, ..., a_{n - 1} + \frac{1}{a_n} ] $$

$\square$

The following theorem helps us to get some better understanding of how the value of a continued fraction depends on the coefficients.

**Theorem**: *Let $f(x) = [a_0; a_1, ..., a_n + x]$. Then $f(x)$ is monotonically increasing if $n$ is even, and monotonically decreasing if $n$ is odd.*

**Proof**: We use induction on $n$. For the base case $n = 0$ we have $f(x) = a_0 + x$ and the result holds. Now assume that the result holds for $n$ and consider $f(x) = [a_0; a_1, ..., a_{n + 1} + x]$. Using the previous lemma we see $[a_0; a_1, ..., a_n, a_{n + 1} + x] = [a_0; a_1, ..., a_n + \frac{1}{a_{n + 1} + x}]$. If $n + 1$ is even, $n$ is odd so this last expression is monotonically decreasing as a function of $a_n + \frac{1}{x}$, which means it's monotonically increasing as a function of $x$. When $n + 1$ is odd, $n$ is even, so this last expression is monotonically increasing as a function of $a_n + \frac{1}{x}$ and monotonically decreasing as a function of $x$.
$\square$

Now, let's try to find a way to evaluate convergents more efficiently.

From the relations we found before, we can derive $x_k = a_k + \frac{1}{x_{k + 1}}$. Simply substituting this in $x = x_0$ repeatedly gives
$$ x = x_0 = a_0 + \frac{1}{x_1} = a_0 + \frac{1}{a_1 + \frac{1}{x_2}} = a_0 + \frac{1}{a_1 + \frac{1}{a_2 + \frac{1}{x_3}}} = ... $$

We see that if we just remove the term $\frac{1}{x_k}$ from the expression, we get the $k$th convergent. We will now make this formal.

**Definition**:
$$ [a_0; a_1, ..., a_k, \infty] = \lim_{x \rightarrow \infty} [a_0; a_1, ..., a_k, x] $$

**Lemma**: *We have*
$$ [a_0; a_1, ..., a_k, \infty] = [a_0; a_1, ..., a_k] $$





**Theorem**: *Define $p_{-2} = 0, q_{-2} = 1, p_{-1} = 1, q_{-1} = 0$. The convergents $\frac{p_0}{k_0}, \frac{p_1}{q_1}, ...$ of a continued fraction $[a_0; a_1, ...]$ satisfy*
$$ \frac{p_{k + 1}}{q_{k + 1}} = \frac{a_{k + 1}p_k + p_{k - 1}}{a_{k + 1}q_k + q_{k - 1}} $$

**Algorithm**: Evaluating convergents

Now, I will continue to prove some basic properties of continued fraction.

**Theorem**: *Let $x \in \mathbb{R}$ be a real number. If $x$ is irrational, then $x$ has a unique representation as a continued fraction.*

**Theorem**: *Every rational number can be expressed as a continued fraction in exactly two ways, which are of the form*
$$ TODO $$

*and*
$$ TODO $$

**Theorem**: *A continued fraction representation of a real number $x$ is periodic if and only if $x$ is a nonrational root of a quadratic equation.*