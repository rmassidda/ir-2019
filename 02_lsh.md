# Locality-Sensitive Hashing
In data analysis a frequent issue is, given a set $S$ of items, each one with $d$ features, to find the largest group of similar items.
The similarity is a function that, taken two items, returns a value in the interval $[0,1]$.

The brute-force approach is not useful because of the practically infinite number of possible groups, even limiting the group size to a constant $L$ requires a huge amount of computational power.
Introducing a certain level of approximation it's possibile to consider a clustering algorithm like the famous machine learning algorithm K-means.

In k-means, fixed a number of $k$ clusters are fixed $k$ random points.
Until convergence the points are assigned to the nearest centroid that are then recomputed.

The LSH technique proposes instead to generate a fingerprint for every item, and then to transform the similarity between items into the equality of fingerprints.
This approach is randomized and correct with high probability, also it guarantees local access to data reducing the number of I/O operations needed.

Given the hamming distance $D(p,q)$ between two binary vectors $p$ and $q$, we define the similarity $s$ as the probability, given an index $i$, that $p[i] = q[i]$, and this is equal to $s = (1 - \frac{D(p,q)}{d})$.

Now consider a set $I$ of $k$ random integers selected from $\{1,\dots,d\}$, we call $h_I(p)$ the projection of $p$ into the $I$ positions.
The probability that two fixed projections of a pair of vectors are equal is $P(h_I(p)=h_I(q))=s^k$.

Using $L$ different projections we can state that $p$ is similar to $q$ with high probability if $\exists i. h_{I_i}(p)=h_{I_i}(q)$. The probability of this event is:

$$
\begin{aligned}
P(p \textrm{ matches } q) = \\
P ( \exists i. h_{I_i}(p)=h_{I_i}(q) ) = \\
1 - P ( \forall i. h_{I_i}(p) \neq h_{I_i}(q) ) = \\
1 - P ( h_{I_i}(p) \neq h_{I_i}(q) ) ^ L = \\
1 - ( 1 - s^k ) ^ L
\end{aligned}
$$

So strictly dependent on the actual similarity $s$, between $p$ and $q$.
It's possibile to notice that while the $k$ value reduces the false positives, the $L$ reduces the false negatives.

In the practice of grouping similar items this technique is applied by generating $L$ sets $I_i$, then computing for each item in the set its sketch, that is the $L$-ple containing all the $h_{I_i}$ projections.
Generating a graph of items where each node has an edge with any node with at least one equal projection in the sketch, permit to define the groups as the connected components in the graph.
Given that this can be implemented via only scan and sort primitives, the number of IO operations to do this is $\tilde{O}(\frac{n}{b})$.

In the case of online queries instead, it's possible to create $L$ hash tables where each table has $2^k$ elements, similar elements to the queried one are the elements that collide with it.

Comparing LSH with K-means we could use this resume:

| Algorithm | Optimality | Cost | Cost per iteration | Number of cluster |
|-----------|------------|------|--------------------|-------------------|
|LSH|Global with high probability|Short sketch comparison|Sort $|S|$ items|Learned|
|K-nn|Local|D-features comparison|$K \times S \times d$|Parameter|
