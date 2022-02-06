# Solving the maximum subarray problem with prefix sums

**Definition**: *Given a sequence $x_0, x_1, ..., x_{N - 1}$, a **subarray** of this sequence is a (possibly empty) subset of contiguous elements*
$$ x_a, x_{a + 1}, ..., x_{b - 1} $$

The [maximum subarray problem](https://en.wikipedia.org/wiki/Maximum_subarray_problem) is the problem of finding the sum of the elements of the maximum subarray for a given array.

**Maximum subarray problem**: *Given an array $x_0, x_2, ..., x_{N - 1} \in \mathbb{R}$, find the sum of the elements of the maximum subarray*
$$ \max_{\substack{a, b \in \mathbb{N}\\ 0 \leq a \leq b \leq N}} \sum_{k = a}^{b - 1} x_k $$

There is also a variation that asks for the maximum over the nonempty subarrays:

**Maximum nonempty subarray problem**: *Given an array $x_0, x_2, ..., x_{N - 1} \in \mathbb{R}$, find the maximum of the sum of elements of all the nonempty subarrays. That is, find*
$$ \max_{\substack{a, b \in \mathbb{N}\\ 0 \leq a \leq b < N}} \sum_{k = a}^{b} x_k $$

If there are only positive numbers, the maximum subarray sum and the maximum nonempty subarray sum both equal the sum of the numbers in the array. If there are only negative numbers, the maximum subarray sum is zero and the maximum nonempty subarray sum equals the largest negative number. So the interesting cases are when the sequence contains both positive and negative numbers.

A naive solution would be to loop over all indices $0 \leq a \leq b \leq N$ and compute $\sum_{k = a}^{b - 1} x_k$. There are $O(n^2)$ such pairs $(a, b)$ and the sum consists of $O(n)$ terms, so this would have a runtime of $O(n^3)$.

A slightly smarter solution would compute the sums in a smarter order so that the previously computed sum can be re-used (e.g. $sum_{k = a}^{b} x_k$ can be found in constant time by adding $x_b$ to $\sum_{k = a}^{b - 1}$). This way, every sum can be computed in $O(1)$, which gives a runtime of $O(n^2)$. 

Can we do better than this?


## A solution using prefix sums

While this problem is not incredibly hard, it is still nontrivial to solve. I will now rephrase the problem so that it has a more obvious solution.

**Definition**: *Given a sequence $x_0, x_2, ..., x_{N - 1} \in \mathbb{R}$, the $n$th **prefix sum** $S_n$ is given by the sum of the $n$ first terms:*
$$ S_n = \sum_{k = 0}^{n - 1} x_k $$

Using $\sum_{k = a}^{b - 1} x_k = S_b - S_a$, we see that the maximum subarray sum is given by
$$ \max_{0 \leq a \leq b < N}\ \sum_{k = a}^{b - 1} x_k = \max_{0 \leq b \leq N} \left(S_b - \min_{0 \leq a \leq b} S_a \right) $$

Now, prefix sums are easy to calculate, and so are the maximum and minimum over the prefix sums. So we have a rather obvious algorithm:

```
function maximum_subarray_sum(double[] x)
    prefix_sum = 0
    min_prefix_sum = 0
    max_subarray_sum = 0
    
    for b = 0 .. length(x) - 1
        prefix_sum += x[b]
        min_prefix_sum = min(min_prefix_sum, prefix_sum)
        max_subarray_sum = max(max_subarray_sum, prefix_sum - min_prefix_sum)
    
    return max_subarray_sum
```

The following invariants hold at the start of the loop:
  - `prefix_sum` equals $S_b$
  - `min_prefix_sum` equals $\min_{0 \leq a \leq b} S_a$
  - `max_subarray_sum` equals $\max_{0 \leq k \leq b} \left(S_k - \min_{0 \leq a \leq k} S_a \right)$

Informally we can see the termination of the loop as the $(N + 1)$th iteration of the loop. So at this point `max_subarray_sum` equals
$$ \max_{0 \leq b \leq N} \left(S_b - \min_{0 \leq a \leq b} S_a \right) = \max_{0 \leq a \leq b < N}\ \sum_{k = a}^{b - 1} x_k $$

The runtime of this algorithm is linear in the number of elements $N$ and the algorithm uses constant memory. This is about as good as we can hope for.


### Maximum nonempty subarray sum

The maximum nonempty subarray problem is slightly trickier to solve. We can adapt the expression we came up with for the maximum subarray problem:
$$ \max_{0 \leq a < b < N}\ \sum_{k = a}^b x_k = \max_{1 \leq b \leq N} \left(S_b - \min_{0 \leq a < b} S_a \right) $$

Comparing this expression with the one for the maximum subarray sum, it appears that we need to make some changes to the algorithm.

We need to ensure that `min_prefix_sum` is only calculated over the prefix sums less than the loop variable `b`. This can be done by swapping the updates of `prefix_sum` and `min_prefix_sum`, so that `min_prefix_sum` is updated against the previous prefix sum (e.g. $S_{b - 1}$ instead of $S_b$). This does mean that `min_prefix_sum` is compared against $S_0$ twice, but this is fine.

Further, we can't initialize `max_subarray_sum` to 0, since this corresponds to the subarray sum of an empty subarray, which are not admitted in this variant of the problem. One option is to initialize `max_subarray_sum` to a [sentinel value](https://en.wikipedia.org/wiki/Sentinel_value) that is less than all the elements in the input array.

Instead, I prefer to "move the first iteration out of the loop". After the first iteration, the only valid configuration is `min_prefix_sum = 0`, `prefix_sum = x[0]`, `max_subarray_sum = 0`. So we can simply initialize the variables to these values and skip the first iteration of the loop.

This will cause an out of bounds exception when the input array is empty. This is fine: it is reasonable and even correct to throw an exception in this case, since an empty array has no nonempty subarrays, so we can't calculate the maximum nonempty subarray. However, we probably want to add a special case to handle an empty input array so that we can return a more appriopriate exception.

With all these changes, we end up with:
```
function maximum_nonempty_subarray_sum(double[] x)
    if length(x) == 0
        throw argument_exception("Empty array has no nonempty subarrays")
    
    min_prefix_sum = 0
    prefix_sum = x[0]
    max_subarray_sum = x[0]
    
    for b = 1 .. length(x) - 1
        min_prefix_sum = min(min_prefix_sum, prefix_sum)
        prefix_sum += x[b]
        max_subarray_sum = max(max_subarray_sum, prefix_sum - min_prefix_sum)
    
    return max_subarray_sum
```

This algorithm is very similar to the one that computes the maximum subarray sum and unsurprisingly this algorithm also runs in linear time and uses constant memory.


### Conditionally allowing empty subarrays

There are only two cases when the solutions to the maximum subarray problem and the maximum nonempty subarray problem are different:
  1. All the numbers in the array are negative. In this case the maximum subarray sum is zero (since the maximum subarray is the empty subarray), while the maximum nonempty subarray is the largest number in the array.
  2. The input array is empty. In this case the maximum subarray sum is zero, while the maximum nonempty subarray sum does not exist.

So, we can make an algorithm that conditionally allows empty subarrays based on a parameter by handling an empty input array as a special case and computing the answer to the maximum nonempty subarray problem. Then, if empty subarrays are allowed, the maximum of 0 and the answer to the maximum nonempty subarray is returned. Else, the answer to the maximum nonempty subarray problem is returned.

```
function maximum_subarray_sum(double[] x, bool allow_empty_subarrays)
    if length(x) == 0
        if allow_empty_subarrays
            return 0
        throw argument_exception("Empty array has no nonempty subarrays")
    
    min_prefix_sum = 0
    prefix_sum = x[0]
    max_subarray_sum = x[0]
    
    for b = 1 .. length(x) - 1
        min_prefix_sum = min(min_prefix_sum, prefix_sum)
        prefix_sum += x[b]
        max_subarray_sum = max(max_subarray_sum, prefix_sum - min_prefix_sum)
    
    return max_subarray_sum
```


## Kadane's algorithm

For the sake of completeness, I'll also describe [Kadane's algorithm](https://en.wikipedia.org/wiki/Maximum_subarray_problem#Kadane's_algorithm) here as well, as it gives a slightly more elegant solution to the maximum subarray and maximum nonempty subarray problem. It is a nice example of [dynamic programming](https://en.wikipedia.org/wiki/Dynamic_programming).

The intuition behind the algorithm is that at any index, there's really only one subarray that you're interested in, which is the longest subarray that ends at the current index and for which all prefix sums are positive. The main insight behind this is that if a subarray has a negative prefix sum, the subarray can not be maximal, since the subarray obtained by leaving out all elements in the (negative) prefix sum is larger.

**Theorem**: *Let $x_0, x_1, ..., x_{N - 1} \in \mathbb{R}$ and let $x_a, x_{a + 1}, ..., x_{b}$ be a maximum subarray. Then all the prefix sums of the maximum subarray are nonnegative.*

**Proof**: Suppose that $x_a, x_{a + 1}, ..., x_{b}$ is a subarray of $x_0, x_1, ..., x_{N - 1}$ with subarray sum $x_0 + x_1 + ... + x_b = S$, and there is a negative prefix sum, that is, $T = x_a + x_{a + 1} + ... + x_{b'}$ is negative for some $b' \leq b$. Then we have
$$ x_{b' + 1} + ... + x_b = (x_a + ... + x_b) - (x_a + ... + x_{b'}) = S - T > S = x_a + ... + x_b$$

The inequality follows from the assumption that $S$ is negative. So a subarray can't be a maximum subarray if it has a negative prefix sum. $\square$

So, the idea is to have a variable to keep track of the prefix sum of the current subarray while we loop over the indices. Once this prefix sum becomes negative, it should be reset to zero, indicating that the current subarray we are considering now starts at the current index.
```
function max_subarray_sum(double[] x)
    current_sum = 0
    max_sum = 0
    
    for x = 0 .. length(x) - 1
        current_sum = max(0, current_sum + x)
        max_sum = max(max_sum, current_sum)
    
    return max_sum
```

This can be adapted to compute the solution to the maximum nonempty subarray problem, analogously to how we adapted the solution that uses prefix sums. We move out the first iteration and instead of resetting `max_sum` to zero, we reset it to the current element (so that the current subarray consists of the current element instead of the empty subarray).
```
function max_subarray_sum(double[] x)
    assert(length(x) > 0)
    
    current_sum = x[0]
    max_sum = x[0]
    
    for x = 1 .. length(x) - 1
        current_sum = max(x, current_sum + x)
        max_sum = max(max_sum, current_sum)
    
    return max_sum
```

Or to conditionally allow empty subarrays:
```
function max_subarray_sum(double[] x, bool allow_empty_subarrays)
    if length(x) == 0
	    if allow_empty_subarrays
			return 0
        throw argument_exception("Empty array has no nonempty subarrays")
    
    current_sum = x[0]
    max_sum = x[0]
    
    for x = 1 .. length(x) - 1
        current_sum = max(x, current_sum + x)
        max_sum = max(max_sum, current_sum)
    
    if allow_empty_subarrays
        max_sum = max(max_sum, 0)
    
    return max_sum
```
