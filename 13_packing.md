# Packing to fewer dimensions
Given the term-document matrix $A \in \mathbb{R}^{m\times n}$, we can associate every document to a column of the matrix, as in $d_i = A^{(i)}$.
In the vector space model we already argued that the computation of the similarities between documents is an hard problem because of the high-dimensionality of the document vectors.
We want to discuss how to speedup the cosine similarity with two techniques that are usually used subsequently, first reducing the number of dimensions by random projection, and then by further reducing via LSI.

## LSI
The Latent Semantic Index is a data-dependent solution that creates a $k$-dim subspace by eliminating redundant axes by pulling together related dimension, aiming to resolve also synonymy and polysemy issues.
To reduce the matrix $A$ the documents are pre-processed by using a linear algebra technique called Singular Value Decomposition (SVD).
The decomposition creates a new smaller vector space, allowing to faster handle queries.

From the matrix $A$ we generate a term-term correlation matrix $T = A A^T$ and a document-document correlation matrix $D = A^T A$.
We can notice that $T_{i,j} = \langle t_i, t_j \rangle$ representing in our vector space model the similarity between the terms $t_i$ and $t_j$, the same can be argued about $D_{i,j}$ in respect to the documents.
Suppose that the matrix $A$ had a rank $r \leq m,n$, we can now define $U$ as the matrix containing the $r$ linearly independent eigenvectors of $T$, and $V$ as the same matrix in respect to $D$.
We have now all the necessary bricks to construct the SVD decomposition of $A = U S V^T$, where $S$ is a diagonal matrix containing in decreasing order the eigenvalues of A[^4].

By taking $k \ll r$ we can select the $k$ biggest eigenvalues and their relative eigenvectors, reducing all the three matrices needed for the SVD and computing so $A_k = U_k S_k V_k^T$.
$A_k$ is provable as the best $k$-rank approximation of $A$, that is:

$$
\|A-A_k\| = \min_{B\in\mathbb{R}^{m\times n}} \|A-B\|
$$

Since we are interested in the similarities between documents we can define a new matrix $X = S V^T$, and its corresponding rank $k$ approximation $X_k = S_k V_k^T$.
This is sound because of $D = A^T A = X^T X$, so that $X$ may substitute $A$ when computing the document similarity.
$X_k$ has one column per document but the features are reduced from $m$ terms to $k$ concepts.

The similarity of the documents $d_i$ and $d_j$ can now be approximated by using only $k$ multiplications if considering the following approximation:

$$
\begin{aligned}
s(d_i, d_j) \\
= \langle d_i, d_j \rangle \\
= \langle A^{(i)}, A^{(j)} \rangle \\
= \langle X^{(i)}, X^{(j)} \rangle \\
\approx \langle X_k^{(i)}, X_k^{(j)} \rangle \\
\end{aligned}
$$

We can further investing the meaning of the concepts by noting that $(U_k)_{i,j}$ is the strength of the association between the term $t_i$ and the concept $t_j$, the same holds for $V_k$ and the document $d_i$.

## Random projection
Another approach to reduce the dimensionality of the document vectors consist in extracting a random number of features.

The Johnson-Linderstrauss lemma can be used to bound the error in computing the euclidean distance between two randomly projected vectors.
Given a set $P$ of $n$ points in $m$-dimensions and $\epsilon>0$, there exists a projection function $f \colon \mathbb{R}^m \to \mathbb{R}^k$, where $k = O(\epsilon^2\log n)$, such that $\forall u,v \in P$ it holds

$$
(1 - \epsilon)\|u-v\|^2 \leq \|f(u)-f(v)\|^2 \leq (1+\epsilon)\|u-v\|^2
$$

The lemma applies to the euclidean distance, while in our discussion we always used the cosine one, that we can actually also bound with some linear algebra manipulation:

$$
\langle f(u), f(v) \rangle \leq \epsilon (\|u\|^2 + \|v\|^2) + (1-\epsilon)\langle u,v\rangle
$$

Therefore, if $u$ and $v$ are normalized vectors, then the cosine distance changes by at most $2\epsilon$.

The Johnson-Linderstrauss lemma proves an existential results, not giving any clue on how to actually find or compute the $f$ function.
We assert, without proving it, that the projection matrix that implements the function $f$ can be constructed using any random distribution with mean $\mu = 0$ and variance $\sigma = 1$, such as for example $P(F_{i,j} = -1) = P(F_{i,j} = 1) = \frac{1}{2}$.

[^4]: Formally $S$ contains the singular values of $A$, but we consider them to be equals to the eigenvalues in our context because of some implicit assumptions that we will not discuss.
