# Computing discrepancy in linear time

This article describes how to compute the one-dimensional discrepancy of a sorted sequence in linear time. In textbooks like Kuipers and Niederreiters "Uniform distribution of sequences", there is an insane amount of detailed and complex information on sequences and discrepancy. However, most mathematic books like this one focus mostly on theoretical results. One thing I could not find in there is a practical algorithm to compute the discrepancy for a given sequence. After some thinking, I was able to come up with a linear algorithm. Given the simplicity it would surprise me if it is novel, but I could not find any references to an algorithm like this.

Given a sequence $X$ on an interval $I$, the [discrepancy](https://en.wikipedia.org/wiki/Equidistributed_sequence#Discrepancy) $D(X) \in (0, 1]$ is a measure of how uniform points are distributed; a low discrepancy means the points are very uniformly distributed while a high discrepancy means they are not.

For simplicity I will always use $I = [0, 1)$ in definitions and results. It should be easy to adapt the results for any interval.

**Definition 1**: *Given a finite sequence $X = x_1, x_2, ..., x_N \in [0, 1)$, the **discrepancy** $D$ is defined as*
$$ D_I(X) = \sup_{0 \leq a \leq b < 1 } \left| \frac{| X \cap [a, b] |}{N} - (b - a) \right| $$

For practical computations, it is useful to remove the supremum and the absolute value. For this purpose, I define the underpopulation and overpopulation.

TODO: definition

First, we note that the local discrepancy is not additive because we take an absolute value. So first we define some quantities that do not have this absolute value, and thus are additive. For this I came up with quantities I dubbed 'underpopulation' and 'overpopulation' (so this is not standard terminology).

**Definition 4**: *Consider a finite sequence $X = x_1, x_2, ..., x_N \in [0, 1)$.*
  - *The **local underpopulation** $U_I(X)$ on the interval $I \subset [0, 1)$ is defined as $U_I(X) = |I| - \frac{|X \cap I|}{N}$*
  - *The **global underpopulation** $U(X)$ is defined as $U(X) = \sup\limits_{0 \leq a \leq b < 1} U_I(X)$*

*Likewise, we define*
  - *The **local overpopulation** $V_I(X)$ on an interval $I \subseteq [0, 1)$ is defined as $V_I(X) = \frac{| X \cap I |}{N} - |I|$*
  - *The **global overpopulation** $V(X)$ is defined as $V(X) = \sup\limits_{0 \leq a \leq b < 1} V_I(X)$*

Now we can express the (global) discrepancy in terms of global underpopulation and overpopulation:
$$ D(X) = \max(U(X), V(X)) $$

It is somewhat easier to reason about the underpopulation and overpopulation. The following two results give us the form of the intervals that maximize the underpopulation and overpopulation.

**Lemma 5**: *Let $X$ be a collection of points of on a nonempty interval $[0, 1)$. Then*
  - *$U(X) = U_{(a, b)}(X)$ for some $a, b \in X \cup \{ 0, 1 \}$ with $a < b$*
  - *$V(X) = V_{[a, b]}(X)$ for some $a, b \in X$ with $a \leq b$*

**Proof**: For the overpopulation we have the following inequalities:
$$ V_{(a, b)}(X) \leq V_{[a, b)}(X), V_{(a, b]}(X) \leq V_{[a, b]}(X) \leq V_{[x_a, x_b]}(X) $$

where $x_a$ is the smallest element from $X$ such that $a \leq x_a$ and $x_b$ is the largest element from $X$ such that $x_b \leq b$. In every step the number of points from $X$ that fall in the interval can not decrease, and the length of the interval can not increase. So the overpopulation can only increase or stay the same between. Hence, for any interval $I$ of the form $(a, b)$, $[a, b)$, $(a, b]$, or $[a, b]$ there is an interval $V_{[x_a, x_b]}(X)$ such that $V_I(X) \leq V_{[x_a, x_b]}(X)$. So the maximum over all intervals $I$ must be attained by an interval of the form $[x_a, x_b]$ with $x_a, x_b \in X$.

Similarly, we have the following for the underpopulation:
$$ U_{[a, b]}(X) \leq U_{[a, b)}(X), U_{(a, b]}(X) \leq U_{(a, b)}(X) \leq U_{(x_a, x_b)}(X) $$

where $x_a$ is the largest element from $X$ such that $x_a \leq a$ and $x_b$ is the smallest element from $X$ such that $b \leq x_b$. This leads to an analogous conclusion for the underpopulation. $\square$

Now, a naive algorithm to compute the discrepancy would be to find the global underpopulation by just looping over all possible intervals that can maximize the underpopulation, and take the maximum of those. Then we have computed the global underpopulation. The same strategy works for the overpopulation, and the discrepancy is now simply the max of the global underpopulation and overpopulation. This gives an $O(n^3)$ algorithm (there are $O(n^2)$ intervals and we need $O(n)$ time to compute the local under- and overpopulation on an interval). If we loop over the intervals in a smart way, we can use the previously calculated result, and compute the discrepancy in constant time. This gives an $O(n^2)$ algorithm.

There is an important property of the under- and overpopulation we haven't used yet: The local under- and overpopulation are *additive* (under some restrictions):

**Theorem**: *If $I$ and $J$ are nonoverlapping intervals and $K = I \cup J$, then $U_K = U_I + U_J$ and $V_K = V_I + V_J$.*
**Proof**: TODO

Now, we can't immediately apply this, because the intervals we are interested in (because they can maximize the local under- of overpopulation) are either of the form $(a, b)$ or $[a, b]$, so they are either not disjoint or the union is not a new interval of the same form.

Now, some observations.

Given two intervals $[a, b)$ and $[b, c)$, we can use theorem TODO and conclude that
$$ U_{[a, c)}(X) = U_{[a, b)}(X) + U_{[b, c)}(X) $$

and
$$ V_{[a, c)}(X) = V_{[a, b)}(X) + V_{[b, c)}(X) $$

Further, if $X$ contains $N$ distinct points, we have
$$ U_{(a, b)}(X) = \frac{1}{N} + U_{[a, b)}(X) $$

and
$$ V_{[a, b]}(X) = \frac{1}{N} + V_{[a, b)}(X) $$

So this suggests that for a sequence $X$ that does not contain duplicate points, we can compute the underpopulations on all intervals of the form $[a, b)$ where $a \in X \cup \{ 0 \}, b \in X \cup \{ 1 \}$. Exploiting the additivity of under- and overpopulation, this reduces to the [maximum subarray problem](TODO).

Since the underpopulation on the interval $(a, b)$ is found by adding $\frac{1}{N}$, if the underpopulation over the halfopen intervals is maximized on $[a, b)$, the underpopulation over the open intervals is maximized by $(a, b)$.

The computation of the overpopulation works analogously.

However, there is an easy better

However, we haven't used the additivity of under- and overpopulation: If $I$ and $J$ are nonoverlapping intervals and $K = I \cup J$, then $U_K = U_I + U_J$ and $V_K = V_I + V_J$.

If $x_k \in X$ and $x_k$ is not a duplicate point in $X$, the interval $(x_k, x_l)$ contains exactly one point less than the interval $[x_k, x_l)$. Using the definition for the local underpopulation, we see that removing one point increases the underpopulation by $\frac{1}{|X|}$, so we have
$$ U_{(x_k, x_l)} = U_{[x_k, x_l)} + \frac{1}{|X|} $$

Likewise, for the local overpopulation we have
$$ V_{[x_k, x_l]} = V_{[x_k, x_l)} + \frac{1}{|X|} $$

when $x_l \in X$ and $x_l$ is not a duplicate point in $X$.

If we have a sorted sequence $X = (x_1, x_2, ..., x_N)$, we can compute the overpopulation $V_k$ for all intervals $I_k = [x_k, x_{k + 1})$ for $k = 1, 2, ..., N - 1$. Then, we want to find the sequence $I_k, I_{k + 1}, ..., I_{k + M}$ that maximizes the sum $V_{I_k}, V_{I_{k + 1}}, ..., V_{I_{k + M}}$ of the local overpopulations. Finally we can add $\frac{1}{N}$ to adjust for the missing point at the end of the interval.

This last step is equivalent to applying the [maximum subarray problem](https://en.wikipedia.org/wiki/Maximum_subarray_problem) to the overpopulations of the intervals. It can be solved with [Kadane's algorithm](https://en.wikipedia.org/wiki/Maximum_subarray_problem#Kadane's_algorithm), which is simple and runs in linear time.

A similar strategy can be used to compute the underpopulation. The only changes are that we allow empty subarrays in the calculation of the overpopulation but not for the underpopulation, and the calculation of the underpopulation needs some additional code to handle the intervals $[0, x_1)$ and $[x_N, 1)$.

Note that we can loop over the overpopulations/underpopulations once, so that we use constant memory:
```
# Calculates the discrepancy of an array of sorted points
#   X: sorted array of points in [0, 1] without duplicates
function discrepancy(array X):
    current_underpopulation = x[0] - 1 / size(X)
    largest_underpopulation = current_underpopulation
    current_overpopulation = 0
    largest_overpopulation = 0

    for every interval [x[i], x[i + 1]) with x[i], x[i + 1] in X:
        local_overpopulation = 1 / size(X) - (x[i + 1] - x[i])
        current_overpopulation = max(0,
		    current_overpopulation + local_overpopulation)
        largest_overpopulation = max(largest_overpopulation,
		    current_overpopulation)

        local_underpopulation = -local_overpopulation
        current_underpopulation = max(local_underpopulation,
		    current_underpopulation + local_underpopulation)
        largest_underpopulation = max(largest_underpopulation,
		    current_underpopulation)
    }

    local_underpopulation = (1 - lastPoint) - 1 / size(X)
    current_underpopulation = max(local_underpopulation,
	    current_underpopulation + local_underpopulation)
    largest_underpopulation = max(largest_underpopulation,
	    current_underpopulation)

    underpopulation = largest_underpopulation + 1 / size(X)
    overpopulation = largest_overpopulation + 1 / size(X)

    return max(underpopulation, overpopulation)
```

I also implemented a C++ version, which can be found on github.

Note that this algorithm only handles sorted sequences without duplicates. It is trivial to add a sorting step, making the algorithm $O(n \log(n))$. It is a bit harder to adapt the algorithm to handle duplicate points. This should be possible, but I haven't tried to implement it.

Finally, the maximum subarray problem can be phrased as a maximization problem. We can use this to write expression for the underpopulation and overpopulation.

**Theorem 6**: *Let $X = (...)$ be a finite sequence with $0 \leq x_0 \leq x_1 \leq ... \leq x_N < 1$. Then the discrepancy $D(X)$ satisfies*
$$ D(X) = \max(U(X), V(X)) $$

*with*
$$ U(X) = \frac{1}{N} + \max_{k = 0, 1, ..., N - 1} \left( \frac{k}{N} - (x_k - x_0) - \min_{j = 0, 1, ..., k} \left( \frac{j}{N} - (x_j - x_0) \right) \right) $$
$$ V(X) = \frac{1}{N} + \max_{k = 1, ..., N - 1} \left( x_k - \frac{k + 1}{N} - \min_{j = 0, 1, ..., k - 1} \left( x_j - \frac{j + 1}{N} \right) \right) $$

**Proof**: TODO

### The star discrepancy

As a curiosity, there is a metric related to the 'normal' discrepancy. The following results are based on those in chapter 2 from "Uniform distribution of sequences" by Kuipers and Niederreiter.

**Definition 7**: *Given a finite sequence $X = (x_1, x_2, ..., x_N)$ on a nonempty interval $[a, b]$, the **star discrepancy** $D^*(X)$ is defined as*
$$ D^*(X) = \sup_{a < d \leq b} D_{[a, d)}(X) $$

**Theorem 8**: *The discrepancy and star discrepancy are related by*
$$ D^*(X) \leq D(X) \leq 2D^*(X) $$

**Proof**: The star discrepancy is the supremum of discrepancies of $X$ over all intervals of the form $[a, d)$ with $a < d \leq b$. The discrepancy is a supremum of the discrepancy over all intervals $I \subseteq [a, b]$. Since the first set of intervals is a subset of the second, we have $D^*(X) \leq D(X)$. Further, for any interval $[c, d) \subset [a, b]$ we have
$$ \frac{| X \cap [c, d) |}{N} - \frac{d - c}{b - a} = \left( \frac{| X \cap [a, d) |}{N} - \frac{d}{b - a} \right) - \left( \frac{| X \cap [a, c) |}{N} - \frac{c}{b - a} \right) $$

Taking the absolute values of both sides, using the triangle inequality, and taking the supremum over all intervals $I \subseteq [a, b]$ we get
$$ \sup_{I \subseteq [a, b]} \left| \frac{| X \cap [c, d) |}{N} - \frac{d - c}{b - a} \right| \leq \sup_{a < d \leq b} \left| \frac{| X \cap [a, d) |}{N} - \frac{d - a}{b - a} \right| + \inf_{a < c \leq b} \left| \frac{| X \cap [a, c) |}{N} - \frac{c - a}{b - a} \right| $$

The left side is just the definition of $D(X)$. For the right side, the infimum is less than or equal to the supremum, so the right side can be bounded by twice the supremum. Since the supremum is just the definition of $D^*(X)$, we have $D(X) < 2D^*(X)$. $\square$

We can see that this bound is sharp by considering some examples. First, consider an example with only points at $0$. In this case both the discrepancy and the star discrepancy are 1, so the lower bound is sharp. On the other hand, consider the example with all the points at $\frac{1}{2}$. The star discrepancy is $\frac{1}{2}$, while the discrepancy is 1. So the upper bound is sharp as well.

The star discrepancy turns out to be easy to calculate. The proof is a bit involved, I will not include and simply refer to theorem 1.4 in Kuipers and Niederreiters "Uniform distribution of sequences".

**Theorem 9**: *Let $X = (x_1, x_2, ..., x_N)$ with $x_1 \leq x_2 \leq ... \leq x_N$ in some interval $I$. Then*
$$ D^*(X) = \max_{i = 1, 2, ..., N} \max \left( \left| x_i - \frac{i}{N} \right|, \left| x_i - \frac{i - 1}{N} \right| \right) = \frac{1}{2N} + \max_{i = 1, 2, ..., N} \left| x_i - \frac{2i - 1}{2N} \right| $$
