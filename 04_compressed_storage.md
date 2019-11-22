# Compressed storage of the web-graph
The directed graph representing the web has three peculiar characteristics:

- Skewed distribution, the probability that a node has $x$ links follows the power law $\frac{1}{x^\alpha}$, where experimentally $\alpha \approx 2.1$.
- Locality, usually most of the hyperlinks point to URLs in the same host.
- Similarity, if two URLs are close in lexicographic order, then they tend to share many hyperlinks

Permuting the host, reversing the dot order, it's possible to create a sequence of adjacency lists that uses the locality and the similarity properties to generate contiguous areas.
This situation can be exploited to reduce space occupancy using the copy-list technique. 
In this technique each list has associated:

- Out-degree, the number of links exiting the page.
- A reference list, in respect to there is the compression.
- A copy list, that is a bit-string of the same length of the out-degree of the reference.
- Extra nodes, that stores the links not shared with the reference.

The bits in the copy list can be compressed using run-length encoding in a copy-block.
This is done by storing the first bit on the list and then the length of each region of consecutive equal bits minus one, the integer representing the last region of the list could be dropped because the number of bits is constrained by the out-degree.
