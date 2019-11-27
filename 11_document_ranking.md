# Document ranking

By considering the matrix containing the relation of terms and documents, we can consider the $i$-th column as a binary vector representing the $i$-th document.
Computing the intersection of the terms in two documents, that is counting the number of components both equal to one in the binary vectors, is a not so useful measure of similarity because it does not depend on the size of the sets.
Two possibile approach to normalize this intersection are the Dice coefficient $D$ and the Jaccard coefficient $J$, that also respect the triangular inequality.
An issue with both this measures resides in the fact that the importance of a term is not considered.

## tf-idf
To start a discussion over the importance of a term in a document, a first approach could be counting the number of occurrences of term $t$ in the document $d$, we call this value term frequency $\mathit{tf}_{t,d}$.
Taking in account only the term frequency could be misleading because of the high frequency of stop words, to balance this situation we can use the inverted document frequency $\mathit{idf}_t = \log\frac{n}{\mathit{df}_t}$, where $n$ is the number of documents in the indexed collection, while $\mathit{df}_t$ is the number of documents in the collection containing the term $t$.
The product of this two measures is considered as a good weight for a term in a document.

$$
w_{t,d} = \mathit{tf_{t,d}} \times \mathit{idf_{t}}
$$

The weight is zero if the term is not present in the document or if is not significative because it appears in all the documents.
In this vector space model every column of the matrix represent a document vector, also the query will be considered as a very short document manageable as a vector in the model.

## Cosine score
One idea to determine the similarity between vectors could be using the euclidean distance in the space, it should be noticed that this approach is dependent on the length of the vector and so it's not a good idea.
A better approach could lead to measuring the angle between two vectors, but we have to consider that being in a very high dimensional space the computation will by hard.

Computing the cosine of the angle is easier, so the cosine similarity is:
$$
\cos(d,q) = \frac{d \cdot q}{||d||\cdot||q||}
$$

If the document is very well written the algorithm works very well, but it's easy exploitable by spammers using term repetition in a document, afflicting $\mathit{tf}$ but not $\mathit{idf}$.
The vector space solution is useful for bag-of-words queries and it's a clean metaphor for similar-document queries, but it's not a good technique to use with operators.

The whole matrix can't be stored, but we have to store the vector by using inverted lists, also the query vector will be largely sparse we have a lot of space for optimizations.
The positions of the terms in a document must be stored (possibly compressed), as seen in phrase queries, so the cardinality of the set of positions of a term in a document is $\mathit{tf}$, while the length of the posting list is $\mathit{df}$, so it's immediate the computation of the weight of a term in a document.

Since only the terms included in a query are interesting a possible algorithm initializes an empty vector of scores and then for every term $t$ in the query it retrieves it's posting list after computing it's weight $w_{t,q}$.
Then for each document $d$ in a posting list, the score $s_d$ is updated as in:

$$
s_d = s_d + w_{t,d} \times w_{t,q}
$$

To conclude the algorithm the scores must be normalized using the length of the documents, so $s_d = \frac{s_d}{l_d}$, taking now the top $k$ components returns the top $k$ similar documents to the query.

## Approximate value
Although the optimization over the sparse vector, the computation of the cosine distance is still costly, so we would like to reduce the computation to a set of documents $A$ much smaller than the collection and then return the top-k docs in $A$ as an approximation of the top-k in the whole collection.
The same approach is also useful for other (non-cosine) scoring functions.

### Index elimination
For a multiterm query it is clear that we only consider documents containing at least one of the query terms.
We can take this a step further using additional heuristics:

- Consider documents containing terms whose idf exceeds a threshold.
- Consider only documents containing at least $|Q|-i$ query terms.

### Champion Lists
To construct a champion list, each term is preprocessed to find the $m$ best documents according to their tf-idf metric, to work well empirically must be $m>k$, also the value of $m$ could be different for each term according to their importance.
At query time the computation of the scores is done only on the champion lists and not on the posting list.

### Fancy-hits
The fancy-hits heuristic is a variation of the champion list that makes use of an additional scoring scheme, the PageRank.
In a preprocessing phase the docIDs are assigned decreasing in respect to their PageRank, and also we compute for each term its champion list, called from now on FH, and the list of documents not in FH, called IL.

At query time first the top-$k$ documents are computed using the cosine similarity algorithm on all the champion list.
To improve the results the IL list is scanned for each term, computing the score and possibly inserting them into the top-$k$, we can use as a stopping criterion the PageRank becoming smaller than some threshold.

A sophisticated stopping criterion makes use of a value defined as the sum of the tf-idf and the PageRank of a document.
Taken the minimum document $x$ in the champion list, we can assert that its tf-idf value is anyway greater than any of the documents in the IL list, and so:

$$
\forall d \in \textrm{IL} . \quad s_d \leq \textrm{tf-idf}_x + \mathit{PageRank}_d
$$

Since the PageRank is decreasing there will exist an element $\bar{d} \in \textrm{IL}$ such that $s_{\bar{d}} < \textrm{tf-idf}_x$, given that this property will be valid for also all the subsequent values the scanning can be stopped.

### Clustering
We can try to solve geometrically by clustering the documents, first of all $\sqrt n$ leaders are randomly extracted between all the documents and all the remaining documents are assigned to the nearest leader.
At query time the result is obtained by seeking the $k$ nearest docs from among the followers of the nearest leader, an obvious variation considers the nearest $m$ leaders.
