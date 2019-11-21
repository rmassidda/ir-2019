# Locality-Sensitive Hashing
In data analysis a frequent issue is, given a set $S$ of items, each one with $d$ features, to find the largest group of similar items.
The similarity is a function that, taken the features of two items, returns a value in the interval $[0,1]$.
The brute-fore approach is unviable because of the practically infinite number of possible groups, even limiting the group size to a constant $L$ requires an incredible computational power.
Introducing a certain level of approximation it's possibile to consider a clustering algorithm like the famous machine learning algorithm K-nn.

The LSH tecnique propose instead to generate a fingerprint for every item, much smaller than the set of features, and then to transform the similarity between items into the equality of fingerprints.
This approach is randomized and correct with high probability, also it guarantees local access to data reducing so the number of I/O operations needed.

The similarity of two binary vectors $p$ and $q$ is formalized as the probability, given an index $i$, that $p[i] = q[i]$, and this is equal to $s = (1 - \frac{D(p,q)}{d})$, where $D(p,q)$ is the hamming distance of the vectors, that is the number of positions where their bit value differs.
Now consider a set $I$ of $k$ random selected integers in the interval $\{1,\dots,d\}$, we call $h_I(p)$ the projection of $p$ into the $I$ positions.
The probability that the projections of a pair of vectors are equals is $s^k$.
Using $L$ different projections and reporting that $p$ is similar to $q$ iff. $\exists i. h_{I_i}(p)=h_{I_i}(q)$, the probability of this event given any pair of vectors is $1 - (1-s^k)^L$;
so strictly dependent on the actual similarity $s$, between them.
It's possibile to notice that while the $k$ value reduces the false positives, the $L$ reduces the false negatives.

In the pratice of grouping similar items this is applied by generating $k$ sets $I_i$, then computing for each item its sketch, that is the $L$-ple containing all the $h_{I_i}$ projections.
Generating a graph of items where each node has an edge with any node with at least one equal projection in the sketch, permit to define the groups as the connected components in the graph.
Given that this can be implemented via only scan and sort primitives, the number of IO operations to do this is $\tilde{O}(\frac{n}{b})$.

In the case of online queries instead, it's possible to create $L$ hash tables where each table has $2^k$ elements, similar elements to the queried one are the elements that collide with it.

Comparing LSH with K-means we could use this resume:

| Algorithm | Optimality | Cost | Cost per iteration | Number of cluster |
|-----------|------------|------|--------------------|-------------------|
|LSH|Global with high probability|Short sketch comparison|Sort $|S|$ items|Not needed|
|K-nn|Locally optima|D-features comparison|$K \times S \times d$|It's a parameter|


