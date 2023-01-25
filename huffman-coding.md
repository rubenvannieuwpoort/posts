# Huffman coding

If we have an alphabet of symbols and each symbol has an associated frequency, Huffman coding is an algorithm that creates an encoding of symbols into bit strings. This encoding has a nice optimality property: It minimizes the product of the frequency and the length of the encoding of a symbol, summed over all the symbols. In practice, this is useful for lossless data compression: If you assume that the symbols are independent and identically distributed, Huffman encoding provides you with the optimal "fixed" encoding. In contrast, if we allow the encoding of a character to be different when the character occurs multiple times, [arithmetic encoding](https://en.wikipedia.org/wiki/Arithmetic_coding) is optimal.

Before defining Huffman trees and investigating some of their properties, we need some definitions:
  - A **bit string** is a sequence of `0`s and `1`s
  - A bit string $P$ is a **prefix** of a bit string $B$ when $B$ starts with $P$
  - An **alphabet** is a countable set of **symbols**
  - For our purposes, an **encoding** of a symbol is a bit string
  - A **full binary tree** is a tree where each node has either 0 or 2 child nodes
  - In a tree, nodes that have no child nodes are called **leafs**

Now, a **Huffman tree** for an alphabet $S$ is a full binary tree with one-to-one correspondence between leaf nodes and symbols in $S$.

By convention, we associate the left child of a node with a `0` and the right child of a node with a `1`. A Huffman tree $T$ provides us with an encoding $E_T(s)$ of a symbol $s$. We can find it by starting at the root and following a path without loops to the leaf node associated to $s$. We start with an empty bit string, and we take the left child node we append a `0` to the end of the bit string -- if we take the right child node we append a `1` to the end. From this, we see that the length of the encoding of a symbol $s$ equals the depth of the node associated to $s$.

**Example here**

This is a Huffman tree $T$ for the alphabet `C`, `D`, `E`. We have $E_T(C) = 1$, $E_T()$ TODO TODO

The encoding process associates a mapping between a node and the bit string representing pattern that you need to follow from the root node to end up at this node. In this mapping, the bit string associated to a node $n$ is a prefix of the bit string associated to another node $m$ if and only if $m$ is a descendant of $n$. By our definition of a Huffman tree, only leaf nodes have symbols associated to them. So, if $T$ is a Huffman tree for an alphabet $S$ and $s_1$ and $s_2$ are different symbols from $S$, then $E_T(s_1)$ and $E_T(s_2)$ are never a prefix of each other.

**Theorem**: TODO

Encodings that have this property are also called **prefix-free codes**, or simply **prefix codes**.

The prefix-free property allows us to decode a bit string created by concatenating encoded symbols. If we would have two symbols $s_1$ and $s_2$ for which $E_T(s_1)$ would be a prefix of $E_T(s_2)$, we would not know how to decode a bit string that starts with $E_T(s_2)$. It could either encode $s_2$ (and then have more bits from it from other encoded symbols), or encode $s_1$ (and then have more bits from it from other encoded symbols).

TODO: EXAMPLE: encoding and decoding

If we want to compress a sequence of symbols, we want to make sure that symbols that occur often have short encodings.

EXAMPLE

So, how we construct our Huffman tree will depend on how often the symbols occur. To capture this idea, we define the **frequency** $f(s)$ of a symbol $s$. You can think of the frequency of a symbol as a weight that encodes how often the symbol occurs. So if $f(s_1)$ is approximately twice $f(s_2)$, we would expect to see $s_1$ approximately twice as often as $s_2$.

Using the frequency of the symbols, we can now define the cost $C$ of an encoding $E$ for an alphabet $S$:
$$ C(E) = \sum_{s \in S} f(s) \cdot |E(s)| $$

If we have an input file that is a sequence of $N$ symbols, we can set the frequency $f(s)$ of a symbol $s$ to the number of times this symbol occurs in the input file. This way, the cost $C(E)$ will equal the length of the encoded input file, counted in bits.

**Definition**: *An encoding $E$ is **optimal** if it minimizes the cost $C(E)$. That is, there is no other encoding $E^*$ with $C(E^*) < C(E)$. We call a Huffman tree optimal if the code it produces is optimal.*

As stated before, we'd like symbols that occur often to have shorter encodings. We can phrase this as a principle: When the frequency of a symbol $s_A$ is higher than the frequency of another symbol $s_B$, the encoding length of $s_A$ should not be longer than the encoding length of $s_B$:
$$ f(s_A) > f(s_B) \rightarrow |E_T(s_A)| \leq |E_T(s_B)| $$

Suppose that we have a Huffman tree $T$ that violates this principle and we have some $s_A, s_B$ with $f(s_A) > f(s_B)$ and $|E_T(s_A)| > |E_T(s_B)|$. Now consider the tree $T'$ obtained by switching the nodes associated to $s_A$ and $s_B$.

and
$$ E_{T'}(s) = E_T(s) $$

for any $s \in S$ with $s \neq s_A, s_B$

From this, we see
$$ C(E_T) - C(E_{T'}) = f(s_A) \cdot (|E_T(s_A)| - |E_{T'}(s_A)|) + f(s_B) \cdot (|E_T(s_B)| - |E_{T'}(s_B)|) = \Delta C $$

Now consider the expression on the right hand side. Since we switched the encoding of $s_A$ and $s_B$ in $T'$, we have
$$|E_{T'}(s_A)| = |E_{T}(s_B)| < |E_{T}(s_A)| = |E_{T'}(s_B)| $$

Setting $c = E_T(s_A) - E_T(s_B)$ we see that
$$ \Delta C = (f(s_A) - f(s_B)) \cdot c $$

Since we assumed that $f(s_A) > f(s_B)$ and $|E_T(s_A)| > E_T(s_B)$ we see that both $f(s_A) - f(s_B)$ and $c = E_T(s_A) - E_T(s_B)$ are positive. So $\Delta C$ is positive as well, and we see that $C(E_{T'}) < C(E_T)$.

**Theorem**: *If $T$ is an optimal Huffman tree for an alphabet $S$, then for any $s_A, s_B \in S$ we have*
$$ f(s_A) > f(s_B) \rightarrow |E_T(s_A)| \leq |E_T(s_B)| $$


Unfortunately this principle on it's own is not enough to decide build an optimal tree.

TODO: example

So, we need to deepen our understanding a bit before we can see how to we can come up with an optimal Huffman tree.

By the principle we discovered earlier, we can deduce that the node associated to the symbol $s$ with the lowest frequency must be a leaf with maximum depth. Now, since a Huffman tree is a full binary tree, this node must have a sibling that is also a leaf. Let's say it's associated to the symbol $t$. Now, this sibling is as deep as the node associated to $s$, so surely, $t$ must also have a very low frequency if $T$ is optimal.

In fact, there is an optimal tree where $t$ is the symbol in $S \setminus \{ s \}$ with lowest frequency. We can see this as follows: Suppose $T$ is an optimal tree where the sibling $t$ of $s$ does not have the minimum frequency. Then we can apply the same trick as before: Let $t'$ be the symbol in $S \setminus \{ s \}$ with lowest frequency. We create another tree $T'$ by switching the nodes associated to $t$ and $t'$. We see that $C(E_T) - C(E_{T'}) \leq 0$ so $T'$ is optimal.

**Theorem**: *Let $S$ be an alphabet of at least two symbols and let $s, t \in S$ be the two symbols with lowest frequency. Then there exists an optimal Huffman tree $T$ where the nodes associated to $s$ and $t$ are siblings and have maximum depth.*

Now, we're getting closer to an algorithm for generating an optimal Huffman tree. We're looking for a Huffman tree containing a particular subtree that has two leafs, associated to symbols $s$ and $t$. From this, we know that for the optimal tree $T$ we're looking for, we have $|E_T(s)| = |E_T(t)|$ since the nodes associated to $s$ and $t$ are at the same depth.

Now, we take a look at the computation of the cost function. Using that $|E_T(s)| = |E_T(t)|$, we see that we can write
$$ C(E_T) = \sum_{r \in S} f(r) \cdot |E_T(r)| = (f(s) + f(t)) \cdot |E_T(s)| + \sum_{r \in S \setminus \{ s, t \} } f(r) \cdot |E_T(r)| $$

In other words, the cost computation behaves identically to the situation where we replace the root of the subtree with a leaf node $u$ with frequency $f(u) = f(s) + f(t)$. This implies the following theorem.

**Theorem**: *Let $T$ be a Huffman tree with subtree $S$, and let $T'$ be the Huffman tree obtained by replacing $S$ with a single leaf node $s$ that has frequency $f(s)$. If $T$ is optimal then $T'$ is optimal.*

**Algorithm: Huffman coding**

Let $A$ be an alphabet, and let $f(s)$ be the frequency for any $s \in A$.
```
function huffman(A, f)
	while |A| > 1
		let s, t be the two nodes with minimal frequency

		u = node(left: s, right: t)
		u.frequency = s.frequency + t.frequency

		A = A - {s, t}
		A = A + {u}

	root = A[0]

	return root
```