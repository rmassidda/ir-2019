# Query Processing

## Phrase queries
A phrase query is a query where multiple words are considered as an atomic unit.
To be able to support such queries, it is no longer sufficient for postings lists to be simply lists of documents that contain individual terms.

One approach is to consider every pair of consecutive terms in a document as a phrase, to generate biwords.
Each of these biwords is treated as a vocabulary term, and inserted as an entry in the dictionary.
The query processing of a two-word phrase is immediate, furthermore longer phrases can also be processed by breaking them down in overlapping pairs and using the AND operator.
Without examining the documents it's not possible to verify the result of this boolean operation, that can possible cause false positive results.
To optimize the results it's possible to use PoS tagging to construct an extended biwords index.

Another approach consists in storing in the postings list also the position in which the term occurs.
Other than for phrase queries this approach can be used to solve free text queries, in which the results are biased according to the close proximity of each other.
This is done according to the assumption that users prefer docs in which query terms occur within close proximity to each other.
The combination of these two schemes is often used in real word search engine.

Usually when dealing with phrase queries the search engine makes use of an iterative process, called soft-AND.
The engine tries to run the query as provided by the user, if the results are too many it tries to run multiple smaller queries and join them.
If even in this case the set of results is too small the search engine must use other techniques, like vector space querying and then ranking the results.

## Zone indexes
Up to now a document has been considered as a sequence of terms, but this view can be enlarged by considering other features like the author, the title, the language and so on.
This accessory features of a document are called its metadata.

A zone is a region of the document that can contain an arbitrary amount of text.
The information about the zones can be stored in the dictionary or in the postings, in any case building an inverted index also on the zones allows the user to query directly on them.

## Tiered index
Caching can be useful to speedup query resolution, there are two possible approaches: to cache the query results, exploiting query locality, or to cache pages of the postings lists, exploiting term locality.

Another possible solution consists in breaking up postings into a hierarchy of lists, sorted by importance.
At query time only the top tier is used, unless it fails to provide a minimum number of documents, if so it recurse onto the lower tier.

## Skip pointers
In a skip list the number of skips is an important trade-off, the more the skips the shorter the spans, in the opposite case is instead more likely to skip but lots of comparisons are required to evaluate skip pointers.
A simple heuristic for posting list of length $L$ is to use $\sqrt{L}$ evenly-spaced skip pointers.
