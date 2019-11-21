# Document duplication
The Web contains multiple copies of the same content, search engines try to avoid indexing multiple copies of the same content, to keep down storage and processing overheads.
Theorically any hash function could be used to store a fingerprint of a web document to be compared for each new document, obviously there are some efficiency and space constraints that makes some solutions more interesting than others.

The rolling hash tecnique described by Karp-Rabin fingerprints it's really common.
Given a prime number $p$ the fingerprint of an $m$-bit string A is $f(A) = A\mod p$, it's possibile to easily compute any shift of $A$.
The probability of a collision between any pair $A$ and $B$ is equivalent to the probability that $p$ divides $A-B$, that is practically zero.

This simplicistic approach fails to capture a crucial and widespread phenomenon on the web: near duplication.
In many cases, the contents of one web page are indentical to those of another except for a few characters.

## Shingling
Given the set of all the possibile $q$-grams of a document and their fingeprints called shingles, the shingling tecnique is used to reduce the near-duplicate document detection problem to the set intersection the shingles of two distinct documents.
We declare that page $A$ and $B$ are near duplicated iff. the insersection of $S_A$ and $S_B$ is large, a possible arbitrary measure could be the Jaccard similarity defined as follows:

$$
\frac{| S_A \cap S_B |}{| S_A \cup S_B|}
$$

This process requires a big amount of space, and the cost of computing the intersection over the whole shingling sets.

## Min-hashing
A possible way to approximate the Jaccard similarity is using $L$ random permutations to generate a sketch given by the minimum of the shingles in each possible permutation.

$$
< \min\pi_1(S_A), \dots, \min\pi_L(S_A) >
$$
This is useful given that is possible to prove that $\mathbb{P}(\min\pi_i(S_A)=\min\pi_i(S_B))=J(S_A,S_B)$, and so:
$$
\frac{\mathbb{E}(\textrm{\#equal components})}{L} \approx \frac{L*\mathbb{P}}{L} = J(S_A,S_B)
$$

The same tecnique used in LSH can be used here to reduce space occupancy, projecting $L'$ times the sketch of each set.

## Cosine distance
The min-hashing procedure maps a document to a multi-dimensional vector on which to compute similarity.
Other possible distances can be used, like for example the cosine distance where $L$ random lines are extracted and compared to a vector $v$ representing a set of features in the document.
