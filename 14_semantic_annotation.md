# Semantic annotation
The vector space model described up to here is mainly term-based, and it's so subject to issues derived from polysemy and synonymy;
more specifically two documents are considered similar only if the share some words.
Using topic-based annotation we pursue the goal to construct algorithms that are able to resolve an information need, overcoming this issues.

An alternative approach consist in using graphs to organize the informations of a knowledge base, we will refer to this concept as knowledge graph[^1].
In the graph each a node is an entity representing a semantic unit, the edges can also be labelled indicating the kind of relation between the concepts.

Wikipedia is one of the simplest possible knowledge graphs, this is because the links are not typed but signify only a generic relation.
The anchor includes a text that possibly different with the name of the entity pointed, this pair of strings constitutes a mention-topic pair.
By collecting of all these pairs relative to a fixed page, we can extract the common sense, that is how the people refer to a certain topic.
Another useful kind of information that can describe a Wikipedia page is made of the categories it appears into, forming a directed acyclic graph.

## TagMe
We propose the definition of TagMe, as it's presented on its live online implementation[^2]:

> TAGME is a powerful tool that is able to identify on-the-fly meaningful short-phrases (called "spots") in an unstructured text and link them to a pertinent Wikipedia page in a fast and effective way.

The system analyses a text, obtaining a set of entities that are then associated to a set of nodes in a knowledge graph.

To define if a given token is a feasible anchor, we can use as an indicator the link probability $lp(a) = \frac{\#\textrm{a as anchor}}{\#\textrm{a in the text}}$.
Let's suppose for now that a text contains only two possible anchors $a$ and $b$.

For each anchor we have now a set of possible pages, disambiguating an anchor means selecting only one page from this set.
The first step consists in pruning from the set the less common pages: after setting a threshold $\tau$, we can compare it against the commonness of a page $p$, with the respect to an anchor $a$, defined as $P(p|a) = \frac{\# \textrm{a linked to p}}{\#\textrm{a as anchor}}$.

We define the similarity between two pages $rel(p,q)$, as the Jaccard similarity between the set of pages pointing to $p$ and the set of those pointing to $q$.

We can now define the score of a candidate page $p_a$ for an anchor $a$, given that the text also contains the anchor $b$, as:

$$
vote_b (p_a) = \frac{\sum_{p_b \in C(b)} rel(p_b,p_a) P( p_b | b)}{|C(b)|}
$$

Where $C(b)$ is the candidate set of pages for the mention $b$.

The constraint of having only two anchors can be relaxed, in the case of multiple mentions we can average over the scores of all the possibile pairs of anchors.
Using the results of the scoring system the top-$\epsilon$ are returned, and if the set is still to big or the results are uncertain we can filter again using the commonness indicator.
Empirical values used are $\tau = 2\%$ and $\epsilon=30\%$.

Another possible mechanism that can be used in the process of disambiguation is considering the words surrounding the anchor, trying to match them with the words in the documents supposedly pointed by the anchor.

[^1]: Knowledge Graph is the name given by Google to its own implementation. Given its diffusion and the simplicity of the name, the term is often used as a synecdoche to refer the general concept.
[^2]: [https://tagme.d4science.org/tagme/](https://tagme.d4science.org/tagme/)
