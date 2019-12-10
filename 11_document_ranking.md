# Document ranking
By considering the matrix containing the relation of terms and documents, we can consider the $i$-th column as a binary vector representing the $i$-th document.
Computing the intersection of the terms in two documents, that is counting the number of components both equal to one in the binary vectors, is a not so useful measure of similarity because it does not depend on the size of the sets.
Two possible approaches to normalize this intersection are the Dice coefficient $D = 2\frac{|X\cap Y|}{|X| + |Y|}$ and the Jaccard coefficient $J = \frac{|X \cap Y|}{|X \cup Y|}$, that also respect the triangular inequality.
An issue with both these measures resides in the fact that the importance of a term is not considered.

## tf-idf
To start a discussion over the importance of a term in a document, a first approach could be counting the number of occurrences of term $t$ in the document $d$, we call this value term frequency $\mathit{tf}_{t,d}$.
Taking into account only the term frequency could be misleading because of the high frequency of stop words, to balance this situation we can use the inverted document frequency $\mathit{idf}_t = \log\frac{n}{\mathit{df}_t}$, where $n$ is the number of documents in the indexed collection, while $\mathit{df}_t$ is the number of documents in the collection containing the term $t$.
The product of these two measures is considered a good weight for a term in a document.

$$
w_{t,d} = \mathit{tf_{t,d}} \times \mathit{idf_{t}}
$$

The weight is zero if the term is not present in the document or if is not significative because it appears in most of the documents.
By computing the weights for all the entries in a term-document matrix we are able to represent a document as a real valued vector, also the query will be considered as a very short document manageable in this vector space model.

## Cosine score
One idea to determine the similarity between the weight vectors could be using the euclidean distance in the space, it should be noticed that this approach is dependent on the length of the vector and so it's not a good idea.
A better approach could lead to measure the angle between two vectors, but we have to consider that being in a very high dimensional space the computation will be hard.

Computing the cosine of the angle is easier, and it is also a good similarity measure, considering that the document vectors are positive definite.
The cosine of the angle between two document vectors can be computed as:
$$
\cos(d,q) = \frac{\langle d , q \rangle}{||d||\cdot||q||}
$$

It should be noticed that if the vectors $d$ and $q$ are already normalized the computation of the cosine it's reduced to the dot product only.

If the document is very well written the algorithm works very well, but it's easy exploitable by spammers using term repetition in a document, afflicting $\mathit{tf}$ but not $\mathit{idf}$.
The vector space solution is useful for bag-of-words queries and it's a good metaphor for similar-document queries, but it's not a good technique to use with operators.

The whole matrix can't be stored because of space issues, but we can store the vector by using inverted lists, also the query vector will be largely sparse we have a lot of space for optimizations.
The positions of the terms in a document must be stored (possibly compressed), as seen in phrase queries, so the cardinality of the set of positions of a term in a document is $\mathit{tf}$, while the length of the posting list is $\mathit{df}$, so it's immediate the computation of the weight of a term in a document.

Since only the terms included in a query are interesting a possible algorithm initializes an empty vector of scores and then for every term $t$ in the query it retrieves it's posting list after computing it's weight $w_{t,q}$.
Then for each document $d$ in a posting list, the score $s_d$ is updated as in:

$$
s_d = s_d + w_{t,d} \times w_{t,q}
$$

To conclude the algorithm the scores must be normalized using the length of the documents, so $s_d = \frac{s_d}{l_d}$, taking now the top $k$ components returns the top $k$ similar documents to the query.

## Approximate retrieval
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
In a preprocessing phase the docIDs are assigned decreasingly with respect to their PageRank, and also we compute for each term its champion list, called from now on FH, and the list of documents not in FH, called IL.

At query time the same process is repeated for each term, first thing the score is computed for all the documents in the fancy-hits list.
Then to improve the results the IL list is scanned, computing the score up to a certain stopping criterion.

A sophisticated stopping criterion makes use of a value defined as the sum of the tf-idf and the PageRank of a document.
Taken the minimum document $x$ in the champion list, we can assert that its tf-idf value is anyway greater than any of the documents in the IL list, and so:

$$
\forall d \in \textrm{IL} . \quad s_d \leq \textrm{tf-idf}_x + \mathit{PageRank}_d
$$

Since the PageRank is decreasing there will exist an element $\bar{d} \in \textrm{IL}$ such that $s_{\bar{d}} < \textrm{tf-idf}_x$, given that this property will be valid for also all the subsequent values the scan can be stopped.

### Clustering
We can try to solve geometrically by clustering the documents, first of all $\sqrt n$ leaders are randomly extracted between all the documents and all the remaining documents are assigned to the nearest leader.
At query time the result is obtained by seeking the $k$ nearest docs from among the followers of the nearest leader, an obvious variation considers the nearest $m$ leaders.

## Exact top-k documents
The problem consists in, given a query $Q$, finding the exact top $K$ documents for $Q$, using some ranking function $r$.
The simplest strategy is to find all the documents interested by the query, compute the score for each document and then sorting to return the best top-$K$.
This is obviously too costly as described in the reasons that introduced the approximate retrieval techniques, we want then to find an admissible heuristic approach the avoid computing the score on documents that won't make it into the top-$K$.

### WAND
The WAND technique is a pruning method which uses a heap over the real documents scores.
We could prove that the document IDs in the heap at the end of the process are the exact top-K.
The algorithm follows a branch and bound approach, we maintain a running threshold $\Theta$ score, pruning away all the documents surely under the threshold and computing the exact scores only for the remaining.

Inside each postings list the documents have to be sorted by document IDs, also we have to assume a special iterator on the postings that can move a pointer to the first document ID greater than a certain value.
This iterator allows only right-moves, and can be implemented by using skip pointer or the Elias-Fano compression technique.

$r(t,d)$ is a generic score function of the term $t$ in the document $d$, also we want to compute the score of a document as $s(d) = \sum_{t \in T} r(t,d)$.
Preprocessing the index we will make use of an upper-bound per term, such that $r(t,d) \leq u(t)$.

The threshold $\Theta$ is initialized to 0, and we want it always to hold that $\forall d \in \textrm{top-K} . r(d) \geq \Theta$, updating it according to the score of the worst document currently in the top-$K$.

The algorithm parallel scans the postings list, at the first step of a generic iteration the terms are sorted according to the value of the document ID pointed. 
For each pointer we sum the terms upper bounds, finding the maximum score that a document can possibly obtain, the first document that possibly could have a score greater than the threshold is taken as pivot.
All the documents between the previous pointers and the pivot have no possibility to enter the top-$K$, so their score computation is skipped.
If the pivot is present in enough postings its score is computed, and if it's actually greater than the threshold it's inserted in the heap.
If a document enters in the top-$K$ the smallest has to exit, and the threshold is updated to the new smallest member.

In practice we can reduce score computation of the 90%.

### Blocked WAND
The WAND algorithm makes use of an upper bound over the full list of a term.
It's possibile to improve this upper bound by partitioning the set of documents, and storing for each bucket in the posting list the maximum score as upper bound.

The algorithm performs now a second check after the movement of the pivot.
Reached the pivot the sum of the current bucket upper bound of the relevant terms is computed, if the sum is smaller then $\Theta$ we can discard all the documents whose right-end of the bucket is the leftmost one.
Otherwise we have to compute the actual score and continue as the original algorithm.

## Relevance feedback
The idea of relevance feedback is to involve the user in the information retrieval process so as to improve the final result set.
This idea was faulty when first created in the 60s, but it's now getting interest because of the conversational queries where the user gives constant feedback.

The basic procedure consists in return in an iterative fashion a certain number of results, then asking to mark some of the returned documents as relevant or non relevant, the system is able now to refine the original query and so improve the result set.
At the time the relevant/irrelevant documents where chosen by hand.
The user should spend a certain amount of time by marking the results, also the final result will depend on the user choices, but also it's biased by the set returned by the search engine.

We are still in the vector space model, so the documents and the query are vectors.

In the Rocchio algorithm the query $q$ it's updated by summing the relevant documents $D_r$ and subtracting the non relevant ones $D_n$.

$$
q = \alpha q + \frac{\beta}{|D_r|} \sum_{d \in D_r} d - \frac{\gamma}{|D_n|} \sum_{d \in D_n} d
$$

The parameter $\alpha, \beta, \gamma$ are used to control the balance between trusting the query, trusting the relevant documents and not trusting the nonrelevant ones.

In pseudo-relevance feedback the choice is automated by the system, so the top-$K$ results are assumed as relevant without asking any feedback to the user.
This approach may works in practice, but can go wrong for some queries because of the bias introduced by the IR system.

While in relevance feedback the users give additional input on documents, used to recompute the scores, in query expansion the user give additional input on query words or phrases, possibly suggesting additional query terms.
Autocompletion is an example of query expansion, as the suggestion of similar queries to the one proposed by the user.

## Quality of a search engine
Information Retrieval has developed as a highly empirical discipline, requiring careful evaluation of the techniques used.
Metrics as indexing speed, search speed and expressiveness of the language are just factors that contributes to the key measure, that is the user happiness.
User groups are usually asked to evaluate the performance of a search engine, and factors like the UI/UX have proved to be crucial for the so called "user happiness".

The standard approach to IR system evaluation revolves around the notion of relevant and non-relevant documents, it should be noticed that the relevance is assessed relative to an information need and not to a query.

We are now able to partition the document set using the notions of relevance and retrieval, consequently defining two metrics.
The Precision $P$ is the fraction of retrieved documents that are relevant, so $P = \frac{\#(\textrm{Relevant}\cap\textrm{Retrieved})}{\#\textrm{Retrieved}}$.
The Recall $R$ is the fraction of the relevant documents that are retrieved, so $P = \frac{\#(\textrm{Relevant}\cap\textrm{Retrieved})}{\#\textrm{Relevant}}$.

By using the following contingency table it's possible to redefine $P$ and $R$.

| | Relevant | Not Relevant |
|-|-|-|
| Retrieved | true positive | false positive |
| Not retrieved | false negative | true negative | 

From the definitions given it's obvious that the Recall can't be estimated, because it would require possibly a vast portion of the web, while we do not have this problem for the precision measure.

It's not trivial if an IR system should be optimized according to precision or to recall, it depends of the context of the application.
We can combine the two values in a single measure, called the $F$ measure, this trades off precision versus recall via the weighted harmonic means of these two.

$$
F = \frac{1}{\alpha\frac{1}{P}+(1-\alpha)\frac{1}{R}}
$$

If $\alpha > 1/2$ the $F$ measure gives more weight to the precision respect that in the recall, we call $F1$ the balanced version of $F$ with $\alpha = \frac{1}{2}$.
