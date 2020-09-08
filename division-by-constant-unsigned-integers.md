
# Division by constant unsigned integers

*Note: This article is a work in progress. The basic ideas are there, but I will try to improve the text and include more and better code in a next iteration.*

Most modern processors have an integer divide instruction which is relatively slow compared to the other arithmetic operations. When the divisor is known at compile-time or the same divisor is used for many divisions, it is possible to transform the single division to a series of instructions which execute faster. Most compilers will optimize divisions in this way. The [libdivide](https://libdivide.com/) library implements this technique for divisors which are constant at runtime but not known at compile-time. In this article, I give an overview of the existing techniques.

First, let me define some terminology. The number that is being divided is called the *dividend*. The dividend is divided by $d$, the *divisor*. The rounded down result $\lfloor \frac{n}{d} \rfloor$ is called the *quotient*.

For some special divisors, the quotient can be found very efficiently. A division by one just returns the dividend and division by another power of two can be implemented by a bit shift. When the divisor is larger than half of the maximum value of the dividend, the quotient is one when $n \geq d$ and zero otherwise. These special cases can be efficiently implemented. I will mostly ignore them and focus on more generally applicable multiplication-based methods.

Multiplication-based techniques uses the idea of multiplying by a constant that approximates the reciprocal of the divisor in fixed point math. This means that, when we want to divide by a divisor $d$, we multiply the dividend by some constant $m$ with
$$ m \approx \frac{2^k}{d} $$

and shift the result $k$ bits to the right. Here, $k$ is a measure of how precise the fixed-point approximation is to $\frac{1}{d}$. Setting $k$ higher will yield a more precise approximation, but also increase the number of bits that we need to store $m$.

Ideally, we would be able to obtain the right answer by simply ignoring everything behind the decimal point, since this can be implemented by a right shift, which is a very efficient operation. Before going any further, let's define the problem more formally. In this article, I'll only cover unsigned integers.

I assume that we are working on a processor with $N$-bit registers. I'll also assume that the processor has a `muluh` instruction, which operates on two $N$-bit unsigned integers and computes the upper $N$ bits of the product. The examples I give will be in pseudocode, in which I will use the `muluh` instruction as if it were a function. Further, I assume that the reader is familiar with the right shift operator `>>`. I will use it as a primitive in the pseudocode without further explanation.

**Definition**: *Let $\mathbb{U}_N$ denote the set of unsigned integers which can be represented in $N$ bits with the binary encoding. That is:*
$$ \mathbb{U}_N = \{ 0, 1, ..., 2^N - 2, 2^N - 1 \} $$

**Problem**: *Given a divisor $d \in \mathbb{U}_N$ which is not a power of two, find $m, k \in \mathbb{N}$ such that*
$$ \lfloor \frac{n \cdot m}{2^k} \rfloor = \lfloor \frac{n}{d} \rfloor $$

*for all $n \in \mathbb{U}_N$.*

Ideally, we would have $m \in \mathbb{U}_N$ so that we can store $m$ in a single register. Later, we will derive a condition that can be used to check if this is the case.

From this formulation we would expect that $m \approx \frac{2^k}{d}$. Suppose we pick some $d \in \mathbb{N}^+$ which is not a power of two and take $m = \lfloor \frac{2^k}{d} \rfloor$. If we now set $n = d$ and try to find the quotient $\lfloor \frac{n}{d} \rfloor$ by evaluating $\lfloor \frac{n \cdot m}{2^k} \rfloor$, we get
$$ \lfloor \frac{d \cdot \lfloor \frac{2^k}{d} \rfloor}{2^k} \rfloor = 0 < 1 = \lfloor \frac{d}{d} \rfloor $$

So we get the result $0$, instead of the expected $1$. So taking $m = \lfloor \frac{2^k}{d} \rfloor$ does not work: The product $n \cdot m$ is too small. To fix this, we can increase $m$, which is equivalent to taking $m = \lceil \frac{2^k}{d} \rceil$ when $d$ is not a power of two. Alternatively, we can slightly alter to the problem formulation so that we are allowed to increase $n$. As we will see later, these are both useful and even complementary techniques. 

If we experiment a bit, setting $m = \lceil \frac{2^k}{d} \rceil$ and picking some high enough $k$ seems to work. If we want to use this as a compiler optimization we need to figure out for which $d$ this works, and that the outcome is correct for all $n \in \mathbb{U}_N$. So, we need to do an error analysis. The following lemma turns out to be very useful.

**Lemma 1**: *Suppose that $d \in \mathbb{N}_+$ is a positive integer. When $0 \leq \epsilon < \frac{1}{d}$ we have*
$$ \lfloor \frac{n}{d} + \epsilon \rfloor = \lfloor \frac{n}{d} \rfloor $$

**Proof**: If $\epsilon = 0$ the result obviously holds. If $0 < \epsilon < \frac{1}{d}$ the expression $\frac{n}{d} + \epsilon$ is not an integer and we have
$$ \lfloor \frac{n}{d} \rfloor \leq \lfloor \frac{n}{d} + \epsilon \rfloor < \lceil \frac{n}{d} + \epsilon \rceil = \lfloor \frac{n}{d} \rfloor + 1 $$

where the equality follows from the fact that $\epsilon < 1$. Since all of these expressions in the inequality are integers, it follows that $\lfloor \frac{n}{d} + \epsilon \rfloor = \lfloor \frac{n}{d} \rfloor$.
$\square$

Now, we can write $m = \lceil \frac{2^k}{d} \rceil = \frac{2^k + e}{d}$ with $e = \text{mod}_d(-2^k)$ and substitute this in $\lfloor \frac{n \cdot m}{2^k} \rfloor$, write the resulting expression in the form $\lfloor \frac{n}{d} + \epsilon \rfloor$ with $\epsilon$ a function of $n$, $d$, $e$, and simply check under which condition we have $0 < \epsilon < \frac{1}{d}$. This gives us the following lemma:

**Lemma 2**: *Let $d \in \mathbb{N}_+$ be a positive integer and $k$ be an integer with $k \geq N$. Define $m = \lceil \frac{2^k}{d} \rceil$. If $\text{mod}_d(-2^k) \leq 2^{k - N}$, then $\lfloor \frac{m \cdot n}{2^k} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.*

**Proof**: Suppose we have $\text{mod}_d(-2^k) \leq 2^{k - N}$. Write $m = \lceil \frac{2^k}{d} \rceil = \frac{2^k + e}{d}$ with $e = \text{mod}_d(-2^k)$. Now we have
$$ \lfloor \frac{m \cdot n}{2^k} \rfloor = \lfloor \frac{\frac{2^k + e}{d} \cdot n}{2^k} \rfloor = \lfloor \frac{n}{d} + \frac{ne}{d2^k} \rfloor = \lfloor \frac{n}{d} + \epsilon \rfloor $$

with $\epsilon = \frac{e \cdot n}{2^k d}$. We have $0 \leq n < 2^N$, and, by assumption, $0 \leq e \leq 2^{k - N}$. Combining these inequalities gives
$$ 0 \leq e \cdot n < 2^k \iff 0 \leq \frac{e \cdot n}{2^k d} < \frac{1}{d} $$

Since $\epsilon = \frac{e \cdot n}{2^k d}$ the right hand side just states that $0 \leq \epsilon < \frac{1}{d}$. So the lemma applies and we have $\lfloor \frac{m \cdot n}{2^k} \rfloor = \lfloor \frac{n}{d} + \epsilon \rfloor = \lfloor \frac{n}{d} \rfloor$.
$\square$

With the two lemmas we can finally prove the following theorem:

**Theorem 1**: *Let $d, N \in \mathbb{N}_+$ be positive integers and $k_\text{min} \in \mathbb{N}$ be the smallest $k \geq N$ such that $\text{mod}_d(-2^k) \leq 2^{N - k}$. Then $k_\text{min} \leq N + \lceil \log_2(d) \rceil$. Furthermore, when $k \geq k_\text{min}$ and $m = \lceil \frac{2^k}{d} \rceil$ we have $\lfloor \frac{m \cdot n}{2^k} \rfloor = \lfloor \frac{n}{d} \rfloor$ for any $n \in \mathbb{U}_N$.*

**Proof**: We have $\text{mod}_d(-2^k) < d \leq 2^{\lceil \log_2(d) \rceil} = 2^{k - N}$. So we have $\text{mod}_d(-2^k) \leq 2^{k - N}$ and it follows that $k_\text{min} \leq N + \lceil \log_2(d) \rceil$. Now suppose that $k \geq k_\text{min}$ and define $p = k - k_\text{min}$. We now have
$$ \text{mod}_d(-2^k) = \text{mod}_d(-2^{k_\text{min} + p}) \leq 2^p \cdot \text{mod}_d(-2^{k_\text{min}}) \leq 2^p \cdot 2^{k_\text{min} - N} = 2^k  $$

So we have $\text{mod}_d(-2^k) \leq 2^{N - k}$. Define $m = \lceil \frac{2^k}{d} \rceil$. By lemma 2 it follows that $\lfloor \frac{m \cdot n}{2^k} \rfloor = \lfloor \frac{n}{d} \rfloor$ for any $n \in \mathbb{U}_N$.
$\square$

From theorem 1 it follows that when we look for the lowest $k \geq N$ such that $\text{mod}_d(-2^k) \leq 2^{k - N}$, we only need to check the values $N, N + 1, ..., N + \lceil \log_2(d) \rceil - 1$. If none of these values succeed, $k = N + \lceil \log_2(d) \rceil$ is the smallest $k$ that satisfies $\text{mod}_d(-2^k) \leq 2^{k - N}$.

Now, we would like to know how many bits we need to hold $m$. We have $1 \leq \frac{2^{\lceil \log_2(d) \rceil}}{d} < 2$, so we would expect that $\lfloor \frac{2^{p + \lceil \log_2(d) \rceil - 1}}{d} \rfloor$ and $\lceil \frac{2^{p + \lceil \log_2(d) \rceil - 1}}{d} \rceil$ both need $p$ bits to represent. The following theorem, which is surprisingly involved to prove, confirms this. Feel free to skip the proof.

**Theorem 2**: *Let $d \in \mathbb{U}_p$ with $d \neq 0$ and let $p \in \mathbb{N}_+$. Then*
$$ 2^{p - 1} \leq \lfloor \frac{2^{p + \lceil \log_2(d) \rceil - 1}}{d} \rfloor \leq \lceil \frac{2^{p + \lceil \log_2(d) \rceil - 1}}{d} \rceil < 2^p $$

**Proof**: The result can easily be checked for $d = 1$. For $p = 1$ the only $d \in \mathbb{U}_p$ is $d = 1$, so we can assume that $d, p > 1$. Note that the result follows from taking the floor and ceiling of
$$ 2^{p - 1} \leq \frac{2^{p + \lceil \log_2(d) \rceil - 1}}{d} \leq 2^p - 1 $$

So it suffices to prove this inequality. From $d \leq 2^{\lceil \log_2(d) \rceil}$ it follows that $2^{p - 1} \leq \frac{2^{p + \lceil \log_2(d) \rceil - 1}}{d}$. So it remains to show that
$$ \frac{2^{p + \lceil \log_2(d) \rceil - 1}}{d} < 2^p - 1 $$

Define $q = \lceil \log_2(d) \rceil - 1$. Since $2^{q + 1}$ is the first power of two that is larger than or equal to $d$ and $d < 2^p$, we have $2^q < d \leq 2^{q + 1} \leq 2^p$. Divide the inequality by $2^p$ to get
$$ \frac{2^q}{d} < \frac{2^p - 1}{2^p} $$

Using $d \geq 2^q + 1$ we see that $\frac{2^q}{d} \leq \frac{2^q}{2^q + 1}$, so it is enough to show
$$ \frac{2^q}{2^q + 1} < \frac{2^p - 1}{2^p} $$

Multiply both sides by $2^p \cdot (2^q + 1)$ to get
$$ 2^{p + q} < (2^p - 1) \cdot (2^q + 1) = 2^{p + q} + 2^p - 2^q - 1$$

This is equivalent to $2^q + 1 < 2^p$. Using $p > 1$ and $p > q$, this equality follows from
$$ 2^q + 1 < 2^q + 2^{p - 1} \leq 2^{p - 1} + 2^{p - 1} = 2^p $$

$\square$

Unfortunately we can now see that when we set $k = N + \lceil \log_2(d) \rceil$ and $m = \lceil \frac{2^k}{d} \rceil$ we have $2^N \leq m \leq 2^{N + 1} - 1$. This means that $m$ is exactly one bit too large to fit in an $N$-bit register. This is unfortunate but not insurmountable. In [1], a technique for multiplying by an $(N + 1)$-bit constant on a processor with $N$-bit registers is presented. This is illustrated in algorithm A.

**Algorithm A**: *Let $d \in \mathbb{U}_N$ be an $N$-bit unsigned integer divisor, let $k = N + \lceil \log_2(d) \rceil$, let $m = \lceil \frac{2^k}{d} \rceil$, and let $m' = m - 2^N$. Then the following algorithm computes $\lfloor \frac{n}{d} \rfloor$ for $n \in \mathbb{U}_N$:*

```
uint fast_divide(uint n) {
	uint hiword = muluh(n, m');
	uint intermediate = (n - hiword) >> min(k - N, 1);
	return (hiword + intermediate) >> max(k - N - 1, 0)
}
```

**Note**: *`min(k - N, 1)` and `max(k - N - 1, 0)` are both constants which can be computed at compile time when the divisor is fixed. For $d = 1$ we can set $m' = 0$ and $k = N$.*

This is a promising first method. If you just don't care about having the absolute fastest algorithm for division, I would advise you to implement this algorithm. It is relatively simple method and works for all integers. It can also be extended to the case of division of signed integers without a lot of effort.

However, it is possible to improve: For some divisors, we might be able to find a smaller $k$, so that $k < N + \lceil \log_2(d) \rceil$ and $m$ fits in a single register. For architectures on which multiplication and/or bit shifts by higher constants are slower, it may be useful to find the smallest $k$ for which $\text{mod}_d(-2^k) < 2^{k - N}$.

Following [3], we will call divisors for which $m$ fits in a single $N$-bit register *cooperative*:

**Definition**: *Let $N \in \mathbb{N}, d \in \mathbb{U}_N$. If there exists a $k \in \mathbb{N}$ such that $N \leq k < N + \lceil \log_2(d) \rceil$ with*
$$ \text{mod}_d(-2^k) \leq 2^{k - N} $$

*then $d$ is called a **cooperative divisor**. Otherwise, $d$ is an **uncooperative divisor**.*

Assuming that an `uint` is an $N$-bit unsigned integer, the following C code can be used to find $k$ given a divisor $d$. It is assumed that $N = 32$, but the code should be easy to extend.

```
uint find_k(uint d, bool *is_cooperative) {
	uint ceillog = ceil_log2(d);
	uint mod_d = ((uint)(-d)) % d; // mod_d = 2^N (mod d)
	*is_cooperative = true;
	
	for (uint k = 32; k < 32 + ceillog; k++) {
		
		// return k if the condition is satisfied
		if (mod_d <= (1 << (k - 32)) {
			return k;
		}
		
		// overflow-safe way of doubling modulo d
		if (mod_d < d - mod_d) mod_d += mod;
		else mod_d += mod_d - d;
	}
	*is_cooperative = false;
	return 32 + ceillog;
}
```

If we found a $k$ such that $N \leq k < N + \lceil \log_2(d) \rceil$ with $\text{mod}_d(-2^k) \leq 2^{k - N}$, we can use a single $N$-bit register to hold $m$ and we don't have to use tricks to multiply by an $(N + 1)$-bit constant. In this case it is simpler to compute $\lfloor \frac{n}{d} \rfloor$:

**Algorithm B**: *Let $d \in \mathbb{U}_n$ be an $N$-bit unsigned integer divisor, let $k \in \mathbb{N}$ be such that $N \leq k < N + \lceil \log_2(d) \rceil$ with $\text{mod}_d(-2^k) \leq 2^{k - N}$, and let $m = \lceil \frac{2^k}{d} \rceil$. Then the following procedure calculates $\lfloor \frac{n}{d} \rfloor$ for $n \in \mathbb{U}_N$:*

```
uint function divide_cooperative(uint n) {
	uint hiword = muluh(n, m);
	return hiword >> (k - N);
}
```

**Note**: *For some divisors $d \in \mathbb{U}_N$ we might have $k = N$ and we don't have to do the shift. In particular, this happens when $d$ is a divisor of $2^N + 1$, since in this case we have $\text{mod}_d(-2^k) = 1 \leq 2^{k - N} = 1$.*

Algorithm B is very efficient, but it can only be used when we find a $k \in \mathbb{N}$ such that $N \leq k < N + \lceil \log_2(d) \rceil$ and $\text{mod}_d(-2^k) \leq 2^{k - N}$. For even divisors for which this is not the case, we can still save the situation by first dividing the dividend $n$ by two with a bit shift. We only need $N - 1$ bits to hold the resulting dividend. Using the bound from theorem 1, we see that we can now decrease $k$ by one from $N + \lceil \log_2(d) \rceil$ to $N + \lceil \log_2(d) \rceil - 1$. So $m$ will fit in a single $N$-bit register.

If $d$ is an even uncooperative divisor, we can apply a trick: Write $d = 2^p \cdot q$ with $q$ odd. Then we can obtain the quotient $\lfloor \frac{n}{d} \rfloor$ as $\lfloor \frac{n'}{q} \rfloor$ with $n' = \lfloor \frac{n}{2^p} \rfloor$. Now $n'$ can be computed by a right shift by $p$ positions, so that $n' \in \mathbb{U}_{N - p}$. Now we can apply theorem 1 to find $m, k$ such that $\lfloor \frac{n' \cdot m}{2^k} \rfloor = \lfloor \frac{n'}{q} \rfloor$ for $n' \in \mathbb{U}_{N - p}$. Now the range of the dividend is smaller and we can certainly find an $m \in \mathbb{U}_N$.

**Corollary 1**: *Let $d, N, p, q \in \mathbb{N}_+$ be positive integers with $d = 2^p \cdot q$, where $q$ is odd. Let $k_\text{min} \in \mathbb{N}$ be the smallest $k \geq N$* such that $\text{mod}_q(-2^k) \leq 2^{N + p - k}$. Then $k_\text{min} \leq N + \lceil \log_2(q) \rceil$. Furthermore, when $k \geq k_\text{min}$, $m = \lceil \frac{2^k}{q} \rceil$, and $n' = \lfloor \frac{n}{2^p} \rfloor$ we have $\lfloor \frac{m \cdot n'}{2^k} \rfloor = \lfloor \frac{n}{d} \rfloor$ for any $n \in \mathbb{U}_N$.

**Proof**: Since $n \in \mathbb{U}_N$ and $n' = \lfloor \frac{n}{2^p} \rfloor$, we have $n' \in \mathbb{U}_{N - p}$. The result now follows from using $N - p$ instead of $N$ in theorem 1.
$\square$

When we can find $k < N$ we want to set $k = N$ since this saves us a shift. In this case the division is as efficient as for most cooperative divisors. This gives us the following algorithm:

**Algorithm C**: *Let $d \in \mathbb{U}_N$ be an $N$-bit unsigned integer divisor with $d = 2^p \cdot q$ with $p, q \in \mathbb{N}$ and $q$ odd, let $k$ be the smallest $k \in \{ N, N + 1, ..., N + \lceil \log_2(q) \rceil - p \}$ for which $\text{mod}_q(-2^k) \leq 2^{N + p - k}$, and let $m = \lceil \frac{2^k}{q} \rceil$. Then the following algorithm computes $\lfloor \frac{n}{d} \rfloor$ for $n \in \mathbb{U}_N$:*

```
uint divide_even(n) {
	uint n' = n >> p;
	uint hiword = muluh(n, m);
	return hiword >> (k - N);
}
```

For odd divisors for which $m$ does not fit in a single word, one option is to resort to algorithm A, which uses a multiplication by an $(N + 1)$-bit constant. Alternatively, we can try the other strategy mentioned before: increase $n$ instead of $m$.

**Theorem 4**: *Let $d \in \mathbb{U}_N$ be a positive number that is not a power of two, let $k \in \mathbb{N}$ such that $N \leq k < N + \lceil \log_2(d) \rceil$, and let $m = \lfloor \frac{2^k}{d} \rfloor$. Then $m \in \mathbb{U}_N$. If $\text{mod}_d(2^k) \leq 2^{k - N}$, then $\lfloor \frac{m \cdot (n + 1)}{2^k} \rfloor = \lfloor \frac{n}{d} \rfloor$ for all $n \in \mathbb{U}_N$.*

**Proof**: Write $m = \lceil \frac{2^k}{d} \rceil = \frac{2^k - e}{d}$ with $e = \text{mod}_d(2^k)$. Now we have
$$ \lceil \frac{m \cdot (n + 1)}{2^k} \rceil = \lceil \frac{\frac{2^k - e}{d} \cdot (n + 1)}{2^k} \rceil = \lfloor (1 - \frac{2}{2^k}) \cdot \frac{n + 1}{d} \rfloor$$

$$ = \lfloor \frac{n}{d} + \frac{1}{d} - \frac{e}{2^k} \cdot \frac{n + 1}{d} \rfloor = \lfloor \frac{n}{d} + \epsilon \rfloor$$

with $\epsilon = \frac{1}{d} - \frac{e}{2^k} \cdot \frac{n + 1}{d}$. Multiplying the inequalities $e \leq 2^{k - N}$ and $n + 1 \leq 2^N$ gives $e(n + 1) \leq 2^k$. Since $d$ is now a power of two, $n + 1$ and $e$ are both positive, so we have $e \cdot (n + 1) > 0$. Combining this, we see
$$ 0 < e \cdot (n + 1) \leq 2^k $$

If we multiply by $\frac{-1}{d2^k}$, which flips the inequalities, and add $\frac{1}{d}$ everywhere, we get
$$ 0 \leq \frac{1}{d} - \frac{e}{2^k} \cdot \frac{n + 1}{d} < \frac{1}{d} $$

The expression in the middle is exactly $\epsilon$, which means we have $0 \leq \epsilon < \frac{1}{d}$. So the lemma applies and we have $\lfloor \frac{n \cdot (n + 1)}{d} \rfloor = \lfloor \frac{n}{d} + \epsilon \rfloor = \lfloor \frac{n}{d} \rfloor$.
$\square$

Based on this, we can come up with the following algorithm:

**Algorithm D**: *Let $d \in \mathbb{U}_N$ be an uncooperative $N$-bit unsigned integer divisor, let $k \in \mathbb{N}$ be the smallest integer such that $N \leq k < N + \lceil \log_2(d) \rceil$ and $\text{mod}_d(2^k) \leq 2^{k - N}$, and let $m = \lfloor \frac{2^k}{d} \rfloor$. Then the following algorithm computes $\lfloor \frac{n}{d} \rfloor$ for $n \in \mathbb{U}_N$:*

```
uint divide_uncooperative(n) {
	uint hi_word = hiword(n * m + m);
	return hi_word >> (k - N);
}
```

The most efficient way to compute `hiword(n * m + m)` depends on the target architecture. If the target supports instructions to calculate the $2N$-bit product $n \cdot m$ and add the $N$-bit operand $m$ to it, these can be used. When the target supports an integer fused multiply-add that produces all $2N$ bits, this can be used. For example, in [2] Intel Celeron's `XMA.HU` instruction, is used for this purpose.

In [3], it is proposed to calculate $n \cdot m + m$ as $(n + 1) \cdot m$. The problem is that the expression `n + 1` can overflow. As a solution, it is proposed to use a *saturating increase*, which increases `n` only if it is smaller than the maximum value that an $N$-bit unsigned integer can hold. Most targets have equivalents of the `add` instruction which sets the carry flag in case of an overflow, and the `sbb` (subtract with borrow) instruction. A saturating increment on the value in register `r0` can be implemented as:
```
add r0, r0, 1
sbb r0, r0, 0
```

The following theorem states that when $d$ is an uncooperative divisor we have $\lfloor \frac{2^N - 1}{d} \rfloor = \lfloor \frac{2^N - 2}{d} \rfloor$. So if we apply algorithm D only to uncooperative divisors, the saturating increase produces the right result even when $n = 2^N - 1$.

**Theorem 5**: *Suppose $d \in \mathbb{U}_N$ is an uncooperative divisor. Then*
$$ \lfloor \frac{2^N - 1}{d} \rfloor = \lfloor \frac{2^N - 2}{d} \rfloor $$

**Proof**: Suppose that $\lfloor \frac{2^N - 1}{d} \rfloor \neq \lfloor \frac{2^N - 2}{d} \rfloor$. Then $2^N \equiv 1 \mod d$. Take $k = \lfloor \log_2(d) \rfloor$ and $e_B = \text{mod}_d(-2^k), e_D = \text{mod}_d(2^k)$. Using $\text{mod}_d(2^N) = 1$ and $2^{\lfloor \log_2(d) \rfloor} < d$, we see:
$$ e_D = \text{mod}_d(2^k) = \text{mod}_d(2^N \cdot 2^{\lfloor \log_2(d) \rfloor}) = \text{mod}_d(2^{\lfloor \log_2(d) \rfloor}) = 2^{\lfloor \log_2(d) \rfloor}$$

Here we used that $2^{\lfloor \log_2(d) \rfloor} < d$. We have $e_B + e_D = d$ since $e_B, e_D \equiv 0 \mod d$ and $e_B, e_D' \in \{ 0, 1, ..., d - 1 \}$. So
$$ e_B = d - e_D \leq 2^{\lceil \log_2(d) \rceil} - 2^{\lfloor \log_2(d) \rfloor}  = 2^{\lfloor \log_2(d) \rfloor} = 2^{k - N} $$

So it follows that $d$ is a cooperative divisor. This is a contradiction, since we assumed that $d$ is uncooperative. So we must have $\lfloor \frac{2^N - 1}{d} \rfloor = \lfloor \frac{2^N - 2}{d} \rfloor$.
$\square$

Note that algorithm D assumes the existence of a $k \in \mathbb{N}$ such that $N \leq k < N + \lceil \log_2(d) \rceil$ and $\text{mod}_d(2^k) \leq 2^{k - N}$. The following theorem states that such a $k$ indeed exists when $d \in \mathbb{U}_N$ is an uncooperative divisor.

**Theorem 6**: *Let $d \in\mathbb{U}_N$ be an uncooperative divisor. Then there exists a $k \in \mathbb{N}$ with $N \leq k < N + \lceil \log_2(d) \rceil$ such that $\text{mod}_d(2^k) \leq 2^{k - N}$.*

**Proof**: Assume that $d$ is an uncooperative divisor. Set $k = N + \lceil \log_2(d) \rceil - 1$ and $e_B = \text{mod}_d(-2^k), e_D = \text{mod}_d(2^k)$. We now have $e_B + e_D = d$ since $e_B + e_D \equiv 0 \mod d$ and $e_B, e_D \in \{ 0, 1, ..., d - 1 \}$. Since we assumed that $d$ is an uncooperative divisor we have $e_B > 2^{k - N} = 2^{\lceil \log_2(d) \rceil - 1}$ so that $e_D \leq d - 2^{\lceil \log_2(d) \rceil - 1}$. Using that $\frac{d}{2} \leq 2^{\lceil \log_2(d) \rceil - 1}$ we see
$$ e_D \leq d - 2^{\lceil \log_2(d) \rceil - 1} \leq \frac{d}{2} \leq 2^{\lceil \log_2(d) \rceil - 1} = 2^{k - N}$$

So we have $e_D = \text{mod}_d(2^k) \leq 2^{k - N}$.
$\square$

So, if we use algorithm B for cooperative divisors and algorithm D for uncooperative divisors, we can always fit $m$ in an $N$-bit word.


## Compiler optimizations

Here, I give a sketch of how the code generation for a compiler backend could look, together with generated code for a hypothetical 32-bit target instruction set with the following instructions:

| Instruction     | Description |
|-----------------|:-------------:|
| `shr a, b, c`   | Shift register `b` by `c` bits, store the result in register `a`.|
| `gte a, b, c`   | Set register `a` to 1 if `b` is greater or equal to `c` and to 0 otherwise.|
| `umulhi a, b, c`| Store the N high bits of the unsigned product `b * c` in `a`.|
|`add a, b, c`    | Store the sum of `b` and `c` in `a`, and set the carry/borrow bit if the sum `b + c` doesn't fit in 32 bits.|
|`sbb a, b, c`   | Store `b - c - <carry bit>` in `a`.|
|`ret`            | Return from function (implementation details are irrelevant). |

The register `c` can be replaced by a 32-bit constant. I will assume that the target uses registers `r0`, `r1`, `r2`, and so on. As far as calling conventions go, the first argument will be passed in `r0`, just like the return value.

I assume that the compiler backend has an `expression` type, which can contain nested calls to instructions like `shr`, `umulhi`, etc. An unsigned integer `v` can be passed as a compile-time constant expression by writing `const(v)`.

I use the following function as an example:

```
const uint32_t d = ...;

uint32_t divide(uint32_t n) {
	return n / d;
}
```

For the optimization of multiplication by a constant, we have the following flowchart. The right arrow should be followed if the question above the node is true, else the left arrow should be followed.
![Flowchart for optimization of a multiplication by a constant unsigned integer](https://i.imgur.com/BugqAFB.png)

Roughly speaking, the algorithms are ordered from least efficient on the left, to most efficient on the right.

In pseudocode, we can implement the compiler backend for the division by a constant unsigned integer as follows:
```
expression div_by_const_uint(const uint d, expression n) {
	if is_power_of_2(d) {
		if (d == 1) return n;
		else return shr(n, log2(d));
	}
	else {
		if (d > max_value / 2) return gte(n, max_value / 2 + 1);
		else return mul_based_div_by_const_uint(d, n);
	}
}
```

When $d = 1$, the divide function will look as follows:
```
divide:
	ret
```

That's right, an empty function. The return value will be the same as the input value, since both the first argument and the return value are passed in `r0`.

When $d = 2^p$ for some $p > 0$ we'll get a bit shift. Say $d = 16$. Then the `divide` function will get compiled to:

```
divide:
	shr r0, r0, 4
	ret
```

For a divisor more than $2^{32}$ we'll get a comparison instruction. For example, taking $d = 2147483649$:

```
divide:
	gte r0, r0, 2147483649
	ret
```

This were the easy special cases. For the multiplication-based techniques, we have the following flowchart:
![Flowchart for multiplication-based optimization of a multiplication by a constant unsigned integer](https://i.imgur.com/lQOKqIe.png)

Algorithms on the right are usually slightly more efficient.

This translates into the following pseudocode:
```
expression mul_based_div_by_const_uint(const uint d, expression n) {
	uint k, m;
	if (is_cooperative(d, &k, &m, N)) {
		return shr(umulhi(n, const(m)), const(k - N);
	}
	else {
		if (d % 2 == 0) {
			uint p = 0, q = d;
			while (q > 0 && q % 2 == 0) {
				q = q / 2;
				p = p + 1;
			}
			calculate_even(p, q, &k, &m, N);
			expression n_prime = shr(n, const(p));
			return shr(umulhi(n_prime, const(m)), const(k - N));
		}
		else {
			calculate_uncooperative(d, &k, &m);
			expression n_sat_inc = sbb(add(n, 1), 0);
			return shr(umulhi(n_sat_inc, const(m)), const(k - N));
		}
	}
}
```

First, for a cooperative divisor like $d = 3$, we have
```
divide:
	umulhi r0, r0, 2863311531
	shr r0, r0, 33
	ret
```

For divisors of $2^{32} + 1$ (the only divisors of $2^{32}$ are $641$ and $6700417$), the right shift can be omitted. For example, for $d = 641$ we have:
```
divide:
	umulhi r0, r0, 6700417
	ret
```

For even non-cooperative divisors, such as $d = 14$, we have
```
divide:
	shr r0, r0, 1
	umulhi r0, r0, 2454267027
	shr r0, r0, 34
	ret
```

For a divisor with more factors two, it might be possible to omit the last shift. For example, when we set $d = 28$:
```
divide:
	shr r0, r0, 1
	umulhi r0, r0, 613566757
	ret
```

Finally, for uncooperative divisors such as $d = 7$, we get:
```
divide:
	add r0, r0, 1
	sbb r0, r0, 0
	umulhi r0, r0, 1227133513
	shr r0, r0, 33
	ret
```


## Runtime optimizations

Suppose we want to perform faster division by a runtime constant. In this case we have to compute the magic value at runtime, and call a function `fast_divide` to perform the division. For the sake of efficiency, we want this function to be branchless (it should not contain any if statements).

Following [2], we note that both algorithm B and algorithm D follow the common pattern of computing $ax + b$ and right shifting it by $k$ bits. So the following struct holds enough information:

```
struct div_info {
	uint a, b, k;
};
```

The `fast_divide` function now looks like
```
uint fast_divide(uint n, div_info di) {
	uint hi_word = (uint)(((big_uint)di.a * n + di.b) >> N);
	return (uint)(hi_word >> (di.k - N));
}
```

For cooperative divisors, we use algorithm B and set $a = m, b = 0$. For uncooperative divisors, we use algorithm D and set $a = m, b = m$. For $d = 1$ we need a special case that sets $a = 2^N - 1, b = 1, k = 32$.

This is certainly not the only way to implement the runtime optimization, but I think it's the most elegant.


## On testing

If you implement these techniques, I recommend that you work out some examples for 4-bit numbers. After that, you can implement the methods for 8- and 16-bit numbers. It should be possible to test these exhaustively. That is, you should be able to test if your code gets the quotient $\lfloor \frac{n}{d} \rfloor$ right for every $n, d \in \mathbb{U}_N$ with $d \neq 0$. For 32-bit numbers, it should still be possible to check, for all $d \neq 0$, the value of the quotient $\lfloor \frac{n}{d} \rfloor$ where $n$ is a multiple of $d$ or the integer before that. You should also test $n = 0$ and $n = 2^{32} - 1$. For 64-bit quotients, it is impossible to check all possibilities within a reasonable time. However, you can just pick a random value for $d$, and check the quotient for $n = 0$ and $n = 2^{64} - 1$ along with some random values for $n$. If your code works for all 32-bit integers and you have no failing cases after a long time of random testing, you have good reason to believe that there are no blatant errors in your code.


## Further reading

The classic work on the subject of dividing by constants is [1], which also also covers division by signed constants and testing for divisibility. Algorithm A is based on this work. In [3] and [4] a very readable introduction to division of unsigned constants is given. Further, the idea of using the saturating increment in algorithm D is introduced. The paper [2] contains many similar ideas, but is a bit more formal.

### References

[1]: [Division by Invariant Integers using Multiplication](https://gmplib.org/~tege/divcnst-pldi94.pdf), Torbjörn Granlund and Peter L. Montgomery, 1994.

[2]: [N-Bit Unsigned Divison Via N-Bit Multiply-Add](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.512.2627&rep=rep1&type=pdf), Arch D. Robinson, 2005.

[3]: [Labor of Divison (Episode I)](https://ridiculousfish.com/blog/posts/labor-of-division-episode-i.html), fish, 2010.

[4]: [Labor of Divison (Episode III): Fast Unsigned Division by Constants](https://ridiculousfish.com/blog/posts/labor-of-division-episode-iii.html), fish, 2011.
