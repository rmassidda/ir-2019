# Web ranking
Considering the web graph we are interested in finding a mechanism for ranking the nodes, representing the pages.
Under the assumption that an hyperlink between pages denotes author perceived relevance, it's possible to model the web graph using a random walker that reached any pages randomly chose where to go next, analyzing the random walker behaviour leads to the most visited pages that are so considered the most important.
This can be done by modelling the web graph as a Markov process, to do so we need to get the transition probability matrix $P$ from the adjacency matrix $A$, as in:

$$
P_{i,j} = \frac{A_{i,j}}{\sum_z A_{i,z}}
$$

Given the transition matrix $P$ and a probability distribution $x_t$, representing the probabilities of being in a certain state at the moment $t$, we can easily update the probabilities by using vector-matrix multiplication: $x_{t+1} = x_t P$

Moreover we can prove that given the initial distribution $x_0$ it's possible to compute the probabilities at time $t$ always using the same multiplication: $x_t = x_0 P^{t}$

A fixed point of the function defined by $P$ is called the steady state distribution and it's equivalent to the eigenvector of the matrix $P$ for the maximum eigenvalue $1$.
In our scenario the stationary distribution represents the proportion of time that a random walker spends visiting each node in an infinite walk, we assume it as a metric of its importance.

The existence of this eigenvector depends on these two conditions:

- The graph must be irreducible, that is $\forall i,j . \exists .$ a path from $i$ to $j$.
- The graph must be aperiodic, that is the MCD of all cycles length is one.

Given these the eigenvector exists unique, it does not depend on the initial distribution $x_0$ and represents the steady state distribution

## Google PageRank
The web graph does not ensure the previous conditions, the intuition of Google inventors was to enforce these constraints.

They assumed the existence of a "teleport" operation, that allows the random walker to visit any node from any other.
Fixing a constant $\alpha \in [0,1]$ the random walker randomly jumps to any other node in the graph with probability $(1-\alpha)$, while it chooses one of its proper neighbors with probability $\alpha$.
Superimposing the teleport graph to the web graph we obtain a complete graph, that is so irreducible and aperiodic, and so it has a unique steady state distribution.

We can formalize this using linear algebra by fixing a vector $e$ where $N$ is the total number of nodes in the web graph:

$$
e = \begin{bmatrix} \frac{1}{\sqrt{N}} \\ \dots \\ \frac{1}{\sqrt{N}} \end{bmatrix}
$$

The transition matrix $P$ can be expanded as $P_\star = \alpha P ^ T + ( 1 - \alpha ) e e^T$, and we can compute its left eigenvector as:

$$
r(i) = \alpha \sum_{j \in B(i)} \frac{r(j)}{\#out(j)} + (1-\alpha) \frac{1}{N}
$$

The computation of this eigenvector is obviously extremely costly, and it's in practice done by extracting a random distribution and applying it to the augmented matrix $P^\star$ using the power law up to obtaining $r \approx x_{2^k} = x_0 P_\star^{2^k}$ for $k \approx 7$.
This is done by periodically preprocessing the web graph and using at query time the computed PageRank.

### Personalized PageRank
A possibile variation on the original PageRank algorithm can be obtained by biasing the teleport step in the direction of a certain subset of pages, without considering the whole web graph.
This can be done by simply mutating the $e$ vector with a preference vector which jumps to preferred pages.

## HITS
The Hypertext Induced Topic Search is a scoring system largely derived from the literature of the scientific papers ranking.
Historically contemporaneous to the PageRank and theoretically more interesting, it didn't made it in the long run because of the high computational cost at query time.

The mechanism is in fact query dependent and considers only a subset of the web graph.
The set of the pages intersected by the query is called the root set, augmenting this with all the pages that links or are linked by the root set generates the so called base set.
The base set is the interesting subset for the algorithm, that will operate using its adjacency matrix $A$.

The mechanism produces two scores per page[^3]:

- Authority score, where a page has a good authority for a certain topic if it's pointed by many good hubs for a topic.
- Hub score, where a page has a good hub score for a topic if it points to many authoritative pages for that topic.

Reminding that the successors of a node can be determined by its adjacency matrix $A$, and symmetrically its predecessor by the transpose matrix $A^T$, the authority/hub mechanism is formalized as follows:

$$
\begin{cases} a = A^T h \\ h = A a \end{cases} =
\begin{cases} a = A^T A a \\ h = A A^T h \end{cases}
$$

So $h$ is the eigenvector of $AA^T$ relative to the eigenvalue 1, the same for $a$ and $A^T A$.
This result is sound because of the symmetry of the multiplication of a matrix for its transposed.
Also it's possible to weight the edges in $A$ without losing any of the useful linear algebra properties.

[^3]: It could be useful to note that in the paper ranking literature the authority score is referred to papers, while the hub score to surveys.
