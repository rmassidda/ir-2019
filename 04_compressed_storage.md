# Compressed storage of the web-graph
The directed graph representing the web has three peculiar characteristics:

- Skewed distribution, the probability that a node has $x$ links follows the power law $\frac{1}{x^\alpha}$, where experimentally $\alpha \approx 2.1$.
- Locality, usually most of the hyperlinks point to URLs in the same host.
- Similarity, if two URLs are close in lexicographic order, then they tend to share many hyperlinks

Permuting the host, reversing the dot order, it's possible to create a sequence of adjacency lists that uses the locality and the similarity properties to generate contiguous areas.
This situation can be exploited to reduce space occupancy using the copy-list tecnique. 
In this tecnique each list has associated:

- Outdegree, the number of links exiting the page.
- A reference list, in respect to there is the compression.
- A copy list, that is a bit-string of the same length of the outdegree of the reference.
- Extra nodes, that stores the links not shared with the reference.
