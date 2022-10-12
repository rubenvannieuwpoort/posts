# Calculating the pseudoinverse

## Defining the pseudoinverse

Our journey starts with the following beautiful theorem about the invertibility of the linear operators corresponding to matrices.

**Theorem**: *Let $A \in \mathbb{C}^{m \times n}$ be a matrix. The function $f(x) = Ax$ is an invertible mapping from $\text{col}(A^*)$ to $\text{col}(A)$.*

**Note**: *When $A \in \mathbb{R}^{m \times n}$ has real entries, $\text{col}(A^*) = \text{row}(A)$.*

**Proof**: It suffices to show that $f$ is injective and surjective. It is easy to see that $f$ is surjective: If $y \in \text{col}(A)$, then, by definition, we can write $y = \sum_{k = 0}^{n - 1} c_k a_k$ where $c_0, c_1, ..., c_{n - 1} \in \mathbb{C}$ are coefficients and $a_0, a_1, ..., a_{n - 1} \in \mathbb{C}^m$ are the columns of $A$. So letting $x = (c_0, c_1, ..., c_{n - 1})^T \in \mathbb{C}^m$ we see that $Ax = y$.

To show that $f$ is injective we need to show that when $x_1, x_2 \in \text{col}(A^*)$ and $Ax_1 = Ax_2$ implies $x_1 = x_2$. Suppose that $x_1, x_2 \in \text{col}(A^*)$ and $Ax_1 = Ax_2$. Let $y = x_1 - x_2$. Then $y$ is a linear combination of vectors in $\text{col}(A^*)$, so $y \in \text{col}(A^*)$. Now since $Ax_1 = Ax_2$ it follows that $A(x_1 - x_2) = Ay = 0$. The $k$th entry of $Ay$ is the inner product of the $k$th column of $A^*$ and $y$. Since the inner products are all zero, $y \in \text{col}(A^*)^\perp$ by definition. Since both $y \in \text{col}(A^*)$ and $y \in \text{col}(A^*)^\perp$ it follows that $y = 0$, so $x_1 = x_2$. $\square$

Of course, when the matrix $A$ is invertible, the inverse of $f(x) = Ax$ is just $f^{-1}(y) = A^{-1}y$. However, this is the least interesting case: The remarkable thing about the theorem is that $f$ is invertible, *even when $A$ is not invertible*. Of course, the trick is the restriction of the domain and range of $f$, as the next example illustrates.

**Example**: Suppose that $A$ is the null matrix. Then $f(x) = 0$. It seems that this $f$ is not invertible. However, the column space of $A$ and $A^*$ are both equal to $\{ 0 \}$. Since $f$ is a function from $\text{col}(A^*)$ to $\text{col}(A)$, the inveres is $f^{-1}(y) = 0$.

Now let's consider an example that is slightly more interesting.

**Example**: Consider the matrix
$A = \left(\begin{array}{cc} 2 & -1 \\ 2 & -1 \end{array}\right)$. This is clearly not an invertible matrix. The column space of $A$ is $\{ \lambda \left( \begin{array}{c} 1 \\ 1 \end{array} \right) : \lambda \in \mathbb{C} \}$ and the column space of $A^*$ is $\{ \lambda \left( \begin{array}{c} 2 \\ -1 \end{array} \right) : \lambda \in \mathbb{C} \}$. We see that $\left(\begin{array}{cc} 2 & -1 \\ 2 & -1 \end{array}\right) \cdot \lambda \left( \begin{array}{c} 2 \\ -1 \end{array} \right) = \lambda \left( \begin{array}{c} 5 \\ 5 \end{array} \right)$. The inverse mapping is then $f^{-1}(\lambda \left( \begin{array}{c} 1 \\ 1 \end{array} \right)) = \frac{\lambda}{5} \left( \begin{array}{c} 2 \\ -1 \end{array} \right)$.

So, for a given matrix $A \in \mathbb{C}^{m \times n}$ the inverse $f^{-1}$ of $f(x) = Ax$ exists. However, the inverse $f^{-1}(x)$ is only defined on $\text{col}(A)$. We would like to find the pseudoinverse $A^\dagger$ that is an extension of $f^{-1}$ to all of $\mathbb{C}^n$.

For $y \in \text{col}(A)$, we have
$$ A^\dagger y = f^{-1}(y)\ \text{for $y \in \text{col}(A)$} $$

This defines $A^\dagger$ for $y \in \text{col}(A)$, but in order to uniquely define $A^\dagger$ we need to define it on $\mathbb{C}^m - \text{col}(A) = \text{col}(A)^\perp$ as well. For $y \in \text{col}(A)$, the simplest condition would be
$$ A^\dagger y = 0\ \text{for $y \in \text{col}(A)^\perp$} $$

Now, we can write any $y \in \mathbb{C}^m$ as $y = y_c + y_{c^\perp}$ with $y_c \in \text{col}(A)$ and $y_{c^\perp} \in \text{col}(A)^\perp$. So we can evaluate $A^\perp y$ as $A^\dagger(y_c + y_{c^\perp}) = A^\dagger y_c + A^\dagger y_{c^\perp} = A^\dagger y_c$. This means these two conditions indeed uniquely define a linear operator $A^\dagger$.

Multiplying the first condition by $A$ on the left makes it a bit nicer. Now, we can define the pseudoinverse:

**Definition**: *Let $A \in \mathbb{C}^{m \times n}$ be a matrix. The **pseudoinverse** $A^\dagger$ is the matrix defined by*
$$ \begin{array}{cc} AA^\dagger y = y& \text{for $y \in \text{col}(A)$} \\ A^\dagger y = 0& \text{for $y \in \text{col}(A)^\perp$} \end{array} $$

With this definition, we are ready to prove some results that help us understand why the pseudoinverse is such a useful tool to 'solve' linear systems that do not have an exact solution.

**Definition**: *For any vector $x \in \mathbb{C}^n$, the **Euclidean norm** of $x$, denoted $||x||_2$, is defined by*
$$ ||x||_2 = \sum_{k = 0}^n |x_k|^2 $$

**Theorem**: *Let $A \in \mathbb{C}^{m \times n}$ be a matrix. Then $x = A^\dagger b$ minimizes $||Ax - b||_2$. Moreover, over all minimizers of $||Ax - b||_2$, $x = A^\dagger b$ is the one that minimizes $||x||_2$.*

**Proof**: Consider $||Ax - b||_2$. Write $b = b_A + b_\perp$ with $b_A \in \text{col}(A)$, $b_\perp \in \text{col}(A)^\perp$. Then $||Ax - b||_2 = ||Ax - b_A - b_\perp||_2$. Since $Ax, b_A \in \text{col}(A)$ and $b_\perp \in \text{col}(A)^\perp$, $Ax - b_A$ and $b_\perp$ are orthogonal to each other so we have $||Ax - b||_2 = \sqrt{||Ax - b_A||_2^2 + ||b_\perp||_2^2}$. In particular, we have $||Ax - b||_2 \geq ||b_\perp||_2$.

Moreover, if $x = A^\dagger b$ then $x = A^\dagger(b_A + b_\perp) = A^\dagger b_A$ because $A^\dagger b_\perp = 0$ by definition of the pseudoinverse and since $b_\perp \in \text{col}(A)^\perp$. Then $||Ax - b||_2 = \sqrt{||Ax - b_A||_2^2 + ||b_\perp||_2^2}$. We have $AA^\dagger b_A = b_A$ so this reduces to $||b_\perp||_2$ so it follows that $||Ax - b||_2 = ||b_\perp||$ when $x = A^\dagger b$. Since $||Ax - b|| \geq ||b_\perp||_2$, we see that $x = A^\dagger b$ minimizes $||Ax - b||_2$.

Now, $||Ax - b||_2$ is minimized by any $x$ of the form $x = A^\dagger b + x_\perp$ where $x_\perp \in \text{col}(A)^\perp$. From the definition of the pseudoinverse it follows that $A^\dagger b \in \text{col}(A)$, so $A^\dagger b$ and $x_\perp$ are orthogonal, so $||x||_2 = \sqrt{||A^\dagger b||_2^2 + ||x_\perp||_2^2}$. It follows that $||x||_2$ is minimized when $x_\perp = 0$, so when $x = A^\dagger b$. $\square$


## Properties of the pseudoinverse

**Lemma**: *Let $A^\dagger \in \mathbb{C}^{n \times m}$ be the pseudoinverse of $A \in \mathbb{C}^{m \times n}$. Then $A A^\dagger$ is an orthogonal projection onto $\text{col}(A)$ and $A^\dagger A$ is an orthogonal projection onto $\text{col}(A^*)$.*

**Proof**: By definition of the pseudoinverse clearly $A^\dagger A$ is the orthogonal projection onto $\text{col}(A)$. Now take any $x \in \mathbb{C}^n$. Write $x = x_A + x_\perp$ with $x_A \in \text{col}(A^*)$ and $x_\perp \in \text{col}(A^*)^\perp$. Then $Ax = A(x_A + x_\perp) = Ax_A + Ax_\perp = Ax_A$ and then it follows from the definition of the pseudoinverse that $A^\dagger A x = x_A$. So $A^\dagger A$ is a projection onto $\text{col}(A^*)$. $\square$

Now we consider some cases for which the pseudoinverse is easy to calculate.

**Lemma**: *If $A \in \mathbb{C}^{n \times n}$ is invertible, then $A^\dagger = A^{-1}$.*

**Proof**: We simply check that the inverse satisfies the defining properties of the pseudoinverse. Since $A$ is invertible we have $\text{col}(A) = \mathbb{C}^n$ and $\text{col}(A)^\perp = \{ 0 \}$. We have $AA^{-1}y = y$ for $y \in \text{col}(A) = \mathbb{C}^n$ by the definition of the inverse. We also have $A^{-1} y = 0$ for all $y \in \text{col}(A)^\perp = \{ 0 \}$. $\square$

**Lemma**: *If $A \in \mathbb{C}^{m \times n}$ has orthonormal rows or orthonormal columns, then $A^\dagger = A^*$.*

**Proof**: Again we simply check if the defining properties of the pseudoinverse are satisfied. If $A$ has orthonormal rows or orthonormal columns, then $AA^*$ is an orthogonal projection onto $\text{col}(A)$, so $AA^*y = y$ for $y \in \text{col}(A)$ and $A^* y = 0$ for $y \in \text{col}(A)^\perp$. $\square$

**Lemma**: *Let $A \in \mathbb{C}^{n \times n}$. Then*
  - $(A^*)^\dagger = (A^\dagger)^*$
  - $(\overline{A})^\dagger = \overline{A^\dagger}$
  - $(A^T)^\dagger = (A^\dagger)^T$

**Proof**: To show that $(A^*)^\dagger = (A^\dagger)^*$ we consider the inner products $\left< (A^*)^\dagger x, y \right>$ and $\left< (A^\dagger)^* x, y \right>$ for arbitrary vectors $x \in \mathbb{C}^m$ and $y \in \mathbb{C}^n$ and show that they are the same.

Write $x = A^*x_A + x_\perp$ with $x_A \in \text{row}(A^*)$ and $x_\perp \in \text{col}(A^*)^\perp$ and $y = Ay_A + y_\perp$ with $y_A \in \text{row}(A)$ and $y_\perp \in \text{col}(A)^\perp$.

Then $\left< (A^*)^\dagger x, y \right> = \left< (A^*)^\dagger (A^*x_A + x_\perp), y \right> = \left< (A^*)^\dagger A^* x_A + (A^*)^\dagger x_\perp, y \right> = \left< (A^*)^\dagger A^* x_A, y \right> = \left< x_A, y \right> = \left< x_A, Ay_A + y_\perp \right> = \left< x_A, Ay_A \right> + \left< x_A, y_\perp \right> = \left< x_A, Ay_A \right>$ and $\left< (A^\dagger)^*x, y \right> = \left< x, A^\dagger y \right> = \left< x, A^\dagger(Ay_A + y_\perp) \right> = \left< x, A^\dagger A y_A + A^\dagger y_\perp \right> = \left< x, y_A \right> = \left< A^* x_A + x_\perp, y_A \right> = \left< A^* x_A, y_A \right> + \left< x_\perp, y_A \right> = \left< x_A, Ay_A \right>$.

So $(A^*)^\dagger = (A^\dagger)^*$.

To see that $(\overline{A})^\dagger = \overline{A^\dagger}$ we take the defining properties of the pseudoinverse for $A$, and take the complex conjugate. From this we see that $\overline{A} \overline{A^\dagger} y = y$ for $y \in \text{col}(\overline{A})$ and $\overline{A^\dagger} y = 0$ for $y \in \text{col}(\overline{A})^\perp$. So $(\overline{A})^\dagger = \overline{A^\dagger}$.

To see that $(A^T)^\dagger = (A^\dagger)^T$ note that $(A^T)^\dagger = (\overline{A^*})^\dagger = \overline{(A^\dagger)^*} = (A^\dagger)^T$. $\square$

**Lemma**: *If $A \in \mathbb{C}^{m \times r}$ has orthonormal rows or $B \in \mathbb{C}^{r \times n}$ has orthonormal columns, then $(AB)^\dagger = B^\dagger A^\dagger$.*

**Proof**: First consider the case that $B$ has orthonormal rows. Then $BB^* = I$, and from lemma TODO we know that $B^\dagger = B^*$. Yet again, we check the defining properties of the pseudoinverse.

Using $\text{col}(AB) = \text{col}(A)$, $B^\dagger = B^*$, and $B^* B = I$, the first property now reduces to $AA^\dagger y = y$ for $y \in \text{col}(A)$, and the second property reduces to $A^\dagger A x = 0$ for $y \in \text{col}(A)^\perp$. These are true by definition of the pseudoinverse of $A$. $\square$


## Calculating the pseudoinverse

Now, we want to calculate the pseudoinverse. From theorem TODO, we know that there's an invertible matrix "hidden inside" any matrix $A \in \mathbb{C}^{m \times n}$.

Now, the idea is to do a [change of basis](https://en.wikipedia.org/wiki/Change_of_basis#Linear_maps) so that instead of operating on $\mathbb{C}^n$, $A$ operates on a basis of $\text{col}(A^*)$, and maps to a basis of $\text{col}(A)$. It will be convenient to use orthonormal bases here, since this will simplify computation.

Call the matrix after the change-of-basis $M$. We change the basis of the range to an orthonormal basis of $\text{col}(A)$ and the basis of the domain to an orthonormal basis of $\text{col}(A^*)$.

For the range, we calculate the change-of-basis matrix $U$ for which the columns form an orthonormal basis for $\text{col}(A)$. For the domain we calculate a change-of-basis matrix $V$ for which the columns form an orthonormal basis of $\text{col}(A^*)$. (These matrices aren't square, strictly speaking they are projections followed by a change of basis)

We end up with the matrix $M = U^* A V \in \mathbb{C}^{r \times r}$. This matrix is an $r$-by-$r$ matrix. Since $U^*$, $A$, $V$ all have rank $r$, $M$ has rank $r$ as well. So $M$ is invertible.

If we take $M = U^* A V$ and multiply on the left by $U$ and on the right by $V^*$ we get $UMV^* = UU^*AV^*V$. Now, it can checked easily that $V^*V = I_r$ and $UU^*$ is an orthogonal projection that projects onto $\text{col}(A)$. Since $UU^*x = 0$ for every $x \in \text{col}(A)^\perp$ we have $UMV^* = A$.

We have proved the following theorem:

**Theorem**: *Any matrix $A \in \mathbb{C}^{m \times n}$ of rank $r$ has a decomposition $A = UMV^*$, where $M \in \mathbb{C}^{r \times r}$ is invertible, the columns of $U \in \mathbb{C}^{m \times r}$ form a orthonormal basis of $\text{col}(A)$, and the columns of $V \in \mathbb{C}^{n \times r}$ form an orthonormal basis of $\text{col}(A^*)$.*

Note that in general the decomposition is not unique, since an orthonormal basis of a space is not unique when the space has dimension greater than one.

It is relatively easy to compute the pseudoinverse for a matrix that is decomposed in the form of theorem 12:

**Theorem**: *If a matrix $A \in \mathbb{C}^{m \times n}$ satisfies $A = UMV^*$, where $M \in \mathbb{C}^{r \times r}$ is invertible and $U \in \mathbb{C}^{m \times r}$ and $V \in \mathbb{C}^{n \times r}$ have orthonormal columns, then*
$$ A^\dagger = V M^{-1} U^* $$

**Proof**: We want to compute $A^\dagger = (UMV^*)^\dagger$. By lemma TODO, we have $(UMV^*)^\dagger = (V^*)^\dagger (M)^\dagger (V)^\dagger$. Now, $M$ is invertible so we can use lemma TODO to see that $M^\dagger = M^{-1}$. Also, $U$ has orthonormal columns and $V^*$ has orthonormal rows, so we can use lemma TODO to see that $U^\dagger = U^*$ and $(V^*)^\dagger = V$. It follows that $A^\dagger = VM^{-1}U^*$. $\square$


## Special cases

In many special cases, we do not need to compute a full decomposition in the form that is used by theorem 12. We already saw some easy cases in the second section. Here, we consider some more special cases for which it is easy to calculate the pseudoinverse.

**Theorem**: *If $P \in \mathbb{C}^{n}$ is a projection to a space $S \subseteq \mathbb{C}^n$, and $P_\text{orth}$ is an orthogonal projection to the same space $S$ then*
$$ P^\dagger = P_\text{orth} $$

**Proof**: As usual, check that the defining properties of the pseudoinverse hold. We see that $P P_\text{orth} y = y$ for $y \in \text{col}(P)$ and $P_\text{orth} y = 0$ for $y \in \text{col}^\perp$, so indeed $P^\dagger = P_\text{orth}$. $\square$

**Theorem**: *If $A$ is full rank, the solution $x$ of*
$$ A^*Ax = Ab $$
*equals $A^\dagger b$. Moreover, if $A$ has linearly independent rows then $A^\dagger = A^*(AA^*)^{-1}$. If $A$ has linearly independent columns then $A^\dagger = (A^*A)^{-1}A^*$.*

**Proof**: Suppose $A$ has linearly independent columns. As usual by now, we check the defining properties of the pseudoinverse hold when we substitute $A^\dagger = (A^*A)^{-1}A^*$. If $y \in \text{col}(A)$ we can write $y = Ax$ for some $x$. It follows that $A(A^*A)^{-1}A^*y = A(A^*A)^{-1}A^*Ax = Ax = y$ for $y \in \text{col}(A)$. If $y \in \text{col}(A)^\perp$ it follows that $A^* y = 0$, so $(A^*A)^{-1}A^*y = 0$ for $y \in \text{col}(A)^\perp$.

If $A$ has linearly independent rows, then $A^*$ has linearly independent columns, so $(A^*)^\dagger = (AA^*)^{-1}A$. By lemma TODO it now follows that $A^\dagger = ((AA^*)^{-1}A)^* = A^*(AA^*)^{-1}$. $\square$
