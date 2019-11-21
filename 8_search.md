# Search

## Prefix-string
Given a dictionary $D$ of $K$ strings, of total length $N$, store them in a way that we can efficiently support prefix searches for a patter P over them.

A trie is a useful data structure for p-search, but every implementation suffers of big space issues.
Using a 2-level indexing solution can mitigate the problem, dividing the sorted elements in pages and constructing a trie on a sampling we can use the trie as a router.
This solution requires typical only one IO operation to visit the trie, and also requires less space given that the trie is built over a subset of strings.
A disadvantage to consider is the tradeoff in speed vs space given by the bucket size used for the sampling.

With front-coding it's possible to further reduce space usage.
To compress a sorted list of strings, the prefix is substituted by one byte that indicates the number of prefix shared characters with the previous word.
Obviously random access is not possible, but in our scenario the scanning of a bucket was already required for p-search in a 2-level indexing solution.
If the strings don't use a termination character, the length of the string also has to be stored.

## Tolerant search
The area of spell correction can be used for two principal use cases: the correction of an indexed document and the correction of the user queries.
Also this correction could occour by analyzing isolated words or by also considering the context of a sentence.

For the rest of the discussion we assume the presence of a lexicon, to correct isolated word in queries.
Given a lexicon and a character sequence Q, return the words in the lexicon closest to Q.
The definition of closeness can vary using different distance measures.

### Brute force
A brute force approach enumerates for each word in $q$ in $Q$, all the possible $\bar{q}$ words with edit distance $d$ from $q$, if $\bar{q}$ is in the dictionary then it's proposed as a suggestion to the user.
The edit distance, or Levensthein distance, is computable via a dynamic programming algorithm where first the matrix $E$ is constructed, and then the value $E[n][m]$ is the edit distance of the two strings.

$$
E_{ij} = 
\begin{cases}
i & j = 0 \\
j & i = 0 \\
\min \begin{cases}
E_{i-1,j-1} + W_! \\
E_{i,j-1} + W_i \\
E_{i-1,j} + W_d
\end{cases} & i \neq 0 \land j \neq 0
\end{cases}
$$

### One-error correction
The enumeration and the dynamic programming solutions are too costly, to improve the performance we have to assume the maximum edit distance as one.
An useful approach is to trasform the problem exploiting the fact that enumeration deletions is easy, so given the dictionary of correct words $D_1$ we can easily generate the dictionary $D_2$ that contains $\Theta(l|D_1|)$ words on average.

Now for each word $q \in Q$ we can operate as follows:

- If $q \in D_1$ the word is correct.
- For each $\bar{q}$ given by dropping a character in $q$, if $\bar{q} \in D_1$, then $q$ had one char more and $\bar{q}$ has to be suggested.
- If $q \in D_2$, then $q$ has one char less and so is not correct.
- For each $\bar{q}$ given by dropping a character in $q$, if $\bar{q} \in D_2$, then $q$ may contain a mismatch and so it's not correct.

This algorithm request linear time respect to the number of words $q \in Q$, but suffers of a space problem and could generate false matching results.

### Overlap distance
The overlap distance is an approximation of edit distance, used as an heuristic to generate a set of candidate words from a dictionary.

Assume that each word is anticipated by $k-1$ special character `$`, this ensures that the number of k-grams will be equal to the length of the string that generated it.
All the words in a dictionary have to be partitioned in k-grams, and then an inverted index is built over the k-grams.

A query Q generates $|Q|$ k-grams, while each error affects $k$ k-grams.
So if Q and a given token have edit distance $i$, they have to share for sure at least $|Q| -ki$ k-grams.

Given a query word Q, by splitting it in k-grams we are able to find the corresponding inverted lists, and so keep track of how many k-grams Q shares with each word in these lists.
Fixed $i$ it's now possible to check all the strings in the list, those that satistfy the filter condition are flagged as candidate strings.

## Wildcard queries
The wildcard queries $\alpha\star$ have been already considered by prefix-search, but also the form $\star\beta$ can be solved with the same techniques by constructing the data structures over the reverse of each term.

More interesting is the case of the form $\alpha\star\beta$.
It's still possible to use the two previous approaches to p-search $\alpha$ and s-search $\beta$ and then intersecting the results.
If the resulting lists are big the intersection could be expensive in terms of time.

The permuterm index proposes an approach that pays with space to reduce time complexity.
Each term in a dictionary is indexed under all the possible rotation of the word, so for example the term `word` is indexed by `word$,ord$w,rd$wo,d$wor,$word`.
Given a query in the form $P\star Q$, the search is reduced to a p-search of the rotated query $Q\$P\star$.

## Soundex
Under the name of soundex goes a class of heuristics to expand a query into phonetic equivalents.
The soundex algorithm are language specific and mainly used for names;
historically the first soundex was used to write down the names of the non-English immigrants in Coney Island.
