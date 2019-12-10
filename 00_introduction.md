---
title: 'Information Retrieval'
classoption:
  - titlepage
toc: true
---
\newpage
# Introduction

Information Retrieval (IR) is finding material of unstructured nature that satisfies an information need from within large collections.
Unstructured data typically refers to free text to query for some keywords or concepts.

The boolean retrieval model is used to respond to a query composed as a boolean expression, that is using `AND`, `OR` and `NOT` operators to join the query terms.
The first idea to maintain the relation between terms and documents containing them could be to use a matrix, but in practice this data structures turns out to be huge and sparse.

To solve this issue we make use of an inverted index where for each term $t$ we must store a list of all documents that contain $t$, each of these identified by a unique id.
The set of terms is called the dictionary, while the documents are called posting and consequently a list is called a postings list.

The `AND` operation can now be implemented by intersecting the postings list, doing this in growing order is one of the possibile optimizations to speedup the computation.
