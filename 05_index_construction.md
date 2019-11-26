# Index construction
In sorting a list of strings, the indirect list containing pointers breaks locality at an higher level, when the number of elements is huge this becomes a considerable problem.

## Blocked sort-based indexing
To construct an inverted index the collection of documents is scanned to generate the term-docID pairs, then the pairs are sorted by term and the docIDs are grouped in the posting list.
For small collections, all this can be done in memory, the problem requires more attention in a scenario where large collections have to be indexed.
To make index construction more efficient, we represent unique terms as termIDs, this requires a first scan to compile the vocabulary, and a second one to construct the inverted index.

With main memory insufficient, we need to use an external sorting algorithm like multi-way MergeSort.
This algorithm sorts $N$ items with a main memory of size $M$ and disk-pages of size $B$, the first pass is to produce $\frac{N}{M}$ sorted runs, and then merge them for a total cost of $O(\frac{N}{B}\log_{\frac{M}{B}} \frac{N}{M})$ IO operations.

## Single-pass in-memory indexing
Blocked sort-based indexing has excellent scaling properties, but it needs a data structure for mapping terms to termIDs.
For very large collection, this data structure does not fit into memory.
A more scalable alternative is SPIMI, that using terms writes each block's dictionary to disk, and the starts a new dictionary for the next block.

The tokens in the document are analyzed one by one, and the relative posting list is filled using a doubling algorithm.
When the memory is full the block is written to the disk, and a new one is started.
At the end of the process the block is merged to generate the full index.
The overall algorithm is faster because it doesn't require external memory sorting nor the creation of a term-termID vocabulary.

## Distributed indexing
When the index dimension becomes to big the only solution is to use distributed indexing algorithms for index construction.
Two obvious alternative index implementations are partitioning by terms and partitioning by documents.

Partitioning by terms means that each node of the cluster contains a partition of the terms and all of their posting lists.
A query is routed to the nodes corresponding to its query terms.
In principle, this allows greater concurreyncy, but in practice this behaviour turns out to be non-trival for multiword queries and load-balancing.

A more common implementation is to partition by documents, each node contains the index for a subset of all documents.
Each query is distributed to all nodes, with the results from various nodes being merged before presentation to the user.
This partitioning simpifies the communication, but requires to contact all the nodes to compute any global operation.

## Dynamic indexing
When dealing with dynamic collections new approaches are needed to correctly index them.
One simple solution, called auxiliary index, consist in using multiple indexes, a main one and other smallest where to insert newly arrived documents and periodically re-index all the collection into one main index;
also deletion can be handled in this situation by invalidating a bit vector.
Storing each postings list as a separate file, then the merges imply consists of extending each postings list of the main index by the corresponding postings list of the auxiliary index, this is unfeasible because the difficulties for a file system in handling a big number of files.
This consolidation process is costly.

A better solution is provided by logarithmic merge, where a series of exponentially increasing indexes are allocated in the disk and in memory an index is present larger as the smallest on disk.
If an index $I_i$ becomes too big its content is merged with the successive $I_{i+1}$, and so on if after merging also this index becomes too big, this maintains the invariant about the fact that each one of the indexes on the disk is either empty or full.

Each text participates to no more than $\log \frac{C}{M}$ mergings because at each merge the text moves to a next index and they are at most $\log \frac{C}{M}$, where $C$ is the total size of the collection.
