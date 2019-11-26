# Document parsing

After the documents are crawled their content has to be tokenized, that is generating a stream of tokens ("words") from a document.
The tokenization is a critical module in a search engine, in OSS frameworks is common to leave the implementation to the user.
After this the tokens are normalized by using one or more linguistic models, and once normalized they can be indexed.
The first issue in parsing a document is identifying features like the format, the language and the character set; all of these are classification problems dealt via heuristics.

## Tokenization
A token is an instance of a sequence of characters in some particular document, that are grouped together as a useful semantic unit for processing.
Each such token is now a candidate for becoming an index entry, that is a term, after further processing.

The major question of the tokenization phase is what are the correct tokens to use?
An easy answer like splitting by space isn't appropriate in the majority of the use cases.
The language identification is so fundamental because of the different ways to separate words that could become terms.

## Stop words
A common practice was to don't index stop words, because they are frequent and meaningless.
But the current trend is away from doing this, good compression techniques and good query optimization techniques enables the search engine to cheaply store stop words.

## Normalization
The normalization is a language-dependent process that transforms tokens into a canonical form, this is done usually by removing hyphens and periods.
Another issue regards case-folding, a best practice is to minimize all the characters and leave the context to the other words.
Other than syntactically we have to handle via a thesauri cases of synonyms and homonyms, historically this was done brute force using an handmade thesauri.

## Stemming and lemmatization
The stemmer is a module able to recognize variations of the same word, this is done by using root prefixes of the tokens.
A possible stemmer can be built by using the Porter's algorithm.

Lemmatization is the process to reduce variant forms to a base form, for example the transformation of a conjugated verb to its infinite form.

# Statistical properties of text
Tokens are not distributed uniformly, but they follow the so called Zipf law.
The Zipf law states that few tokens are very frequent, a middle sized set has medium frequency and many are rare, formally: $f_s(k) = \frac{c}{k^s}$.
This empirical law has been found true in all the known languages, most of the more frequent are stop words.
The Zipf law is a power law, so its log-log plot is approximately a straight line.

Also the number of distinct tokens grows sublinearly according to the Heaps law $n^\beta$ where $\beta \approx 0.5 < 1$ and $n$ is the number of total tokens.
The interesting words from an IR point-of-view are, according to Luhn law, the ones with medium frequency.

# Keyword extraction
One interesting aspect in the process of keyword extraction is the analysis of the collocations.
A collocation is a sequence of two or more words that correspond to some conventional way of saying where their constituent words are not substitutable or modifiable and can't fully infer the meaning of the collocation without the others.

Frequency sorting of all the adjacent pairs in a document is not useful because of the high frequency of prepositions, articles and other stop words.
Better result can be obtained by using Part-of-Speech tagging, and by allowing only certain pairs to be ranked.

This solutions doesn't consider flexibility, often words are not adjacent to each other.
A possible approach is computing the mean and the variance of the distance within a window of all the possible pairs of words.
We can conclude that if the mean is large the collocation is not interesting, whilst if the mean is very small the pair should be treated as a collocation.

## Bi-grams
Pearson's chi-squared test ($\chi^2$) is a statistical test applied to sets of categorical data to evaluate how likely it is that any observed difference between the sets arose by chance.
It tests a null hypothesis stating that the frequency distribution of certain events observed in a sample is consistent with a particular theoretical distribution.
The events considered must be mutually exclusive and have total probability 1.

In inferential statistics, the null hypothesis $H_0$ is a general statement or default position that there is nothing new happening, like there is no association among groups, or no relationship between two measured phenomena.

In our scenario taking as a null-hypothesis that two words are independent, and so $P(A \circ B) = P(A) P(B)$ it's possible to perform Pearson's chi-squared test to assess whether observations consisting of measures on two variables, expressed in a contingency table, are independent of each other 

A contingency table $O$, that counts the occurrences of all the possible pairs fixed $A$ and $B$, has to be computed.
The so called degree of freedom is dependent on the size of the contingency matrix, as in $df = (\textrm{rows}-1)(\textrm{columns}-1)$.

|| $w_1 = A$ | $w_1 \neq A$ |
|-|-|-|
| $w_2 = B$ |$O_{11}$|$O_{12}$|
| $w_2 \neq B$ |$O_{21}$|$O_{22}$|


Than we define $E_{ij} = N * p_i * p_j$, where $p_i = \sum_j \frac{O_{ij}}{n}$ and $p_j = \sum_i \frac{O_{ij}}{n}$, and consequently to this:

$$
\chi ^ 2 = \sum_{i,j} \frac{(O_{ij}-E_{ij})^2}{E_{ij}}
$$

The $\chi^2$ has to be compared against the critical value of the distribution, dependent of the degrees of freedom and a certain confidence level arbitrary chosen, if the value is smaller than the threshold the null hypothesis is plausible, and so it's not a good pair for a collocation.

## Rapid Automatic Keyword Extraction
The algorithm works on single, not much long, documents and provides fast and unsupervised keyword extraction exploiting the fact that keywords frequently contain multiple words but rarely contain punctuation or stop words.

Given a set of word delimiters, a set of phrase delimiters and a list of stop words, in the first phase the algorithm splits the document by words, then by phrases and then by stop words: the remaining words are considered candidate keywords

The ordered sequence of candidate keywords is scanned to compute the table of co-occurences where the item $M_{ij}$ counts the number of pairs that contains both the $i$-th and the $j$-th word.
Given this table is possible to compute the frequency of a word as $M_{ii}$, and the degree of a word as the sum over its row $\sum_j M_{ij}$.
We define the score of a word as the ratio $\frac{deg(w)}{freq(w)}$.

A set of adjoining keywords is a candidate if it appears at least twice in the document, if so the score for the new keyword is the sum of its member keyword scores.
Sorting by score enables the user to select the $k$-th most significative keywords.
