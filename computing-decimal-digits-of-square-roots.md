# Computing decimal digits of square roots

We define the integer square root $n$ of a number $N \in \mathbb{N}_0$ as the largest integer $n$ such that $n^2 \leq N$ (so we have $n = \lfloor \sqrt{N} \rfloor$). For given numbers $n$ and $N$ it easy to check if $n$ is the integer square root of $N$, because in that case we have
$$ n^2 \leq N < (n + 1)^2> $$

The difference of the square of two consecutive numbers $k, k + 1$ is the $k$th odd number:
$$ (k + 1)^2 - k^2 = 2k + 1 $$

So by summing the first $n$ odd numbers we find $n^2$:
$$ n^2 = \sum_{k = 0}^{n - 1} 2k + 1 $$

Indeed we see that $1^2 = 1$, $2^2 = 1 + 3$, $3^2 = 1 + 3 + 5$, etc.

This can be used to find the integer square root of a number $N$; We just keep on adding odd numbers until adding the next number would make the sum bigger than $N$.

Now, we have $\sqrt{N} = \frac{\sqrt{100^k N}}{10^k}$ so that $\frac{\lfloor 100^k N \rfloor}{10^k}$ gives us $\sqrt{N}$ with the first $k$ digits after the decimal dot. For example, to find the first 3 digits of $11.66$ we compute the integer square root of $100^3 \cdot 11.66 = 11660000$ and we see that
$$ \frac{\sqrt{11660000}}{10^3} = 3.414 $$

Note that this way of computing does not round the digits. The actual value of $\sqrt{11.66}$ is $3.41467$ so it would be more accurate to round up to $3.415$ here. One way to do this is to compute one extra digit $d$, and do not include it in the decimal expansion, but only to round the previous digits (so increase the last digit by one if $d \geq 5$).

It is possible to convert this algorithm to a streaming algorithm, which produces digits one by one. We already saw how to compute the digits before the decimal point. We use this to calculate an approximation $x$ to $\sqrt{N}$. When we have found a digit of $x$, we simply start increasing the next digit until we find that $x^2$ would become larger than $N$ if we increase it further.

```
x      x^2      (x + 1)^2 - x^2
-------------------------------
0      0
                1
1      1
                3
2      4
                5
3      9
                0.61
3.1    9.61               (4^2 > 11.66, go to the next digit)
                0.63
3.2    10.24
                0.65
3.3    10.89
                0.67
3.4    11.56
                0.0681
3.41   11.6281
                0.006821
3.411  11.634921          (3.42^2 > 11.66, go to the next digit)
                0.006823
3.412  11.641744
                0.006825
3.413  11.648569
                0.006827
3.414  11.655396
...                       (3.415 > 11.66, go to the next digit)
```

We see that when we move to the $n$th digit after the decimal dot, the delta becomes `2 * x / 10^n + 10^(2n)`. this makes sense because we're computing the next digit $k$ to compute a new approximation $x'$ of the form $x' = x + \frac{k}{10^n}$, which makes the next delta
$$ (x + \frac{k + 1}{10^n}) - (x + \frac{k}{10^n}) = 2x + \frac{1}{10^{2n}} $$

```
from decimal import Decimal, getcontext

def compute_sqrt_digits(input: Decimal, num_digits=10) -> Decimal:
    getcontext().prec = num_digits  # set precision of Decimal arithmetic
    precision = Decimal('0.1') ** num_digits

    delta = Decimal(1)
    digit_value = Decimal(1)
    current_approximation = Decimal(0)
    current_square_approximation = Decimal(0)

    while True:
        if current_square_approximation + delta <= input:
            current_approximation += digit_value
            current_square_approximation += delta
            delta += 2 * digit_value * digit_value
        else:
            digit_value /= 10
            delta = (2 * current_approximation + digit_value) * digit_value
            if input - current_square_approximation <= precision:
                return current_approximation
```

If $x$ is our approximation of $N$ and $N - x^2$ becomes smaller than `precision`, the function stops improving and returns $x$. This is guaranteed to return only correct digits (e.g. it can’t happen that it outputs 1.99 but the number actually turns out to be two or larger).

This code can be slightly improved by nothing that we really only `compare current_square_approximation` with `input`, so that we can use a single variable `rest = input - current_square_approximation` instead.

```
from decimal import Decimal, getcontext

def compute_sqrt_digits(input: Decimal, num_digits=10) -> Decimal:
    getcontext().prec = num_digits  # set precision of Decimal arithmetic
    precision = Decimal('0.1') ** num_digits

    delta = Decimal(1)
    digit_value = Decimal(1)
    current_approximation = Decimal(0)
    rest = input

    while True:
        if delta <= rest:
            current_approximation += digit_value
            rest -= delta
            delta += 2 * digit_value * digit_value
        else:
            digit_value /= 10
            delta = (2 * current_approximation + digit_value) * digit_value
            if rest <= precision:
                return current_approximation
```

If you want the digits to come out one by one, shift the decimal dot left or right an even number of places, so that the input number has either one or two digits before the decimal dot. If you shifted the decimal dot in the input by $2N$ places, the decimal dot in the output needs to be shifted in the other direction by $N$ places. If you do not do this, there might be a series of zeroes before the first non-zero digit. Another possibility is that the first 'digit' that is printed actually contains multiple digits. For example, if you compute the square root of 10000, the accumulating loop will take 100 steps, and then output 100 as a single digit. This is inefficient, since it will take the loop 100 iterations before moving on the next digit, instead of the maximum of 9 iterations it would take when the input would have one or two digits before the decimal dot.

A similar algorithm is described by Lionel Dangerfield on [Paul Bourke's site](http://paulbourke.net/miscellaneous/numbers/).
