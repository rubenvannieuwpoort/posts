# The singular value decomposition

In this article, I will prove that every matrix has a *compact singular value decomposition*.


**Theorem**: *Any matrix $M \in \mathbb{C}^{m \times n}$ of rank $r$ has a decomposition*
$$ M = U \Sigma V^* $$

*where $U \in \mathbb{C}^{m \times r}$ and $V \in \mathbb{C}^{n \times r}$ are unitary matrices, and $\Sigma \in \mathbb{R}^{r \times r}$ is a diagonal matrix with positive elements on the diagonal.*

**Proof**: Suppose that $m \leq n$. If $m > n$, we can look at the singular value decomposition $M^* = U \Sigma V^*$ of the conjugate transpose $M^*$ of $M$. Using $M = (M^*)^* = (U \Sigma V^*)^* = V \Sigma U^*$, we find the singular of $M$. So we can assume $m \leq n$ without loss of generality.

Now consider the $n \times n$ matrix $M^* M$, which is Hermitian. Using the spectral theorem for Hermitian matrices, we have
$$ M^* M = V \Lambda V^* $$

where $V$ is a unitary matrix with $n$ orthonormal eigenvectors $v_1, v_2, ..., v_n$ of $M^* M$, and $\Lambda$ is a diagonal matrix, where the $k$th element $\lambda_k$ on the diagonal is the eigenvalue of $v_k$. According to the first lemma, all the $\lambda_k$ are nonnegative.

We can assume that $\lambda_1 \geq \lambda_2 \geq ... \geq \lambda_n$. If this is not the case, we can simply permute the columns of $V$ and $\Sigma$ so that this condition holds. Since all the eigenvectors are orthogonal, and the rank of $M^* M$ is $r$ by the first lemma, the eigenvalues $\lambda_1, \lambda_2, ..., \lambda_r$ are positive, and $\lambda_{r + 1}, \lambda_{r + 2}, \lambda_n$ are zero.

Now define
$$\sigma_k = \sqrt{\lambda_k}$$

$$ u_k = \frac{A v_k}{\sigma_k} $$

for
$$ k = 1, 2, ..., r $$

and let $U \in \mathbb{C}^{m \times r}$ be the matrix with $u_1, u_2, ..., u_r$, $V \in \mathbb{C}^{n \times r}$ be the matrix with $v_1, v_2, ..., v_r$ as columns, and $\Sigma \in \mathbb{R}^{r \times r}$ be the diagonal matrix with $\sigma_1, \sigma_2, ..., \sigma_r$ on the diagonal.

I claim now that $M = U \Sigma V^*$ is the desired decomposition. First, the elements of $U^* U$ are given by $u_i^* u_j = \frac{(Av_i)^* Av_j}{\sigma_i \sigma_j} = \frac{v_i^* (A^* A v_j)}{\sigma_i \sigma_j} = \lambda_j \frac{\left< v_i, v_j  \right>}{\sigma_i \sigma_j} = \delta_{i, j}$, so $U$ is unitary. I proceed with showing that $U \Sigma V^* v_k = M v_k$ for $k = 1, 2, ..., n$. The eigenvectors $v_1, v_2, ..., v_n$ of $M^*M$ form an orthonormal basis of $\mathbb{C}^n$, so if the equality holds for each of these vectors, it holds for the whole of $\mathbb{C}^n$ by linearity.

For $1 \leq k \leq r$, we have that $V^* v_k = e_k$. So $Mv_k = U \Sigma V^* v_k = U \sigma_k e_k =\sigma_k U e_k = \sigma_k u_k = A v_k$. On the other hand, if $r < k \leq n$, we have $V^* v_k = 0$, so $U \Sigma V^* v_k = 0$. This is correct, since $v_k$ is an eigenvector of $M^* M$ with eigenvalue zero, which implies that $|| M v_k || = \sqrt{ v_k ^* M^* M v_k} = \left< v_k, 0 \right> = 0$. This means that $M v_k = 0$.

$\square$


**Lemma**: *$M^* M$ has only real, nonnegative eigenvalues.*
**Proof**: Let $v$ be any eigenvector of $M^* M$ and $\lambda$ be the corresponding eigenvalue, so that $M^* M v = \lambda v$. We have $\lambda = \frac{|| M v ||^2}{||v||^2}$ since $|| M v ||^2 = v^* M^* M v = \lambda v^* v = \lambda || v ||^2$. Now, $\frac{|| M v ||^2}{||v||^2}$ is obviously real and nonnegative, so $\lambda$ is as well.
$\square$

**Lemma**: Let $r$ be the rank of a complex matrix $M$. Then $M^* M$ has rank $r$ as well.
**Proof**: The rank of a matrix is determined by the dimension of the null space, so it suffices to show that $Mv = 0 \iff M^* M v = 0$, since this implies that the null spaces are the same.

Now, from $Mv = 0$ it directly follows that $M^* M v = M^* 0 = 0$. It is a property of the norm that $|| v || = 0 \iff v = 0$. So, conversely, we have that if $M^* M v = 0$, then $v^* M^* M v = (Mv)^* Mv = || Mv ||^2 = 0$, which implies $v = 0$.

$\square$


I have chosen to prove the existence of the compact singular value decomposition instead of the `normal' singular value decomposition, because it feels much more natural. The existence of the normal singular value decomposition follows easily, since we can just add extra zeroes to the diagonal, and add more columns to $U$ and $V$, without changing the product $U \Sigma V^*$.

The singular value decomposition has numerous applications. It can be used to compute the pseudoinverse of a matrix, to perform principal component analysis, and it can be used to approximate a matrix $M$ by a low-rank approximation $\tilde{M}$.
