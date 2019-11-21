# Compression of documents
In a modern search engine the raw documents are needed for various applications, one of the more common is the dynamic extraction of a snippet depending on the query.
The most important tradeoff in data compressione is given by the inverse proportionality of the compression rate and the decompression speed.
Recently many technologies have been developed, among the others we recall Snappy and Brotly by Google and LZFSE by Apple.

## LZ77
The LZ77 algorithm, used by gzip, compresses data by exploiting repeated substrings in a document.
Given a document it generates a set of triplets of the form $\langle$ distance, length, next-char $\rangle$ by scanning the document using a fixed size buffer window.
So, at position $i$ the algorithm checks if a substring $A[i,k]$ is repeated at least once in the substring $A[i-b,i-1]$, where $b$ is the buffer size, if so it generates a triplet and skips to the next non compressed character.
The decompression reverses this process, by applying the sorted triplets to the initially empty string $\epsilon$.

## Z-delta compression
This algorithm is used to compress a new file $f_n$ by using another known file $f_k$.
Reprising the idea of the LZ77 algorithm the file $f_k$ and $f_n$ are concatenated, then by scanning from $f_n$ the set of triplets $f_d$ is generated, it's possibile to prove that $f_d$ it's an optimal coverage, contaning the minimal number of triplets.
Now it's possibile to obtain $f_n$ concatenating $f_k$ to the empty string $\epsilon$ and then by applying the triplets as in the LZ77 algorithm.

The Z-delta algorithm can also be used to compress a cluster of files by constructing a graph where all the files are represented by a node.
Any directed edge $<i,j>$ is weighted by the number of triples needed to compress $f_j$ using $f_i$ as common knowledge.
By adding a dummy node $\epsilon$ an optimal compression of the cluster is found by computing a minimum directed spanning tree rooted in the dummy node.
It should be noticed that the number of edges is quadratic in respect to the number of files, and it's costly to generate them.
Since we are intereseted in using only edges between similar files, we can use locality sensitive hashing as an heuristic in the construction of the graph.

## rsync
This algorithm is used to synchronize the content of two files in a client-server scenario, let's suppose that the old one $f_o$ is in the client, and the new one $f_n$ is in the server and they're possibly very different, so we can't assume common knowledge between them.
Given a block size $b$ the clients generates a non-overlapping sequence of blocks, then each of these is hashed and sent to the server.
Once that the server received the hashes it insercts them in a dictionary.
After this it scans $f_n$ with a window of size $b$ and hashes each buffer, if it doesn't match any of the ones in the dictionary the character in the first position of the buffer is sent to the client, otherwise it sends the index of the block.

## zsync
The zsync algorithm is useful to reduce the work load of the server and could be considered as a symmetrical approach to rsync.
This time is the server that computes the $\frac{n}{b}$ blocks, and then sends them to the client.
Using a rolling hash scanning the client looks for the received hashes is $f_o$ and replies to the server with a bitmask of length $\frac{n}{b}$ where $B[i]$ is set to one iff. the block $H_i$ is present in $f_o$.
Given the bitmask the server is able to z-delta compress the block that the client doesn't have according to the ones it owns, enabling the client to construct the full file.

