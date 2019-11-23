# Document ranking

By considering the matrix containing the relation of terms and documents, we can consider the $i$-th column as a binary vector representing the $i$-th document.
Computing the intersection of the terms in two documents, that is counting the number of components both equal to one in the binary vectors, is a not so useful measure of similarity because it does not depend on the size of the sets.
Two possibile approach to normalize this intersection are the Dice coefficient $D$ and the Jaccard coefficient $J$, that also respect the triangular inequality.
Even with this measures the importance of a term is not considered.

## tf-itf
Taking in account the term frequency $\textrm{tf}_{td}$ could be misleading because of the frequency of stop words, so also the inverted term frequency $idf_t$ has to be considered where $n$ is the number of documents in the indexed collection, while $n_t$ is the number of documents containing the term $t$.
The product of this two measures is considered as a good weight for a term in a document.

$$
\begin{aligned}
w_{td} = tf_{td} \log \frac{n}{n_t}
\end{aligned}
$$

For example an article will have a big $tf$, compensated by a really small $itf$, and experimentally works in a lot of contexts.
The weight is zero if the term is not present in the document or if is not significative because it appears in all the documents.
In this vector space model every column of the matrix represent a document vector, also the query will be considered as a very short documents manageable as a vector in the model.

## Cosine score
One idea to compare vectors could be to use the euclidean distance in the space, given that this is also dependent on the length of the vector it's not a good idea.
Better to use angles, but computing angles in multidimensional space could be hard.
Computing the cosine of the angle is easier, and also the smaller the angle the nearer to $1$ similar.
$$
\cos(d,q) = \frac{d \cdot q}{||d||\cdot||q||}
$$

The computational problem is that a query will have $1$ in very few components, and $0$ in all the other.
If the document is very well written the algorithm works very well, but it's easy exploitable by spammers using term repetition in a document, afflicting $tf$ but not $itf$.

The whole matrix can't be stored, but we have to store the vector by using inverted lists.
As seen in phrase queries also the positions of the terms in a document must be stored, computing the tf by summing over the positions (preprocessed and storade with unary/gamma compresison) and the itf using the lenght of the list the computation of a component is immediate.

For every document $d \in D$ scan the terms to reconstruct the vector, this is obviously not possibile.
Considering that the similarity $d \cdot q$ is a summation over the non-zero values, it's possible to compute the components only on the terms in the query.

```python
for term in query:
  for doc in posting[term]:
    score[d] += w_td * w_tq
```

The vector space solution is useful for bag-of-words queries and it's a clean metaphor for similar-document queries, but it's not a good technique to use with operators.

### Approximate value
The computation of the cosine distance is costly, so we would like to reduce the computation to a set of documents $A$ much smaller than the collection and then return the top-k docs in $A$ as an approximation of the top-k in the whole collection.
The same approach is also useful for other (non-cosine) scoring functions.

The first filter has already been applied in the previous optimization, that is considering documents that contains at least one query term.
This can be taken further by considering only documents containing at least $|Q|-i$ query terms, algorithmically this can be done by set intersection or by scanning and flagging the documents.

One problem of this improvement is the fact that all the terms are considered equivalent, and so the presence of articles and so on is then considered.
This problem can improved by considering only terms with an high idf value.

The champion lists approach assigs to each term its $m$ best documents according their ft-itf metric, to work well empirically must be $m>k$.

We can try to solve geometrically by clustering the documents, for every group of documents a leader is elected and so the query is compared only with the leaders, and consequently with all the documents in the group in the nearest leader.
An obvious variation considers the nearest $m$ leaders.

The fancy-hits heuristic considers an additional scoring scheme: the PageRank.
Suppose that the score of the document $d$ is given by the summation of the PageRank with the tf-idf score.
The champion list, here called fancy-list, of a document is sorted by decreasing page rank, also all the other documents in another list are stored by decreasing page rank.
By construction assing the docIDs in increasing order respect to their PageRank.
Compte the score of all docs in FH, keep the top-k docs, scans the other list up to a certain threshold is reached.
