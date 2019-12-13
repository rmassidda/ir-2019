# Document duplication
The Web contains multiple copies of the same content that search engines try to avoid indexing, to keep down storage and processing overheads.
Theoretically any hash function could be used to store a fingerprint of a web document to be compared for each new document, obviously there are some efficiency and space constraints that makes some solutions more interesting than others.

The rolling hash technique described by Karp-Rabin fingerprints is commonly used.
Given a prime number $p$, the fingerprint of an $m$-bit string A is $f(A) = A\mod p$, it's possible to easily compute any shift of $A$.
The probability of a collision between any pair $A$ and $B$ is equivalent to the probability that $p$ divides $A-B$, that is practically zero.

This simplistic approach fails to capture a crucial and widespread phenomenon on the web: near duplication.
In many cases, the contents of one web page are identical to those of another except for a few characters.

## Shingling
Given the set of all the possible $q$-grams of a document and their fingerprints called shingles, the shingling technique is used to reduce the near-duplicate document detection problem to intersection of the set of shingles of two distinct documents.
We declare that page $A$ and $B$ are near duplicated if the intersection of $S_A$ and $S_B$ is large according to an arbitrary measure, like the Jaccard similarity defined as follows:

$$
J(S_A,S_B) = \frac{| S_A \cap S_B |}{| S_A \cup S_B|}
$$

This process requires a big amount of space, and the full cost of computing the intersection over the whole shingling sets.

## Min-hashing
A possible way to approximate the Jaccard similarity between two sets is using $L$ random permutations to generate a sketch given by the minimum of the set in each considered permutation.

$$
< \min\pi_1(S_A), \dots, \min\pi_L(S_A) >
$$
To share the same minimum value in the same permutation $\pi_i$, the minimum must have been taken from the intersection between all the possible values, so it's immediate that $p = P(\min\pi_i(S_A)=\min\pi_i(S_B))=J(S_A,S_B)$.

We are now able to approximate the Jaccard similarity of the two sets by counting the number of equal components between two sketches and normalizing it via $L$, this is sound because of this observation:
$$
\frac{\mathbb{E}(\textrm{\#equal components})}{L} = \frac{L*p}{L} = J(S_A,S_B)
$$

The space occupancy of this technique can be reduced at the cost of introducing approximate results by projecting each sketch into a set of $k$ integers for $L'$ times, and then group them by equal component as in LSH.

## Cosine distance
Another possible way to compute the sketch of a vector is using the cosine distance.
Given $L$ random lines, we can compute the $i$-th element of the sketch as the result of the hash function $h_i(p) = \textrm{sign} ( r_i \cdot p )$, where $r_i$ is the $i$-th random line.

Two sketches share the same value iff. the line $r$ doesn't lie in the angle $\alpha$ between the original vectors of the sketches, so the probability of this event is $P(h_i(p)=h_i(q)) = \frac{\pi - \alpha}{\pi} = 1 - \frac{\alpha}{\pi}$.

By dividing the number of not shared components between two sketches, divided by the number of components $L$ we obtain an approximation of $\cos(\alpha)$ for small $\alpha$ angles.
